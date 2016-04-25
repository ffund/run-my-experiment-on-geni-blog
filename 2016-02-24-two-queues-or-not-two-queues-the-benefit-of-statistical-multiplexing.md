This experiment reproduces a classic result in queueing theory, statistical multiplexing gain in queues. (See e.g. Example 3.9 in Bertsekas, Dimitri, and Robert Gallager. "Data networks." Second Edition.) It answers the basic question: is it faster to serve a set of requests with one fast server, or many slower servers?

It should take about 60-180 minutes to run this experiment from start (reserve resources) to finish (plot experiment results.) 


To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes).

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

In this experiment, we compare statistical multiplexing with time or frequency division multiplexing. 

Assume we have *m* separate identically distributed and independent Poisson arrival streams each with parameter &lambda;/m (i.e. packets in each stream arrive at an average rate of &lambda;/m packets per second, with each stream having independent, exponentially distributed interarrival times.) We can serve these with two possible configurations:

 * We may serve all of the streams with a single service with mean service time 1/&mu; (statistical multiplexing, shown on the left), or
 * We may serve each stream with its own service, with each of the _m_ services having a mean service time of m/&mu; (TDM, shown on the right).

![](/blog/content/images/2016/02/tdm-vs-sm.svg)

In either case, the utilization &rho; of each service, defined as the rate of packet arrivals divided by the service rate, is the same:

$$ \rho\_{SM} = \frac{\sum^m \frac{\lambda}{m}}{\mu} = \frac{\lambda}{\mu}$$

$$ \rho\_{TDM} = \frac{\frac{\lambda}{m}}{\frac{\mu}{m}} = \frac{\lambda}{\mu}$$

and the mean number of packets being served or waiting for service in each queue will be

$$N = \frac{\rho}{1-\rho}$$

in either case. However, the average delay in the system, which is equal to _N_ times the service rate, will be longer in the TDM case by a factor of _m_:

$$ T\_{TDM} = \frac{m}{\mu - \lambda}$$

vs. 

$$ T\_{SM} = \frac{1}{\mu - \lambda}$$


This delay reduction is known as a "statistical multiplexing gain."

## Results

Our results show that the queuing delay is smaller for the statistical multiplexing case (red line) relative to the TDM case (blue line):

![](/blog/content/images/2016/02/tdm-sm-delay-1.svg)

The 25th, 50th, and 75th percentile values were 2.520, 7.595, and 15.600 ms for the statistical multiplexing case, and 3.38, 13.90, and 30.40 ms for the TDM case. The mean delay in the statistical multiplexing case was 0.546% of the mean delay in the TDM case.  


## Run my experiment


We are going to run two versions of this experiment: the statistical multiplexing version, and the TDM version. In each experiment, we will generate two Poisson traffic streams with &lambda;=110 packets/second and exponentially distributed packet sizes with a mean of 512 B. In each experiment, we will produce a CDF of the queue length and a CDF of delay measurements from each sender.

### Reserve resources

