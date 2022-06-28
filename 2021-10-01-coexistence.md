## Background

With the emergence of delay-sensitive applications like cloud gaming, remote driving, and virtual/augmented reality (VR/AR), low latency networking is more important than ever. Consequently, there has been renewed research interest in the development of low latency congestion control protocols. However, these efforts have been impeded by the difficulty associated with designing an end-to-end protocol that reacts to delay signals, but that also shares a link effectively (i.e., without causing harm to other flows and without being starved by other flows). This is a critical requirement for use in the public internet. To help protocol designers, we summarize the literature on coexistence of delay-based congestion controls. We implement a selection of the coexistence techniques described in the literature, and evaluate them in a set of controlled experiments designed to showcase the benefits and limitations of each approach. 

Through our experimental analysis, we highlight coexistence-related pitfalls of delay as a congestion signal. We hope that this work will help inform the design of new low latency congestion control protocols or improvements to existing protocols.

## Results 

![](/blog/content/images/2021/10/1flow_10ms-1.svg)
<center>_Figure 1: 1 delay-based flow vs 1 loss-based flow_</center>

![](/blog/content/images/2021/10/5flows_10ms.svg)
<center>_Figure 2: 5 delay-based flows vs 5 loss-based flows_</center>

![](/blog/content/images/2021/10/10flows_10ms.svg)
<center>_Figure 3: 10 delay-based flows vs 1 loss-based flow result_</center>

In the figures above, we show the average of the sum of throughput of the delay-based TCP flows, across five experiment runs, during each of the three phases of the experiment. We also show the average transport layer RTT across the network in each phase.

These are considered in comparison to the **ideal response** of a low latency protocol with good coexistence behavior:

* **First 60 seconds**: A delay-based TCP should fully utilize the bottleneck link capacity (100 Mbps) and maintain low delay.
* **60 - 120 seconds**: For Figures 1 and 2, the delay-based flows together should get close to half the link capacity (50 Mbps). For Figure 3, the delay-based flows together should get close to the fair share of 10 flows competing against 1 loss-based flow, i.e., 91 Mbps. In all three scenarios, we expect to  see a delay of 10 + 15 = 25 ms (the sum of the base delay and full buffer delay).
* **120 - 180 seconds**: After the loss-based flow(s) leave the network, the delay-based flow(s) should resume low latency operation (10 ms delay) and also fully utilize the link capacity (100 Mbps throughput).

Here are our observations for each of the candidate solutions.

* **TCP Vegas:** TCP Vegas is used as the benchmark for comparing the coexistence performance of the different solutions. As expected, TCP Vegas gets starved for throughput while competing with loss-based traffic. 

* **Basic threshold adaptation:** In all three scenarios of our experiment, this scheme improves the throughput share of delay-based flows while competing with loss-based traffic, as compared to TCP Vegas. As mentioned earlier, this scheme by itself cannot resume low delay operation.

* **Threshold adaptation with delay gradient signal:** This scheme works well in Scenarios 1 and 2. It ensures decent throughput for delay-based traffic during the competition phase (60 - 120 seconds), and also works well for reducing queueing delay once the loss-based flows leave (120 - 180 seconds). In Scenario 3, this method is unable to adapt the threshold for resuming low delay. This is because with only one loss-based flow in the network, the observed RTT gradient when this flow leaves is too small to resume low delay operation. 

* **Threshold adaptation with periodic backoff:** While this scheme works well in light multiplexing scenarios, it breaks down when multiple delay-based flows share a bottleneck with loss-based flows. In these cases, we see that delay remains high even after loss-based flows leave the network. As the instances of threshold backoff for all delay-based flows are not synchronized, any flow which drops its threshold will not observe a subsequent dip in RTT as the other competing delay-based flows, which are still operating at a high threshold, will send in more packets into the queue. Such a synchronization is not possible in end-to-end, distributed CC. Decreasing the periodicity of threshold backoff, i.e., increasing the frequency of backoff, is also not a viable option as it will result in poor throughput while competing with loss-based traffic.  

* **Cx-TCP:** Coexistent TCP is known to not perform well in light multiplexing scenarios e.g., when a single Cx-TCP flow shares a bottleneck with a single TCP Cubic flow. We observe this behavior in our single-flow experiments as well,  where it loses out to loss-based flows during the competition phase of the experiment. 

