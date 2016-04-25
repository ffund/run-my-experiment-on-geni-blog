# TCP congestion control in lossy wireless networks

This experiment shows some of the basic issues affecting TCP congestion control in lossy wireless networks. 

It should take about 60-90 minutes to run, but you will need to have [reserved that time](http://witestlab.poly.edu/respond/sites/witest/tutorial/make-a-reservation) in advance. This experiment uses wireless resources (specifically, on the [WITest](http://witestlab.poly.edu/) testbed), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have [reserved time on the WITest testbed](http://witestlab.poly.edu/respond/sites/witest/tutorial/make-a-reservation), and you must run this experiment during your reserved time.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

TCP is arguably one of the most important Internet protocols, as it carries a much higher volume of traffic on the Internet than any other transport-layer protocol.

Because TCP carries so much traffic, its congestion control algorithm is the main technique which prevents the Internet from slowing to a crawl due to over-utilization. This is a state known as congestion collapse, and occurs when the link is "busy" but not getting useful work done - for example, if the network is clogged with unnecessary retransmissions.

In fact, an Internet collapse has occurred in the past, due to insufficient congestion control. A [1988 paper by V. Jacobson](http://dl.acm.org/citation.cfm?id=52356) explains:

> In October of '86, the Internet had the first of what became a series of 'congestion collapses.' During this period, the data throughput from LBL to UC Berkeley (sites separated by 400 yards and three IMP hops) dropped from 32 Kbps to 40 bps.

In addition to preventing congestion collapse, TCP congestion control is also responsible for dividing bandwidth between flows in a "fair" manner, without requiring per-flow scheduling by routers.

The congestion control mechanism in TCP is constantly evolving - it is a popular research topic, and the mechanism is constantly undergoing improvement and refinement. As the Internet (and the characteristics of Internet traffic) changes, the TCP congestion control mechanism must evolve along with it.


### Original concepts

The first "flavors" of TCP introduce several techniques to prevent over-utilization of a link: additive increase/multiplicative decrease (AIMD) and slow start.

All of these follow the same general strategy: the sender transmits TCP packets on the network, then reacts to observable events to either increase or decrease its sending rate.

#### Additive increase/multiplicative decrease (AIMD)


Additive-increase/multiplicative-decrease (AIMD) is a feedback control algorithm that is the primary mechanism for adjusting the rate of a TCP flow, i.e. answering the question "How fast should I send?"

For each connection, TCP maintains a congestion window (`cwnd`), limiting the total number of unacknowledged packets that may be in transit end-to-end. In other words: if the `cwnd` is reached (i.e., the number of unacknowledged segments is equal to the `cwnd`), the sender stops sending data until more acknowledgements are received. If the acknowledgements are received after some timeout has passed, then it is considered an indication of packet loss, and the segments are retransmitted.

The `cwnd` variable is not advertised or exchanged between the sender and receiver - it is a private variable maintained by each end host. You can not find the `cwnd` by inspecting packet headers.

When a connection is set up, the `cwnd` is set to a small multiple of the maximum segment size (MSS) allowed on that connection.

The basic AIMD algorithm then goes as follows:

* Additive increase: Increase the `cwnd` by a fixed amount (e.g., one MSS) every round-trip time (RTT) that no packet is lost.
* Multiplicative decrease: Decrease the congestion window by a multiplicative factor (e.g., 1/2) every RTT that a packet loss occurs.

This manifests as a classic "sawtooth" `cwnd` pattern.

The rule is then: the maximum amount of data in flight (i.e., not acknowledged) between the sender and the receiver is the minimum of the receive window size (`rwnd`) and `cwnd` variables.

The following animation shows AIMD at work in a network with a buffering router sitting between two hosts. Data (in blue) flows from the host at the left to the host at the right; acknowledgements (in red) return over the reverse link. Utilization on the second hop (from the router to the receiver) is noted. The plot at the bottom shows the sender congestion window as a function of time.

<video controls="controls" width="528" height="304">
	<source src="http://witestlab.poly.edu/respond/sites/ee136s15/files/tcp-aimd.ogv" type="video/ogg">
	<span title="No video playback capabilities, please visit the link below to view the animation.">Router Buffer Animation</span>
</video>

Animation Source: Guido Appenzeller and Nick McKeown, [Router Buffer Animations](http://guido.appenzeller.net/anims/).

#### Slow start

Under AIMD, it can take some time for a flow to actually reach link capacity. Slow start was introduced to help TCP flows reach link capacity faster.

Slow start is in effect during the congestion control phase. During this phase, the `cwnd` is increased by the number of segments acknowledged each time an acknowledgement is received. This phase continues until a loss event occurs, or until a slow start threshold (`ssthresh`) is reached.

With slow start, the `cwnd` grows exponentially (rather than linearly) at the beginning of the connection. Once the `ssthresh` is reached (or a packet loss is detected), then the TCP flow enters the congestion avoidance phase. At this point, the `cwnd` of the flow grows linearly.

![](https://upload.wikimedia.org/wikipedia/commons/2/24/TCP_Slow-Start_and_Congestion_Avoidance.svg)

Image source: By user Fleshgrinder [GPLv3](http://www.gnu.org/licenses/gpl-3.0.html), via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:TCP_Slow-Start_and_Congestion_Avoidance.svg).

#### TCP over wireless

While early versions of TCP congestion control with basic AIMD and slow start were effective at controlling congestion over wired links, these turned out to have poor performance on wireless networks.

There are several major reasons for this:

* TCP congestion control reacts to delay in the network - a long RTT is taken as an indicator of congestion, and a reason to reduce sending rate. This is not necessarily a good idea. For example, some wireless networks impose a delay due to the overhead of requesting and scheduling bandwidth on the link, even when there is no load on the network.
* Similarly, TCP congestion control assumes that loss is an indicator of congestion (i.e., of a router dropping packets due to buffer overflow), and that the sender should therefore reduce its sending rate. This, too, is not always accurate: on a wireless network, there is often some loss due to random errors on the wireless link, which are not indicative of congestion.
* TCP also underutilizes links that are part of "long fat networks" - networks with high delay (i.e., "long") and high bandwidth (i.e., "fat"). On these links, the sending rate is limited by traditional TCP congestion control algorithms to max(cwnd)/RTT - which, if RTT is very high (and the maximum allowed `cwnd` is not very high), may be much less than the link capacity.

Recently, new TCP congestion control algorithms have been designed to adapt to these conditions and give better performance on diverse networks. These mainly vary in:

* How much they decrease `cwnd` after a packet loss is detected (often, with different rules for different indicators of packet loss),
* How quickly they increase `cwnd` at the beginning of the flow,
* How quickly they increase `cwnd` after a packet loss or other indicator of congestion.

TCP congestion control algorithms are typically referred to by name. Some TCP variants include: Veno, Westwood, CUBIC, and Vegas.

Recent Linux kernels use a TCP variant known as [CUBIC](http://dl.acm.org/citation.cfm?id=1400105). Microsoft Windows operating systems may be using [New Reno](https://tools.ietf.org/html/rfc3782) or [Compound TCP](http://en.wikipedia.org/wiki/Compound_TCP).



## Results

TCP congestion window over a lossy wireless link with TCP Reno. Even though there is no congestion on the link, there are many instances of packet loss, causing TCP to reduce its congestion window:

![](/blog/content/images/2016/04/iperf3-allmcs-4.svg)

TCP congestion window over the same wireless link, also with TCP Reno, with link-layer-retransmission (ARQ) enabled. ARQ compensates for some of the physical layer error at the link layer, so there are fewer instances in which TCP reduces its congestion window:

![](/blog/content/images/2016/04/iperf3-allmcs-arq-2.svg)

## Run my experiment


To measure TCP internal variables, we'll use the [iperf3](https://github.com/esnet/iperf) network measurement tool, which reports TCP congestion window size. It gets the CWND measurements from the `tcp_info` structure in the Linux kernel - more info on that [here](http://linuxgazette.net/136/pfeiffer.html).  We will use iperf3 to send TCP traffic from witestlab.poly.edu over a WiMAX cellular link to one of the testbed nodes, and monitor `cwnd` at the sender side  over the duration of the connection.

### 1. Set up resources

First, start by logging in to the console (witestlab.poly.edu) with SSH using your GENI wireless username and key. Load the baseline disk image onto a node on the testbed (here, node7, but you can choose [any available node](http://witestlab.poly.edu/site/tutorial/check-node-status) - node8 or node10 are good alternatives):

```
omf load -i baseline-witest.ndz -t omf.witest.node7
```

When this process ends, turn the node on:

```
omf tell -a on -t omf.witest.node7
```

Then, once the node boots up (after a few minutes), SSH into the node (from the WITest console):

```
ssh root@node7
```

Retrieve and build iperf3 on the node:

```
apt-get update
apt-get install -y git automake libtool
```

```
# To access the public Internet from a wireless node,
# you need to use a proxy.
export HTTPS_PROXY="http://10.0.0.200:3128"
git clone https://github.com/esnet/iperf.git
cd ~/iperf
./bootstrap.sh
./configure
make
make install
ldconfig
cd
```


To make things more interesting, we will enable higher-order [modulation and coding schemes](http://witestlab.poly.edu/blog/adaptive-modulation-and-coding-in-cellular-networks/) (64QAM). These increase the spectral efficiency of the link, at the cost of a higher bit error rate (which is why we have disabled them by default). 


Run the following command **on the witestlab.poly.edu console** to enable higher-order modulation and coding (note that this entire command should be on one line):

```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/dlprofile?dlprof1=13&dlprof2=15&dlprof3=16&dlprof4=17&dlprof5=18&dlprof6=19&dlprof7=20&dlprof8=21"  
```


### 2. Connect to WiMAX network

On your wireless node, connect to the WiMAX network:

```
wimaxcu scan
wimaxcu connect network 51
```

Set the IP address on the WiMAX interface to "10.43.4.X" where "X" is the number of the node you are connected to, e.g.

```
n=$(hostname)  
n=${n:4}  
ifconfig wmx0 10.43.4.$n netmask 255.255.255.0  
```

Verify that you have IP connectivity on the cellular link:
```
ping -c 5 10.43.4.200
```


### 3. Run iperf3

On the **wireless node**, run the iperf3 receiver:

```
iperf3 -s
```

On the **witestlab.poly.edu console**, run the iperf3 sender, sending traffic to the IP address of the node - "10.43.4.X" where "X" is the node number - as the destination address:

```
iperf3 -C reno -c 10.43.4.7 -i 0.1 -J -t 60 | tee iperf3.json
```

Here, the "-C reno" argument says to use TCP Reno congestion control, the "-i 0.1" argument says to report on the connection at 0.1 second intervals, the "-J" argument says to report the output in JSON format, the "-t 60" says to run for 60 seconds, and we use "tee iperf3.json" to redirect the output to a file called "iperf3.json" while simultaneously displaying it on the terminal.

Repeat this step several times, saving the output to a new JSON file each time.

### 4. Visualize `cwnd` over time

We have written a simple R script to plot congestion window over time for this experiment. On the witestlab.poly.edu console, run `R` and then copy and paste the following into your R terminal:

```r
library(jsonlite)
library(data.table)
library(ggplot2)

# Replace "iperf3.json" with the output filename
# you saved your iperf data to, if different
jsonData <- fromJSON("iperf3.json")
df <- rbindlist(jsonData$intervals$streams)

q <- ggplot(df) + theme_minimal()
q <- q + geom_line(aes(x=start, y=snd_cwnd))
q <- q + scale_x_continuous("Time (s)")
q <- q + scale_y_continuous("CWND")
# Saves output to a file called "iperf3.png"
# Use a different output filename, if you prefer
png("iperf3.png")
print(q)
dev.off()

quit('no')
```

This will create a file called "iperf3.png". You may use `scp` to copy this file back to your own computer. 

If you look at this file, you may see many instances where the `cwnd` drops to zero in response to packet loss on the wireless link.  (Depending on which node you use, the quality of its wireless link, and the modulation and coding scheme assigned to it, you may experience more or less loss.)

### 5. Turn on link layer retransmission

When the bit error rate of the physical layer is high, we can compensate by enabling link layer retransmission (ARQ). With [ARQ](https://en.wikipedia.org/wiki/Automatic_repeat_request), frames that are not acknowledged by the receiver within some time interval may be retransmitted.

To turn on ARQ, run the following command on the witestlab.poly.edu console:

```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/set?arq=1"
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/set?arq_block_size=1024"
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/restart" 
```

This last command will return the error message `Failed: 'Exception in snmp_set 'host 192.168.1.10 not responding''`. Don't worry, this is normal! Wait about five minutes for the link to come back up.

Then, repeat Step 2, Step 3, and Step 4.

ARQ reduces error rate, but at a cost: the link-layer acknowledgements and retransmission increase latency of the connection. Also, like any [sliding window protocol](https://en.wikipedia.org/wiki/Sliding_window_protocol), it may throttle the overall data rate on the link, depending on the windows size and block size you select.

If you observed a lot of packet loss when sending TCP traffic without ARQ, then turning on ARQ will probably improve the overall throughput, because the `cwnd` will generally be higher, allowing more data "in flight". But if you didn't experience much packet loss in the no-ARQ experiment, then turning on ARQ might reduce the overall throughput, for the reasons described above.

### Notes

If you want to turn off ARQ again (to repeat the earlier experiments), just run


```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/set?arq=0"
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/restart" 
```

and wait five minutes for the base station to restart.
