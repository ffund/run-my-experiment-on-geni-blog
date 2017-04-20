This experiment shows the basic behavior of TCP congestion control. You'll see the classic "sawtooth" pattern in a TCP flow's congestion window, and you'll see how a TCP flow responds to indications of congestion.

It should take about 1 hour to run this experiment.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Results

In this experiment, we see the classic "sawtooth" pattern of the TCP congestion window, shown as the solid line in the plot below:

![](/blog/content/images/2017/04/tcp-cwnd-2.svg)
<small><i>Figure 1: Congestion window size (solid line) and slow start threshold (dotted line) of three TCP flows sharing the same bottleneck.</i></small>

The slow start threshold is shown as a dashed line.

We can also identify

* "Slow start" periods
* "Congestion avoidance" periods (when the congestion window is greater than the slow start threshold)
* Instances where 3 duplicate ACKs were received. (We are using [TCP Reno](http://intronetworks.cs.luc.edu/current/html/reno.html), which will enter "fast recovery" in response to 3 duplicate ACKs.)
* Instances of timeout. This will cause the congestion window to go back to 1 MSS, and trigger a "slow start" period.

For example, the following annotated image shows a short interval in the top TCP flow of Figure 1:

![](/blog/content/images/2017/04/tcp-one.svg)
<small><i>Figure 2: Events in a TCP flow. Slow-start periods are marked in yellow; congestion avoidance periods are in purple. We can also see when indicators of congestion occur: instances of timeout (red), and instances where three duplicate ACKs are received (blue) triggering fast recovery (green).</i></small>

And we can see how the TCP congestion control algorithm responds to indications of packet loss (timeout and duplicate ACKs). 



## Run my experiment

First, reserve a topology on GENI including two end hosts, and a router between them. The router will buffer traffic between the sender and the receiver. If the buffer in the router becomes full, it will drop packets, triggering TCP congestion control behavior.

In the GENI Portal, create a new slice, then click "Add Resources". Scroll down to where it says "Choose RSpec" and select the "URL" option, the load the RSpec from the URL: [https://git.io/vSioM](https://git.io/vSioM)

This will load the following topology in your canvas:

![](/blog/content/images/2017/04/tcp-topology.svg)

Click on "Site 1" and choose an InstaGENI site to bind to, then reserve your resources. Wait for your nodes to boot up (they will turn green in the canvas display on your slice page in the GENI portal when they are ready). Then, SSH into each node. 

### Set up experiment


On the end hosts ("sender" and "receiver"), install the `iperf` network testing tool, with the command 

```
sudo apt-get update
sudo apt-get install iperf
```

Also set TCP Reno as the default TCP congestion control algorithm on both, with

```
sudo sysctl -w net.ipv4.tcp_congestion_control=reno
```

On the router, turn on packet forwarding with the command

```
sudo sysctl -w net.ipv4.ip_forward=1
```

Also, set each experiment interface on the router so that the router is a 1 Mbps bottleneck, and buffers up to 0.1 MB, using a [token bucket](https://linux.die.net/man/8/tc-tbf) queue: 

```
sudo tc qdisc replace dev eth1 root tbf rate 1mbit limit 0.1mb burst 32kB peakrate 1.01mbit mtu 1600
sudo tc qdisc replace dev eth2 root tbf rate 1mbit limit 0.1mb burst 32kB peakrate 1.01mbit mtu 1600
```

Finally, on the sender host, load the `tcp_probe` kernel module, and tell it to monitor traffic to/from TCP port 5001:

```
sudo modprobe tcp_probe port=5001 full=1
sudo chmod 444 /proc/net/tcpprobe
```

We will use this TCP probe module to monitor the behavior of the TCP congestion control algorithm on the sender.

### Generate data

Next, we will generate some TCP flows between the two end hosts, and use it to observe the behavior of the TCP congestion control algorithm.

On the "receiver", run

```
iperf -s
```

On the "sender", run

```
dd if=/proc/net/tcpprobe ibs=128 obs=128 | tee /tmp/tcpprobe.dat
```

Open another SSH session to the "sender" and run

```
iperf -t 60 -c receiver -P 3
```

to start the TCP flows. Here

* `-t 60` says to run for 60 seconds
* `-c receiver` says to send traffic to the host named "receiver"
* `-P 3` says to send 3 parallel TCP flows


You will see a final status report in the `iperf` window after the `iperf` sender finishes, which should take about a minute.  In the window where the TCP probe is running, you should see a line of output for each TCP packet.

The output of the TCP probe will be saved in `/tmp/tcpprobe.dat` on the sender. Use `scp` to transfer this file to your own laptop for processing.

### Understanding the TCP Probe output

The TCP module probe records a line of output every time a packet is sent, if either the destination or source port number in the TCP packet header is 5001 (since we loaded it with the `port=5001` option).

Each line of output will include the following fields, in order:


<table id="table-1" class="table table-striped table-bordered col-2" data-columns="2"><thead>  
<tr><th class="col-1">Field</th><th class="col-2">Explanation</th></tr></thead>  
<tbody>  
<tr class="row-1"><td class="col-1">Time</td><td class="col-2">Time (in seconds) since beginning of probe output</td></tr>  
<tr class="row-2"><td class="col-1">Sender</td><td class="col-2">Source address and port of the packet, as IP:port</td></tr>  
<tr class="row-3"><td class="col-1">Receiver</td><td class="col-2">Destination address and port of the packet, as IP:port</td></tr>  
<tr class="row-4"><td class="col-1">Bytes</td><td class="col-2">Bytes in packet</td></tr>  
<tr class="row-5"><td class="col-1">Next</td><td class="col-2">Next send sequence number, in hex format</td></tr>  
<tr class="row-6"><td class="col-1">Unacknowledged</td><td class="col-2">Smallest sequence number of packet send but unacknowledged, in hex format</td></tr>  
<tr class="row-7"><td class="col-1">Send CWND</td><td class="col-2">Size of send congestion window for this connection (in MSS)</td></tr>  
<tr class="row-8"><td class="col-1">Slow start threshold</td><td class="col-2">Size of send congestion window for this connection (in MSS)</td></tr>
<tr class="row-9"><td class="col-1">Send window</td><td class="col-2">Send window size (in MSS). Set to the minimum of send CWND and receive window size</td></tr>
<tr class="row-10"><td class="col-1">Smoothed RTT</td><td class="col-2">Smoothed estimated RTT for this connection (in ms)</td></tr>
<tr class="row-11"><td class="col-1">Receive window</td><td class="col-2">Receiver window size (in MSS), received in the lack ACK. This limit prevents the receiver buffer from overflowing,
 i.e. prevents the sender from sending at a rate that is faster than the receiver can process the data.</td></tr></tbody>  
</table>

By looking at your TCP probe data, you can observe the behavior of the TCP congestion control algorithm. 

(The last line in your data file may be truncated - you may remove that line.)

Note that in Linux, the slow start threshold takes on the value 2147483647 when it is initialized. It's set to this extreme number - which is not a realistic value for this metric - to show that it hasn't been computed yet. You can ignore that value when it occurs (but you should still use the congestion window that is recorded for those rows - don't throw them out entirely). For repeated connections, the slow start threshold is usually "remembered" from the last connection, so if you run this experiment multiple times, you may not see that extreme value in later experiments.



## Notes

### Exercise

Create a plot of the congestion window size and slow start threshold for each TCP flow over the duration of the experiment, similar to Figure 1 in the [Results](#results) section.

Annotate your plot, similar to Figure 2 in the [Results](#results) section, to show:

* Periods of "Slow Start" 
* Periods of "Congestion Avoidance"
* Instances where 3 duplicate ACKs were received (which will trigger "fast recovery")
* Instances of timeout

Using your plot and/or experiment data, explain how the behavior of TCP is different in the "Slow Start" and "Congestion Avoidance" phases. Also, using your plot, explain what happens to both the congestion window and the slow start threshold when 3 duplicate ACKs are received.