## Background

We are looking at the paper [``Are we heading towards a BBR-dominant internet?''](https://dl.acm.org/doi/abs/10.1145/3517745.3561429)

The main contribution of this work is a new mathematical model to estimate the throughput of TCP BBR when competing with TCP CUBIC flows. Using their model, the authors show that 

> even though BBR currently has a throughput advantage over CUBIC, this advantage will be diminished as the proportion of BBR flows increases.

What this means for the future of the internet's transport layer is that TCP traffic will reach a stable mixed distribution of BBR and CUBIC flows. This will be a Nash equilibrium, where none of the flows will be incentivised to switch their congestion control protocol. If true, this would mean that 

> BBR is unlikely to replace CUBIC on the internet in the near future. 

To validate their model's predictions, the authors also execute several testbed experiments in their paper to experimentally analyze the competition between TCP CUBIC and TCP BBR (or other low latency congestion control protocols). In this blog, we will reproduce a subset of the experimental results from this paper.

## Target Result

We want to reproduce the following results from the paper:

- **Figure 8**

![](/blog/content/images/2022/12/Screen-Shot-2022-12-09-at-3.39.32-PM-2.png)

Figure 8 above shows the results of an experiment with 10 long flows running for a duration of 120 seconds. They pass through a bottleneck link of capacity 100 Mbps, with a 2 BDP buffer and 40 ms base RTT. The number of BBR flows is varied from 1 to 10, while the rest of the flows use the loss-based TCP CUBIC protocol. 

This result verifies the prediction made by the model proposed in the paper. As the proportion of BBR flows in the network increase, the average per-flow throughput of BBR goes down. After a certain point, TCP CUBIC starts getting higher per-flow throughput. It is also interesting to note that the queuing delay is mostly unaffected by the proportion of BBR vs CUBIC flows, except when all flows are TCP BBR. Even though BBR is a low latency congestion control that tries to minimize queuing delay, it cannot do so in the presence of competing CUBIC flows. 

- **Figure 7** 
![](/blog/content/images/2022/12/Screen-Shot-2022-12-09-at-7.38.06-PM.png)

Figure 7 above shows the results of a similar experiment with 10 long flows running for a duration of 120 seconds. They pass through a bottleneck link of capacity 100 Mbps, with a 2 BDP buffer and 40 ms base RTT. The number of low-latency TCP flows varies from 1 to 10, while the rest use the loss-based TCP Cubic protocol. The low-latency TCP protocols tested are PCC-Vivace, TCP BBR, TCP BBR v2, and TCP Copa. It is seen here that most of the low latency congestion controls have similar coexistence behavior to the one observed for TCP BBR, except for the delay-based congestion control, Copa. 

The per-flow throughput for Copa flows is much lower than the fair share point for this experiment setting, i.e., 10 Mbps. This is in line with previous observations for other delay-based congestion control protocols, which have been found to perform poorly when coexisting with loss-based TCP, hindering their deployment in the public internet. For a more detailed analysis of the coexistence behavior of delay-based congestion control, please refer to [our blog](https://witestlab.poly.edu/blog/coexistence/) and [our recently published paper](https://ieeexplore.ieee.org/abstract/document/9918593) on this topic.

## Run my experiment

To run these experiments, you will need an account on [CloudLab](https://cloudlab.us/). If you already have a GENI account, you can use it to log in to CloudLab. Once you are logged in to CloudLab, open our  profile named `bbr-cubic-coexistence`: [https://www.cloudlab.us/p/a92003a65cac52a65965623af9a895414ed4bce9](https://www.cloudlab.us/p/a92003a65cac52a65965623af9a895414ed4bce9)

Click on ``Instantiate``, then choose CloudLab Wisconsin and start your experiment. Wait until all nodes have booted and are ready to log in, then click on the ``List view`` tab to get SSH login instructions for your nodes. Then, use SSH to open a shell session on each node in the experiment.

After instantiating the experiment on Cloudlab, follow these instructions : 

First, we install software tools (`iperf3`, `moreutils`) needed to run this experiment. 

On the `src` and `dstn` nodes, do:
```
sudo apt update; sudo apt -y install iperf3
```

On the `src`, `router` and `dstn` nodes, do:

```
sudo apt -y install moreutils
```

Use `htb` to create a bottleneck link with capacity 100 Mbps between the `router` and the `dstn` node. Also, set the bottleneck queue size to 1 MB.

On the `router` node:
```
sudo tc qdisc del dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root 
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root handle 1: htb default 3 
sudo tc class add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1: classid 1:3 htb rate 100mbit
sudo tc qdisc add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 1000000
```
Use `netem` to add a based delay of 40 ms at the `emulator` node.

```
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root netem delay 40ms 
```
#### Running the experiment for TCP BBR

We will start with the first version of the TCP BBR congestion control protocol, also known as BBRv1. 

On `src`:
```
sudo modprobe tcp_bbr
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



Open three terminals on the `src` node, and one on the `router` node.

On each of the four open terminals, set the number of BBR flows, i.e., `n1`, and the number of Cubic flows, i.e., `n2 = 10 - n1`:

```
n1=1;n2=$(expr 10 - $n1)
EXPID=bbr-$n1
```

First enter the following set of commands on the respective nodes, without actually executing them:

* First terminal of `src` node (C1):
```
iperf3 -f m -c dstn -C bbr -P $n1 -i 0.1 -t 120 -p 5201 | ts '%.s' | tee bbr-$n1-iperf.txt
```
* Second terminal of `src` node (C2):
```
iperf3 -f m -c dstn -C cubic -P $n2 -i 0.1 -t 120 -p 5202 | ts '%.s' | tee cubic-$n1-iperf.txt
```

* Third terminal of `src` node (C3):
```
rm -f "$EXPID"-ss.txt; while true; do ss --no-header -eipn dst 10.0.3.1 | ts '%.s' | tee -a "$EXPID"-ss.txt; sleep 0.1; done
```
* Terminal on `router` node (C4):
```
rm "$EXPID"-queue.txt; while true; do tc -p -s -d qdisc show dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") | tr -d '\n' | ts '%.s' | tee -a "$EXPID"-queue.txt; echo "" | tee -a "$EXPID"-queue.txt; sleep 0.1; done
```

Now execute the above set of commands one-by-one, in quick succession, in the following order : C3 -> C4 -> C2 -> C1

This single experiment runs for 120 seconds. Wait for the experiment to finish. Once these commands finish running, manually shut down (using Ctrl+C) the commands running in terminals 3 and 4 (C3 and C4).

You should now be able to see that the data from this experiment (n1=1) will be stored in: 

* `bbr-1-iperf.txt` on the `src` node: `iperf` statistics of the BBR flows in the experiment.
* `cubic-1-iperf.txt` on the `src` node: `iperf` statistics of the CUBIC flows in the experiment.
* `bbr-1-ss.txt` on the `src` node: `ss` statistics of all the TCP flows in the experiment.
* `bbr-6-queue.txt` on the `router` node: queue backlog statistics during the experiment.

Now, you can change the parameters `n1` and `n2` and re-run the experiment to get similar data for different values of `n1`, i.e. number of non-CUBIC flows at the bottleneck (x-axis of Figure 8 in the paper). For example, for `n1=2`, do:
```
n1=10;n2=$(expr 10 - $n1)
EXPID=bbr-$n1
```

Once you have completed the experiment, runs for `n1 = 1,2, ..., 10`; you can use the following data processing scripts.


On the `router` node:
```
for EXPID in bbr-1 bbr-2 bbr-3 bbr-4 bbr-5 bbr-6 bbr-7 bbr-8 bbr-9 bbr-10
do
    cat "$EXPID-queue.txt" | awk '{print $1","$24$30","$31}' | tr -d 'b' | tr -d 'p' | sed 's/K/000/' | sed 's/M/000000/' > "$EXPID-queue.csv"
done
```

This will create a file for each experiment ID whose columns are: timestamp, number of packets dropped, queue backlog in bytes, and queue backlog in packets. This can be used to plot the result of Figure 8(b), i.e., average queueing delay. 

For reproducing figure 8(a), you need the average per-flow throughput of the TCP BBR and TCP CUBIC flows, which can be directly taken from the `iperf` data. For further processing of the `iperf` and `ss` data, you may use the following scripts.


First, on the `src` node:
```
for EXPID in bbr-1 bbr-2 bbr-3 bbr-4 bbr-5 bbr-6 bbr-7 bbr-8 bbr-9 bbr-10
do

    cat $EXPID-ss.txt | sed -e ':a; /<->$/ { N; s/<->\n//; ba; }'  | grep "iperf3" > $EXPID-ss-processed.txt
    cat $EXPID-ss-processed.txt | awk '{print $1}' > ts-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\bcwnd:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' ' > cwnd-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\brtt:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' '  | cut -d '/' -f 1   > srtt-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\bfd=.*?(\s|$)' |  awk -F '[=,]' '{print $2}' | tr -d ')' | tr -d ' '   > fd-$EXPID.txt

    paste ts-$EXPID.txt fd-$EXPID.txt cwnd-$EXPID.txt srtt-$EXPID.txt -d ',' > $EXPID-ss.csv

    cat $EXPID-iperf.txt | grep "\[ " | grep "sec" | grep -v "0.00-1" | awk '{print $3","$4","$8","$10}' | tr -d ']' | sed 's/-/,/g' > $EXPID-iperf.csv

done

```

This will create a file for each experiment ID whose columns are: timestamp, file descriptor of flow, CWND, and smoothed RTT in ms. It will also create another file for each experiment ID whose columns are: file descriptor of flow, relative timestamp from beginning of flow at beginning of interval, relative timestamp at end of interval, throughput in Mbps, and number of retransmissions.

Again on the `src` node:

```
for EXPID in cubic-1 cubic-2 cubic-3 cubic-4 cubic-5 cubic-6 cubic-7 cubic-8 cubic-9
do

    cat $EXPID-iperf.txt | grep "\[ " | grep "sec" | grep -v "0.00-1" | awk '{print $3","$4","$8","$10}' | tr -d ']' | sed 's/-/,/g' > $EXPID-iperf.csv

done 

```
This will do a similar processing of the `iperf` data for the TCP CUBIC flows in the experiment.

#### Running the experiment with other low latency congestion control protocols

##### TCP Vegas

Instead of the delay-based TCP Copa used in the paper, we will use another popular delay-based congestion control, i.e., TCP Vegas. On the `src` node, run the following commands to enable the use of TCP Vegas:

```
sudo modprobe tcp_vegas
sudo sysctl -w net.ipv4.tcp_congestion_control=vegas
sudo sysctl -w net.ipv4.tcp_congestion_control=cubic
```

The rest of the experiment follows the same instructions as the BBR experiment. Please do not forget to change the congestion control from `bbr` to `vegas` in the following commands:
```
EXPID=vegas-$n1
```
```
iperf3 -f m -c dstn -C vegas -P $n1 -i 0.1 -t 120 -p 5201 | ts '%.s' | tee vegas-$n1-iperf.txt
```
You would also need to make this change in the data analysis script, if used. 

##### TCP BBR v2

To change the congestion control, you will need to boot the TCP sender into a different kernel. Three kernels are installed on the TCP sender:

* The default kernel is the 5.3-rc3 kernel, which you can use for TCP BBR and TCP Vegas experiments.
* To run experiments with TCP BBR v2, boot into the 5.2-rc3 kernel.
* To run experiments with PCC-Vivace, boot into the 4.15.0-55-generic kernel.

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

Then you can choose the new default kernel, e.g. for switching to the BBR2 kernel run, 

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

Once you have done this, add the BBRv2 congestion control module using:

```
sudo modprobe tcp_bbr2
```
The rest of the instructions for the experiment remain the same as the `BBRv1` experiment. Please change the congestion control name wherever appropriate in the above set of instructions.


##### PCC-Vivace
Boot into the `4.15.0-55-generic` following the same instructions as in the previous subsection (BBRv2). 

Next, we will install the PCC linux kernel module from the [PCCProject Github repository](https://github.com/PCCproject/PCC-Kernel)

```
sudo git clone https://github.com/PCCproject/PCC-Kernel.git
cd PCC-Kernel
sudo git checkout vivace
cd src
sudo make
sudo insmod tcp_pcc.ko
```

Once you have done this, you will be able to use the PCC-Vivace congestion control protocol while running `iperf` :

```
iperf3 -f m -c dstn -C pcc -P $n1 -i 0.1 -t 120 -p 5201 | ts '%.s' | tee vegas-$n1-iperf.txt
```

The rest of the experiment follows the same set of instructions as before. 



## Results

The results obtained from our experiment on the Cloudlab testbed are shown below.

- **Figure 8(a)**
![](/blog/content/images/2022/12/BBR_tput.png)
The plot obtained from our Cloudlab testbed experiment closely matches the corresponding figure from the reference paper.

- **Figure 8(b)**
![](/blog/content/images/2022/12/BBR_queue.png)
Here, we observe two key differences as compared to the original paper:

-  The absolute queuing delay values are much lower. Given  the experiment conditions, i.e., 40 ms base RTT and 2 BDP buffer, the maximum queuing delay at the bottleneck should not exceed 80 ms. This is in line with what we observe in our experiment, but in the paper, the authors observe queuing delay values as high as 100 ms.
- Although the queuing delay drops at `n1=10` (when all flows use TCP BBR), the drop is not as significant as the one observed in the paper.

- **Figure 7**
![](/blog/content/images/2022/12/tput_all.png)
The plot obtained from our Cloudlab testbed experiment closely matches the corresponding figure from the reference paper.