This experiment emulates a multipath wireless scenario by "playing back" wireless network link traces that were collected in the same time and place.

It should take about 60-120 minutes to run this experiment.

You can run this experiment on CloudLab or on FABRIC! Refer to the testbed-specific prerequisites listed below.


<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">
<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Prerequisites</h4>

To reproduce this experiment on Cloudlab, you will need an account on <a href="https://cloudlab.us/">Cloudlab</a>, you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._join-project%29">joined a project</a>, and you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._ssh-access%29">set up SSH access</a>.

</div>
<br>

<div style="border-color:#47aae1; border-style:solid; padding: 15px;">
<h4 style="color:#47aae1;">FABRIC-specific instructions: Prerequisites</h4>

To run this experiment on <a href="https://fabric-testbed.net/">FABRIC</a>, you should have a FABRIC account and be part of a FABRIC project. You will need to have set up SSH keys and understand how to use the Jupyter interface in FABRIC.

</div>
<br>

## Background
As mobile devices with dual WiFi and cellular interfaces become widespread, network protocols have been developed that utilize the availability of multiple paths. 

To evaluate the performance of these protocols on a testbed, researchers often use link emulation with either synthetic traces or real traces from live WiFi and cellular networks. However, in a multipath scenario, the two links are not necessarily independent. The available bandwidth on the WiFi and cellular links may be negatively correlated (e.g. when moving from indoors to outdoors or vice versa) or positively correlated (e.g. when the local density of other users increases, so that both networks become more congested at the same time), and the result of an experimental evaluation of a multipath protocol will depend heavily on this relationship. Link traces of WiFi and cellular links measured at the same time, in the same place are necessary for the reproducible evaluation of multipath protocols in a testbed environment. 

The contributions of this work, therefore are:
 
* first ever data set of wireless link traces measured simultaneously on WiFi and cellular interfaces of a mobile device, 
* and reference experiments for replaying these traces on emulated links on the CloudLab and FABRIC testbeds.

## Run my experiment 

In this section, we show how to "play back" the wireless link traces on emulated links, using either the CloudLab or FABRIC testbed.

The links traces and all of the experiment artifacts are available in the following Github repository: https://github.com/aydini/Multipath-Wireless-Link-Traces


### CloudLab experiment

In this section, we will describe how to:

* reserve and configure resources on CloudLab
* validate the multipath configuration
* "play back" link traces on the multipath configuration

####  Reserve and configure resources 

To run this experiment on CloudLab, we will instantiate the profile named `mptcp-demo`. You can open this profile directly from the following link: https://www.cloudlab.us/p/17ab2ba76d30848f3667ca2e8652035b1db9c148

You can instantiate this topology on your preferred CloudLab cluster.

