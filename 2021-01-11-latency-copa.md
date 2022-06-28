
As voice- and video-based services that require low latency become increasingly essential, a tremendous research effort has been devoted to improving the latency performance of networks. However, there remains a "latency divide" between those whose Internet access is subject to high latency and those with low-latency access. Furthermore, it is unclear whether the aforementioned research efforts narrow this divide (by improving the latency performance of poor-quality connections the most), maintain the divide (by improving poor- and high-quality connections equally), or even widen the divide (by mostly reducing latency of high-quality network connections). In this work, we design a reproducible experiment for evaluating the performance of Copa, a delay-based congestion control protocol, in different categories of residential broadband connections. We use empirical data from U.S. home network connections to evaluate whether Copa leads to more equitable network access. We confirm that Copa has potential to narrow the ``digital divide'' by improving latency performance for users with a low-quality residential broadband connection, although care must be taken to set appropriate parameters for best performance.


This experiment will take approximately one hour set up, and then approximately 15 minutes for each experiment "run" (each household profile).

To reproduce this experiment, you will need an account on [CloudLab](https://cloudlab.us/), and you should have some basic familiarity with the platform. For more details, please refer to the [CloudLab user manual](https://docs.cloudlab.us/index.html).


## Background

Internet connectivity is essential in the modern world: for access to information, work, education, public services, and entertainment. However, high-quality Internet access is _not_ universal, and those with no network connection or a poor network connection suffer because of it. This phenomenon is known as the _digital divide_, a term that describes the significant disadvantage those without high-quality Internet access have in today’s society. Especially during the COVID-19 pandemic, when essential services such as healthcare are accessed through virtual platforms, it is more important than ever to bridge the divide.

Access to high-quality Internet access is often defined in terms of data rates (for example: what percent of U.S. households are within the service area of a provider that offers access according to the FCC definition of broadband,  25 Mbps down/3 Mbps up). In this research, however, we focus specifically on the _latency_ aspect of the digital divide. Latency is essential for Internet usability; in particular, high latency negatively impacts the performance of the "live" voice and video services that so many currently rely on. A great deal of research effort has been devoted recently to reducing latency in the Internet. However, for many of the proposed techniques, it is not clear whether they narrow the digital divide (by improving the latency performance of poor-quality connections the most), maintain the divide (by improving poor-quality and high-quality connections equally), or even widen the digital divide (by mostly improving the latency performance of high-quality network connections).


### Copa

Copa [[1]](#references) is a delay-based congestion control protocol that aims to reduce latency. Based on the statistics of a flow, Copa tries to:

1. identify a *target rate* that optimizes a function of throughput and queuing delay, and
2. adjust the congestion window to move towards that target rate.

In the original Copa paper, the authors present experimental results showing similar throughput but much lower delay than CUBIC (a standard loss-based congestion control that is the current default in the Linux kernel), and better fairness and less delay than BBR and PCC (two recently proposed low-delay congestion control protocols). However, while their experiments represent a wide variety of networks (including emulated datacenter networks and satellite links, and real Internet paths using both wired and cellular networks), it is not clear from their results under what conditions the benefit of Copa is _greatest_. Are the greatest benefits of Copa realized on network links that are already high quality (high capacity, low latency, and low loss), or on network links that are poor quality? Does Copa widen the digital divide or narrow it?

![](/blog/content/images/2021/01/copa-original-results.svg)
<center><small><i>Figure 1: Results from the original Copa paper show benefits in a wide variety of network environments. Image from [[1]](#references).</i></small></center>

### Facebook experiment

A recent evaluation at Facebook (presented at IETF 106 [[2]](#references) and in the Facebook Engineering blog [[3]](#references)), sheds some light on this issue.  Facebook added the Copa congestion control algorithm to their _mvfst_ [[4]](#references)  implementation of QUIC, a UDP-based transport protocol. (CUBIC and BBR are also implemented in _mvfst_.) Using this implementation, they conducted a series of A/B tests on Facebook Live users around the world, with users randomly assigned a congestion control. Facebook Live is a product that enables users to stream live video to friends; low latency is essential for a good quality of experience. 

The results of these experiments suggested that Copa yields the greatest benefits for users with the "worst" connections. The following two figures show the transport RTT of the QUIC flows for users with the "best" connections (10th percentile, 25th percentile, and 50th percentile) and for users with the "worst" connections (75th percentile, 90th percentile, and 95th percentile). For users who already had very low latency connections (the 10th percentile and 25th percentile connections), the transport RTT was similar with CUBIC, BBR, and Copa. For users with a poor network connection, however, the transport RTT was dramatically lower with Copa compared to CUBIC. For the 75th percentile connections, Copa reduced transport RTT by 35% relative to CUBIC; for the 95th percentile connections (the 5% of users with the worst connections), Copa reduced transport RTT by 45% relative to CUBIC!

![](/blog/content/images/2021/01/transport-rtt-best.jpg)
<center><small><i>Figure 2: In the Facebook Live experiments, Copa has minimal effect on RTT for users who already have low-latency connections. Image from [[3]](#references).</i></small></center>

![](/blog/content/images/2021/01/transport-rtt-worst.jpg)
<center><small><i>Figure 3: In the Facebook Live experiments, Copa reduced transport RTT dramatically for users with the worst Internet connections. Image from [[3]](#references).</i></small></center>

Based on these results, Copa may be a promising tool to narrow the "digital divide" and improve latency performance for users with low-quality network connections. The first goal of our research was therefore to confirm this result in a controlled, reproducible experiment on a research testbed.

### Effect of δ

A secondary goal of our research was to explore the parameter settings for the latency factor, δ, in Copa. Recall that Copa seeks to optimize a function of delay and throughput, _U_:

$$ U = \log (\lambda) - \delta \log (d)$$

where λ is the flow throughput and _d_ is the total delay minus propagation delay. The latency factor δ may be set to any value in the range 0 to 1, and determines how much to prioritize low delay vs. high throughput, as shown in the following figure:

![](/blog/content/images/2021/01/copa-fb-delta.jpg)
<center><small><i>Figure 4: Effect of δ on throughput and latency of a Copa flow. Image from [[3]](#references).</i></small></center>

With a lower value of δ, Copa will focus more on maximizing throughput, and with a higher value, the protocol will work to minimize delay.

In the Facebook Live experiment, they used a value of δ=0.04, which they "found to provide a reasonable quality vs. latency trade-off for the application" [[3]](#references). (With this value of δ, the bottleneck queue should have a steady state queue length of 1/δ = 25 packets.)  However, in evaluating the extent to which Copa can narrow the "digital divide", we also seek to measure the effect of δ on different types of connections - should users with a poor connection use a different value of δ than those with a strong connection?

(Note that Copa also has a competitive mode, which seeks to adapt the value of δ so that it competes effectively with other congestion controls sharing a bottleneck link. We did not evaluate competitive mode in this experiment.)


### Experiment design

To address the research questions described above, we implement and execute an experiment on the CloudLab testbed [[5]](#references). Our experiment includes four nodes in a linear topology, connected by 1 Gbps links: a server node with _mvfst_, a gateway node that implements the bottleneck queue, a delay emulation node, and finally a client node also with _mvfst_.

To investigate a range of households from both sides of the "divide:, we use the "digital divide" utility described in [[6]](#references).  This tool uses data from FCC and the U.S. Census Current Population Survey Computer and Internet Use Supplement, and produces `tc` commands that can be used to make the link between client and server nodes mimic the residential broadband connection of a randomly sampled U.S. household. Furthermore, we can specify the price range from which we want to sample households. To understand the impact of Copa on households on either side of the "divide", we sampled approximately 10 households from each of four price ranges (estimated monthly cost of service): $0-30, $30-40, $40-50, and $50+. Some statistics of the randomly sampled households are shown in Table I; also provided are the household identifiers, which may be used to reproduce exactly the results of our research.



<table>
<thead>
  <tr>
    <th>Price range ($)</th>
    <th>Uplink rate (Mbps)</th>
    <th>Downlink rate (Mbps)</th>
    <th>Base RTT (ms)</th>
    <th>Household IDs</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>$0-30</td>
    <td>1.1</td>
    <td>3.2</td>
    <td>31</td>
    <td>799254, 11406, 13570, 7884, 797760, 13013, 28928, 9747, 799322, 32648</td>
  </tr>
  <tr>
    <td>$30-40</td>
    <td>0.79</td>
    <td>4.0</td>
    <td>32 <sup>*</sup></td>
    <td>9254, 615514, 615016, 12512, 6046, 216020, 10232, 10428, 10373, 615520</td>
  </tr>
  <tr>
    <td>$40-50</td>
    <td>1.9</td>
    <td>15</td>
    <td>22</td>
    <td>27082, 5781, 12334, 619832, 1290, 12491, 9960, 13133, 215248</td>
  </tr>
  <tr>
    <td>$50+</td>
    <td>4.4</td>
    <td>35</td>
    <td>26</td>
    <td>8268, 214030, 5749, 12430, 27337, 11750, 734, 32501, 12251</td>
  </tr>
</tbody>
</table>
<center><small><i>Table I: Mean uplink rate, downlink rate, and base RTT of the sampled households in each price range, using the digital divide utility [[6]](#references). The household identifiers may be used to reproduce exactly the results of our research. <br> <sup>*</sup> Mean excludes one outlier household with extreme delay.</i></small></center>

In addition to setting the uplink and downlink rate and base delay, we also set up a FIFO queue with capacity of 2BDP at the bottleneck router.

For each of the ~40 household profiles, we used a bulk transfer utility included in _mvfst_ to transfer data across the link for 60 seconds. This experiment was repeated for each of four different congestion control variants: CUBIC, Copa with δ=0.2, Copa with δ=0.5, and Copa with δ=0.8. For each experiment trial, we collected data on flow throughput and delay (as measured by a concurrent `ping`). 

## Results

The following figure shows the mean self-induced delay (i.e. queueing delay) and the mean throughput normalized to link capacity, for each of the four congestion control variants and four price ranges. 


![](/blog/content/images/2021/03/copa-results-more.svg)

For households in the least expensive price range, Copa reduced the self-induced delay with only a moderate decrease in throughput. The effect on self-induced delay was much smaller for the households in the more expensive price ranges. Furthermore, for household profiles in the highest price ranges, we observed the negative effect on throughput was greater. It is not clear to what extent this effect is due to issues with the _mvfst_ implementation or its bulk transfer utility, and to what extent it is due to the congestion control protocol. 

The results further show that the latency factor, δ, is not a "one size fits all" parameter. For household profiles in the less expensive price ranges, a _large_ value of δ was necessary to achieve low delay. For household profiles in the more expensive price ranges, a _small_ value of δ was necessary to achieve a relatively high throughput. 

## Run my experiment

## Reserve resources

To reserve the resources needed for this experiment, you will need a CloudLab account.  Then, instantiate an experiment on the CloudLab Wisconsin cluster, using the profile: [copa-delay-gw](https://www.cloudlab.us/p/CloudLab/copa-delay-gw). You may need to reserve resources in advance, if the testbed is experiencing heavy load.

## Setup

### On client and server

Clone the `mvfst` repository:

```
git clone https://github.com/facebookincubator/mvfst.git
cd mvfst
sudo apt update
```

on each.

Install the dependencies needed for `mvfst`:

```
sudo apt-get install         \
    g++                      \
    cmake                    \
    libboost-all-dev         \
    libevent-dev             \
    libdouble-conversion-dev \
    libgoogle-glog-dev       \
    libgflags-dev            \
    libiberty-dev            \
    liblz4-dev               \
    liblzma-dev              \
    libsnappy-dev            \
    make                     \
    zlib1g-dev               \
    binutils-dev             \
    libjemalloc-dev          \
    libssl-dev               \
    pkg-config               \
    libsodium-dev
```
Run the build helper script to finish the `mvfst` install:

```
./build_helper.sh
```

Come out of the `mvfst` directory back to your home directory using `cd` and save [this bash script](https://gist.github.com/ffund/c9d40b303944e8056e22f2c4cd91b9f4) in a file named `copa_test`:

<pre>
wget -O copa_test https://gist.githubusercontent.com/ffund/c9d40b303944e8056e22f2c4cd91b9f4/raw/bef6ce1aeedc15981a65318c2574e222e8d46207/copa_test.sh
</pre>

On the server, change the IP address 10.10.2.2 to 10.10.1.1 wherever it occurs in the `copa_test` file:

```
sed -i 's/10.10.2.2/10.10.1.1/g' copa_test
```

Also, set up the digital divide tool:

```
git clone https://github.com/csmithsalzberg/digitaldivide.git
cd digitaldivide
cd util
bash install.sh
cd ~
```

### On gateway

Set up the digital divide tool:

```
git clone https://github.com/csmithsalzberg/digitaldivide.git
cd digitaldivide
cd util
bash install.sh
cd ~
```

Save the following bash script as `setup_queues.sh` (come out of the digital divide directory before doing this using `cd`) :

```
#!/bin/bash

household="$1"
if [ "$#" -ne 1 ]; then
        echo "Household ID needed to run setup_queues."
        exit
fi

cd ~/digitaldivide
capup=$(python src/digitaldivideutil.py --houseid $household | grep "Upload rate" | cut -d" " -f9)
capdown=$(python src/digitaldivideutil.py --houseid $household | grep "Download rate" | cut -d" " -f7)
latency=$(python src/digitaldivideutil.py --houseid $household | grep "Round-trip delay" | cut -d" " -f6)
cd

ifaceup=$(ip route get 10.10.2.2 | head -n 1 | cut -d" " -f 3)
ifacedown=$(ip route get 10.10.1.1 | head -n 1 | cut -d" " -f 5)
owd=$(bc <<< "scale=2; $latency/2") # in ms
qfactor=2
qup=$(bc <<< "scale=2; $qfactor*$latency*$capup/1000.0") # in kbit
qdown=$(bc <<< "scale=2; $qfactor*$latency*$capdown/1000.0") # in kbit

echo "Latency: $latency"
echo "Uplink: $capup on $ifaceup, $qup queue"
echo "Downlink: $capdown on $ifacedown, $qdown queue"

# Uplink direction
## Delete any existing queues
sudo tc qdisc del dev $ifaceup root
## Create an htb qdisc
sudo tc qdisc replace dev $ifaceup root handle 1: htb default 3
## Limit the queue traffic to the upload rates
sudo tc class add dev $ifaceup parent 1: classid 1:3 htb rate "$capup"kbit
## Set up queue limit
sudo tc qdisc add dev $ifaceup parent 1:3 bfifo limit "$qup"kbit

# Downlink direction
## Delete any existing queues
sudo tc qdisc del dev $ifacedown root
## Create an htb qdisc
sudo tc qdisc replace dev $ifacedown root handle 1: htb default 3
## Limit the queue traffic to the upload rates
sudo tc class add dev $ifacedown parent 1: classid 1:3 htb rate "$capdown"kbit
## Set up queue limit
sudo tc qdisc add dev $ifacedown parent 1:3 bfifo limit "$qdown"kbit
```

Also prepare to save the queue backlog. Run

```
sudo apt update; sudo apt -y install moreutils
```

### On delay

Set up the digital divide tool:

```
git clone https://github.com/csmithsalzberg/digitaldivide.git
cd digitaldivide
cd util
bash install.sh
cd ~
```

Save the following as `setup_delay.sh`:

```
#!/bin/bash

household="$1"
if [ "$#" -ne 1 ]; then
        echo "Household ID needed to run setup_delay."
        exit
fi

cd ~/digitaldivide
capup=$(python src/digitaldivideutil.py --houseid $household | grep "Upload rate" | cut -d" " -f9)
capdown=$(python src/digitaldivideutil.py --houseid $household | grep "Download rate" | cut -d" " -f7)
latency=$(python src/digitaldivideutil.py --houseid $household | grep "Round-trip delay" | cut -d" " -f6)
cd

ifaceup=$(ip route get 10.10.2.2 | head -n 1 | cut -d" " -f 5)
ifacedown=$(ip route get 10.10.1.1 | head -n 1 | cut -d" " -f 3)
owd=$(bc <<< "scale=2; $latency/2") # in ms
qfactor=2
qup=$(bc <<< "scale=2; $qfactor*$latency*$capup/1000.0") # in kbit
qdown=$(bc <<< "scale=2; $qfactor*$latency*$capdown/1000.0") # in kbit

sudo tc qdisc del dev $ifaceup root
sudo tc qdisc del dev $ifacedown root
sudo tc qdisc add dev $ifaceup root netem delay "$owd"ms limit 50000
sudo tc qdisc add dev $ifacedown root netem delay "$owd"ms limit 50000
```

### Run experiment

First, configure the link. On the gateway, run the digital divide tool:

```
cd ~/digitaldivide; python src/digitaldivideutil.py | tee household.txt
```
The output of the above command will show a household ID and the characteristics of the internet connection of that particular household. Make a note of the household ID.

Then run 

```
bash setup_queues.sh HOUSEID
```

(substituting value above).

On the delay host,

```
bash setup_delay.sh HOUSEID
```

(again, substituting the value).

#### Uplink experiment

On the server, open 4 terminal windows (or 4 screen sessions!) and start a tperf server on each:

For Cubic:

<pre>
cd mvfst/_build/build/quic/tools/tperf
./tperf -mode server -host 10.10.2.2 -port 5550  -congestion=cubic 
</pre>

For Copa with delay factor 0.2:

<pre>
cd mvfst/_build/build/quic/tools/tperf
./tperf -mode server -host 10.10.2.2 -port 5552  -congestion=copa -latency_factor=0.2
</pre>

For Copa with delay factor 0.5:

<pre>
cd mvfst/_build/build/quic/tools/tperf
./tperf -mode server -host 10.10.2.2 -port 5555  -congestion=copa -latency_factor=0.5
</pre>

For Copa with delay factor 0.8:

<pre>
cd mvfst/_build/build/quic/tools/tperf
./tperf -mode server -host 10.10.2.2 -port 5558 -congestion=copa -latency_factor=0.8
</pre>

On the gateway, start monitoring the uplink queue:

```
EXPID=queue-1      # you can change the 1 to your household number

rm "$EXPID"-queue.txt; while true; do tc -p -s -d qdisc show dev $(ip route get 10.10.1.1 | grep -oP "(?<=dev )[^ ]+") | tr -d '\n' | ts '%.s' | tee -a "$EXPID"-queue.txt; echo "" | tee -a "$EXPID"-queue.txt; sleep 0.1; done
```

On the client, begin the Copa test by running

```
bash copa_test HOUSEID
```

When it finishes, stop the queue monitoring with Ctrl+C.

#### Downlink experiment

On the client, open 4 terminal windows (or 4 screen sessions!) and start a tperf server on each by running

For Cubic:
c
<pre>
cd mvfst/_build/build/quic/tools/tperf
./tperf -mode server -host 10.10.2.2 -port 5550  -congestion=cubic 
</pre>

For Copa with delay factor 0.2:

<pre>
cd mvfst/_build/build/quic/tools/tperf
./tperf -mode server -host 10.10.2.2 -port 5552  -congestion=copa -latency_factor=0.2
</pre>

For Copa with delay factor 0.5:

<pre>
cd mvfst/_build/build/quic/tools/tperf
./tperf -mode server -host 10.10.2.2 -port 5555  -congestion=copa -latency_factor=0.5
</pre>

For Copa with delay factor 0.8:

<pre>
cd mvfst/_build/build/quic/tools/tperf
./tperf -mode server -host 10.10.2.2 -port 5558 -congestion=copa -latency_factor=0.8
</pre>


On the gateway, start monitoring the downlink queue:

<pre>
EXPID=queue-1      # Change the 1 to your household number

rm "$EXPID"-queue.txt; while true; do tc -p -s -d qdisc show dev $(ip route get 10.10.2.2 | grep -oP "(?<=dev )[^ ]+") | tr -d '\n' | ts '%.s' | tee -a "$EXPID"-queue.txt; echo "" | tee -a "$EXPID"-queue.txt; sleep 0.1; done
</pre>

On the server, begin the Copa test by running

```
bash copa_test HOUSEID
```

When it finishes, stop the queue monitoring with Ctrl+C.

Save all data.


## Notes

This post describes joint work with Ashutosh Srivastava, Fraida Fund, and Shivendra Panwar.

This research is supported by the New York Center for Advanced Technology in Telecommunications (CATT), by the Center for K12 STEM Education at NYU Tandon, and by the Pinkerton Foundation.


### References

[1] Venkat Arun and Hari Balakrishnan. "Copa: Practical delay-based congestion control for the internet." 15th USENIX Symposium on Networked Systems Design and Implementation (NSDI 18). 2018. [PDF](https://www.usenix.org/system/files/conference/nsdi18/nsdi18-arun.pdf).

[2] Nitin Garg. "Comparing COPA with CUBIC, BBR for
live video upload." Presentation at IETF 106, Internet Congestion Control Research Group session, November 2019. [Video](https://youtu.be/i3CpETXwA7Q?t=4053), [Slides](https://datatracker.ietf.org/meeting/106/materials/slides-106-iccrg-experiments-at-facebook-with-copa-00).

[3] Nitin Garg. "Evaluating COPA congestion control for improved video performance". Facebook Engineering Blog, November 2019. [URL](https://engineering.fb.com/2019/11/17/video-engineering/copa/).

[4] Facebook _mvfst_   GitHub   repository.   https://github.com/facebookincubator/mvfst

[5] R. Ricci, E. Eide, and C. Team, "Introducing CloudLab: Scientific infras-tructure for advancing cloud architectures and applications,". login:: the magazine of USENIX & SAGE, vol. 39, no. 6, pp. 36–38, 2014. [CloudLab.us](https://cloudlab.us/).

[6] Caleb Smith-Salzberg, Fraida Fund, and Shivendra Panwar. "Bridging the digital divide between research and home networks." 2017 IEEE 2017 INFOCOM International Workshop on Computer and Networking Experimental Research Using Testbeds (CNERT ’17). [PDF](http://catt.poly.edu/~panwar/publications/Bridging%20the%20digital%20divide%20between%20research%20and%20home%20networks.pdf).