Use [this RSpec](https://nyu.box.com/shared/static/zn9ij1eq12rqupwq19u715t5odqu5eog.xml) to reserve the two-queue topology. (In the GENI Portal, in your slice, click "Add Resources", then "Choose RSpec" from file. After loading the RSpec, bind it to a *different* InstaGENI site):

![](/blog/content/images/2016/02/two-queues.png)

Wait until your nodes are ready to log in.


### Statistical multiplexing version


First we'll run the statistical multiplexing version of this experiment. 

In this topology, we will send two traffic streams to a single receiver, via a single egress queue on an outgoing interface on the router.

To set the service rate of the single queue, SSH into the router node. Identify the name of the interface connected to receiver-1 (e.g. eth1, eth2) by running

```
ifconfig
```

The interface connected to the receivers is the one with IP address 10.10.10.1.

When you have the name of the interface connected to the receivers, run

```
sudo tc qdisc replace dev eth2 root netem rate 1000kbit
```

to apply a simple rate limiting queue to the egress chain of that interface. (Substitute "eth2" in the command above with the name of the correct interface.) This sets the "service rate" of the queue to 1 Mbps. When we generate traffic with a mean packet size of 512 B, the service rate of our queue will be &mu; = 1e6/(8*512) &approx; 244.14 packets/second.

Then, open another terminals and log into the receiver-1 node. Run

```
ITGRecv
```

to start the D-ITG receiver. Leave this running.

Open another two terminals and log into each sender node.

On sender-1, generate a Poisson traffic stream to receiver-1 with a mean packet rate of 110 packets per second and exponentially distributed packet sizes with a mean of 512 B using

```
ITGSend -a receiver-1 -E 110 -e 512 -T UDP -t 600000
```

and run the same command on sender-2.

These streams will run for 600 seconds.

Finally, open two more terminals, and log on to each sender node again. On sender-1, run

```
ping -i 0.2 -c 1000 receiver-1 | tee pingtimes.txt
```

and on sender-2, run the same command.

This will measure the round-trip delay between each sender and receiver-1 1000 times, with a delay of 200 ms between each sample. The results will be displayed in the terminal output and are also redirected to a file named "pingtimes.txt".

### Data analysis for statistical multiplexing case

On sender-1, run

```
cat pingtimes.txt | grep "bytes from" | awk 'BEGIN { FS = " |=" } ; { print "1,SM," $7 "," $11 }' > pingtimes-1-sm.csv
```

to turn the ping output into a CSV file, and on sender-2 run

```
cat pingtimes.txt | grep "bytes from" | awk 'BEGIN { FS = " |=" } ; { print "2,SM," $7 "," $11 }' > pingtimes-2-sm.csv
```

Keep these two files ("pingtimes-1-sm.csv" and "pingtime-2-sm.csv") in a safe place for later use.

### TDM version

Now we're going to run the TDM version of this experiment. 

In this topology, the router has two outgoing interfaces, one for each receiver. We will set up a separate egress queue on each interface. Traffic destined to exit from a particular interface will only wait in the queue associated with that interface.

SSH into the router node. Identify the names of the _two_ interfaces connected to the receivers (e.g. eth1, eth2) by running

```
ifconfig
```

The interface connected to the receiver-1 node is the one with IP address 10.10.10.1, and the interface connected to the receiver-2 node is the one with IP address 10.10.20.1.

When you have the names of the interfaces connected to the receivers, run

```
sudo tc qdisc replace dev eth2 root netem rate 500kbit
```

for *each* of those two interfaces to apply a simple rate limiting queue to the egress chain of that interface. (Substitute "eth2" in the command above with the name of the correct interface.) This sets the "service rate" of each queue to 500 kbps. When we generate traffic with a mean packet size of 512 B, the service rate of our queue will be &mu; = 500e3/(8*512) &approx; 122.07 packets/second.

Then, open two more terminals and log into each receiver node and run

```
ITGRecv
```

to start the D-ITG receiver. Leave these running.

On sender-1, generate a Poisson traffic stream to receiver-1 with a mean packet rate of 110 packets per second and exponentially distributed packet sizes with a mean of 512 B using

```
ITGSend -a receiver-1 -E 110 -e 512 -T UDP -t 600000
```

and on sender-2,

```
ITGSend -a receiver-2 -E 110 -e 512 -T UDP -t 600000
```

These streams will run for 600 seconds. 

Finally, open two more terminals, and log on to each sender node. On sender-1, run

```
ping -i 0.2 -c 1000 receiver-1 | tee pingtimes.txt
```

and on sender-2 run

```
ping -i 0.2 -c 1000 receiver-2 | tee pingtimes.txt
```

### Data analysis for TDM case

On sender-1, run

```
cat pingtimes.txt | grep "bytes from" | awk 'BEGIN { FS = " |=" } ; { print "1,TDM," $7 "," $11 }' > pingtimes-1-tdm.csv
```

to turn the ping output into a CSV file and on sender-2 run

```
cat pingtimes.txt | grep "bytes from" | awk 'BEGIN { FS = " |=" } ; { print "2,TDM," $7 "," $11 }' > pingtimes-2-tdm.csv
```

Keep these two files ("pingtimes-1-tdm.csv" and "pingtime-2-tdm.csv") in a safe place for later use.

### Visualization of experiment results

Find out the hostname and port of all four sender nodes (from both experiments) from the GENI Portal. Then, in a local shell, use SCP to retrieve all four CSV files from the two sender nodes. For example, I ran:

```
scp -P 32058 ffund01@pc2.instageni.iu.edu:~/pingtimes-1-sm.csv .
scp -P 32058 ffund01@pc1.instageni.iu.edu:~/pingtimes-2-sm.csv .

scp -P 32058 ffund01@pc2.instageni.iu.edu:~/pingtimes-1-tdm.csv .
scp -P 32058 ffund01@pc1.instageni.iu.edu:~/pingtimes-2-tdm.csv .

```

but you will have to use your own username, port numbers, and hostnames.

Then copy all four files back onto any one node in your experiment. For example, I ran

```
scp -P 32058 pingtimes* ffund01@pc1.instageni.iu.edu:~/
```

Log onto that node and concatenate the four files together with

```
cat pingtimes-1-sm.csv pingtimes-2-sm.csv pingtimes-1-tdm.csv pingtimes-2-tdm.csv > pingtimes-all.csv
```

Then run 

```
R
```

and paste the following commands into your R terminal:

```r
library(ggplot2)

exp <- read.table("pingtimes-all.csv", header=FALSE, sep=",")  
names(exp) <- c("sender", "type", "seq", "rtt")  

q <- ggplot(exp, aes(x=rtt, 
                     linetype=as.factor(sender),
                     colour=as.factor(type))
            )
q <- q + ggtitle("CDF of delay for M/M/1 queue")
q <- q + theme_minimal(base_size = 18)
q <- q + stat_ecdf()
q <- q + scale_x_continuous("Round trip time (ms)")
q <- q + scale_y_continuous("")
q <- q + scale_linetype_discrete("Sender")
q <- q + scale_colour_discrete("Type")
q <- q + theme(legend.position="bottom")

svg("delay.svg")  
print(q)  
dev.off()


png("delay.png")  
print(q)  
dev.off()

# print 25th, 50th, and 75th percentiles
with(exp, tapply(rtt, type, quantile, probs=c(0.25,0.5,0.75,1.0)))


quit("no")  
```

The output will include the percentile values from each experiment, like this:

```
> with(exp, tapply(rtt, type, quantile, probs=c(0.25,0.5,0.75,1.0)))
$SM
    25%     50%     75%    100% 
  2.520   7.595  15.600 110.000 

$TDM
   25%    50%    75%   100% 
  3.38  13.90  30.40 193.00 
```

and it will also create a CDF plot in both PNG and SVG format, which you can retrieve with SCP. To view the results quickly, you can run

```
imgurbash delay.png
```

to upload the image to imgur.com. Copy the link from the terminal into your browser to see the results.

Please delete your resources when you're finished to free them up for other experimenters!

## Notes

Several applications and scripts have been pre-installed on one or more nodes in this experiment as part of the RSpec:

- D-ITG
- R and ggplot2
- curl, which is required for the imgur upload script
- [*Bart's Bash Script Uploader*](http://imgur.com/tools/imgurbash.sh) by Bart Nagel for uploading to imgur
- The queue monitor script developed for [this experiment](http://witestlab.poly.edu/blog/average-queue-length-of-an-m-m-1-queue/)
