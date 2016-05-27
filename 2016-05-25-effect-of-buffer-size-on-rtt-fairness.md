In this experiment, I am going to reproduce the experiment for congestion window behavior of two HighSpeed TCP (HSTCP) flows as it relates to drop tail queue size for both the under-buffered and over-buffered case, as described in [[1](#references)]. Moreover, I would like to also conduct the same experiments with the CUBIC congestion control algorithm and compares its behavior to HSTCP under the same conditions.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you have never used GENI before, you may want to try running [Lab Zero](http://groups.geni.net/geni/wiki/GENIExperimenter/Tutorials/jacks/GettingStarted_PartI/Procedure) first.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background ##

Bandwidth usage between competing TCP flows is a function of RTT, with short flows receiving more bandwidth than longer ones. This is intuitive, since the flow with shorter RTT increases congestion window more rapidly, and larger congestion window allows sending more packets per RTT. 

This "RTT unfairness" may be alleviated by forcing two congestion window converge to a similar size. Therefore, for the competing flows with different RTTs, fast congestion window convergence means a more fair share of the bandwidth. 

The experiment we are going to reproduce considers RTT fairness in two cases:

* when the buffer size at the bottleneck link has a size equal to the bandwidth-delay product (BDP) of the link (over-buffered case), and 
* when the buffer size at the bottleneck link has a size equal to 10% of the bandwidth-delay product (BDP) of the link (under-buffered case).  

##  Results ##

The results of our experiment are not consistent with the  published result.


![](/blog/content/images/2016/05/mz-overbuffered.svg)

![](/blog/content/images/2016/05/mz-underbuffered.svg)


The original paper states that HSTCP congestion window converges faster and achieves greater RTT fairness with *large* buffer size. However, the congestion window behavior shown is not consistent with HSTCP, e.g., after loss event, HSTCP should decrease a small portion of congestion window, but in the window plot from the original paper it always halves the congestion window.

As the figures show, on the GENI testbed I get the opposite result, which suggests that a *small* buffer helps HSTCP congestion window converge. 

The last finding is that CUBIC (not considered in the original paper, which was from 2004) shows a faster window convergence in both small and large buffer case, which indicates CUBIC has a better RTT fairness than HSTCP in this specific experiment.


## Run My Experiment ##

In a new slice, load the RSpec located at [https://bitbucket.org/mz1192/project-mz1192/raw/master/topo2.xml](https://bitbucket.org/mz1192/project-mz1192/raw/master/topo2.xml) into the Jacks tool in the GENI portal. This will prepare the following topology on your canvas:

![](/blog/content/images/2016/05/mz-topology.png)

Click on "Site 1" and bind your RSpec to an InstaGENI site, then reserve the resources.

After all the nodes are ready, log into them. At the two servers (s1, s2) and the two devices (d1, d2), run the following commands to install iperf3:

```
sudo apt-get update
sudo apt-get -y install git

git clone https://github.com/esnet/iperf.git  
cd ~/iperf  
./bootstrap.sh
./configure
make  
sudo make install  
sudo ldconfig  
cd  
```

Also, on s1 and s2, make sure that the kernel modules for TCP CUBIC and HighSpeed are available to be loaded:

```
sudo modprobe tcp_highspeed
sudo modprobe tcp_cubic
```

On g1, download a script that will monitor the queue length, and make it executable:

```
wget https://bitbucket.org/mz1192/project-mz1192/raw/master/queuemonitor.sh
chmod a+x queuemonitor.sh
```

Now we need to set the delay of all the links. The total delay of the s1-d1 flow will be approximately 100 ms, and the delay of the s2-d2 flow will be approximately 50 ms.

On s1, set the delay of the s1-g1 link to 45 ms:

```
iface=$(route -n  | grep "10.10.10.0" | awk '{print $8}')
sudo tc qdisc add dev "$iface" root netem delay 45ms
```

On s2, set the delay of the s2-g1 link to 20 ms:

```
iface=$(route -n  | grep "10.10.10.0" | awk '{print $8}')
sudo tc qdisc add dev "$iface" root netem delay 20ms
```

On g2, set the delay of the g1-g2 link to 10 ms:


```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')
sudo tc qdisc add dev "$iface" root netem delay 10ms
```

On d1, set the delay of the d1-g2 link to 45 ms:

```
iface=$(route -n  | grep "10.10.20.0" | awk '{print $8}')
sudo tc qdisc add dev "$iface" root netem delay 45ms
```

And on d2, set the delay of the d2-g2 link to 20 ms:

```
iface=$(route -n  | grep "10.10.20.0" | awk '{print $8}')
sudo tc qdisc add dev "$iface" root netem delay 20ms
```


You can verify the total delay by running

```
ping d1 -c 5
```

on s1, and 

```
ping d2 -c 5
```

on s2.

We are going to run four experiments:

1. [HSTCP, overbuffered case](#1hstcpoverbufferedcase)
2. [HSTCP, underbuffered case](#2hstcpunderbufferedcase)
3. [CUBIC, overbuffered case](#3cubicoverbufferedcase)
4. [CUBIC, underbuffered case](#4cubicunderbufferedcase)

### 1. HSTCP, overbuffered case

On g1, set the buffer size of the g1-g2 link to be 1.25MB (overbuffered case), and the link rate to 100 Mbps:


```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')
sudo tc qdisc replace dev "$iface" root tbf rate 100mbit limit 1.25Mb burst 32kB

```


Start iperf3 servers at d1 and d2 using the following command,

```
iperf3 -s
```

Then run the following three commands at the same time, to start the two TCP flows and to monitor the queue on the bottleneck link. 

On s1:

```
sudo iperf3 -c d1 -C highspeed -i 0.1 -t 120 -f K | tee node1-highspeed-overbuffer.txt  
```

On s2:

```
sudo iperf3 -c d2 -C highspeed -i 0.1 -t 120 -f K | tee node2-highspeed-overbuffer.txt  
```

On g1:

```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')  
./queuemonitor.sh "$iface" 120 0.1 | tee router-highspeed-overbuffer.txt
```


When the execution terminates, run the following command at g1 to obtain the queue length trace file:


```
cat router-highspeed-overbuffer.txt | sed 's/\p / /g' | awk  '{print $31}' > queue-highspeed-overbuffer.txt

```


On s1, to get the CWND trace:

```
grep Bytes node1-highspeed-overbuffer.txt | awk '{print $3, $10, $11}' | grep KBytes | sed 's/-/ /' | awk '{print $1, $3}' > kout1-highspeed-overbuffer.txt
grep Bytes node1-highspeed-overbuffer.txt | awk '{print $3, $10, $11}' | grep MBytes | sed 's/-/ /' | awk '{print $1, $3*1e3}' >> kout1-highspeed-overbuffer.txt

```

And on s2:

```
grep Bytes node2-highspeed-overbuffer.txt | awk '{print $3, $10, $11}' | grep KBytes | sed 's/-/ /' | awk '{print $1, $3}' > kout2-highspeed-overbuffer.txt
grep Bytes node2-highspeed-overbuffer.txt | awk '{print $3, $10, $11}' | grep MBytes | sed 's/-/ /' | awk '{print $1, $3*1e3}' >> kout2-highspeed-overbuffer.txt

```

### 2. HSTCP, underbuffered case

On g1, set the buffer size of the g1-g2 link to be 0.125MB (underbuffered case), and the link rate to 100 Mbps:


```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')
sudo tc qdisc replace dev "$iface" root tbf rate 100mbit limit 0.125Mb burst 32kB

```


Start iperf3 servers at d1 and d2 using the following command,

```
iperf3 -s
```

Then run the following three commands at the same time, to start the two TCP flows and to monitor the queue on the bottleneck link. 

On s1:

```
sudo iperf3 -c d1 -C highspeed -i 0.1 -t 120 -f K | tee node1-highspeed-underbuffer.txt  
```

On s2:

```
sudo iperf3 -c d2 -C highspeed -i 0.1 -t 120 -f K | tee node2-highspeed-underbuffer.txt  
```

On g1:

```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')  
./queuemonitor.sh "$iface" 120 0.1 | tee router-highspeed-underbuffer.txt
```


When the execution terminates, run the following command at g1 to obtain the queue length trace file:


```
cat router-highspeed-underbuffer.txt | sed 's/\p / /g' | awk  '{print $31}' > queue-highspeed-underbuffer.txt

```


On s1, to get the CWND trace:

```
grep Bytes node1-highspeed-underbuffer.txt | awk '{print $3, $10, $11}' | grep KBytes | sed 's/-/ /' | awk '{print $1, $3}' > kout1-highspeed-underbuffer.txt
grep Bytes node1-highspeed-underbuffer.txt | awk '{print $3, $10, $11}' | grep MBytes | sed 's/-/ /' | awk '{print $1, $3*1e3}' >> kout1-highspeed-underbuffer.txt
```

And on s2:

```
grep Bytes node2-highspeed-underbuffer.txt | awk '{print $3, $10, $11}' | grep KBytes | sed 's/-/ /' | awk '{print $1, $3}' > kout2-highspeed-underbuffer.txt
grep Bytes node2-highspeed-underbuffer.txt | awk '{print $3, $10, $11}' | grep MBytes | sed 's/-/ /' | awk '{print $1, $3*1e3}' >> kout2-highspeed-underbuffer.txt

```

### 3. CUBIC, overbuffered case

On g1, set the buffer size of the g1-g2 link to be 1.25MB (overbuffered case), and the link rate to 100 Mbps:


```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')
sudo tc qdisc replace dev "$iface" root tbf rate 100mbit limit 1.25Mb burst 32kB

```


Start iperf3 servers at d1 and d2 using the following command,

```
iperf3 -s
```

Then run the following three commands at the same time, to start the two TCP flows and to monitor the queue on the bottleneck link. 

On s1:

```
sudo iperf3 -c d1 -C cubic -i 0.1 -t 120 -f K | tee node1-cubic-overbuffer.txt  
```

On s2:

```
sudo iperf3 -c d2 -C cubic -i 0.1 -t 120 -f K | tee node2-cubic-overbuffer.txt  
```

On g1:

```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')  
./queuemonitor.sh "$iface" 120 0.1 | tee router-cubic-overbuffer.txt
```


When the execution terminates, run the following command at g1 to obtain the queue length trace file:


```
cat router-cubic-overbuffer.txt | sed 's/\p / /g' | awk  '{print $31}' > queue-cubic-overbuffer.txt

```

On s1, to get the CWND trace:

```
grep Bytes node1-cubic-overbuffer.txt | awk '{print $3, $10, $11}' | grep KBytes | sed 's/-/ /' | awk '{print $1, $3}' > kout1-cubic-overbuffer.txt
grep Bytes node1-cubic-overbuffer.txt | awk '{print $3, $10, $11}' | grep MBytes | sed 's/-/ /' | awk '{print $1, $3*1e3}' >> kout1-cubic-overbuffer.txt
```

And on s2:

```
grep Bytes node2-cubic-overbuffer.txt | awk '{print $3, $10, $11}' | grep KBytes | sed 's/-/ /' | awk '{print $1, $3}' > kout2-cubic-overbuffer.txt
grep Bytes node2-cubic-overbuffer.txt | awk '{print $3, $10, $11}' | grep MBytes | sed 's/-/ /' | awk '{print $1, $3*1e3}' >> kout2-cubic-overbuffer.txt  
```


### 4. CUBIC, underbuffered case

On g1, set the buffer size of the g1-g2 link to be 0.125MB (underbuffered case), and the link rate to 100 Mbps:


```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')
sudo tc qdisc replace dev "$iface" root tbf rate 100mbit limit 0.125Mb burst 32kB

```


Start iperf3 servers at d1 and d2 using the following command,

```
iperf3 -s
```

Then run the following three commands at the same time, to start the two TCP flows and to monitor the queue on the bottleneck link. 

On s1:

```
sudo iperf3 -c d1 -C cubic -i 0.1 -t 120 -f K | tee node1-cubic-underbuffer.txt  
```

On s2:

```
sudo iperf3 -c d2 -C cubic -i 0.1 -t 120 -f K | tee node2-cubic-underbuffer.txt  
```

On g1:

```
iface=$(route -n  | grep "10.10.30.0" | awk '{print $8}')  
./queuemonitor.sh "$iface" 120 0.1 | tee router-cubic-underbuffer.txt
```


When the execution terminates, run the following command at g1 to obtain the queue length trace file:


```
cat router-cubic-underbuffer.txt | sed 's/\p / /g' | awk  '{print $31}' > queue-cubic-underbuffer.txt

```

On s1, to get the CWND trace:

```
grep Bytes node1-cubic-underbuffer.txt | awk '{print $3, $10, $11}' | grep KBytes | sed 's/-/ /' | awk '{print $1, $3}' > kout1-cubic-underbuffer.txt
grep Bytes node1-cubic-underbuffer.txt | awk '{print $3, $10, $11}' | grep MBytes | sed 's/-/ /' | awk '{print $1, $3*1e3}' >> kout1-cubic-underbuffer.txt

```

And on s2:

```
grep Bytes node2-cubic-underbuffer.txt | awk '{print $3, $10, $11}' | grep KBytes | sed 's/-/ /' | awk '{print $1, $3}' > kout2-cubic-underbuffer.txt
grep Bytes node2-cubic-underbuffer.txt | awk '{print $3, $10, $11}' | grep MBytes | sed 's/-/ /' | awk '{print $1, $3*1e3}' >> kout2-cubic-underbuffer.txt
```




### Visualize experiment results

When this is done, use `scp` to transfer all the "kout"  and "queue" files to a local machine. You should have the following files:

Then, to create the queue length plots, in MATLAB run the script available at [https://nyu.box.com/v/mz-hstcp](https://nyu.box.com/shared/static/ghiyp184nk3eczbzjyh9q4qe6w34z7i7.m).



### Clean up


When you have finished your experiment, please [delete your resources](http://www.cs.unc.edu/Research/geni/geniEdu/Shutdown.html) to free them up for other experimenters.

## Notes ##

* System information: Linux kernel version 3.13.0-33-generic (buildd@tipua) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) )
* GENI aggregate: GPO InstaGENI
* Other Software: Matlab R_2014b for creating plots

## References #

[1] D. Barman, G. Smaragdakis and I. Matta, The effect of router buffer size on HighSpeed TCP performance, Global Telecommunications Conference, 2004. GLOBECOM 04. IEEE, 2004, pp. 1617-1621 Vol.3. ([Link](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=1378255))