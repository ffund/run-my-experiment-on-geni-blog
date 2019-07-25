In this experiment, we will show how to replicate the key result of the following published paper: 

> Rajeev Kumar, Athanasios Koutsaftis, Fraida Fund, Gaurang Naik, Pei Liu, Yong Liu, Shivendra Panwar, "TCP BBR for Ultra-Low Latency Networking: Challenges, Analysis, and Solutions", in proceedings of IFIP Networking 2019, Warsaw, Poland.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.


### Background

Emerging applications require high throughput and low latency which cannot be supported by current LTE networks. 5G mmWave links promise to support these requirements, and are designed from the ground up for low latency. However, for low end-to-end delay, we will need low latency transport protocols to operate over these links.

Google's TCP BBR is a potential candidate for this use case. (See N. Cardwell et al., "[BBR: congestion-based congestion control](https://queue.acm.org/detail.cfm?id=3022184)", Queue, vol. 14, no. 5, pp. 50:20–50:53, Oct. 2016.) BBR is designed to achieve maximum throughput with minimum latency. It works by estimating the bottleneck link bandwidth, and sending at that rate. It also estimates what the delay should be when there is no congestion, and computes the link's bandwidth delay product (BDP) and the product of this estimated delay and the estimated bottleneck link bandwidth. Then it caps the amount of data in flight at a small multiple of the link's BDP (specifically, 2 x BDP).

TCP BBR has been successfully deployed over Google’s WAN and over other wired networks, with higher throughput and less latency relative to TCP Cubic. However, when we tried BBR over a prototype mmWave backhaul link in our lab, we observed throughput much lower than the link capacity. We were able to confirm delay jitter on the link as the cause of the reduced throughput.

Why does delay jitter cause throughput collapse? Under ordinary circumstances, BBR estimates bandwidth as follows: when a packet Y is sent, we note the ACK that arrived most recently, X. Then when ACK Y arrives, we compute the delivery rate as (Y – X) / (time between ACK X and ACK Y). Then the bandwidth is estimated as the maximum over ten delivery rate samples.

![](/blog/content/images/2019/06/NoCWNDbubble--1--3.svg)

However, when BBR is CWND-limited, there is an idle period that leads to bandwidth estimates that are much lower than the actual link capacity:

![](/blog/content/images/2019/06/CWNDbubble--1--1.svg)

Since CWND is computed as the product of the bandwidth estimate, this leads to CWND dropping lower, more CWND exhaustion while estimating bandwidth, and even lower bandwidth estimates - a feedback loop that results in complete collapse.

The feedback loop that leads to throughput collapse is triggered when BBR becomes CWND-limited as a result of jitter. To find the link BDP, BBR uses the minimum RTT as an estimate of "typical RTT without queueing delay". However, when there is delay variation that is not due to congestion, the minimum RTT can be much less
than "typical RTT without queueing delay". When the minimum RTT is less than half of the "typical RTT without queueing delay", the estimated BDP is half of the "true" link BDP, the CWND will be lower than the "true" link BDP. and BBR becomes CWND limited. 





### Results

In "TCP BBR for Ultra-Low Latency Networking: Challenges, Analysis, and Solutions", we observe that BBR suffers from a "throughput collapse" when the delay variation is high, specifically when the minimum RTT estimate is small enough that the CWND falls below half of the link BDP. In this experiment, you will replicate this result for both the initial release of BBR (BBR 2016) and a recent version (BBR 2019):

![](/blog/content/images/2019/06/bbr-published-results.svg)

### Run my experiment

To replicate the experimental results in the published paper, you will run a series of experiments:

