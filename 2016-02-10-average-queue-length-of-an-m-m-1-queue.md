This experiment reproduces a classic result in queueing theory: the length of the M/M/1 queue as its utilization approaches 100%. (See e.g. Kleinrock, Leonard (1975). *Queueing Systems: Theory, Volume 1*. Wiley. ISBN 0471491101.)

It should take about 60 minutes of active time and 8 hours of inactive time to run completely, from start (reserve resources) to finish (plot experiment results.) However, the first set of data (with only one experiment run for each value of utilization) will be available in 45 minutes.


To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes).

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background 

In queueing theory, an M/M/1 queue is a system with a single queue, where arrivals at the queue are determined by a Poisson process (i.e. with exponentially distributed times between packets) and job service times have an exponential distribution. (In the context of a network, the queue is at a switch or a router, and the arrivals are packet arrivals.)


It is known that the mean number of packets in this system is given by

$$ \frac{\rho}{1-\rho} $$

where

$$\rho =  \frac{\lambda}{\mu} $$ 
                                                                                                             
&lambda; is the rate at which packets arrive at the queue (in packets/second), and &mu; is the rate at which packets may be served by the queue (in packets/second). 

![](https://upload.wikimedia.org/wikipedia/commons/thumb/6/65/Mm1_queue.svg/640px-Mm1_queue.svg.png)

*<sup>Image by Tsaitgaist [CC BY-SA 3.0](http://creativecommons.org/licenses/by-sa/3.0) via Wikimedia Commons</sup>*


Usually, the distribution of &mu; comes from variations in the size of packets arriving at the queue, which has a constant service rate in bits/second. In other words, packet sizes are exponentially distributed and &mu; is computed by dividing the queue service rate in bits/second by the average packet size in bits.

From the mean number of packets in the system, we can compute the mean number of 
packets in the queue by substracting the mean number of packets in service:
 
$$ \frac{\rho}{1-\rho} - \rho = \frac{\rho^2}{1-\rho} $$                         

As &rho; grows close to 1, the mean queue length grows very quickly.                                                                                    

In this experiment, we will reproduce the theoretical result for mean queue length.


## Results

The results of our measurements are as follows:

![](/blog/content/images/2016/02/mm1-qlenexp-1.svg)

<sup>*Mean queue length for varying &rho; by analytical approach and testbed measurement. (The points represent the mean value and the lines show the minimum and maximum values across five independent trials for each &rho;.)*</sup>

We can see that the testbed measurements are generally consistent with the analytical results. For small values of &rho;, the queue length in the testbed experiment is smaller than the theoretical prediction. This may be due to the "burst" behavior of the practical queue, as well as fragmentation (for packets that are larger than the MTU of the link). For large values of &rho;, where small variations in &rho; lead to large changes in average queue length, the measurement results are less consistent.

The difference between analytical and experimental results becomes clearer when we plot the queue length on a log10 scale:

![](/blog/content/images/2016/02/mm1-qlenexp-log.svg)

## Run my experiment


To start, create a new slice on the [GENI portal](https://portal.geni.net/). Create a client-router-server topology with three standard VMs, like this:

![](http://witestlab.poly.edu/~ffund/el7353/images/jacks-topology.png)

Next, click on the small square boxes represeting the links, and assign IP addresses to each network interface:

* Assign 10.10.1.1 to the client 
* Assign 10.10.1.2 to the interface on the router that is connected to the client
* Assign 10.10.2.1 to the interface on the router that is connected to the server
* Assign 10.10.2.2 to the server

Alternatively, you may download [this RSpec](https://gist.github.com/ffund/d147dfe585d6b78dfb8ec7b8bab5bb3c) and then load the RSpec into the GENI Portal from the file. Or, you can load the RSpec directly from this URL: [https://git.io/vPyaO](https://git.io/vPyaO) (If you use this RSpec, it will also install some software onto your resources when they boot up.)

Then, click on "Site 1", choose an InstaGENI aggregate, and reserve your resources. Wait until your resources are ready, and then log in.

### Sending traffic through a bottleneck queue

In this experiment, we will send Poisson traffic through a bottleneck queue with an exponentially distributed service time, like this:

![](/blog/content/images/2017/09/mm1-scenario-1.svg)

We will use an mean packet size of 512 bytes and a consistent queue rate of 1 Mbps, so for all experiment trials, the mean service rate &mu; = 1000000/(512*8) = 244.14. To get the queue length for different values of &rho; we will only vary &lambda;.


To generate traffic with exponentially distributed interarrival times and exponentially distributed packet sizes, we will use the [D-ITG traffic generator](http://traffic.comics.unina.it/software/ITG/) ([manual available here](http://traffic.comics.unina.it/software/ITG/manual/).) You will need to install D-ITG on your client and server nodes. Log in to each and run

```
sudo apt-get update # refresh local information about software repositories
sudo apt-get -y install d-itg # install D-ITG software package
```



So, first find the name of the interface on the router (e.g. eth1, eth2) that has the IP address 10.10.2.1 - this is the interface that packets will leave from en route to the server. Use `ifconfig` on the router node to find out the interface name. Then, to set the queue to operate at 1 Mbps, run:

<pre>
sudo tc qdisc replace dev <b>eth1</b> root tbf rate 1mbit limit 200mb burst 32kB peakrate 1.0001mbit mtu 1600
</pre>

where you replace the part in bold with the relevant interface name. This command sets up a [token bucket filter queue](https://linux.die.net/man/8/tc-tbf) for outgoing traffic on that interface, which throttles traffic to 1 Mbps.

At any given time, we can see the current state of the queue, including the queue length, by running

<pre>
tc -p -s -d qdisc show dev <b>eth1</b>
</pre>

again, substituting the appropriate interface name. It should show that it is rate limited to 1 Mbps:

<pre>
qdisc tbf 8001: root refcnt 2 <b>rate 1Mbit</b> burst 32Kb/1 mpu 0b peakrate 1Mbit mtu 1599b/1 mpu 0b lat 1677.5s linklayer ethernet 
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0) 
 backlog 0b 0p requeues 0 
</pre>

It also shows:

* the total number of bytes/packets - `Sent 0 bytes 0 pkt`
* the total number of packets dropped by the queue (e.g. because the queue was full when the packet arrived or because the queue is configured to drop backets) - `dropped 0`
* the total number of packets that have exceeded the configured rate limit - `overlimits 0`
* the total number of packets that were removed from the queue, but then reinserted back into the queue because they could not be transmitted - `requeues 0`

since the queue began operating. It also shows the _current_ status of the queue, i.e. the number of bytes and packets currently in the queue: here `backlog 0b 0p` shows that the queue is empty.


For convenience, we've written a script that runs this commands at regular intervals and records the queue information together with the timestamp. You can download this script file on the router node and then make it executable with 

```
wget https://git.io/v6KVn -O queuemonitor.sh
chmod a+x queuemonitor.sh
```

(You can see the contents of this script in [this gist](https://gist.github.com/ffund/b4fb116094e4dcd3ec46af14648ba75e).)

This script accepts three arguments: the name of the interface on which the queue is running, the duration (in seconds) for which you want to record measurements, and the interval between measurements (in seconds).

Now let's send some traffic to test our setup. On the server node, which will act as the traffic sink, run:

```
ITGRecv
```


On the client node, which will be the traffic source, run:

```
ITGSend -a server -l sender.log -x receiver.log -E 220 -e 470 -T UDP -t 120000
```

which will send UDP traffic for 120 seconds (120000 milliseconds) with an exponentially distributed time between packets at an average rate of 220 packets/second, exponentially distributed packet sizes with an average size of 512 bytes. (We use 470 for the D-ITG argument to specify the mean packet size, rather than 512, because some packet headers will be added: an 8-byte UDP header, a 20-byte IP header, and a 14-byte Ethernet header.)

Within a few seconds of starting the sender, on the router node, start the queue monitor script, simultaneously redirecting its output to a file "router.txt":

<pre>
bash queuemonitor.sh <b>eth1</b> 120 0.1 | tee router.txt
</pre>

where you substitute for `eth1` the name of the interface connecting the router node to the server node. This will run queue monitor script for 120 seconds and report data every 0.1 seconds.


After the experiment is over, get the average queue size by running on the router node:

```
cat router.txt | sed 's/\p / /g' | awk  '{ sum += $37 } END { if (NR > 0) print sum / NR }'
```

This prints the contents of the file to the terminal, then uses `sed` and `awk` to find the mean value queue size.

We have used &lambda; = 220 for this experiment, so &rho; = 220/244.14 = 0.90 and the average queue length is expected to be about 0.90<sup>2</sup>/(1-0.90) = 8.1 packets. 

You can also collect information from the D-ITG logs by running

```
ITGDec sender.log
```

on the client node, and 

```
ITGDec receiver.log
```

on the receiver node.


### Run a systematic experiment

Now we would like to systematically run this experiment many times for different values of &lambda;. To do this, we can use a simple shell script to run the experiment in a loop, and to coordinate between the nodes in the experiment. Save the following on the client and router nodes in a script called `mm1experiment.sh`:

```bash
#!/bin/bash

# Find name of interface through which traffic goes to server
IFACE=$(ip route get 10.10.2.255 | head -n 1 | cut -d' ' -f 4)

l_start=120
l_end=240

for i in {1..10}; do
    for (( l = ${l_start}; l <=${l_end}; l=l+5 )); do

        if [ "$(hostname --short)" = "client" ]; then
            echo "START" | netcat router 8888  # contact router
            echo "Sending traffic at rate $l ($i)"
            ITGSend -a server -l "sender-$l-$i.log" -x "receiver-$l-$i.log" -E "$l" -e 470 -T UDP -t 110000
        fi

        if [ "$(hostname --short)" = "router" ]; then
            netcat -l 8888  > /dev/null # Wait for contact from client
            sleep 10
            bash queuemonitor.sh "$IFACE" 90 0.1 > "router-$l-$i.txt"
            echo -n "$l,$i,"
            cat "router-$l-$i.txt" | \
                sed 's/\p / /g' | \
                awk  '{ sum += $37 } END { if (NR > 0) print sum / NR }'
        fi
    done
done
``` 

Since this script will take a long time to run, we will want to set it up so that it will continue running even when we log off from our SSH session. To do this, we will need to install a utility called [screen](https://kb.iu.edu/d/acuy). Install `screen` on all three nodes - the server, the router, and the client - as follows:

```
sudo apt-get update
sudo apt-get -y install screen
```

To run the experiment, save the shell script above as "mm1experiment.sh" on the client and router nodes. Then, perform the following steps in order:

1. Run `screen` on the server. In the terminal that opens, run `ITGRecv`.
2. Run `screen` on the router. In the terminal that opens, run `bash mm1experiment.sh | tee mm1experiment.csv`.
3. Run `screen` on the client. In the terminal that opens, run `bash mm1experiment.sh`.

This script will then run the experiment ten times each for values of &lambda; from 120 to 240, in increments of 5. The duration of each experiment will be 90 seconds, and the entire script will take about 8 hours.

The value of &lambda;, the experiment number, and the average queue length will be printed to STDOUT on the router node at the end of each experiment. The output is also redirected to a file "mm1experiment.csv".

While the experiment runs, you can detach from your `screen` session in each of the three terminals with Ctrl+A followed by D. When your session is in a detached state, you can close the terminal or lose your SSH connection and the processes running in the `screen` session will continue. At any time, you can log back in and re-attach to the screen session to check on its progress - use 

```
screen -r
```

to reattach. 

### Analyze experiment results

You can plot the experiment results, which are stored on the router node, with R. (You can plot results as they become available, and update the plots whenever you want - you don't have to wait for the experiment to finish completely before you start looking at the data.)

To install R on the router node, run 

```
gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9
gpg -a --export E084DAB9 | sudo apt-key add -

echo "deb http://cran.wustl.edu/bin/linux/ubuntu $(lsb_release -cs)/" | sudo tee --append /etc/apt/sources.list

sudo apt-get update
sudo apt-get -y install r-base r-cran-ggplot2

```

Then, assuming the output on the router has been redirected to a file named "mm1experiment.csv", we can run `R` and paste the following into the R terminal (or alternatively, save it as a script "mm1experiment.R" and then run `R CMD BATCH mm1experiment.R`):

```r
library(ggplot2)

exp <- read.table("mm1experiment.csv", header=FALSE, sep=",")
names(exp) <- c("lambda", "exp", "qlen")
exp$rho <- exp$lambda/244.140625

rhos <- seq(from = 0.45, to = 0.999, by = 0.001)
qlens <- rhos*rhos/(1-rhos)
analytical <- data.frame(rho=rhos, qlen=qlens)

q <- ggplot() + theme_minimal(base_size = 18)

q <- q + geom_line(data=analytical, aes(x=rho,
    y=qlen, 
    colour="Analytical"))
q <- q + stat_summary(data=exp, aes(x=rho, y=qlen, 
    colour="Testbed Experiment"), 
    fun.y = mean, fun.ymin = min, 
    fun.ymax = max,alpha=0.4)
q <- q + scale_colour_discrete("")
q <- q + scale_x_continuous("ρ", limits=c(0.45,1))
q <- q + scale_y_continuous("Average queue length")
q <- q + theme(legend.position="bottom")

png("mm1-qlenexp.png")
print(q)
dev.off()

quit("no")
```

You can then use `scp` to retrieve the "mm1-qlenexp.png" file from the router node.


To create the version of the plot with the log10 scale on the y-axis, replace 

```r
q <- q + scale_y_continuous("Average queue length")
```

with 

```r
q <- q + scale_y_continuous("Average queue length", trans = "log10")
q <- q + annotation_logticks(sides = "l")
```



Note that the experiment script also saves all the log files (from sender, receiver, and router) from the experiment. Use these to validate the experiments (e.g. verify the actual values of &lambda; and &mu;, make sure no packets were dropped, etc.)

Please delete your resources when you're finished to free them up for other experimenters!

## Notes


### Common problems

If you try to start an `ITGRecv` when one is already running, you may encounter this error message:

```
** ERROR_TERMINATE **
Function main aborted caused by general parser 
** Cannot bind a socket on port 9000 for signaling ** 
Finish requested caused by errors! 
```

To kill the already running `ITGRecv` so that you can start a new one, run

```
sudo killall ITGRecv
```

### Software versions

This experiment was developed on the following software versions:

 * Ubuntu 14.04.1
 * Linux kernel 3.13.0-33-generic
 * D-ITG 2.8.1-r1023-3 from the Ubuntu repositories
 * R 3.2.3
 * ggplot2 0.9.3.1
