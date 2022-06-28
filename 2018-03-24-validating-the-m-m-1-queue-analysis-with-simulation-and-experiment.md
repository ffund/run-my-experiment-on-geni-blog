In this assignment, you will validate the classic result for the queue length of the M/M/1 queue with a series of experiments on the GENI testbed and the ns2 simulator.

After completing this lab assignment, you should:

* Know how to run a basic queuing experiment on GENI
* Be able to use `tc` to manipulate queues on Linux
* Be able to use the D-ITG traffic generator to generate traffic with specific properties on Linux
* Know how to run a basic queuing experiment in ns2
* Understand how to "sanity-check" an experiment to verify that it is a valid experiment
* Compare analytical, simulation, and testbed experiment results for the average length of the M/M/1 queue

In future lab assignments, you will use the same tools to design, implement, execute, and validate your own experiments.


### Background

In this experiment, you're going to measure the average queue length in the following scenario:

![](http://witestlab.poly.edu/~ffund/el7353/images/mm1-scenario-1.svg)


We know that the mean number of packets in this system is given by


$$\frac{\rho}{1-\rho}$$

where

$$\rho =  \frac{\lambda}{\mu}$$

and λ and μ are the rate at which packets arrive at the queue (in packets/second) and the rate at which packets may be served by the queue (in packets/second). In our experiment, the distribution of μ comes from variations in the size of packets arriving at the queue, which has a constant service rate in bits/second - μ is computed by dividing the queue service rate in bits/second by the average packet size in bits.

From the mean number of packets in the system, we can compute the mean number of packets in the queue by subtracting the mean number of packets in service:


$$\frac{\rho}{1-\rho} - \rho = \frac{\rho^2}{1-\rho}$$


This mean queue length is the quantity we will be measuring in this lab assignment. You will create a plot similar to this one:


![](http://witestlab.poly.edu/~ffund/el7353/images/mm1-qlen.svg)

You will also include some discussion of the validity of the three different approachs: the analytical, simulation, and testbed.

### Run my experiment

To start, create a new slice on the GENI portal. Create a client-router-server topology like the one shown here, and assign IP addresses to each host (you can use the Auto IP button in the GENI portal, or manually assign addresses).

![](/blog/content/images/2021/03/line-topo.png)

Once your resources come up, verify that traffic between client and server goes via the router, and find out the properties (latency, capacity) of the individual links: the link between client and router, and the link between router and server. To estimate the latency of a link, you can use `ping` between two endpoints of a link, and to estimate the capacity of a link, you can use `iperf` or `iperf3`.

To generate traffic with exponentially distributed interarrival times and exponentially distributed packet sizes, we will use the D-ITG traffic generator. The user manual for this software is available [here](http://traffic.comics.unina.it/software/ITG/manual/).

You will need to install D-ITG on your client and server nodes. Log in to each and run

```
sudo apt update # refresh local information about software repositories
sudo apt -y install d-itg # install D-ITG software package
```


Now, the D-ITG tools should be installed on both the client and server nodes.

Also, install `tshark` on the router node:

```
sudo apt update # refresh local information about software repositories
sudo apt install tshark
```
You will also designate one node (any node) to run your simulations. On your designated `ns2` node, run

```
sudo apt update # refresh local information about software repositories
sudo apt install ns2 # install ns2 network simulator
```

The ns2 manual is available [here](http://www.isi.edu/nsnam/ns/doc/index.html). We will also refer to [these ns2 lecture notes](https://www-sop.inria.fr/members/Eitan.Altman/COURS-NS/n3.pdf).

Finally, run

```
ifconfig
```

on all three hosts. Use the IP addresses assigned to each interface to identify the names of the interfaces (e.g. "eth1", "eth2") on the router

* through which traffic from the client enters the router. This will be the interface whose IP address is in the same subnet as the client's experiment interface.
* through which traffic leaves the router towards the server. This will be the interface whose IP address is in the same subnet as the server's experiment interface.

(Make sure you are using the experiment interfaces, not the control interface through which you log in to your resources.)


#### Using D-ITG

D-ITG comes with a suite of utilities for generating traffic, receiving traffic, parsing log files, and automating experiments. We will use learn how to use three of the D-ITG utilities in: the sender (traffic generator), the receiver (traffic sink), and the log file decoder. We will run the sender on the client node, the receiver on the server node, and the log file decoder on both the client and server node.

Try it now with the following command on the receiver (server) node:

```
ITGRecv
```

Leave that running, and on the sender (client) nodes, run:

```
ITGSend -a server -l sender.log -x receiver.log -E 200 -e 512 -T UDP -t 240000
```

This will send UDP traffic to the server with a packet interarrival time that is exponentially distributed with mean 200 packets/second, packet size that is exponentially distributed with mean 512 bytes. It will run for 4 minutes (240000 ms), and will save the log files at the client node and server node with file names "sender.log" and "receiver.log" respectively.

Wait for the sender to finish and then stop the receiver with Ctrl+C. Then, open the log on the receiver side with

```
ITGDec receiver.log
```

and on the sender side with

```
ITGDec sender.log
```


Here you can see some basic aggregate information about the generated traffic flow(s). More detailed information is available from the log file using various arguments; see the documentation for details. (Note that the delay information reported is not reliable, since the clocks on the two nodes are not synchronized. You may even see a negative delay reported.)

In general, the basic usage of the sender will be as follows:

```
ITGSend -l SENDER-LOG -x RECEIVER-LOG -E MEAN-ARRIVAL-RATE -e MEAN-PACKET-SIZE  -t EXPERIMENT-DURATION -T UDP
```

where SENDER-LOG is the name to use for the log file at the sender side, RECEIVER-LOG is the name to use for the log file at the receiver side, MEAN-ARRIVAL-RATE is the mean rate at which to send traffic in packets per second (the -E indicates that the arrival rate will be exponentially distributed), MEAN-PACKET-SIZE is the mean size of the data payload of the packet in bytes (the -e indicates that the packet size will be exponentially distributed), and EXPERIMENT-DURATION is the length of the experiment in milliseconds (the default is 10 seconds). You can choose MEAN-ARRIVAL-RATE and MEAN-PACKET-SIZE to create a traffic pattern with any λ and any μ that you want. (For our experiments today, we will only vary λ and will keep the mean packet size constant.)

> **Note**: As you run your experiments, if you become disconnected from your server node and have to reconnect, you may encounter this error when you try to run the D-ITG receiver:
>
> ```
> ** ERROR_TERMINATE **
> Function main aborted caused by general parser 
> ** Cannot bind a socket on port 9000 for signaling ** 
> Finish requested caused by errors! 
> ```
>
> This indicates that a D-ITG receiver is already running in the background. To kill it (so that you can start a new one), run `sudo killall ITGRecv`.

Open another terminal and log in to the router node. On this terminal, we will use `tshark` to capture incoming traffic on the experiment interface.


```
sudo tshark -i eth1 -f "udp and port 8999" -T fields -e frame.time_delta_displayed -e frame.len -E separator=, | tee output.csv
```


Here,

* the `-i` flag specifies which network interface to listen on (in your experiment, use the experiment/data interface through which traffic enters the router from the client - either eth1 or eth2),
* we use `-f "udp and port 8999"` to filter out traffic that is not generated by D-ITG,
* we specify a list of fields to print with `-T` fields:
  * `frame.time_delta_displayed` is the time delta between displayed frames
  * `frame.len` is the size of the frame, in bytes
* and we use a comma to separate fields in the output, with `-E separator=,`

We also multiplex the output using `tee`, so it appears on the terminal but is also saved in a file called "output.csv".

Re-run the D-ITG sender and receiver on your client and server nodes. After these have finished, stop the `tshark` with Ctrl+C. Then, you can transfer the "output.csv" file to your laptop (e.g. with scp) and open it in Excel, MATLAB, or any other data analysis tool.

#### Learn basic usage of token bucket filter queue in `tc`

The traffic source and sink applications will run on the client and server nodes. On the router node, we will deploy a queue, using the Linux Traffic Control (`tc`) utility to manipulate queue settings. 

For this assignment, we will use a kind of token bucket filter queue, documented [here](https://linux.die.net/man/8/tc-htb). This queue shapes traffic using tokens, which arrive at a steady rate into a token bucket (which has a maximum capacity equal to the "burst size" of the queue). Meanwhile, traffic arrives at the token bucket from a separate queue. To leave the queue (and be transmitted), a packet must consume a number of packets roughly equal to its size in bytes. If there is not a sufficient number of tokens available when a packet arrives at the head of the queue, it must wait for more tokens to be generated. The queue, like the token bucket, has a finite length; if a packet arriving at the queue finds it full, the packet is dropped.

The model of the token bucket filter looks something like this (image via [unix.se](http://unix.stackexchange.com/a/100797)):

![](http://i.stack.imgur.com/200us.png)


Try setting the queue to serve traffic at a rate of 1Mbps by running the following command on the router (in the following commands, change `eth2` to the name of the router interface "facing" the server):

```
# Delete any existing queues
# don't worry about error message on this command
sudo tc qdisc del dev eth2 root  

# Create an htb queue
sudo tc qdisc replace dev eth2 root handle 1: htb default 3  

# Set up rate limiting
sudo tc class add dev eth2 parent 1: classid 1:3 htb rate 1mbit  

# Set FIFO queue capacity in bytes
sudo tc qdisc add dev eth2 parent 1:3 bfifo limit 500mbit  
```

Here, I have specified the interface on which to queue outgoing traffic (use the interface through which traffic leaves the router towards the server - in my case, `eth2`), the rate at which the queue should operate (1 Mbps), and the maximum size of the queue (500 megabits, i.e. "very large"). 

Use `iperf` or `iperf3` again to verify the new bottleneck link rate.

The `tc` tool includes a command to show the current state of a queue. Try running it, specifying as the last interface the name of the network interface you want to monitor (e.g. `eth1` in this example):


```
tc -p -s -d qdisc show dev eth2
```


The output of this command may look something like this:

```
qdisc htb 1: root refcnt 2 r2q 10 default 3 direct_packets_stat 8995 ver 3.17 direct_qlen 1000
 Sent 159261600 bytes 1043 pkt (dropped 0, overlimits 0 requeues 0) 
 backlog 119606b 58p requeues 0
qdisc bfifo 8002: parent 1:3 limit 64000Kb
 Sent 31322502 bytes 849 pkt (dropped 0, overlimits 0 requeues 0) 
 backlog 119606b 58p requeues 0
```

This output shows that the active queuing discipline (`qdisc`) on my interface is Hierarchy Token Bucket (`htb`). I can also see the current queue statistics. The most relevant of these (for our purposes) is the "backlog" information, which tells us the number of bytes and the number of packets currently in the queue. Also important is the "dropped" value, which tells you whether packets were dropped by the queue, and how many.

For your convenience, I have written a simple bash script that you can run on the router node to watch the queue size:

```
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

Save the contents of this script in a file on your router node, e.g. as `queuemonitor.sh`, and then make it executable (e.g. `chmod a+x queuemonitor.sh`).

To use this script, run

```
./queuemonitor.sh INTERFACE DURATION INTERVAL
```


where INTERFACE is the name of the network interface on which the queue is running, DURATION is the total time for which to monitor the queue, and INTERVAL specifies how long to sleep in between measurements.

Try running it now, on the router node, while you repeat your D-ITG experiment. You may find it useful to simultaneously watch the output on stdout and redirect it to a file at the same time with `tee`, e.g.

```
./queuemonitor.sh eth2 240 0.1 | tee router.txt
```

(make sure to specify the appropriate interface name for your experiment.) Each line of output may look something like this:

```
1616593533.619288051 qdisc htb 1: root refcnt 2 r2q 10 default 3 direct_packets_stat 8995 ver 3.17 direct_qlen 1000 Sent 195768011 bytes 3796 pkt (dropped 131, overlimits 54406 requeues 0) backlog 1651840b 633p requeues 0 qdisc bfifo 8002: parent 1:3 limit 64000Kb Sent 67828913 bytes 3602 pkt (dropped 0, overlimits 0 requeues 0) backlog 1651840b 633p requeues 0
```


After an experiment is over, you can process this file with your data analysis tool of choice. For your convenience, if you have redirected the output to `router.txt`, you can retrieve the average queue size (in packets) with:

```
cat router.txt | sed 's/\p / /g' | awk  '{ sum += $54 } END { if (NR > 0) print sum / NR }'
```


This prints the contents of the file to the terminal, then uses `sed` to replace instances of "p" with a blank space " " (since the value we are interested in is in the form "2p"). Finally, it uses `awk` to find the mean value of the 54th column.

To remove a traffic shaping queue, you can run

```
sudo tc qdisc del dev eth2 root
```

(substituting the name of the interface that you want to modify). This will remove any non-default queues you have added, and restore the default queue on that interface.


#### ns2 simulation

For a quick intro to `ns2`, you can skim Chapter 2 of [these ns2 lecture notes](https://www-sop.inria.fr/members/Eitan.Altman/COURS-NS/n3.pdf), "ns Simulator Preliminaries."

You have previously installed `ns2` on a designated node on GENI. Log into that node now. To run an ns2 simulation, we simply run

```
ns SIMULATION.tcl
```

where SIMULATION.tcl is the name of the simulation script we want to run. These are written in the tcl scripting language.

To get a feel for this, we'll start by running a simulation right away. Read Chapter 10, "Classical queuing models", in [the ns2 lecture notes](https://www-sop.inria.fr/members/Eitan.Altman/COURS-NS/n3.pdf). This chapter includes a script for a simulation of an M/M/1 queue, which we're going to use as the base for our simulation.

On your ns2 node, run

```
wget https://www-sop.inria.fr/members/Eitan.Altman/COURS-NS/TCL+PERL/mm1.tcl # download the script
ns mm1.tcl # run it
```

This script produces two output files: "out.tr" and "qm.out". The "out.tr" file is known as a "trace file," and its [format is described here](http://www.isi.edu/nsnam/ns/doc/node289.html). Chapter 3 of the [ns2 lecture notes](https://www-sop.inria.fr/members/Eitan.Altman/COURS-NS/n3.pdf) also describes how to process trace files using standard Linux tools like `awk` and `grep`.

The "qm.out" file is a trace file that specifically monitors the queue. Its columns are:

1. Time,
2. From node,
3. To node,
4. Queue size (B),
5. Queue size (packets),
6. Arrivals (packets),
7. Departures (packets),
8. Drops (packets),
9. Arrivals (bytes),
10. Departures (bytes),
11. Drops (bytes)

We can use `awk` to compute the average value of the fifth column (queue size in packets) without having to transfer the trace file to another computer to use another data analysis program. Simply run

```
cat qm.out | awk  '{ sum += $5 } END { if (NR > 0) print sum / NR }' 
```

Having run the experiment with its default paramaters, now we will modify the values in the script to match the scenario described at the top of this page:

* total experiment duration should be 240 seconds (modify the line `$ns at 1000.0 "finish"`)
* queue service rate should be 1 Mbps -
modify the link rate, by changing the line `set link [$ns simplex-link $n1 $n2 100kb 0ms DropTail]`
* average packet size should be 512 Bytes - modify two lines: `set mu 33.0` and `$pktSize set avg_ [expr 100000.0/(8*$mu)]` - the numerator in the latter should be the new link rate. Note that the value you use for mu must be a float, not an integer value - add a `.0` to explicitly make it a float if it is not.
* set the rate at which packets arrive to 200 packets per second (as in the testbed experiment you just ran) - modify the line `set lambda 30.0`. Note that this value must be a float, not an integer value - add a `.0` to explicitly make it a float if it is not.

Then, run your experiment again and compute the mean queue length.



#### Validating the experiment

In our experiment, we assume that:

* The packet sizes are exponentially distributed with mean size 512 bytes, so that the mean service rate at the router is exponentially distributed with mean 244.1 packets/second.
* The traffic is Poisson with λ = 200 packets/second (at least, for the initial experiment that we ran on each platform).
* The queue capacity is effectively infinite (i.e. no packets are dropped).


Now, we will try and validate how well these assumptions are realized in our experiments.

First, we will consider the testbed experiment.

Traffic generators may not perfectly represent the model they are supposed to - traffic generators (including D-ITG) do not always achieve exactly what they have promised. To understand whether or not this can be expected to affect our experiment, we will have to validate the performance of D-ITG in our specific experiment scenario. Specifically, we want to see how well D-ITG mimics an exponential packet interarrival time and exponential packet size distribution under the circumstances we are running it in: with data rates of approximately 200 packets/second, and an average packet size of 512 bytes.

You may have noticed from the output of D-ITG that you did not get the exact rate of packets/second that you requested. Similarly, if you divide the reported number of bytes sent by the reported number of packets sent, you may find that it isn't exactly the average packet size you asked for. In fact, the situation may be even worse than that!

After going through the steps in the previous sections, you should have four records of the mean packet size from an M/M/1 experiment: the two D-ITG log files, the tshark output, and the queue monitor output (where you can compute the difference in the number of bytes sent between the last and first lines of the experiment, compute the different in the number of packets sent, and divide to get the mean packet size). Create a table listing the four records in the first column, and the mean packet size suggested by this record in the second column. Do those four numbers agree?

You may observe that the mean packet size measured by the router is larger than expected. This is because the size that we pass as an argument to D-ITG is only for the data payload, but the total size of the frame as it transits through the router will also include:

* an 8-byte UDP header
* a 20-byte UDP header
* a 14-byte Ethernet header

Thus the mean packet size seen by the router is slightly more than 512 B, and the mean service rate will therefore be a bit lower than anticipated. To have D-ITG generate packets with mean size 512 B as seen by the router, we should actually tell it to use a mean data payload size of 470 B - another 42 B of header will be added to each packet by the OS.

That's not the only difference between our model and the realization of it, though. From the `tshark` output, find out the frequency of each packet size in the trace (e.g. number of packets of size 1, number of packets of size 2, ... up to the number of packets of size 2500). Then plot these frequencies: put "Packet size in bytes" on the x-axis and "Number of packets of this size observed" on the y-axis. Use a log10 scale for the y-axis, and a range of 0 to 2500 B on the x-axis. Then, on top of this plot, draw a line showing the expected number of packets of each size if packet sizes are exponentially distributed with mean 512 B.

If our packets were generated according to an exponential distribution with mean 512 B, we would expect to see something like this:



![](http://witestlab.poly.edu/~ffund/el7353/images/packetsize-distribution-ideal.svg)

In your output,

* Do you observe a much higher number of ~1514-byte frames than expected?
* Do you observe any frames larger than ~1514 bytes?
* Do you observe more frames in the range 62-1514 bytes than expected?
* Do you observe a much higher number of ~62-byte frames than expected?
* Do you observe any frames smaller than 62 bytes?

You are likely to see the effects of fragmentation in your plot. The network interfaces in our experiment have an MTU of 1500 B; this is the maximum size IP packet it will transmit. Any packet larger than this will be fragmented. Thus, large packets generated by D-ITG will be transmitted as a 1514 B frame (1500 B + 14 B Ethernet header) followed by one or more fragments. We therefore see many more 1514 B frames than expected, and no frames larger than 1514 B. We also see more frames than expected between 62-1514 bytes, because the fragments will be counted in this range.

We also note that the minimum size of an Ethernet frame is 64 bytes. (In our experiment, we see frames as small as 62 bytes because 2 bytes of padding will be added to the Ethernet header by the router when it is transmitted.) Smaller packets will be padded to 62 bytes. We therefore see many more 62 B frames than expected, and no frames that are smaller.

Because of these effects - header overhead, MTU, and minimum frame size - the actual mean packet size in the testbed experiment will not be 512 B. We can give D-ITG a mean payload size of 470 B, but even then we won't get a mean frame size of 512 B. This is partly because small frames will be padded to 62 B and also because, if a packet is fragmented, then a 42 B header is appended to each of its fragments. (And of course, we also won't get a perfect exponential distribution of packet sizes.)

Next, we will consider the packet interarrival time for our testbed experiment. We expect a mean interarrival time of 0.005 second (1/200 packets/second). Compute the observed mean interarrival time using the CSV file produced by tshark - how close is it to 0.005 seconds? Also create a plot showing the expected and observed distribution. From the tshark output, find out the frequency of each interarrival time in the trace in 0.001 second bins. Then plot these frequencies: put "Interarrival time in seconds" on the x-axis and "Number of packets of this size observed" on the y-axis. Use a log10 scale for the y-axis, and a range of 0 to 0.1 seconds on the x-axis. Finally, on top of this plot, draw a line showing the expected number of packets of each size if interarrival times are exponentially distributed with mean 0.005 seconds. Comment on your observations.

Finally, validate the assumption that the queue size is effectively infinite - make sure no packets are dropped. Look at the D-ITG receiver and sender logs, and make sure the same number of packets are received as are sent. Also look at the queue monitor output - make sure that the number of packets dropped is zero, or at least that it is the same at the beginning and end of the queue monitor output file (indicating that if any packets were dropped, they were dropped before the experiment began.


Before we turn to look at the ns-2 experiment more closely, there is one more thing we need to verify: that, if we run the experiment repeatedly, we will have a valid stochastic experiment. Run the D-ITG experiment again with exactly the same arguments as before. Then, compare the sequence of packet sizes and interarrival times observed by tshark - are they the same from one experiment to the next, or are they different? Refer to the D-ITG documentation. What kind of PRNG is used to generate random numbers in D-ITG? How is it seeded?



Let us now consider the ns2 experiment. From the trace file, extract the timestamp and size of each received packet:

```
cat out.tr | grep "r" | cut -f2,6 -d' ' > out.dat
```

Transfer this "out.dat" to your laptop (e.g. with `scp`.) Then, create a plot showing the distribution of packets sizes compared to the expected distribution, like you did for the testbed experiment.

In this experiment, too, you will notice that the distribution is truncated at the top end. The [ns2 lecture notes](https://www-sop.inria.fr/members/Eitan.Altman/COURS-NS/n3.pdf) explain the reason, and how to fix it:

> The simulated packets turn out to be truncated at the value of 1kbyte, which is the default size of a UDP packet. Thus transmission times are a little shorter than we intend them to be. To correct that, one should change the default maximum packet size, for example to 100000. This is done by adding the line
> 
> `$src set packetSize_ 100000`
>
> after the command `set src [new Agent/UDP]`.


Make the suggested change, run the simulation again, and check the distribution of packet sizes and arrival times again. Has the issue been resolved? What is the mean observed packet size?


In the testbed experiment, you observed that the distribution was truncated at the low end, too. Do you see a similar effect in the simulation experiment?

Next, we will consider the packet interarrival time for our simulation experiment. We expect a mean interarrival time of 0.005 second (1/200 packets/second). Compute the observed mean interarrival time using the timestamps of received packets in the output file - how close is it to 0.005 seconds? Also create a plot showing the expected and observed distribution, as in the testbed experiment. Find out the frequency of each interarrival time in the trace in 0.001 second bins. Then plot these frequencies: put "Interarrival time in seconds" on the x-axis and "Number of packets of this size observed" on the y-axis. Use a log10 scale for the y-axis, and a range of 0 to 0.1 seconds on the x-axis. Finally, on top of this plot, draw a line showing the expected number of packets of each size if interarrival times are exponentially distributed with mean 0.005 seconds. Comment on your observations.

Third, validate the assumption that the queue size is effectively infinite - at least, for this experiment. In the "out.tr" file, look for lines beginning with a "d" - indicating dropped packets. Also check the last line of the queue monitor output file, and see if any packets are reported as being dropped (8th column in the "qm.out" file.)

Finally, we want to make sure that we are running a valid stochastic experiment. Run the script a few times and look at the trace files. Are these separate runs independent trials, or are the interarrival times and packet sizes identical on each run?

To get statistically independent trials, we will need to seed the random number generator with a different value each time we run the experiment. Also, we will want to set up the two distributions in the experiment (packet interarrival time and packet size) to be statistically independent, using different substreams of the random number generator.

To do this, add a few lines immediately after `set ns [new Simulator]`:

```
set rep [lindex $argv 0]

set rng1 [new RNG]
set rng2 [new RNG]

for {set i 1} {$i<$rep} {incr i} {
        $rng1 next-substream;
        $rng2 next-substream
}
```

The first line says to take one argument from the command line and store it in the variable "rep". We then create two random number generator objects. Then, we explicitly set them to use different random substreams of the ns2 random number generator, using `next-substream`. We use the "rep" value passed from the command line to determine how many times to call `next-substream`, so that simulations run with different values of "rep" will have different random numbers.


We still need to assign the random number generator objects to the distributions that they will feed. Underneath the line

```
set InterArrivalTime [new RandomVariable/Exponential]
```

add

```
$InterArrivalTime use-rng $rng1
```

and underneath

```
set pktSize [new RandomVariable/Exponential]
```

add

```
$pktSize use-rng $rng2
```

Now, you will run your simulation with an argument, e.g.

```
ns mm1.tcl 1
```


where independent results will be generated when you supply different arguments. Run your script a few times with a few different values of "rep", and verify that different values give different results.

ns2 uses a multiple recursive generator called MRG32k3a, which contains streams from which numbers picked sequentially seem to be random (uniformly distributed.) These in turn are transformed to create variates of other desired distributions. More information about the current random number generator in ns2 is available [here](http://www.isi.edu/nsnam/ns/doc/node267.html).


#### Putting it together

Putting together everything we have learned, we're going to experimentally measure the queue length of an M/M/1 queue for a range of ρ values. For convenience, we will keep everything constant and only vary λ from one experiment to the next.

Record your experiment results in a spreadsheet with the following columns:


* Experiment type (simulation, testbed)	
* Requested λ	
* Requested μ	
* Requested ρ	
* Realized mean packet size	
* Realized mean interarrival time	
* Realized λ	
* Realized μ	
* Realized ρ	
* Mean queue length measured in experiment	
* Expected mean queue length based on requested λ and μ	
* Expected mean queue length based on realized λ and μ


Start with the testbed experiment. You will run `ITGSend` with a mean packet size of 512 B (but use 470 B as the argument to D-ITG, to account for packet header overhead), and varying packet rate. Run experiments for the following values of λ: 225, 200, 175, 150, 125. For each value of λ you should run 5 experiments.

Given that you have already set up the queue as described above, to run a single experiment, you will:

* Record the requested λ and μ in your experiment log.
* Start `tshark` to record traffic patterns on the router node, on the interface through which traffic from the client enters the router.
* Start `ITGRecv` on the server node.
* Start `ITGSend` with the appropriate arguments and with a long experiment duration (e.g. 240 seconds) on the client node.
* Start the queue monitor script on the router node, and let it run until just before the experiment is over. (Don't let it keep running after the experiment is over.)
* Find the mean queue size and mean interarrival time from the `tshark` output, and record these in your experiment log. Compute the "realized" λ and μ based on these values, and then find the "realized" ρ.
* In your experiment log, record three values for the mean queue size: the actual mean queue size from your experimet, the expected mean queue size based on the "requested" λ and μ values, and the expected mean queue size based on the "realized" λ and μ values.

Your experiment log should have 25 lines total (5 different values of λ and 5 experiment runs of each) after finishing the testbed experiments.
Then we'll run the simulation version. Assuming you have modified the "mm1.tcl" script appropriately as discussed above,

* Run experiments for the following values of λ: 225, 200, 175, 150, 125.
* For each value of λ you should run 5 experiments (with five different seeds.)
* For each experiment, fill in all of the fields in the experiment log, as discussed above.


## Notes

### Exercise

In your report, you should:

* Compare the analytical, simulation, and testbed measurement results for the M/M/1 queue length, (create a figure like the one in the "Background" section, using your own experiment results.)
* Discuss the results of the "Validating the experiment" section, (make sure to address the questions posed in that section). Include the figures you were asked to create in that section, showing how well D-ITG and ns2 approximate exponential distributions of packet size and interarrival time (both before and after any changes you might have made to the experiment design).
* In your experiment log, you recorded three values for mean queue length for each experiment. The "expected mean queue length based on realized ρ" represents the value we would expect to see if the realized traffic patterns were the only difference between the M/M/1 queue model and the experiment version. Based on your measurements, do you think that the difference between the requested and realized traffic patterns completely explain any differences between the M/M/1 queue model and the measured mean queue length? Explain (with reference to specific, numeric data from your experiments).
* Finally, discuss what the analytical, simulation, and testbed experiments represent - which experiment most closely represents the "real" scenario of a router queue with M/M/1 traffic, and which most closely represents the mathematical model of it?

Do not include raw data - only figures and discussion. Also, do not list the commands you ran, or any similar details.

However, you should include as appendices:

* A copy of your final `mm1.tcl` script.
* Your collected data ("experiment log") from the experiments (it should have 50 lines of data).