When it first boots, each node will retrieve the [Github repository](https://github.com/aydini/Multipath-Wireless-Link-Traces) to the directory `/local/repository`, and then run its corresponding setup script - you can see these scripts in the [CloudLab directory](https://github.com/aydini/Multipath-Wireless-Link-Traces/tree/main/CloudLab) in the Github  repository.

In the CloudLab "Topology View", wait until each node turns green (indicating that it is ready to log in) and has a small check mark in the top right corner (indicating that it has completed the startup script), as illustrated below. The server and client nodes may take longer to come up, because they use a custom disk image with the MPTCP kernel installed. 

![](/blog/content/images/2023/01/cloudlab-done.svg)

Then, you can SSH into each of the hosts, either using your local terminal or the built-in terminal in the CloudLab web interface.


#### Validate the multipath configuration

In the multipath topology, both the client and server nodes have two network interfaces, with two independent paths connecting them. Along each path there is a node named "emulator", which will be used to set the base delay on each path, and a node named "router", where the link capacity will be set (or played back from network traces). The client and server nodes have multipath TCP (MPTCP) [1] installed, and have routing rules configured to enable MPTCP. 

To validate the multipath configuration, we will initiate a TCP flow between client and server using `iperf3`, and verify that it uses both paths. The initial setup scripts configure the base delay on both paths to 30 ms, and the link capacity on both paths to 100 Mbps, so the multipath flow should have a throughput of approximately 200 Mbps.

On the "server" node, run

```
iperf3 -1 -s
```

and leave this running. On the "client" node, initiate the MPTCP flow (using the "balia" MPTCP congestion control algorithm) with

```
iperf3 -c 192.168.3.1 -C "balia" -i 10
```

A sample `iperf3` session output at the client is given below.  You should see that the bandwidth at the client is close to the total capacity of both paths (200 Mbps).

```
aydini1@client:~$ iperf3 -c 192.168.3.1 -C "balia" -i 10
Connecting to host 192.168.3.1, port 5201
[  4] local 192.168.10.2 port 50660 connected to 192.168.3.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-10.00  sec   224 MBytes   188 Mbits/sec    0   14.1 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec   224 MBytes   188 Mbits/sec    0             sender
[  4]   0.00-10.00  sec   221 MBytes   185 Mbits/sec                  receiver

iperf Done.
   
```

#### "Play back" link traces

Finally, we will show how to "play back" wireless link traces - collected at the same time and place - on the multipath topology. The dynamic network capacity in the link traces will be emulated at "router1" (emulating the WiFi link) and "router2" (emulating the cellular link).

When you play back a link trace pair, you will specify the "path" and the "trial" that you want to emulate. In the following experiment, we will emulate map path 8, trial 1. However, you can change the numbers in the trace file names to emulate a different trace pair.

At "router1", to replay the WiFi trace data, you will run:

```
cd /local/repository
bash CloudLab/tputVary.sh  Traces/8_1_wifi.csv
```

At "router2", to replay the cellular trace data, you will run:

```
cd /local/repository
bash CloudLab/tputVary.sh  Traces/8_1_cellular.csv
```

To make sure the traces are approximately synchronized in the emulated network as they were in the live network, you should try to run these commands in quick sequence, so that the traces start together. The script will print the emulated data rate (in kbps) to the terminal every second, so that you can monitor the experiment.

 
Then, at the server run: 
```
iperf3 -1 -s
```

Finally, at the client run: 
```
iperf3 -c 192.168.3.1 -C "balia" -f m -i 1 -t 10 
```
(to run a 10-second experiment).

A sample experiment output at router1, router2, client and server nodes is shown below. 
```
aydini1@router1:/local/repository$ bash CloudLab/tputVary.sh  Traces /8_1_wifi.csv
51598.40
50887.68
51634.80
57467.76
51780.80
50819.04
54844.40
54432.24
52342.40
55505.60
54684.40
56941.44
61584.88
55238.24
50190.16
^C
aydini1@router1:/local/repository$

```

```
aydini1@router2:/local/repository$ bash CloudLab/tputVary.sh  Tr
aces/8_1_cellular.csv                                           
8511.20                                                         
15116.80                                                        
22383.20                                                        
46808.40                                                        
50745.20                                                        
35686.32                                                        
42424.56                                                        
38703.76                                                        
30426.08                                                        
33841.04                                                        
34224.64                                                        
39270.00                                                        
38047.04                                                        
36917.84                                                        
38133.04                                                        
37095.04                                                        
32118.32                                                        
^C                                                              
aydini1@router2:/local/repository$                              
```
```
aydini1@server:~$ iperf3 -1 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 192.168.10.2, port 43652
[  5] local 192.168.3.1 port 5201 connected to 192.168.10.2 port 43654
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  7.94 MBytes  66.6 Mbits/sec

[  5]   1.00-2.00   sec  11.7 MBytes  98.0 Mbits/sec

[  5]   2.00-3.00   sec  11.0 MBytes  92.2 Mbits/sec

[  5]   3.00-4.00   sec  10.1 MBytes  85.0 Mbits/sec

[  5]   4.00-5.00   sec  10.6 MBytes  88.5 Mbits/sec

[  5]   5.00-6.00   sec  10.0 MBytes  84.0 Mbits/sec

[  5]   6.00-7.00   sec  9.63 MBytes  80.8 Mbits/sec

[  5]   7.00-8.00   sec  9.91 MBytes  83.1 Mbits/sec

[  5]   8.00-9.00   sec  10.3 MBytes  86.5 Mbits/sec

[  5]   9.00-10.00  sec  10.6 MBytes  89.2 Mbits/sec

[  5]  10.00-10.11  sec  1.15 MBytes  89.3 Mbits/sec

- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-10.11  sec  0.00 Bytes  0.00 bits/sec
   sender
[  5]   0.00-10.11  sec   103 MBytes  85.4 Mbits/sec
     receiver
```
```
aydini1@client:~$ iperf3 -c 192.168.3.1 -C "balia" -f m -i 1 -t 10
Connecting to host 192.168.3.1, port 5201
[  4] local 192.168.10.2 port 43654 connected to 192.168.3.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  11.6 MBytes  97.6 Mbits/sec    0   14.1 KBytes
[  4]   1.00-2.00   sec  11.7 MBytes  98.2 Mbits/sec    0   14.1 KBytes
[  4]   2.00-3.00   sec  10.8 MBytes  91.0 Mbits/sec    0   14.1 KBytes
[  4]   3.00-4.00   sec  10.2 MBytes  85.3 Mbits/sec    0   14.1 KBytes
[  4]   4.00-5.00   sec  10.5 MBytes  88.4 Mbits/sec    0   14.1 KBytes
[  4]   5.00-6.00   sec  9.93 MBytes  83.3 Mbits/sec    0   14.1 KBytes
[  4]   6.00-7.00   sec  9.68 MBytes  81.2 Mbits/sec    0   14.1 KBytes
[  4]   7.00-8.00   sec  9.87 MBytes  82.8 Mbits/sec    0   14.1 KBytes
[  4]   8.00-9.00   sec  10.4 MBytes  86.9 Mbits/sec    0   14.1 KBytes
[  4]   9.00-10.00  sec  10.7 MBytes  89.4 Mbits/sec    0   14.1 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec   105 MBytes  88.4 Mbits/sec    0
 sender
[  4]   0.00-10.00  sec   103 MBytes  86.4 Mbits/sec
 receiver

iperf Done.
aydini1@client:~$
```

When you are finished, click "Terminate" in the CloudLab web interface to free your resources and make them available for other experimenters.

### FABRIC experiment


First login to your FABRIC testbed account, then go to your Jupyter hub. 
![](/blog/content/images/2023/01/fabric1.PNG)

Then from the Launcher, open a terminal window.
![](/blog/content/images/2023/01/fabric2.PNG)

Then clone the github repo to your FABRIC environment by typing the following in the terminal window.
```
git clone https://github.com/aydini/Multipath-Wireless-Link-Traces.git
```
Finally, open the notebook called `fabric-demo-mptcp-trace.ipynb` from your local repo that has all the instructions to 

* reserve and configure resources on FABRIC
* validate the multipath configuration
* "play back" link traces on the multipath configuration

![](/blog/content/images/2023/01/1.PNG)
![](/blog/content/images/2023/01/2.PNG)
![](/blog/content/images/2023/01/3.PNG)






## References

[1] Raiciu, Costin, Christoph Paasch, Sebastien Barre, Alan Ford, Michio Honda, Fabien Duchene, Olivier Bonaventure, and Mark Handley. *"How hard can it be? designing and implementing a deployable multipath TCP."* In 9th USENIX symposium on networked systems design and implementation (NSDI 12), pp. 399-412. 2012.

