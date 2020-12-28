In this experiment, we will show how to replicate some key results relating to low latency congestion control over mmWave links.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.


## Background

Low latency applications such as AR/VR and autonomous vehicles are expected to be a major driver for 5G and future WLAN networks. To fulfill the high throughput and low latency requirements of such applications, 5G networks will utilize the mmWave frequency bands (30-300 GHz) due to the large amounts of spectrum available in these bands. 

However, mmWave links can experience frequent, sudden changes in link capacity due to obstructions in the signal path. These dramatic variations in link capacity cause a temporary "bufferbloat" condition during which delay may increase by a factor of 2-10. Low latency congestion control protocols, which manage bufferbloat by minimizing queue occupancy, represent a potential solution to this problem, however their behavior over links with dramatic variations in capacity is not well understood. In this work, we explore the behavior of two major low latency congestion control protocols, TCP BBR and TCP Prague (as part of L4S), using link traces collected over mmWave links under various conditions. Our evaluation reveals potential problems associated with use of these congestion control protocols for low latency applications over mmWave links.

The figure below shows the traces of link capacity collected over a 60GHz WLAN testbed hosted at NYU Wireless. The four different mmWave scenarios represented here are : Static Link (sl), Short Blockages (sb), Long Blockages (lb) and Mobility & Blockages (mobb). We play back these traces in our CloudLab experiment to replicate the frequent variation in mmWave link capacity.

![link traces](/blog/content/images/2020/03/link-traces.svg)


## Results

The figure below shows the Transport RTT experienced by each of the congestion control protocols under the four different mmWave link scenarios.

![RTT Results](/blog/content/images/2020/03/rtt-all-1.svg)
 
The RTT results for TCP CUBIC are in line with the expected behavior of loss-based congestion control. The queueing delay with a static link is around 20ms (7.5MB buffer / 3Gbps typical capacity) and we see spikes in the RTT whenever the LoS path of the mmWave link is blocked.The delay can reach as high as 150ms, which is more than seven times the base delay with a static link. 

With L4S's TCP Prague, the delay stays near the 5ms ECN marking threshold. The delays increase in presence of blockage events but remain much lower compared to loss-based TCP. TCP BBR is also able to maintain small delays which mostly stay in the range of 5-10ms for all mmWave link conditions. BBR achieves this without any ECN support using its model-based algorithm at the sender side. 

The next figure shows the plot for throughput achieved (in Mbps) by the ten competing bulk flows for each congestion control scheme and the four mmWave link scenarios.


![TPUT Results](/blog/content/images/2020/03/tput-all-1.svg)

The sum throughput of the 10 flows for all link scenarios and all congestion control schemes was close to the link capacity. However, TCP BBR periodically reduces its sending rate in order to drain the queue and measure min RTT. This is called the "Probe RTT" which also helps in flow synchronization but can be problematic for streaming applications which require consistently high throughput in addition to low latencies.

While TCP CUBIC is able to maintain a fair share of capacity between the 10 flows, TCP Prague has poor fairness behavior where some flows are  starved completely. This unfairness becomes worse in presence of blockages and mobility. TCP BBR (both v1 and v2) is able to maintain decent fairness due to its "Probe RTT" phase explained earlier but BBR is known to have problems with fairness for flows with different RTTs and for flows that share a link with loss-based congestion control.


## Run my experiment

To produce the results above, we ran a sequence of experiments on CloudLab to explore the behavior of several congestion control protocols over mmWave links. In this profile, we provide instructions for running these experiments, in order to reproduce our results.

### Reserve Resources

