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

This mean queue length is the quantity we will be measuring in this experiment, in an attempt to reproduce the theoretical result


## Results

The results of our measurements are as follows:

![](/blog/content/images/2016/02/mm1-qlenexp-1.svg)

<sup>*Mean queue length for varying &rho; by analytical approach and testbed measurement. (The points represent the mean value and the lines show the minimum and maximum values across five independent trials for each &rho;.)*</sup>

We can see that the testbed measurements are generally consistent with the analytical results. For small values of &rho;, the queue length in the testbed experiment is smaller than the theoretical prediction. This may be due to the "burst" behavior of the practical queue. For large values of &rho;, where small variations in &rho; lead to large changes in average queue length, the measurement results are less consistent.

The difference between analytical and experimental results becomes clearer when we plot the queue length on a log10 scale:

![](/blog/content/images/2016/02/mm1-qlenexp-log.svg)

## Run my experiment


To start, create a new slice on the [GENI portal](https://portal.geni.net/). Create a client-router-server topology with three standard VMs, like this:

![](http://witestlab.poly.edu/~ffund/el7353/images/jacks-topology.png)

Use the "Auto IP" tool to automatically assign appropriate IPs. Then choose an InstaGENI aggregate and log in to your resources. Alternatively, you may download [this RSpec](https://gist.github.com/ffund/d147dfe585d6b78dfb8ec7b8bab5bb3c) and then load the RSpec into the GENI Portal from the file. Or, you can load the RSpec directly from this URL: [https://git.io/vPyaO](https://git.io/vPyaO)

To generate traffic with exponentially distributed interarrival times and exponentially distributed packet sizes, we will use the [D-ITG traffic generator](http://traffic.comics.unina.it/software/ITG/) ([manual available here](http://traffic.comics.unina.it/software/ITG/manual/).) You will need to install D-ITG on your client and server nodes. Log in to each and run

```
sudo apt-get update # refresh local information about software repositories
sudo apt-get install d-itg # install D-ITG software package
```


In this experiment, I have used an average packet size of 512 bytes and a consistent queue rate of 1 Mbps, so for all experiment trials, &mu; = 1000000/(512*8) = 244.14. To get the queue length for different values of &rho; we will only vary &lambda;.

So, to set the queue to operate at 1 Mbps, log in to the router node and run:

```
sudo tc qdisc replace dev eth2 root tbf rate 1mbit limit 200mb burst 32kB peakrate 1.0001mbit mtu 1600
```

where `eth2` is the interface connecting the router node to the server node.

At any given time, we can see the current state of the queue, including the queue length, by running

```
tc -p -s -d qdisc show dev eth2
```

For convenience, we've written a script that runs this at regular intervals and records the queue information together with the timestamp. You can download this script file on the router node and then make it executable with 

```
wget https://git.io/v6KVn -O queuemonitor.sh
chmod a+x queuemonitor.sh
```

The contents of this script are:

```bash
#!/bin/bash

if [ "$#" -ne 3 ]; then
    echo "Usage: $0 interface duration interval"
    echo "where 'interface' is the name of the interface on which the queue"
    echo "running (e.g., eth2), 'duration' is the total runtime (in seconds),"
    echo "and 'interval' is the time between measurements (in seconds)"
    exit 1
fi

interface=$1
duration=$2
interval=$3

# Run for the number of seconds specified by the "duration" argument
end=$((SECONDS+$duration))

while [ $SECONDS -lt $end ]
do
        # print timestamp at the beginning of each line;
        # useful for data analysis. (-n argument says not to
        # create a newline after printing timestamp.)
        echo -n "$(date +%s.%N) "
        # Show current queue information on the specified
        # interface, all on one line.
        echo $(tc -p -s -d qdisc show dev $interface)
        # sleep for the specified interval
        sleep $interval
done
```

Now let's send some traffic to test our setup. On the server node, which will act as the traffic sink, run:

```
ITGRecv
```


On the client node, which will be the traffic source, run:

```
ITGSend -a server -l sender.log -x receiver.log -E 220 -e 512 -T UDP -t 60000
```

which will send UDP traffic for 60 seconds (60000 milliseconds) with an exponentially distributed time between packets at an average rate of 220 packets/second, exponentially distributed packet sizes with an average size of 512 bytes.

And within a few seconds of starting the sender, on the router node, start the queue monitor script, simultaneously redirecting its output to a file "router.txt":

```
bash queuemonitor.sh eth2 40 0.1 | tee router.txt
```

where `eth2` is the interface connecting the router node to the server node, and we want the queue monitor to run for 40 seconds and report data every 0.1 seconds.


After the experiment is over, get the average queue size by running on the router node:

```
cat router.txt | sed 's/\p / /g' | awk  '{ sum += $37 } END { if (NR > 0) print sum / NR }'
```

This prints the contents of the file to the terminal, then uses `sed` to replace instances of "p" with a blank space " " (since the value we are interested in is in the form "2p"). Finally, it uses `awk` to find the mean value of the 37th column.

We have used &lambda; = 220 for this experiment, so &rho; = 220/244.14 = 0.90 and the average queue length should be about 0.90<sup>2</sup>/(1-0.90) = 8.1 packets.

You can also collect information from the D-ITG logs by running

```
ITGDec sender.log
```

on the client node, and 

```
ITGDec receiver.log
```

on the receiver node.

Assuming everything is set up correctly, we would like to systematically run this experiment many times for different values of &lambda;. To do this, I have written a script:

```bash
#!/bin/bash

l_start=120
l_end=240

for i in {1..10}; do
    for (( l = ${l_start}; l <=${l_end}; l=l+5 )); do

        if [ "$(hostname --short)" = "client" ]; then
            echo "START" | netcat router 8888  # contact router
            echo "Sending traffic at rate $l ($i)"
            ITGSend -a server -l "sender-$l-$i.log" -x "receiver-$l-$i.log" -E "$l" -e 512 -T UDP -t 110000
        fi

        if [ "$(hostname --short)" = "router" ]; then
            netcat -l 8888  > /dev/null # Wait for contact from client
            sleep 10
            bash queuemonitor.sh eth2 90 0.1 > "router-$l-$i.txt"
            echo -n "$l,$i,"
            cat "router-$l-$i.txt" | \
                sed 's/\p / /g' | \
                awk  '{ sum += $37 } END { if (NR > 0) print sum / NR }'
        fi
    done
done
``` 


To run the experiment, save the script above as "mm1experiment.sh" on the client and router nodes. Then, perform the following steps in order:

1. Run `ITGRecv` on the server.  (Or, if you want to log off and leave it running, `nohup ITGRecv &`.)
2. Run `bash mm1experiment.sh | tee mm1experiment.csv` on the router. (Or, if you want to log off and leave it running in the background, `nohup bash mm1experiment.sh | tee mm1experiment.csv &`.)
3. Run `bash mm1experiment.sh` on the client. (Or, if you want to log off and leave it running, `nohup bash mm1experiment.sh &`.)

This script will then run the experiment ten times each for values of &lambda; from 120 to 240, in increments of 5. The duration of each experiment will be 90 seconds, and the entire script will take about 8 hours.

The value of &lambda;, experiment number, and average queue length will be printed to STDOUT on the router node at the end of each experiment. The output is also redirected to a file "mm1experiment.csv".

We can then plot the experiment results with R. (You can plot results as they become available, and update the plots whenever you want - you don't have to wait for the experiment to finish completely before you start looking at the data.)

To install R on the router node, become root with `sudo su` and then run 

```
gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9
gpg -a --export E084DAB9 | apt-key add -

echo "deb http://cran.wustl.edu/bin/linux/ubuntu trusty/" >> /etc/apt/sources.list

apt-get update
apt-get install r-base r-cran-ggplot2

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

This experiment was developed on the following software versions:

 * Ubuntu 14.04.1
 * Linux kernel 3.13.0-33-generic
 * D-ITG 2.8.1-r1023-3 from the Ubuntu repositories
 * R 3.2.3
 * ggplot2 0.9.3.1