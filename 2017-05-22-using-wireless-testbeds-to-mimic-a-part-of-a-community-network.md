In this experiment, we will set up a mesh network similar to a part of a live community network, and observe how packets are routed across multiple hops in this network.

It should take about 120 minutes to run this experiment, but you will need to have reserved that time in advance. This experiment uses wireless resources, and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have reserved time on the sb4 testbed at [ORBIT](http://geni.orbit-lab.org), and you must run this experiment during your reserved time.  

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

Community networks are wireless mesh networks that are built from the ground up; as each member joins the network, it becomes a node in the network and may carry other members' traffic. Some members also have conventional Internet connectivity through an ISP, and share their service with the community network. 

The [Representing community network topologies on GENI](https://witestlab.poly.edu/blog/representing-community-network-topologies-on-geni/) experiment describes how to set up a topology on GENI that has the same structure as a part of the [Funkfeuer Graz](https://www.ffgraz.net/) community network. However, that experiment runs on wired GENI resources. For some applications, it may be desirable to set up a topology that mimics a part of a community network on a testbed with real wireless links.

In this experiment, we will show how to do this on the sb4 testbed on ORBIT. This testbed has a [programmable attenuation matrix](http://www.orbit-lab.org/wiki/Hardware/bDomains/cSandboxes/dSB4), which we can use to set up any arbitrary wireless topology. This allows us to run experiments involving a small subset (9 nodes or fewer) of a community network.

Using a similar procedure as in [Representing community network topologies on GENI](https://witestlab.poly.edu/blog/representing-community-network-topologies-on-geni/), we split off two "subgraphs" in the FF Graz community network that have nine nodes or fewer. Here, we see these smaller groups colored green (network 1) or blue (network 2) within the larger community network:


![](/blog/content/images/2017/05/community-within-network.svg)

And here we see each small group on its own, together with its realization on the sb4 testbed (each node in each image is marked with the number of the sb4 testbed node on which it will be realized, and its MAC address):

<table>
<tr>
<td><b>Network 1</b><br> <a href=/blog/content/images/2017/05/subgraph-1-label.svg><img style="width:100%" src=/blog/content/images/2017/05/subgraph-1-label.svg></a> </td><td> <b>Network 2</b><br><a href=/blog/content/images/2017/05/subgraph-2-label.svg><img style="width:100%" 
 src=/blog/content/images/2017/05/subgraph-2-label.svg></a>
 </td></tr>
</table>

In this experiment, we will set up one of these topologies with real wireless links, using the programmable attenuation matrix on sb4. Then, to validate our setup, we'll observe the behavior of a mesh routing protocol in the network.

## Results

We will use the output of a mesh routing protocol to validate that our network topology on sb4 mimics the community network subgraph that we have selected. 

For example, we created the following table to show how we validate our setup for network #2. In the following table, each row represents a different node. On the left, we show the node (marked in green in the figure) and its neighbors (in pink). On the right, we show the neighbor table created by the B.A.T.M.A.N. mesh routing protocol. We can see that the network topology is correctly represented on the testbed:

<table>
<tr><td><a href=/blog/content/images/2017/05/subgraph-2-node1-1.svg><img style="max-width: 100%" src=/blog/content/images/2017/05/subgraph-2-node1-1.svg></a></td><td><br>
<pre>
node1-1: 00:60:b3:25:c0:14

IF        Neighbor             last-seen
wlan0	  00:15:6d:84:92:cb    0.452s
wlan0	  00:15:6d:84:92:cd    0.260s
wlan0	  00:60:b3:b0:c6:6c    0.264s
</pre></tr>

<tr><td><a href=/blog/content/images/2017/05/subgraph-2-node2.svg><img style="max-width: 100%" src=/blog/content/images/2017/05/subgraph-2-node2.svg></a></td><td><br>
<pre>
node1-2: 00:60:b3:b0:c6:6c

IF        Neighbor             last-seen
wlan0	  00:60:b3:25:c0:14    0.892s
wlan0	  00:15:6d:84:92:cd    0.480s
</pre></tr>
<tr><td><a href=/blog/content/images/2017/05/subgraph-2-node3.svg><img style="max-width: 100%" src=/blog/content/images/2017/05/subgraph-2-node3.svg></a></td><td><br>
<pre>
node1-3: 00:15:6d:85:e0:c8

IF        Neighbor             last-seen
wlan0	  00:15:6d:85:e0:c6    0.624s
</pre></tr>
<tr><td><a href=/blog/content/images/2017/05/subgraph-2-node4.svg><img style="max-width: 100%" src=/blog/content/images/2017/05/subgraph-2-node4.svg></a></td><td><br>
<pre>
node1-4: 00:15:6d:84:92:cb

IF        Neighbor             last-seen
wlan0	  00:60:b3:25:c0:14    0.500s
wlan0	  00:60:b3:25:c0:37    0.376s
wlan0	  00:15:6d:85:e0:c6    0.008s
</pre></tr>

<tr><td><a href=/blog/content/images/2017/05/subgraph-2-node5.svg><img style="max-width: 100%" src=/blog/content/images/2017/05/subgraph-2-node5.svg></a></td><td><br>
<pre>
node1-5: 00:15:6d:84:92:cd

IF        Neighbor             last-seen
wlan0	  00:60:b3:25:c0:14    0.908s
wlan0	  00:60:b3:b0:c6:6c    0.436s
</pre></tr>

<tr><td><a href=/blog/content/images/2017/05/subgraph-2-node6.svg><img style="max-width: 100%" src=/blog/content/images/2017/05/subgraph-2-node6.svg></a></td><td><br>
<pre>
node1-6: 00:15:6d:85:e0:c6

IF        Neighbor             last-seen
wlan0	  00:60:b3:25:c0:37    0.840s
wlan0	  00:15:6d:84:92:cb    0.688s
wlan0	  00:15:6d:85:e0:c8    0.416s
</pre></tr>

<tr><td><a href=/blog/content/images/2017/05/subgraph-2-node7.svg><img style="max-width: 100%" src=/blog/content/images/2017/05/subgraph-2-node7.svg></a></td><td><br>
<pre>
node1-7: 00:60:b3:25:c0:37

IF        Neighbor             last-seen
wlan0	  00:15:6d:85:e0:c6    0.964s
wlan0	  00:15:6d:84:92:cb    0.160s
</pre></tr>
</table>



We can also trace the path a packet takes through the network. For example, we can trace the path from node1-3 to node1-5:

```
root@node1-3:~# batctl traceroute 192.168.1.5
traceroute to 192.168.1.5 (00:15:6d:84:92:cd), 50 hops max, 20 byte packets
 1: 00:15:6d:85:e0:c6  4.172 ms  1.341 ms  1.578 ms
 2: 00:15:6d:84:92:cb  3.476 ms  2.865 ms  3.370 ms
 3: 00:60:b3:25:c0:14  4.147 ms  4.211 ms  4.088 ms
 4: 00:15:6d:84:92:cd  5.418 ms  5.926 ms  5.196 ms
```

and then verify from the graph that this is the expected path (through nodes marked in pink):

![](/blog/content/images/2017/05/subgraph-2-path.svg)


## Run my experiment

At your reserved time, SSH into 

```
sb4.orbit-lab.org
```

with your GENI wireless username and associated keys.

When you first log in to the "sb4" console, you should [reset sb4's programmable attenuation matrix](http://www.orbit-lab.org/wiki/Hardware/bDomains/cSandboxes/dSB4) to zero attenuation between all pairs of nodes. From the "sb4" console, run

```
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/setAll?att=0"


wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=1&port=1"
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=2&port=1"
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=3&port=1"
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=4&port=1"
```


### Set up the testbed nodes

Then, load the "mesh-protocols" disk image onto the testbed nodes:

```
omf load -i mesh-protocols.ndz -t system:topo:all
```

Wait for this process to finish. If some nodes are not imaged (e.g. they do not "check in" to the experiment, or they "time out"), repeat the imaging process on the individual nodes that failed. For example, if node1-2 and node1-9 did not load the disk image successfully, then run 

```
omf load -i mesh-protocols.ndz -t node1-2.sb4.orbit-lab.org,node1-9.sb4.orbit-lab.org
```

It may take a few tries to get the image onto all of the nodes. 

After the disk image has been loaded onto all 9 nodes in sb4, turn them on with

```
omf tell -a on -t system:topo:all
```

### Set up the community network topology

Next, we will use the [programmable attenuation matrix](http://www.orbit-lab.org/wiki/Hardware/bDomains/cSandboxes/dSB4) on sb4 to set up one of the community network topologies. We will set 10 dB of attenuation between all "connected" nodes, and 63 dB of attenuation (the maximum) between nodes that are not connected.

To set up network #1, run [this gist](https://gist.github.com/ffund/c741b74ab1d246055f25d0a9ebd2e8dc):

```
wget https://git.io/vHk6K | bash
```

on the sb4 console. Or, to set up network #2, run [this gist](https://gist.github.com/ffund/bfd8c4eb5299be63d41cecd5b686dd37):

```
wget -qO- https://git.io/vHk6r | bash
```

on the sb4 console. 

You can see a visual representation of the current sb4 topology at [http://www.orbit-lab.org/status/sb4/network](http://www.orbit-lab.org/status/sb4/network).

### Set up a mesh network with B.A.T.M.A.N.

Open up eight terminal sessions (if using network #1) or seven terminal sessions (if using network #2). In each, SSH into the sb4 console, and from there, SSH into each of the nodes as the root user. For example, to log in to node1-1 from the sb4 console run

```
ssh root@node1-1
```
For network #1, log into nodes 1-1 through 1-8, and for network #2 log into nodes 1-1 through 1-7.

Then, on each node, run


```
modprobe ath9k
modprobe ath5k
sleep 1

ifconfig wlan0 up
ifconfig wlan0 0.0.0.0 down
ifconfig wlan0 mtu 1532
iwconfig wlan0 mode ad-hoc essid community-mesh ap 02:B8:C0:08:78:80 channel 11
```

to set up a mesh network (at layer 2). You can run 

```
iwconfig wlan0
```

to verify layer 2 connectivity. If connected, the output should look like this:

```
wlan0     IEEE 802.11abg  ESSID:"community-mesh"  
          Mode:Ad-Hoc  Frequency:2.462 GHz  Cell: 02:B8:C0:08:78:80   
          Tx-Power=27 dBm   
          Retry  long limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:off
```

Now, we set up the [B.A.T.M.A.N. routing protocol](https://en.wikipedia.org/wiki/B.A.T.M.A.N.) on this mesh. On each node, run:

```
modprobe batman-adv
batctl if add wlan0

sysctl -w net.ipv4.ip_forward=1

# Set up IP address
y=$(hostname -s | cut -f2 -d'-')
ifconfig wlan0 up
ifconfig bat0 192.168.1.$y
```

Wait a few minutes for the B.A.T.M.A.N. messages to propagate through the network. Then, you can find out what neighbors each node can "see" by running

```
batctl n
```

on each node. You should be able to verify that each node can "see" exactly the neighbors that it is supposed to, depending on which network we chose to mimic.

You can also trace the route a packet takes through the network by running

<pre>
batctl traceroute <b>DESTINATION</b>
</pre>

on the source node, where in place of "DESTINATION" you give the IP address of the destination node. (The destination IP address will be 192.168.1.X, where X is that node number.) It will show the MAC address of each intermediate hop along the path to the destination.

### Building on this experiment: varying link attenuation

With the programmable attenuation matrix, you can set any attenuation you want - so you can degrade some links and see what would happen to the performance of the community network.

To change the attenuation between a pair of nodes, run

<pre>
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/set?portA=<b>1</b>&portB=<b>2</b>&att=<b>20</b>"
</pre>

where in place of the bolded values, you specify the node numbers of the two nodes, and the attenuation (in dB, any integer value from 0 to 63) that you want between them.