To run these experiments, you will need an account on [CloudLab](https://cloudlab.us/). If you already have a GENI account, you can use it to log in to CloudLab. Once you are logged in to CloudLab, open our `low_latency_tcp_mmwave` profile: 
[https://www.cloudlab.us/p/b4993875-fa74-11e9-b1eb-e4434b2381fc](https://www.cloudlab.us/p/b4993875-fa74-11e9-b1eb-e4434b2381fc)

Click on "Instantiate", then choose CloudLab Wisconsin and start your experiment. Wait until all nodes have booted and are ready to log in, then click on the "List view" tab to get SSH login instructions for your nodes. Then, use SSH to open a shell session on each node in the experiment. 

### Set Up Resources

This section describes how to prepare your nodes to run this experiment:

Install `moreutils` on the TCP source node:

```
sudo apt update
sudo apt -y install moreutils
```

Configure the interface on the router through which traffic goes to the destination node. Use `tc` to set up an HTB queue with an egress rate of 3Gbps. Then, add a FIFO queue with a limit of 7.5MB. On the router, run:

```
sudo tc qdisc del dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root handle 1: htb default 3
sudo tc class add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1: classid 1:3 htb rate 3gbit

sudo tc qdisc add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 7500000
```

Next, retrieve the link capacity traces on the router node, and the tputvary.sh script with which you can vary the link capacity. On the router, run:

```
wget https://gist.githubusercontent.com/Shreeshail-Hingane/ea39871943010085953f1bd7ef691d35/raw/8b4586054ab6d0cbb73f703c2508745596291c98/lb-tput.csv
wget https://gist.githubusercontent.com/Shreeshail-Hingane/ea39871943010085953f1bd7ef691d35/raw/4bf1add9a8c32d9c553f86a4ec45954283f9ad4b/mobb-tput.csv
wget https://gist.githubusercontent.com/Shreeshail-Hingane/ea39871943010085953f1bd7ef691d35/raw/4bf1add9a8c32d9c553f86a4ec45954283f9ad4b/sb-tput.csv
wget https://gist.githubusercontent.com/Shreeshail-Hingane/ea39871943010085953f1bd7ef691d35/raw/4bf1add9a8c32d9c553f86a4ec45954283f9ad4b/sl-tput.csv
wget https://gist.githubusercontent.com/ffund/7a02086edcbff5af7db025e08621f08d/raw/01bacb40db9ed5492f3f78b512971893d62c39bc/tputvary.sh
```

### Running a Single Experiment

To run a basic experiment with TCP CUBIC and a static link scenario, you will:

* Run an `iperf3` receiver at the destination node: 

```
iperf3 -s -i 0.1 -1
```

* Use `ss` at the sender node to capture TCP statistics:

```
EXPID="sl-cubic"
rm -f "$EXPID"-ss.txt; while true; do ss --no-header -eipn dst 10.0.3.1 | ts '%.s' | tee -a "$EXPID"-ss.txt; sleep 0.1; done
```

* Use the `tputvary.sh` script to start shaping the link capacity. In this first experiment, use sl as the argument to the script, for the Static Link scenario.

```
bash tputvary.sh sl
```

* Right away, start an `iperf3` client at the sender node, with ten parallel flows, for 120 seconds (you will need to open a second SSH session to the sender node): 

```
EXPID="sl-cubic";
iperf3 -f m -c dstn -C cubic -P 10 -i 0.1 -t 120 | ts '%.s' | tee "$EXPID"-iperf.txt
```

After the experiment concludes (120 seconds), you can use Ctrl+C to stop the `ss` process. Retrieve the two data files from the TCP sender node using `scp`.

###  Sequence of experiments

You will run a total of 16 experiments, for every possible combination of four link scenarios (Static Link, Short Blockages, Long Blockages, Mobility & Blockages) and four congestion control protocols (TCP CUBIC, TCP Prague, TCP BBRv1, TCP BBRv2). For each, you will save the output of the `iperf3` sender and the `ss` data at the sender. 

The following sections describe how to modify the instructions above to run the alternative versions (different link scenarios and different congestion controls).

In addition to the other changes described below, you will set a unique value for the `EXPID` variable in both of the commands you run at the TCP sender. We recommend using the link scenario, followed by a `-`, then the TCP congestion control. For example, you would choose the appropriate entry from this list:

```
EXPID="sl-cubic"
EXPID="sb-cubic"
EXPID="lb-cubic"
EXPID="mobb-cubic"
EXPID="sl-prague"
EXPID="sb-prague"
EXPID="lb-prague"
EXPID="mobb-prague"
EXPID="sl-bbr1"
EXPID="sb-bbr1"
EXPID="lb-bbr1"
EXPID="mobb-bbr1"
EXPID="sl-bbr2"
EXPID="sb-bbr2"
EXPID="lb-bbr2"
EXPID="mobb-bbr2"
```

This will ensure that a unique filename is used for each experiment.

Note that the `EXPID` variable should be set in each shell session where it is used, just before it is used.


### Changing the Link Scenario
	
To vary the link scenario, you will change the argument to the `tputvary.sh` script. The four scenarios are: Static Link (sl), Short Blockages (sb), Long Blockages (lb), and Mobility and Blockages (mobb). In other words, you will run

```
bash tputvary.sh sl
```

for the static link experiment,

```
bash tputvary.sh sb
```

for the short blockages experiment,

```
bash tputvary.sh lb
```

for the long blockages experiment, and 

```
bash tputvary.sh mobb
```

for the experiment with mobility and blockages.

### Changing the Congestion Control

To change the congestion control, you will need to boot the TCP sender into a different kernel. Two kernels are installed on the TCP sender:

* The default kernel is the 5.3-rc3 kernel, which you should use for TCP CUBIC and TCP Prague experiments.
* To run experiments with TCP BBR, boot into the 5.2-rc3 kernel.

To switch kernels, at the source node, do 

```
sudo vim /etc/default/grub
```

and edit the line configuring `GRUB_DEFAULT` to look like this:

```
GRUB_DEFAULT=saved
```

Then, get a list of kernel "names":

```
awk -F\' '/menuentry / {print $2}' /boot/grub/grub.cfg
```

Then you can choose the new default kernel, e.g. for switching to the BBR kernel run, 

```
sudo grub-set-default "Advanced options for Ubuntu>Ubuntu, with Linux 5.2.0-rc3+"
sudo update-grub
sudo reboot
```

Wait for the src node to reboot , then SSH into it and run : 

```
uname -r 
```

to verify that the correct kernel was loaded.

Similarly, to get back to the L4S kernel 

```
sudo grub-set-default "Advanced options for Ubuntu>Ubuntu, with Linux 5.3.0-rc3+"
sudo update-grub
sudo reboot
```

To use any congestion control besides for TCP CUBIC, once you have booted into the appropriate kernel, you will have to use `modprobe` to load the corresponding kernel module - i.e. choose the appropriate command from

```
sudo modprobe tcp_prague
sudo modprobe tcp_bbr
sudo modprobe tcp_bbr2
```

For TCP Prague, you should also enable accurate ECN at **both** the sender and receiver nodes:

```
sudo sysctl -w net.ipv4.tcp_ecn=3
```

 Also for TCP Prague, change the FIFO queue at the router to a FQ with one "bucket" and a 5ms ECN marking threshold:

```
sudo tc qdisc del dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 7500000
sudo tc qdisc add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: fq limit 5000 flow_limit 5000 orphan_mask 0 ce_threshold 5ms
```

To change back from an FQ to a FIFO queue for the non-Prague congestion controls, run:

```
sudo tc qdisc del dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: fq limit 5000 flow_limit 5000 orphan_mask 0 ce_threshold 5ms
sudo tc qdisc add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 7500000
```

### Data analysis

For your convenience, we provide the following Bash script, which you can run at the sender to extract relevant data from the output of the `iperf` and `ss` commands. 

<pre>
for EXPID in sl-cubic sb-cubic lb-cubic mobb-cubic sl-prague sb-prague lb-prague mobb-prague sl-bbr1 sb-bbr1 lb-bbr1 mobb-bbr1 sl-bbr2 sb-bbr2 lb-bbr2 mobb-bbr2
do

    cat $EXPID-ss.txt | sed -e ':a; /<->$/ { N; s/<->\n//; ba; }'  | grep "iperf3" > $EXPID-ss-processed.txt
    cat $EXPID-ss-processed.txt | awk '{print $1}' > ts-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\bcwnd:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' ' > cwnd-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\brtt:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' '  | cut -d '/' -f 1   > srtt-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\bfd=.*?(\s|$)' |  awk -F '[=,]' '{print $2}' | tr -d ')' | tr -d ' '   > fd-$EXPID.txt

    paste ts-$EXPID.txt fd-$EXPID.txt cwnd-$EXPID.txt srtt-$EXPID.txt -d ',' > $EXPID-ss.csv

    cat $EXPID-iperf.txt | grep "\[ " | grep "sec" | grep -v "0.00-1" | awk '{print $3","$4","$8","$10}' | tr -d ']' | sed 's/-/,/g' > $EXPID-iperf.csv

done
</pre>

This script will produce two CSV files (one with `iperf` data, one with `ss` data), for each of the 16 experiments. The columns in the `iperf` CSV file are: flow ID , timestamp relative to the start of flow, throughput in Mbps and number of retransmissions. The columns in the `ss` CSV file are: timestamps, flow ID , cwnd and smoothed RTT (srtt) in ms. Note that in addition to the ten flow IDs for each of the ten bulk data flows in the experiment, there will be an additional flow ID corresponding to a flow used to share information about the `iperf3` process between sender and receiver; this flow should be excluded from the analysis.

You can use your preferred plotting tool to show throughput vs. time for each of the ten flows, and smoothed RTT vs. time for each of the ten flows, for each of the 16 experiments, to reproduce the figures in the [Results](#results) sections.
