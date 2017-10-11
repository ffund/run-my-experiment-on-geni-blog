
In this experiment, we reproduce the results of

> Amogh Dhamdhere and Constantine Dovrolis. Open issues in router buffer sizing. SIGCOMM Comput. Commun. Rev., 36(1):87–92, January 2006.

on the GENI testbed. 

Prior results in [[2](#references)] had suggested that as long as the link carries many TCP flows, only a very small router buffer is needed to fully utilize the link.  In “Open Issues in Router Buffer Sizing” [[3](#references)] Dhamdhere and Dovrolis argued _against_ that conclusion. They say that the results in [[2](#references)] are based on average performance of the network, using metrics such as utilization, mean packet-loss rate, and mean queuing delay that are used for aggregate traffic analysis. However, they fail to consider the performance of individual flows. They show using ns-2 simulations that some kinds of traffic flows have substantially worse performance with a very small buffer.In this experiment, we have extended this simulation results to the Global Environment for Network Innovations (GENI) testbed. Our results mostly agree with the results in “Open Issues in Router Buffer Sizing”, with some slight variations. 

This experiment should take approximately 2-3 hours to run.


## Background

Finding optimal buffer size for routers has been a subject of debate for a long time. Overly large buffer sizes can lead to excessive delay, while buffers that are too small lead to packet loss, which in turn reduces throughput, link utilization, and might degrade the performance of individual flows. 

Traditionally, the rule of thumb has been to set the router buffer size to some multiple of the bandwidth-delay product [[1](#references)]: the product of the bandwidth _C_ of the bottleneck link and the longest round trip time (RTT) _T_ experienced by a TCP connection that uses this link. However, in 2004, Appenzeller, Keslassy, and McKeown argued that buffer sizes much smaller than the bandwidth delay product can still achieve high link utilization if the number of flows is very large [[2](#references)], an approach known as the “Stanford Model”. According to this model, the minimum buffer size _B_ required to saturate bottleneck link is given by

$$B = \frac{CT}{\sqrt{N}}$$

where _N_ is the number of large TCP flows at that link. Thus, the minimum required buffer size decreases inversely with the number of long TCP flows. 

However, Dhamdhere and Dovrolis challenge the Stanford Model, arguing that such small buffers lead to high packet loss [[3](#references)]. Furthermore, TCP throughput for some flows may suffer significantly due to packet losses since TCP throughput can be approximated by [[4](#references)]

$$R = \frac{0.87}{T\sqrt{p}}$$

where _p_ is the packet loss ratio of the link.

## Results

We have reproduced experiments from Dhamdhere and Dovrolis, with two traffic models:

* Persistent connections (long-lived flows) running together with mice (short-lived flows)
* Closed-loop traffic, which is supposed to model heavy-tailed traffic with realistic flow sizes and waiting time between flows

Furthermore, we run each experiment with two buffer sizes:

* _B<sub>s</sub>_ is the Stanford model buffer size, which is 18 packets for the scenario in our experiments according to [[3](#references)].
* _B<sub>l</sub>_ is the buffer size suggested by the bandwidth delay product formula, which is 500 packets for the scenario in our experiments according to [[3](#references)]

For each experiment, we separately measure the CDF of per-flow throughput and the CDF of per-flow packet loss ratio for the large flows (> 100 KB) and small flows (< 100 KB). 

In the figures below, the simulation results of Dhamdhere and Dovrolis are shown on the right, and the results of my testbed reproduction are shown on the left.



###  Throughput for persistent connections with mice 

For the experiment with persistent connections with mice, we have validated the results of Dhamdhere and Dovrolis for throughput for both large flows and short flows. 

![](/blog/content/images/2017/09/tput-per.svg)

Our observations are as follows compared to reference paper:

* **Observation 1:** In the reference paper, around 80% of the persistent flows get a higher throughput with _B<sub>l</sub>_ than with _B<sub>s</sub>_. Similarly, in our result, around 80% of the persistent flows get a higher throughput with _B<sub>l</sub>_ than with _B<sub>s</sub>_. 

* **Observation 2:** In the reference paper, around 50% of the small flow do better with _B<sub>s</sub>_. In our results, only around 40% of the small flows do better with _B<sub>s</sub>_.  We attribute this difference to the higher packet loss in our results, compared to the refe rence paper. 

* **Observation 3:** In the reference paper, the variability of the per flow throughput is higher with _B<sub>s</sub>_ than with _B<sub>l</sub>_, i.e., with _B<sub>s</sub>_ the range of per flow throughput is wider. This observation holds true for our experiments, too.

We note that "steps" can be observed in the CDF plot above, especially for _B<sub>s</sub>_ and for large flows. This is attributed to the effect of RTT on throughput; in the experiment, there are groups of flows with the same RTTs. 

###  Packet loss for persistent connections with mice 

Further, we compare the packet loss observed in our experiment with results presented in the reference paper for both small and large flows. The observations are as follows:

![](/blog/content/images/2017/09/loss-per.svg)

_(Note that in the results for the reference paper, the x-axis range is from 0-12% for large flows (top) and from 0-60% for small flows (bottom).)_

* **Observation 4:** In the reference paper, a smaller packet drop rate is observed for the larger buffer size. We observe a similar trend. Similar to the reference paper, we observe around 2%-4% packet loss for persistent flow with buffer size _B<sub>l</sub>_. However, for small buffer size (Stanford model), the packet drop rates were higher in our experiments than in the reference paper. 

* **Observation 5:** In the reference paper, around 40% of the small flows do not have any packet loss even with the small buffer size. The authors cite this as the reason why those flows have higher throughput with _B<sub>s</sub>_. We observe that only 25% of the small flows do not suffer any packet loss with _B<sub>s</sub>_. (This is consistent with our finding that fewer small flows have a higher throughput with _B<sub>s</sub>_ than _B<sub>l</sub>_ in our experiment, relative to the reference experiment.)  For small flows and _B<sub>l</sub>_, our results are consistent with the reference paper where around 75% of the flows do not observe any packet drops.

### Throughput for closed loop traffic

In the closed-loop traffic experiment, observations for throughput in our testbed experiments compared to reference paper are as follows:

![](/blog/content/images/2017/09/tput-cl.svg)

* **Observation 6:** In the reference paper, most of the large flows achieve higher throughput for _B<sub>l</sub>_. This is consistent with our testbed experiment. We observe that for both testbed results and the reference paper results, around 75%-80% of the large flows achieve higher throughput with buffer size _B<sub>l</sub>_.

* **Observation 7:** In the reference paper, most of the small flows achieve higher throughput with buffer size _B<sub>s</sub>_ than with _B<sub>l</sub>_. However, our testbed results disagree; in our experiments, around 75% of the small flows obtain higher throughput with buffer size _B<sub>l</sub>_.

* **Observation 8:** In the reference paper, there is higher variability in the throughput with buffer size _B<sub>s</sub>_. This is also observed in our testbed results.

### Packet loss for closed loop traffic

Finally, we consider the packet loss for closed loop traffic:

![](/blog/content/images/2017/09/loss-cl.svg)

_(Note that in the reference paper results, the x-axis range is 0-20% for the large flows (top) and 0-40% for the small flows (bottom).)_

* **Observation 9:** In the reference paper, around 15% of large flows experience no packet loss at all with buffer size _B<sub>l</sub>_. We observe similar results for large flows. For a small buffer size, some large flows experience up to 15% packet loss in the reference paper, while in our results, large flows may see up to 25% packet loss with _B<sub>s</sub>_. On the other hand, we observe much more packet loss for small flows. In our testbed results, a small flow may experience up to 90% packet loss for small buffer size, which is much higher than the 25% maximum packet loss observed in the reference paper.

### Overall conclusions

Overall we have similar observations in our testbed results as in the reference paper.

* **Observation 10:** Small buffers have the advantage that they reduce queuing delay, while still maintaining full utilization at the target bottleneck link. However, this comes at the cost of a higher packet loss rate.

* **Observation 11:** This high packet loss rate can harm the performance of certain applications.

* **Observation 12:** Small buffer size results in lower throughput for most of the large TCP flows.

* **Observation 13:** Small buffers would increase the variability in the throughput of TCP flows, making applications layer performance less predictable.    

However, we do observe overall higher packet loss rates for small buffer sizes in our testbed experiment than in the ns-2 simulations of the reference paper, sometimes affecting throughput as well. 

## Run my experiment

Start by reserving resources to set up the topology described in [[3](#references)]. In the GENI Portal, using Jacks, load our RSpec from the following URL:

[https://bitbucket.org/rajeevpnj/el7353finalproject/raw/master/EL7353Project\_request\_rspec.xml](https://bitbucket.org/rajeevpnj/el7353finalproject/raw/master/EL7353Project_request_rspec.xml)

The topology is a balanced binary tree, with source nodes (that generate traffic flows) located two, three, or four hops away from the sink node:

![](/blog/content/images/2016/06/open-issues-topology-1.svg)

The link between target1 and target2 is a 50 Mbps bottleneck link. All other links in the network have the capacity as 100 Mbps. Furthermore, each link has some artificial delay added by `netem`:

* Between sink and target1: 2ms
* Between target1 and target2: 5ms
* Between target2 and h1 nodes: 10ms
* Between h1 nodes and h2 nodes: 100ms
* Between h2 nodes and h3 nodes: 150ms

The RSpec sets this up, and also includes some "execute" commands to install software and download scripts on each node. (It also requests extra RAM for the target2 node, where we will run our data analysis.)

In Jacks, bind to any InstaGENI site, then reserve your resources.

Once your resource reservation finishes, download the Manifest RSpec from the portal:

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZKwYTD4P8BU" frameborder="0" allowfullscreen></iframe>

Then, go back to your slice page and wait for all of your resources to be ready to log in (turn green). After that, wait another ~15 minutes for all of the software to be installed and configured on the nodes.

Then, you are ready to run the experiment. We will describe two ways to run it:

* [Run the experiment manually](#runtheexperimentmanually) - You can manually run each command on each node. This is complicated and error-prone, and can also be problematic because it is difficult to synchronize events that are supposed to happen at certain times.
* [Use the experiment script](#usetheexperimentscript) - Alternatively, you can run this experiment with one command, using an experiment script provided by us. (However, in case this fails, you may want to try the manual method to help debug the problem.)

This experiment has some software dependencies for the host from which you run it. You will need to have `clusterssh` installed. If your system is a recent Ubuntu or Debian host, you can install clusterssh with

```
sudo apt-get install clusterssh
```

For Mac or other \*nix-based systems, you can get it from [SourceForge](https://sourceforge.net/projects/clusterssh/). Windows users should be able to [install clusterssh via Cygwin](http://lifeofageekadmin.com/how-to-install-clusterssh-on-cygwin/).

Once installed, you can [use clusterssh](http://linux.die.net/man/1/cssh) to log into N hosts simultaneously with:

```
cssh [options] [user1@]<server1>[:port1] [user2@]<server2>[:port2] ... [userN@]<serverN>[:portN] 
```  

Furthermore, to extract relevant information from the manifest RSpec, you will need Python and the [xmltodict](https://pypi.python.org/pypi/xmltodict) Python package.

### Run the experiment manually

<iframe width="560" height="315" src="https://www.youtube.com/embed/d39BpdyLwek" frameborder="0" allowfullscreen></iframe>

To run this experiment, you are going to need:

 * A terminal window logged in to the "sink" node
 * A terminal window logged into the "target2" node
 * A clusterssh window (more on this below) in which to run commands on all 18 source nodes (nodes with names beginning with "h")


For your convenience, we have written a Python script that parses your manifest RSpec file and gives you the clusterssh command to run to log in to your source nodes. To use it download the [parseManifest.py](https://bitbucket.org/rajeevpnj/el7353finalproject/raw/master/parseManifest.py) script, and then run the following:

```
python parseManifest.py /path/to/manifestRSpec geniUsername sources
```

substituting the path to the manifest RSpec you have downloaded from the Portal and your GENI username in the arguments to the Python script. Note that this script requires Python and the [xmltodict](https://pypi.python.org/pypi/xmltodict) Python package.


Once you have terminal windows prepared to run commands on the "sink", "target2", and source nodes, we are going to run four experiments:

1. Small buffer size, persistent connections with mice
2. Large buffer size, persistent connections with mice
3. Small buffer size, closed loop traffic
4. Large buffer size, closed loop traffic

#### 1. Small buffer size, persistent connections with mice

On the "target2" node, set the buffer size with

```
buffsize=18  
iface=$(route -n  | grep "10.20.1.0" | awk '{print $8}')  
sudo tc qdisc replace dev "$iface" root pfifo limit "$buffsize"  
```

On the "sink" node, start the iperf receivers:

```
bash /receiverPer.sh
```

On the "target2" node, start tcpdump:

```
iface=$(route -n  | grep "10.20.1.0" | awk '{print $8}')  
sudo tcpdump -enx -i $iface -w perConnectionSmall.pcap
```

In the clusterssh terminal (on all 18 source nodes), start the traffic flows with:

```
bash /Perflow.sh | tee out-small-per-"$(hostname -s)".txt
```


After the experiment finishes (150 seconds), the iperf instances on all the sources will have stopped. At the target2 terminal, use Ctrl+C to stop the tcpdump, then run

```
tshark -T fields -n -r perConnectionSmall.pcap -e frame.time_epoch -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport -e frame.len -e tcp.analysis.retransmission -E separator=, > perConnectionSmall.csv
```

and on the "sink" node, stop the script with Ctrl+C and run

```
pkill -f "iperf"
```

#### 2. Large buffer size, persistent connections with mice

On the "target2" node, set the buffer size with

```
buffsize=500
iface=$(route -n  | grep "10.20.1.0" | awk '{print $8}')  
sudo tc qdisc replace dev "$iface" root pfifo limit "$buffsize"  
```

On the "sink" node, start the iperf receivers:

```
bash /receiverPer.sh
```

On the "target2" node, start tcpdump:

```
iface=$(route -n  | grep "10.20.1.0" | awk '{print $8}')  
sudo tcpdump -enx -i $iface -w perConnectionLarge.pcap
```

In the clusterssh terminal (on the source nodes):

```
bash /Perflow.sh | tee out-large-per-"$(hostname -s)".txt
```

After the experiment finishes (150 seconds), the iperf instances on all the sources will have stopped. At the target2 terminal, use Ctrl+C to stop the tcpdump, then run

```
tshark -T fields -n -r perConnectionLarge.pcap -e frame.time_epoch -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport -e frame.len -e tcp.analysis.retransmission -E separator=, > perConnectionLarge.csv
```

and on the "sink" node, stop the script with Ctrl+C and run

```
pkill -f "iperf"
```
 
#### 3. Small buffer size, closed loop traffic

On the "target2" node, set the buffer size with

```
buffsize=18  
iface=$(route -n  | grep "10.20.1.0" | awk '{print $8}')  
sudo tc qdisc replace dev "$iface" root pfifo limit "$buffsize"  
```

On the "sink" node, start the iperf receivers:

```
bash /receiverCL.sh
```

On the "target2" node, start tcpdump:

```
iface=$(route -n  | grep "10.20.1.0" | awk '{print $8}')  
sudo tcpdump -enx -i $iface -w CLConnectionSmall.pcap
```


In the clusterssh terminal (on the source nodes), start the traffic flows with:

```
bash /CLflow.sh | tee out-small-cl-"$(hostname -s)".txt
```

After the experiment finishes (150 seconds), the iperf instances on all the sources will have stopped. At the target2 terminal, use Ctrl+C to stop the tcpdump, then run

```
tshark -T fields -n -r CLConnectionSmall.pcap -e frame.time_epoch -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport -e frame.len -e tcp.analysis.retransmission -E separator=, > CLConnectionSmall.csv
```

and on the "sink" node, stop the script with Ctrl+C and run

```
pkill -f "iperf"
```

#### 4. Large buffer size closed loop traffic

On the "target2" node, set the buffer size with

```
buffsize=500
iface=$(route -n  | grep "10.20.1.0" | awk '{print $8}')  
sudo tc qdisc replace dev "$iface" root pfifo limit "$buffsize"  
```

On the "sink" node, start the iperf receivers:

```
bash /receiverCL.sh
```

On the "target2" node, start tcpdump:

```
iface=$(route -n  | grep "10.20.1.0" | awk '{print $8}')  
sudo tcpdump -enx -i $iface -w CLConnectionLarge.pcap
```

In the clusterssh terminal (on the source nodes), start the traffic flows with:

```
bash /CLflow.sh | tee out-large-cl-"$(hostname -s)".txt
```

After the experiment finishes (150 seconds), the iperf instances on all the sources will have stopped. At the target2 terminal, use Ctrl+C to stop the tcpdump, then run

```
tshark -T fields -n -r CLConnectionLarge.pcap -e frame.time_epoch -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport -e frame.len -e tcp.analysis.retransmission -E separator=, > CLConnectionLarge.csv
```

and on the "sink" node, stop the script with Ctrl+C and run

```
pkill -f "iperf"
```


#### Data analysis

For evaluation of all the results, we are going to use the packets captured at target2 to find the throughput and packet loss of each individual flow.

This is not the most accurate way to compute throughput, since we compute the start time of the flow as the time the first packet is received at the middlebox, and we compute the end time of the flow as the time the last packet is received at the middlebox. It does not include the time from the end host to the middlebox for the first and last packet. However,


1. It would be much more complicated to capture traffic separately on each of the 18 hosts. 
2. We cannot use the throughput reported by iperf at the end hosts, because it is not accurate for small flows with a very short duration.
3. In most cases, a single round trip time is much less than the overall connection duration, so the throughput computed in this manner is close to the actual throughput.

and so we consider this approximation to be an acceptable tradeoff.

To produce the figures shown in the [Results](#results), run


```
cp /dataAnalysis.R .
Rscript dataAnalysis.R
```

on the target2 node. This will create four PDF files, which you can transfer to your local host with `scp`.

### Use the experiment script

<iframe width="560" height="315" src="https://www.youtube.com/embed/Sj0tHky1gS4" frameborder="0" allowfullscreen></iframe>

Alternatively, we provide a Bash script that will run the entire experiment. To use it, 

1. Download [runExperiment.sh](https://bitbucket.org/rajeevpnj/el7353finalproject/raw/master/runExperiment.sh) to your local host. You will need to have `clusterssh` installed already.
2. Download the [parseManifest.py](https://bitbucket.org/rajeevpnj/el7353finalproject/raw/master/parseManifest.py) script (if you haven't already). You will need Python and the `xmltodict` package.
3. Run the script with two arguments, the path to the `parseManifest.py` script and your GENI username:

```
bash runExperiment.sh /path/to/parseManifest.py GENI-username
```

Once the experiment finishes running, the figures it produces will be transferred to a "data" directory inside your working directory. (The last step - the data analysis step - will take a while, so don't be concerned if it appears as if nothing is happening.)

## Notes

We used following software versions for our experiment:

1. iperf 2.0.5
2. Python 3.6
3. R 3.4.1

## References

[1] Curtis Villamizar and Cheng Song. High performance TCP in ANSNET. SIGCOMM Comput. Commun. Rev., 24(5):45–60, October 1994.

[2] Guido Appenzeller, Isaac Keslassy, and Nick McKeown. Sizing router buffers. In Proceedings of the 2004 Conference on Applications, Technologies, Architectures, and Protocols for Computer Communications, SIGCOMM ’04, pages 281 292, New York, NY, USA, 2004. ACM.

[3] Amogh Dhamdhere and Constantine Dovrolis. Open issues in router buffer sizing. SIGCOMM Comput. Commun. Rev., 36(1):87–92, January 2006.

[4] Jitendra Padhye, Victor Firoiu, Don Towsley, and Jim Kurose. Modeling TCP throughput: A simple model and its empirical validation. In Proceedings of the ACM SIGCOMM ’98 Conference on Applications, Technologies, Architectures, and Protocols for Computer Communication, SIGCOMM’98, pages 303–314, New York, NY, USA, 1998. ACM.

    
    