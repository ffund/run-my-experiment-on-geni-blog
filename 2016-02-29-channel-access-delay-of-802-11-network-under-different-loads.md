<div class="st-alert">
<b>Note</b>: This post is preserved for reference; however, the WiMAX base station at WITest is not currently operational, so this experiment will not run as described here. It may be possible to run this experiment with some modifications on another testbed.
</div>

This experiment highlights the difference in channel access delay between a contention-based WiFi network and a scheduled WiMAX network. 

It should take about 60-120 minutes to run this experiment, but you will need to have [reserved that time](http://witestlab.poly.edu/respond/sites/witest/tutorial/make-a-reservation) in advance. This experiment uses wireless resources (specifically, on the [WITest](http://witestlab.poly.edu/) testbed), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have [reserved time on the WITest testbed](http://witestlab.poly.edu/respond/sites/witest/tutorial/make-a-reservation), and you must run this experiment during your reserved time.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

Different classes of applications have different needs with regard to network-related metrics like:

* **Bandwidth**: The rate at which traffic is carried by the network
* **Latency**: The delay in data transmission from source to destination
* **Jitter**: The variation in latency
* **Reliability**: The percentage of traffic that is transported end-to-end (i.e., not lost)


The QoS requirements of an application describe its needs with respect to these metrics.

For example, voice and video chat applications have a low delay tolerance - customer satisfaction with the quality of a voice call deteriorates significantly beyond 300 ms latency.

In contrast, email applications can tolerate delays of minutes or even hours. Even if the email is delivered with a delay, user's overall experience is not affected.

QoS directly impacts Quality of Experience (QoE). QoS describes objective measurements of network performance, e.g., "delay jitter is high." QoE describes the customer’s subjective perception of service, e.g., "voice quality is very bad"

In this experiment, we'll focus on measuring latency and jitter. These quantities are both very important for video and voice communications.

Network operators may to choose offer different kinds of QoS guarantees - i.e., certify that the traffic they carry will meet certain QoS goals. There are several types of guarantees of this kind:

* **Guaranteed Service**: Network resources are reserved for traffic in this class so that QoS goals are guaranteed to be met.
* **Differentiated Service**: Traffic in this class gets special treatment (e.g. more average bandwidth) but no explicit service guarantees.
* **Best Effort Service**: Basic connectivity with no guarantees.


Network operators can enforce these classes in several ways, including:

* **Admission Control**: The network will only establish a connection with a new client if the network has enough capacity to support the QoS requirements of the new client and its current clients. For example, in telephone networks, a call is connected only if the network has enough capacity to support the needs of the voice call.
* **Traffic Control**: Routers in the network classify, schedule, and route packets with QoS needs in mind. If there's a bottleneck, some traffic will be prioritized over others. For example, an Internet service provider might throttle BitTorrent traffic to reserve capacity for other services.

In the experiment we will do today, both networks offer BE service with no guarantees.


QoS metrics are used to evaluate the practical performance of a network. There is generally a relationship between the underlying network technology - especially the design of its MAC layer - and the QoS as observed by a network user.


###  WiFi networks

WiFi networks use CSMA/CA, which is a kind of contention-based MAC mechanism. In WiFi networks, devices compete in a random access game to use the channel. If multiple devices try to transmit at the same time, a collision will occur, and the devices will engage in a procedure to reduce the probability that they both try again at the same time.

This approach is efficient under light load, since devices can access the channel quickly without waiting to request bandwidth and get permission from a central scheduler. With many contenders, however, its performance deteriorates because channel time is wasted on collisions, and there may be a very long delay to access the channel.

You may be interested in this animated [CMSA/CA Demo](http://www.ccs-labs.org/teaching/rn/animations/csma/index.htm).

### Cellular networks

Cellular networks (including the WiMAX network used in this experiment) use a scheduled MAC. In order to transmit, devices on the network first contact a centralized scheduler, ask for some uplink resources, and get instructions from the scheduler telling it when and on what frequency it can transmit.

This takes more time than immediately accessing the network, as in WiFi. But when there are many contenders, the scheduler can balance resources across devices and make sure that traffic belonging to a high-priority class does not have to wait an unreasonably long time to access the network.


### Access rules

The access rules for the wireless frequency impact the QoS of applications using the network. WiFi networks operate on unlicensed wireless frequency bands, which may be used by anyone subject to certain conditions. This means that there may be many networks all trying to use the same physical medium, which causes more transmission failures. WiMAX and other cellular networks use licensed frequency bands, which are regulated by the government such that only one network is permitted to use the physical medium in a given geographic area.

### Summary

In this experiment, we'll measure delay in accessing the channel for two kinds of wireless networks:

* **WiFi**: contention-based MAC, operates in unlicensed frequency bands.	
* **WiMAX**: centralized scheduler MAC, operates in licensed frequency bands.



## Results

The following image shows that under no load, we may access the WiFi channel much more quickly, since it is not scheduled. However, when the network is loaded, access delay may be much longer and the jitter is much larger relative to the scheduled WiMAX network.

![](/blog/content/images/2016/02/delay.svg)

The 25th, 50th, and 75th percentile values for the delay (in ms) of each network under each load condition were:

```
  Group.1 Group.2    x.25%    x.50%    x.75%
1    WiFi    Load 228.0000 355.0000 412.0000
2   WiMAX    Load 129.0000 137.0000 141.0000
3    WiFi No Load   2.4800   2.7800   3.7525
4   WiMAX No Load 123.0000 127.0000 131.0000
```


## Run my experiment



### Set up resources

SSH into witestlab.poly.edu at the beginning of your reserved time. (If you were already logged in at the beginning of your reserved time, log out and then log back in again.)

This experiment requires three wireless "nodes". The instructions here assume you are using node4, node7, and node8. These three nodes were chosen because their WiFi and WiMAX signal quality is acceptable for this experiment. If those are [not available](http://witestlab.poly.edu/site/activity/node-status) at the time of your reservation, choose any three nodes of those numbered 1-13, and modify the instructions accordingly.


Load the baseline disk image onto the nodes with the following command:

```
omf load -i baseline-witest.ndz -t omf.witest.node4,omf.witest.node7,omf.witest.node8
```

When the disk load process finishes successfully, turn the nodes on:

```
omf tell -a on -t omf.witest.node4,omf.witest.node7,omf.witest.node8
```


Use the [node status page](http://witestlab.poly.edu/respond/sites/witest/activity/node-status) to find out when your nodes are ready to log in to.

Then, open three terminals, and in each, SSH into the witestlab.poly.edu console and from there, into the console of each of your nodes:

```
# run this on WITest console
ssh root@nodeX
# where X is the node number, e.g. node4
```

Our next step will be to set up WiFi connectivity. On each wireless node, run

```
modprobe ath5k # load ath5k driver
ifconfig wlan0 up # bring up WiFi interface
iwconfig wlan0 essid 'witestlab' # connect to WiFi AP
```

(Source: [WiFi connectivity instructions](http://witestlab.poly.edu/respond/sites/witest/tutorial/wifi-connectivity-manual) for WITest)

Then assign an IP address to the wireless interface. On each node, run

```
n=$(hostname)
n=${n:4}
ifconfig wlan0 192.168.0.$n netmask 255.255.255.0 
```

This will set the IP address of the WiFi interface to 192.168.0.X where X is the number of the node (e.g. 192.168.0.4 for node4.)


Verify that you have connectivity between the nodes, e.g. run

```
ping -c 5 192.168.0.4
ping -c 5 192.168.0.7
ping -c 5 192.168.0.8
```

from any one node.

Next, set up WiMAX connectivity. On each node, run

```
wimaxcu scan
wimaxcu connect network 51
```

You should see the following output:

```
Connecting to GENI 4G Network...
Connection successful
```

Then set a static IP address on each node with 

```
n=$(hostname)
n=${n:4}
ifconfig wmx0 10.43.4.$n netmask 255.255.255.0 
```

This sets the IP address on the WiMAX interface to 10.43.4.X, where X is the number of the node (e.g. "10.43.4.4" for node4.)

Verify WiMAX connectivity by running the following on each node:

```
ping -c 5 10.43.4.200
```

### Delay without load

First we are going to measure the round-trip delay on the network when there is no load.

On node7, send a series of pings to node8 over the WiFi network:


```
ping -i 0.001 -c 2000 192.168.0.8 | tee wifipings.txt
```

The results will be displayed on the terminal and also redirected to a file "wifipings.txt".

Then, also on node7, send a series of pings to node8 over the WiMAX network:

```
ping -i 0.001 -c 2000 10.43.4.8 | tee wmxpings.txt
```

### Delay with load

Now we will measure the delay of each network when there is other traffic (load) on the network.

On node8, run

```
iperf -s -i 1
```

to prepare it to receive this other traffic. Leave this running.

We will initiate the other traffic flow ("load") from node4. On node4, run

```
iperf -c 192.168.0.8 -i 1 -t 60
```

While that is running, run the ping test on node7 again:

```
ping -i 0.001 -c 2000 192.168.0.8 | tee wifipings-load.txt  
```

Now we will repeat this test on the WiMAX network.

On node4, run

```
iperf -c 10.43.4.8 -i 1 -t 60
```

While that is running, run the ping test on node7 again:

```
ping -i 0.001 -c 2000 10.43.4.8 | tee wmxpings-load.txt  
```


### Data analysis

Open a new terminal and log in to witestlab.poly.edu with your GENI wireless username and key.

Transfer the ping output files from your node to the WITest console:

```
scp root@node7:/root/wifipings.txt .
scp root@node7:/root/wmxpings.txt .
scp root@node7:/root/wifipings-load.txt .
scp root@node7:/root/wmxpings-load.txt .

```

The convert these to a single CSV file:

```
cat wifipings.txt | grep "bytes from" | awk 'BEGIN { FS = " |=" } ; { print "WiFi,No Load," $6 "," $10 }' > pingtimes.csv
cat wmxpings.txt | grep "bytes from" | awk 'BEGIN { FS = " |=" } ; { print "WiMAX,No Load," $6 "," $10 }' >> pingtimes.csv
cat wifipings-load.txt | grep "bytes from" | awk 'BEGIN { FS = " |=" } ; { print "WiFi,Load," $6 "," $10 }' >> pingtimes.csv
cat wmxpings-load.txt | grep "bytes from" | awk 'BEGIN { FS = " |=" } ; { print "WiMAX,Load," $6 "," $10 }' >> pingtimes.csv

``` 

Then, run 

```
R
```

In your R terminal, paste the following commands:

```
library(ggplot2)

exp <- read.table("pingtimes.csv", 
        colClasses=c("factor","factor","integer","numeric"),
        header=FALSE, sep=",")  
names(exp) <- c("network", "load", "seq", "rtt")  

q <- ggplot(exp, aes(y=as.factor(network),x=rtt))
q <- q + theme_minimal(base_size = 18)  
q <- q + geom_jitter(alpha=0.1)  
q <- q + scale_x_continuous("Round trip time (ms)")  
q <- q + scale_y_discrete("")  
q <- q + theme(legend.position="bottom")
q <- q + facet_grid(~load)

svg("delay.svg")  
print(q)  
dev.off()


png("delay.png")  
print(q)  
dev.off()

# print 25th, 50th, and 75th percentiles
aggregate(exp$rtt, by = list(exp$network, exp$load), FUN = function(x) quantile(x, probs = c(0.25,0.5,0.75)))

quit("no")  
```

The last command above will print the 25th, 50th, and 75th percentile values for the delay of each network under each load condition, e.g. 

```
  Group.1 Group.2    x.25%    x.50%    x.75%
1    WiFi    Load 228.0000 355.0000 412.0000
2   WiMAX    Load 129.0000 137.0000 141.0000
3    WiFi No Load   2.4800   2.7800   3.7525
4   WiMAX No Load 123.0000 127.0000 131.0000
```

The plot of delay values will be saved as "delay.png" and "delay.svg" in your home directory on the WITest console. To transfer these to your laptop, run (in a local console):

```
scp GENI-USERNAME@witestlab.poly.edu:~/delay.png .
scp GENI-USERNAME@witestlab.poly.edu:~/delay.svg .
```

substituting your GENI wireless username (and possibly, location of your keys.)