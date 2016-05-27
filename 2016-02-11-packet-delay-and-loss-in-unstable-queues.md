This experiment is about the effect of queue size on packet delay in a queue where the service time is longer than the inter-arrival time. We reproduced the known effect that larger queue sizes lead to longer delays in this scenario. 

This experiment is based on [Exploring Queues](http://www.cs.unc.edu/Research/geni/geniEdu/09-queues.html), which is for a D/D/1 queue.

This experiment should take about 60 to 120 minutes.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes).

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background 

An M/M/1 queue is a type of queue for which the probability distributions of the arrival and service processes are exponential. 

When the mean service time is greater than the mean inter-arrival time, the queue is said to be unstable and it will grow infinitely unless the size of the queue is limited. 

When the size of the queue is limited, the queue will grow to reach that size and then enter steady state. At this point, each arriving packet will either wait at the back of the line to be serviced, or get dropped since there is no room in the queue. Thus each arriving packet that *is* served sees a full queue (or almost full queue) on arrival, and has to wait for N seconds to be served, where N is the time is takes to serve a full queue. The larger the queue size is, the longer each packet that is not dropped will be delayed.

More specifically, as 

$$\rho \rightarrow \infty $$

(where &rho; is the ratio of packet arrival rates to service rate), then packets that are not dropped will see a delay of

$$ T = \frac{m}{\mu} $$

where _m_ is the size of the queue and &mu; is the service rate of the queue.


## Results

Below is graph showing our experiment results. Three experiments were performed and the peak delay of each queue size was plotted against its corresponding queue size. 

![](/blog/content/images/2016/02/Peakdelaygraph.png)

Even though the utilization was not much greater than 1 (&rho; = 1.25), the worst-case delay observed in a 10-second experiment was the same as what we would expect from a queue with &rho; = &infin;.

(The peak delay most closely represents how long a packet arriving at a full queue would have to wait. In our experiment &rho; is not infinite so not all packets arrive at a full queue.)

We also use peak delay to avoid the transient effects that occurred at the beginning of the experiment. The queue slowly filled up until the delay reached a steady state value.

For example for a queue of size 2 mbytes, this is what the delay looks like for the first 10 seconds of the experiment:

```
64 bytes from server-link-1 (10.10.2.2): icmp_seq=2 ttl=63 time=1.26 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=3 ttl=63 time=26.4 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=4 ttl=63 time=155 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=5 ttl=63 time=239 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=6 ttl=63 time=314 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=7 ttl=63 time=350 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=8 ttl=63 time=288 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=9 ttl=63 time=414 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=10 ttl=63 time=525 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=11 ttl=63 time=590 ms
64 bytes from server-link-1 (10.10.2.2): icmp_seq=12 ttl=63 time=619 ms
```

## Run my experiment


To start, create a new slice on the [GENI portal](https://portal.geni.net/). Create a client-router-server topology with three standard VMs, like this:

![](http://witestlab.poly.edu/~ffund/el7353/images/jacks-topology.png)

Use the "Auto IP" tool to automatically assign appropriate IPs. Then choose an InstaGENI aggregate and log in to your resources.

To generate traffic with exponentially distributed interarrival times and exponentially distributed packet sizes, we will use the [D-ITG traffic generator](http://traffic.comics.unina.it/software/ITG/) ([manual available here](http://traffic.comics.unina.it/software/ITG/manual/).) You will need to install D-ITG on your client and server nodes. Log in to each and run

```
sudo apt-get update # refresh local information about software repositories
sudo apt-get install d-itg # install D-ITG software package
```
To create an unstable queue you need the arrival rate, the '-E' parameter in the ITGSend command, to be set to a value greater then the service rate at the router.

Let us choose to set the value of '-E', the mean arrival rate, to 2000 packets/sec. Converting this to Megabits/sec gives a rate of 8.192. We will set the service rate to 6.55 Megabits/sec. Then &rho; = 8.192/6.55 &#8773; 1.25, so the queue is unstable.

Perform the following steps for 8 queue length sizes in the range of 5kbits to 200kbits:  

### Traffic sink

First run `ITGRecv` in the server window, this only needs to be done once.

### Router 

Enter the following code in the router window, making sure to change the queue size:

```
sudo tc qdisc replace dev eth2 root tbf rate 6.55mbit limit 2mb burst 10kb
```
(In this example 2mb is the queue size and is 2 megabytes.)
### Traffic source

Open two client windows and run the following simultaneously (remember to change the log file names to reflect the queue size):

```
ITGSend -a server -l sender2m.log -x receiver2m.log -E 2000 -e 512 -T UDP
```

```
ping server -c 10
```

Record the peak delay time. 

### Visualize your results

You can visualize your results in Excel or any other spreadsheet program. In one column record the peak queue size. In another column record your peak delay data that you collected. In a third column calculate the expected delay with the following equation:

$$ T = \frac{m}{\mu} $$

Where m = queue size in megabits, &mu; = 6.55 Megabits/sec.  
You can multiply the results by 1000 to get it in milliseconds.

Graph the peak delay and expected delay against the corresponding queue size.

### Delete your resources

Please delete your resources when you're finished to free them up for other experimenters!