* First, you will [reserve the resources](#reserveresources) you will use for all of the experiments.
* To generate the BBR 2016, 0 ms jitter points in Figure 2, you will follow the instructions in [Run the basic experiment](#runthebasicexperiment).
* To generate the rest of the BBR 2016 points in Figure 2, you will follow the instructions to [vary the emulated delay jitter](#varytheemulateddelayjitter) as you repeat the experiment.
* To generate the BBR 2019 points in Figure 2, you will follow the instructions to [repeat with BBR 2019](#repeatwithbbr2019).

#### Reserve resources

This experiment runs on [CloudLab](https://cloudlab.us/). If you have  GENI account, you can sign in to CloudLab as follows:

* Visit https://cloudlab.us/ and click on "Log In"
* Click on the button that says "Geni User?"
* In the pop-up window, click on the GENI logo
* In the new pop up window, log on to your GENI Portal account
* Authorize CloudLab to use your GENI account

Once you are signed in to CloudLab, reserve resources by visiting the link for our profile, [BBR with RTT variation](https://www.cloudlab.us/p/CloudLab/bbr-with-rtt-variation). Click on "Instantiate", accept the default bottleneck link bandwidth (1Gbps), then choose the "CloudLab-Wisconsin" cluster and start your experiment. Wait until all four nodes have booted and are ready to log in, then click on the "List view" tab to get SSH login instructions for your four nodes. Use SSH to open:

* One terminal window on the emulator
* One terminal window on the TCP sink
* Two terminal windows on the TCP source


#### Run the basic experiment

In the basic experiment, you will generate the BBR 2016, 0 ms jitter points in Figure 2 in the published paper.

First, you will need to set up 5ms fixed delay on the link between the TCP sender and the bottleneck router. On the emulator node, run:

<pre>
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root netem delay 5ms
</pre>

Now you are ready to run the experiment. You will need two SSH sessions connected to the TCP source, and one to the TCP sink.

On the TCP sink, run:

```
iperf3 -s -i 0.1 -f m -1 -V | tee iperf-receiver.txt
```

On the TCP source, shell #1, run:

```
bbrwatch 10.0.3.1 | tee bbrstats.csv
```

And finally, on the TCP source, shell #2, run:

```
iperf3 -c tcp-sink -f m -i 0.1 -C bbr -t 90 -V | tee iperf-sender.txt
```

Your experiment will conclude after 90 seconds. When it does, use Ctrl+C to stop any running processes. 

Then, use `scp` to retrieve the data files (`iperf-receiver.txt` on the TCP sink, `bbrstats.csv` and `iperf-sender.txt` on the TCP source) from the testbed. Compute the "Average TCP throughput" for this experiment from the `iperf-receiver.txt` and `iperf-sender.txt` files; compute the average CWND, BBR bandwidth estimate, and BBR minimum RTT estimate from the `bbrstats.csv` file. The columns in the `bbrstats.csv` file are: (1) timestamp, (2) CWND, (3) SRTT (smoothed round-trip time), (4) RTTVAR (round-trip time variation), (5) BBR bandwidth estimate, (6) BBR minimum RTT estimate, (7) BBR pacing gain, (8) BBR CWND gain.


#### Vary the emulated delay jitter

To generate the rest of the BBR 2016 data points in Figure 2 in the published paper (the red lines), we will add delay jitter. 

On the emulator node, run:

<pre>
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root netem delay 5ms <b>0.25ms</b> dist normal
</pre>

to add 5ms fixed delay with 0.25ms normally distributed jitter on the link between the TCP sender and the bottleneck router.


Then we can repeat the experiment by running the `iperf3` and `bbrwatch` commands in the previous section. After each experiment, retrieve the data files with `scp`, and compute the average TCP throughput and CWND, BBR bandwidth estimate, and BBR minimum RTT estimate.

To generate the rest of the data points, repeat this procedure for six values of delay jitter: 0.25ms, 0.5ms, 0.75ms, 1ms, 1.25ms, 1.5ms

#### Repeat with BBR 2019

Finally, we can generate the data points for the more recent BBR version. T change the BBR version  to BBR 2019, run:

```
sudo grub-set-default "Advanced options for Ubuntu>Ubuntu, with Linux 5.1.0-rc3+"
sudo update-grub
sudo reboot
```

on the TCP source node.

After it reboots, run

```
uname -r
```

to verify that it has booted into the 5.1 kernel.

Then, repeat the basic experiment and the experiment with delay jitter to generate the blue lines in Figure 2 of the published paper.


<blockquote>
<b>Note</b>: To change back to BBR-2016, you can run 
<pre>sudo grub-set-default "Advanced options for Ubuntu>Ubuntu, with Linux 4.12.0+"
sudo update-grub
sudo reboot
</pre>
</blockquote>