* **Cx-TCP with shadow CWND:** This variant of Cx-TCP did not show any major improvements to Cx-TCP's throughput performance in light multiplexing scenarios.

* **Cx-TCP with RTT variance scaling:** We observe that while this idea improves Cx-TCP's throughput during competition, it increases the overall delay, thus losing the low latency benefits of Cx-TCP.



## Run my experiment

To run these experiments, you will need an account on [CloudLab](https://cloudlab.us/). If you already have a GENI account, you can use it to log in to CloudLab. Once you are logged in to CloudLab, open our `TCP_coexistence_study` profile: [https://www.cloudlab.us/show-profile.php?uuid=5f931846-2d71-11ec-bba7-e4434b2381fc](https://www.cloudlab.us/show-profile.php?uuid=5f931846-2d71-11ec-bba7-e4434b2381fc)

Once you reach the profile page, click on the `version_history` button in the `Show Profile` section of the page. Then select the appropriate version of the experiment based on the coexistence solution you want to test, as follows : 

* For running the experiment with the TCP_Vegas congestion control, select version `0`.
*  Basic threshold adaptation : version `1`
*  Threshold adaptation with delay gradient : version `1`
*  Threshold adaptation with periodic backoff : version `2`
*  Cx-TCP : version `3`
* Cx-TCP with rttvar scaling : version `3`
*  Cx-TCP with shadow CWND : version `4`

After selecting the profile version, click on "Instantiate", then choose CloudLab Wisconsin and start your experiment. Wait until all nodes have booted and are ready to log in, then click on the "List view" tab to get SSH login instructions for your nodes. Then, use SSH to open a shell session on each node in the experiment.

### Set Up Resources

This section describes how to prepare your nodes to run this experiment:

Install `iperf3` on the src and dstn nodes:

```
sudo apt update
sudo apt -y install iperf3
```
Install `moreutils` on the src node : 

```
sudo apt -y install moreutils
```

Configure the interface on the router through which traffic goes to the destination node. Use `tc` to set up an HTB queue with an egress rate of 100Mbps. Then, add a FIFO queue with a limit of 187.5kB, which corresponds to a full buffer delay of 15ms. On the router, run:

```
sudo tc qdisc del dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root handle 1: htb default 3
sudo tc class add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1: classid 1:3 htb rate 100mbit

sudo tc qdisc add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 187500
```

Add a 10ms delay at the emulator node : 

```
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root netem delay 10ms
```

On the dstn node, open a new screen using : 

```
screen
```

Then, inside this new screen at the dstn node, start an `iperf3` server : 

```
iperf3 -s -i 0.1
```

Leave the `iperf3` server running and close the screen using the keyboard shortcut `Ctrl+A+D`. You may also logout of your SSH session at the dstn node. You can resume the `screen` session at a later stage using `screen -r`

Similarly, open another screen on the dstn node, and start another `iperf3` server on port 5202 as follows:

```
iperf3 -s -i 0.1 -p 5202
```
Both these `iperf3` servers need to be running for the entire duration of the experiment, which is why we are using the `screen` utility. 


### Configure coexistence protocol parameters

On your SSH session at the src node, run :

```
sudo modprobe tcp_vegas
```
This loads the linux kernel module for the delay based congestion control. The coexistence solution implemented in this module depends on your choice of the profile version, as explained above.

For changing module parameters, you need root access. Run : 

```
cd /sys/module/tcp_vegas/parameters
sudo su
```


As an example, if you have to change parameter `X` to a value of `4`, run : 

``` 
sudo echo `4` > gamma
```
Configure protocol specific parameters as follows : 

* TCP_Vegas : no change to the default parameters 

* Basic threshold adaptation : `compete` = 1, `comeback` = 0

* Threshold adaptation with delay gradient : `compete` = 1, `comeback` = 1

* Threshold adaptation with periodic backoff : `compete` = 1, `comeback` = 1, `P1` = 200, `P2` = 5, `drop` = 95

*  Cx-TCP : `mdev_scaling` = 0, `betao` = 2000, `del_th` = 4000, `del_max` = 150000

* Cx-TCP with rttvar scaling : `mdev_scaling` = 1, `betao` = 2000, `del_th` = 4000, `del_max` = 150000

* Cx-TCP with shadow CWND : `mdev_scaling` = 0, `betao` = 2000, `del_th` = 4000, `del_max` = 150000

### Running a single experiment

Save the following bash script inside a file named ``src-run-1v1.sh``. 

```
EXPID=vegas
runtime="180 seconds"
t=180
for j in {1..5}
do
	endtime=$(date -ud "$runtime" +%s)
	rm -f $EXPID-cubic-$j-ss.txt;while [[ $(date -u +%s) -le $endtime ]]; do ss --no-header -eipn dst 10.0.3.1 | ts '%.s' | tee -a $EXPID-cubic-$j-ss.txt > /dev/null ;sleep 0.1; done & iperf3 -f m -c dstn -C vegas -P 1 -i 0.1 -t 180 | ts '%.s' | tee $EXPID-$j-iperf.txt > /dev/null & sleep 60; iperf3 -f m -c dstn -C cubic -P 1 -i 0.1 -t 60 -p 5202 | ts '%.s' | tee cubic-$j-iperf.txt > /dev/null;sleep 60;

	sleep 5;
done
```

The above script executes the following :

* Starts a single ``iperf3`` flow using the delay based congestion control with the coexistence protocol of choice. This flow runs from T = 0s  to T = 180s.
* Starts a single ``iperf3`` flow using loss based CC,i.e., TCP Cubic. This flow runs from T = 60s to T = 120s
*  Records TCP socket data using the ``ss`` tool and also saves the throughput statistics recorded by ``iperf3``
* Executes five trials of this experiment.


To run the experiment, simply do ``bash src-run-1v1.sh`` at the src node. You may want to use the ``screen`` utility before starting the experiment, similar to how it was done at the dstn node. 

After the experiment is done, you will need to process the raw data. Save the following script as ``processing-1v1.sh``

```
EXPID=vegas

for j in {1..5}  
do
	EXPID=vegas
	echo "$j" | tr '\n' ',' >>  results-trial.txt
	cat $EXPID-$j-iperf.txt | grep "\[ " | grep "sec" | grep -v "0.00-1" | awk '{print $3","$4","$8","$10}' | tr -d ']' | sed 's/-/,/g' > $EXPID-$j-iperf.csv
	head -599 $EXPID-$j-iperf.csv > $EXPID-$j-start-iperf.csv
	sed '1,599d' $EXPID-$j-iperf.csv > $EXPID-$j-iperf-1.csv
	head -599 $EXPID-$j-iperf-1.csv > $EXPID-$j-compete-iperf.csv
	sed '1,599d' $EXPID-$j-iperf-1.csv > $EXPID-$j-resume-iperf.csv
	rm $EXPID-$j-iperf-1.csv; rm $EXPID-$j-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-start-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-compete-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-resume-iperf.csv
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-start-iperf.csv | tr '\n' ',' >>  results-trial.txt
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-compete-iperf.csv | tr '\n' ',' >>  results-trial.txt
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-resume-iperf.csv | tr '\n' ',' >>  results-trial.txt

	EXPID=vegas-cubic
	rm test-$j-ss.txt
	cp $EXPID-$j-ss.txt test-$j-ss.txt
	sed -i '/fd=3/d' test-$j-ss.txt
	sed -i '/10.0.3.1:5202/d' test-$j-ss.txt
	EXPID=test
	cat $EXPID-$j-ss.txt | sed -e ':a; /<->$/ { N; s/<->\n//; ba; }'  | grep "iperf3" > $EXPID-$j-ss-processed.txt
	cat $EXPID-$j-ss-processed.txt | awk '{print $1}' > ts-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\bcwnd:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' ' > cwnd-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\brtt:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' '  | cut -d '/' -f 1   > srtt-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\bfd=.*?(\s|$)' |  awk -F '[=,]' '{print $2}' | tr -d ')' | tr -d ' '   > fd-$EXPID-$j.txt
	paste ts-$EXPID-$j.txt fd-$EXPID-$j.txt cwnd-$EXPID-$j.txt srtt-$EXPID-$j.txt -d ',' > $EXPID-$j-ss.csv
	rm ts-$EXPID-$j.txt;rm fd-$EXPID-$j.txt;rm cwnd-$EXPID-$j.txt;rm srtt-$EXPID-$j.txt;rm $EXPID-$j-ss-processed.txt;rm $EXPID-$j-ss.txt
	EXPID=vegas-cubic
	mv test-$j-ss.csv $EXPID-$j-ss.csv

	datetime1=$(head -n1 $EXPID-$j-ss.csv | cut -d, -f1)
	datetime2=$(echo "$datetime1 + 60" | bc)
	datetime3=$(echo "$datetime1 + 120" | bc)
	awk -v datetime2=$datetime2 -F, '$1 < datetime2' $EXPID-$j-ss.csv  > $EXPID-$j-start-ss.csv
	awk -v datetime2=$datetime2 -v datetime3=$datetime3 -F, '$1 >= datetime2 && $1 < datetime3' $EXPID-$j-ss.csv  > $EXPID-$j-compete-ss.csv
	awk -v datetime2=$datetime2 -v datetime3=$datetime3 -F, '$1 >= datetime3' $EXPID-$j-ss.csv  > $EXPID-$j-resume-ss.csv
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-start-ss.csv | tr '\n' ',' >>  results-trial.txt
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-compete-ss.csv | tr '\n' ',' >>  results-trial.txt
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-resume-ss.csv  >>  results-trial.txt
done
```

Run ``bash processing-1v1.sh`` to execute the processing script. The results will be saved in a file named ``results-trial.csv``. The columns of this file represent the following data : 

* Col 1 : average throughput of delay based flows in the first 60 seconds of the experiment.
* Col 2 : average throughput of delay based flows during time T = 60s to T = 120s.
* Col 3 : average throughput of delay based flows during time T = 120s to T = 180s.
* Col 4 : Average of the smoothed RTT (sRTT) observed by delay-based flows in the first 60 seconds of the experiment.
* Col 5 : Average of the smoothed RTT (sRTT) observed by delay-based flows during time T = 60s to T = 120s.
* Col 6 : Average of the smoothed RTT (sRTT) observed by delay-based flows during time T = 120s to T = 180s.

Each row represents one trial of the experiment ( 5 trials in total).

### Experiments with multiple flows

In the above experiment, a single delay-based flow competed with a single loss-based flow. 

In our second experiment, we look at a heavy multiplexing scenario where five delay-based flows coexist with five loss-based flows. 

The script for running this experiment (``src-run-5v5.sh``) is : 

```
EXPID=vegas
runtime="180 seconds"
t=180
for j in {1..5}
do
	endtime=$(date -ud "$runtime" +%s)
	rm -f $EXPID-cubic-$j-ss.txt;while [[ $(date -u +%s) -le $endtime ]]; do ss --no-header -eipn dst 10.0.3.1 | ts '%.s' | tee -a $EXPID-cubic-$j-ss.txt > /dev/null ;sleep 0.1; done & iperf3 -f m -c dstn -C vegas -P 5 -i 0.1 -t 180 | ts '%.s' | tee $EXPID-$j-iperf.txt > /dev/null & sleep 60; iperf3 -f m -c dstn -C cubic -P 5 -i 0.1 -t 60 -p 5202 | ts '%.s' | tee cubic-$j-iperf.txt > /dev/null;sleep 60;

	sleep 5;
done

```

The data processing script for this experiment (``processing-5v5.sh``) is : 

```
EXPID=vegas

for j in {1..5}  
do
	EXPID=vegas
	echo "$j" | tr '\n' ',' >>  results-trial.txt
	cat $EXPID-$j-iperf.txt | grep "\[ " | grep "sec" | grep -v "0.00-1" | awk '{print $3","$4","$8","$10}' | tr -d ']' | sed 's/-/,/g' > $EXPID-$j-iperf.csv
	head -2995 $EXPID-$j-iperf.csv > $EXPID-$j-start-iperf.csv
	sed '1,2995d' $EXPID-$j-iperf.csv > $EXPID-$j-iperf-1.csv
	head -2995 $EXPID-$j-iperf-1.csv > $EXPID-$j-compete-iperf.csv
	sed '1,2995d' $EXPID-$j-iperf-1.csv > $EXPID-$j-resume-iperf.csv
	rm $EXPID-$j-iperf-1.csv; rm $EXPID-$j-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-start-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-compete-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-resume-iperf.csv
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-start-iperf.csv | tr '\n' ',' >>  results-trial.txt
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-compete-iperf.csv | tr '\n' ',' >>  results-trial.txt
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-resume-iperf.csv | tr '\n' ',' >>  results-trial.txt

	EXPID=vegas-cubic
	rm test-$j-ss.txt
	cp $EXPID-$j-ss.txt test-$j-ss.txt
	sed -i '/fd=3/d' test-$j-ss.txt
	sed -i '/10.0.3.1:5202/d' test-$j-ss.txt
	EXPID=test
	cat $EXPID-$j-ss.txt | sed -e ':a; /<->$/ { N; s/<->\n//; ba; }'  | grep "iperf3" > $EXPID-$j-ss-processed.txt
	cat $EXPID-$j-ss-processed.txt | awk '{print $1}' > ts-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\bcwnd:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' ' > cwnd-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\brtt:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' '  | cut -d '/' -f 1   > srtt-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\bfd=.*?(\s|$)' |  awk -F '[=,]' '{print $2}' | tr -d ')' | tr -d ' '   > fd-$EXPID-$j.txt
	paste ts-$EXPID-$j.txt fd-$EXPID-$j.txt cwnd-$EXPID-$j.txt srtt-$EXPID-$j.txt -d ',' > $EXPID-$j-ss.csv
	rm ts-$EXPID-$j.txt;rm fd-$EXPID-$j.txt;rm cwnd-$EXPID-$j.txt;rm srtt-$EXPID-$j.txt;rm $EXPID-$j-ss-processed.txt;rm $EXPID-$j-ss.txt
	EXPID=vegas-cubic
	mv test-$j-ss.csv $EXPID-$j-ss.csv

	datetime1=$(head -n1 $EXPID-$j-ss.csv | cut -d, -f1)
	datetime2=$(echo "$datetime1 + 60" | bc)
	datetime3=$(echo "$datetime1 + 120" | bc)
	awk -v datetime2=$datetime2 -F, '$1 < datetime2' $EXPID-$j-ss.csv  > $EXPID-$j-start-ss.csv
	awk -v datetime2=$datetime2 -v datetime3=$datetime3 -F, '$1 >= datetime2 && $1 < datetime3' $EXPID-$j-ss.csv  > $EXPID-$j-compete-ss.csv
	awk -v datetime2=$datetime2 -v datetime3=$datetime3 -F, '$1 >= datetime3' $EXPID-$j-ss.csv  > $EXPID-$j-resume-ss.csv
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-start-ss.csv | tr '\n' ',' >>  results-trial.txt
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-compete-ss.csv | tr '\n' ',' >>  results-trial.txt
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-resume-ss.csv  >>  results-trial.txt
done
mv results-trial.txt results-trial.csv

```

Next, we look at a third network traffic scenario where ten delay-based flows coexist with a single loss-based flow. 

Experiment execution script (``src-run-10v1.sh``) :  

```
EXPID=vegas
runtime="180 seconds"
t=180
for j in {1..5}
do
	endtime=$(date -ud "$runtime" +%s)
	rm -f $EXPID-cubic-$j-ss.txt;while [[ $(date -u +%s) -le $endtime ]]; do ss --no-header -eipn dst 10.0.3.1 | ts '%.s' | tee -a $EXPID-cubic-$j-ss.txt > /dev/null ;sleep 0.1; done & iperf3 -f m -c dstn -C vegas -P 10 -i 0.1 -t 180 | ts '%.s' | tee $EXPID-$j-iperf.txt > /dev/null & sleep 60; iperf3 -f m -c dstn -C cubic -P 1 -i 0.1 -t 60 -p 5202 | ts '%.s' | tee cubic-$j-iperf.txt > /dev/null;sleep 60;

	sleep 5;
done
```

Data processing script (``processing-10v1.sh``) : 

```
EXPID=vegas

for j in {1..5}  
do
	EXPID=vegas
	echo "$j" | tr '\n' ',' >>  results-trial.txt
	cat $EXPID-$j-iperf.txt | grep "\[ " | grep "sec" | grep -v "0.00-1" | awk '{print $3","$4","$8","$10}' | tr -d ']' | sed 's/-/,/g' > $EXPID-$j-iperf.csv
	head -5990 $EXPID-$j-iperf.csv > $EXPID-$j-start-iperf.csv
	sed '1,5990d' $EXPID-$j-iperf.csv > $EXPID-$j-iperf-1.csv
	head -5990 $EXPID-$j-iperf-1.csv > $EXPID-$j-compete-iperf.csv
	sed '1,5990d' $EXPID-$j-iperf-1.csv > $EXPID-$j-resume-iperf.csv
	rm $EXPID-$j-iperf-1.csv; rm $EXPID-$j-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-start-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-compete-iperf.csv
	sed -i 's/,/ /g' $EXPID-$j-resume-iperf.csv
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-start-iperf.csv | tr '\n' ',' >>  results-trial.txt
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-compete-iperf.csv | tr '\n' ',' >>  results-trial.txt
	awk '{ total += $4; count++ } END { print total/count }' $EXPID-$j-resume-iperf.csv | tr '\n' ',' >>  results-trial.txt

	EXPID=vegas-cubic
	rm test-$j-ss.txt
	cp $EXPID-$j-ss.txt test-$j-ss.txt
	sed -i '/fd=3/d' test-$j-ss.txt
	sed -i '/10.0.3.1:5202/d' test-$j-ss.txt
	EXPID=test
	cat $EXPID-$j-ss.txt | sed -e ':a; /<->$/ { N; s/<->\n//; ba; }'  | grep "iperf3" > $EXPID-$j-ss-processed.txt
	cat $EXPID-$j-ss-processed.txt | awk '{print $1}' > ts-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\bcwnd:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' ' > cwnd-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\brtt:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' '  | cut -d '/' -f 1   > srtt-$EXPID-$j.txt
	cat $EXPID-$j-ss-processed.txt | grep -oP '\bfd=.*?(\s|$)' |  awk -F '[=,]' '{print $2}' | tr -d ')' | tr -d ' '   > fd-$EXPID-$j.txt
	paste ts-$EXPID-$j.txt fd-$EXPID-$j.txt cwnd-$EXPID-$j.txt srtt-$EXPID-$j.txt -d ',' > $EXPID-$j-ss.csv
	rm ts-$EXPID-$j.txt;rm fd-$EXPID-$j.txt;rm cwnd-$EXPID-$j.txt;rm srtt-$EXPID-$j.txt;rm $EXPID-$j-ss-processed.txt;rm $EXPID-$j-ss.txt
	EXPID=vegas-cubic
	mv test-$j-ss.csv $EXPID-$j-ss.csv

	datetime1=$(head -n1 $EXPID-$j-ss.csv | cut -d, -f1)
	datetime2=$(echo "$datetime1 + 60" | bc)
	datetime3=$(echo "$datetime1 + 120" | bc)
	awk -v datetime2=$datetime2 -F, '$1 < datetime2' $EXPID-$j-ss.csv  > $EXPID-$j-start-ss.csv
	awk -v datetime2=$datetime2 -v datetime3=$datetime3 -F, '$1 >= datetime2 && $1 < datetime3' $EXPID-$j-ss.csv  > $EXPID-$j-compete-ss.csv
	awk -v datetime2=$datetime2 -v datetime3=$datetime3 -F, '$1 >= datetime3' $EXPID-$j-ss.csv  > $EXPID-$j-resume-ss.csv
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-start-ss.csv | tr '\n' ',' >>  results-trial.txt
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-compete-ss.csv | tr '\n' ',' >>  results-trial.txt
	awk -F, '{ total += $4; count++ } END { print total/count }' $EXPID-$j-resume-ss.csv  >>  results-trial.txt
done
mv results-trial.txt results-trial.csv
```

### Experiments with different coexistence solutions

The set of three experiments (``1v1``, ``5v5``, ``10v1``) can be done for any of the following coexistence schemes for delay-based congestion control : 

1. TCP Vegas congestion control ( with no coexistence mechanisms)
2. Threshold adaptation 
3. Threshold adaptation with delay gradient
4. Threshold adaptation with periodic backoff
5. Coexistent TCP (Cx-TCP)
6. Cx-TCP with rttvar scaling
7. Cx-TCP with shadow CWND

You will need to select the appropriate version of our Cloudlab profile, and also change the configuration of the Linux kernel module parameters. Instructions for doing this have already been provided in previous sections. 




