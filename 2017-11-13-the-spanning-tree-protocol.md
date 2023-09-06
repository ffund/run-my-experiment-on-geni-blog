In this experiment, we will see how broadcast storms can occur in a network with bridge loops (multiple Layer 2 paths between endpoints). Then, we will see how the spanning tree protocol creates a loop-free logical topology in a network with physical loops, so that a broadcast storm cannot occur. We will also see how the spanning tree protocol reacts when the topology changes.

It should take about 1 hour to run this experiment. (This does not include the time to answer the questions in the [Exercise](#exercise) section, which will take some additional time.)

<div style="border-color:#FB8C00; border-style:solid; padding: 15px;">
<h4 style="color:#FB8C00;"> GENI-specific instructions: Prerequisites</h4>

To reproduce this experiment on GENI, you will need an account on the <a href="http://groups.geni.net/geni/wiki/SignMeUp">GENI Portal</a>, and you will need to have <a href="http://groups.geni.net/geni/wiki/JoinAProject">joined a project</a>. You should have already <a href="http://groups.geni.net/geni/wiki/HowTo/LoginToNodes">uploaded your SSH keys to the portal and know how to log in to a node with those keys</a>.

</div>
<br>

<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">
<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Prerequisites</h4>

To reproduce this experiment on Cloudlab, you will need an account on <a href="https://cloudlab.us/">Cloudlab</a>, you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._join-project%29">joined a project</a>, and you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._ssh-access%29">set up SSH access</a>.

</div>
<br>



* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

An Ethernet switch or bridge relays Ethernet frames between devices connected to different ports. It thereby links multiple hosts (or smaller network segments each with multiple hosts) into a single connected network.

When a frame arrives at a switch or a bridge, the forwarding table is used to determine whether to forward, filter, or flood the frame, depending on its destination address:

* When the switch receives a frame that is destined for an address that is not in the table, the switch will flood the frame out of all ports other than the port on which it arrived.
* If the destination address is in the table, and is known to be reachable on the same port that the frame was received on, the switch will filter (drop) the frame.
* Otherwise, the switch will forward the frame out of the port corresponding to its destination address in the forwarding table.

While this usually works well, problems arise in the event of a bridging loop. For various reasons (e.g. redundancy in case of a link failure), a network may have multiple Layer 2 paths between two endpoints - a bridging loop. This can lead to a _broadcast storm_:

* When a broadcast packet arrives at a switch, copies will be flooded out all ports other than the one it arrives on.
* Other switches in the network will also flood copies of the packet out all ports other than the one it arrives on. 
* Eventually, it will re-appear at the switch that first flooded the packet - but on a different port than the one it originally arrived on. Copies will be flooded out all other ports, including the port that the packet was first seen on.
* As more and more copies are created, the large volume of copies can saturate the network, preventing other traffic from getting through.
* Because Ethernet frames do not have a time-to-live (TTL) field, like IP packets, they will not be discarded - copies of the frame can keep circulating in the network forever.

The spanning tree protocol addresses this issue in a network with physical loops by setting some redundant paths to a blocked state and leaving others in a forwarding state. This creates a loop-free logical topology, so that a broadcast storm cannot occur. However, the network still benefits from the added reliability of redundant paths: if a link that is in a forwarding state becomes unavailable, then the protocol will reconfigure the tree and re-active blocked paths as necessary, to restore connectivity.

![](/blog/content/images/2017/11/spanning-tree-example.svg)
_<small>Example of a bridged network with a loop, and the minimum spanning tree with the loop removed. (From "TCP/IP Essentials: A Lab-Based Approach", Figure 3.3.)</small>_

To create a loop-free tree, bridges in the network  exchange BPDUs, and execute the spanning tree protocol as follows:

1. **Elect a root switch or bridge**. Each bridge is assigned a unique bridge ID, usually formed from a priority concatenated with the MAC address of one of the bridge ports. The bridge or switch with the lowest bridge ID is elected as the root bridge.
2. **Elect a root port on each non-root bridge**.  Each bridge (except the root bridge) computes the _root path cost_, i.e. cost of the path to the root bridge, through each port. Then, the _root port_ is elected - the one with the lowest root path cost.
3. **Select a designated bridge and port on each network segment**.  The bridge on each network segment with the lowest root path cost will be selected as the designated bridge, and the port that connects that bridge to the network segment is the designated port. (The sender's bridge ID, then bridge port, is used as the tie-breaker in case there are multiple bridges on a network segment with the same root path cost. The lowest bridge ID or bridge port "wins".)
4. **Set bridge ports' states**. On a bridge that is not the root, only root ports or designated ports can forward frames on a network segment. Other bridge ports are set to the "blocked" state.

In this experiment, we will create a topology with a loop, then watch as the spanning tree algorithm creates a logical loop-free topology.

## Results

When the spanning tree protocol is not enabled, we observe that sending a broadcast frame on the network leads to a broadcast storm: 

<iframe width="560" height="315" src="https://www.youtube.com/embed/TsOf-Z9UNJc" frameborder="0" allowfullscreen></iframe>


Once we start the spanning tree protocol, we will see how a logical loop-free topology is created in the network, with at least one bridge port ending up in the "blocked" state. We can also identify the root bridge (with a path cost of 0), and the designated port for each network segment:

<iframe width="560" height="315" src="https://www.youtube.com/embed/TX5PUxCVAHM" frameborder="0" allowfullscreen></iframe>

We will see that a broadcast storm is _not_ triggered in the spanning tree-enabled network:

<iframe width="560" height="315" src="https://www.youtube.com/embed/K5A6VsfchiE" frameborder="0" allowfullscreen></iframe>

Finally, we will observe how the spanning tree protocol adapts to changes in the topology. After bringing down the root bridge, we will see a temporary loss in connectivity between two end hosts, then a change in bridge port state on some bridges and re-establishment of a link (following a different Layer 2 path):

<iframe width="560" height="315" src="https://www.youtube.com/embed/lxQRsTIpu8A" frameborder="0" allowfullscreen></iframe>


## Run my experiment


First, you will reserve a topology on GENI that includes four bridges connected in a loop, with one host also connected to each LAN:

![](/blog/content/images/2017/10/spanning-tree-topo.svg)


with each interface assigned an IP address as follows:


<table id="table-1" class="table table-striped table-bordered col-3" data-columns="3">
<thead>
<tr><th class="col-1">Host</th><th class="col-2">on network segment</th><th class="col-3">IP address </th></tr>
</thead>
<tbody>
<tr class="row-1"><td class="col-1">romeo</td><td class="col-2">3-4</td><td class="col-3">10.10.0.100</td></tr>
<tr class="row-3"><td class="col-1">hamlet</td><td class="col-2">1-2</td><td class="col-3">10.10.0.102</td></tr>
<tr class="row-5"><td class="col-1">othello</td><td class="col-2">2-3</td><td class="col-3">10.10.0.104</td></tr>
<tr class="row-5"><td class="col-1">petruchio</td><td class="col-2">1-4</td><td class="col-3">10.10.0.106</td></tr>
</tbody>
</table>


#### Setup

Follow the instructions for the testbed you are using (GENI, Cloudlab, or FABRIC) to reserve the resources and log in to each of the hosts in this experiment. 


<div style="border-color:#FB8C00; border-style:solid; padding: 15px;">

<h4 style="color:#FB8C00;"> GENI-specific instructions: Reserve resources</h4>

<p>To reserve these resources on GENI, create a new slice on GENI. Click on “Add Resources”, and load the RSpec from the following URL:</p>

<p>https://gist.githubusercontent.com/ffund/76522ca62a06c7a259eb62d9fa402f08/raw/842460d4c33ce1815333d7e859b40051e6142188/spanning-tree.xml</p>

<p>You can ignore the warnings indicating that a duplicate IP address is assigned - we are deliberately assigning an IP address of 0.0.0.0 to each bridge interface. Since a bridge operates at Layer 2, it does not need an IP address.</p>

<p>Click on "Site 1" and choose an InstaGENI site to bind to. (There have been some reports of problems with this exercise at the Illinois InstaGENI and Virginia Tech InstaGENI sites; I recommend avoiding those.) Then reserve your resources. </p>

<p>Wait until the resources have turned green, indicating that they are ready to log in. Then, use the details given in the GENI Portal to SSH into each node.</p>


</div>

<br>

<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">

<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Reserve resources</h4>

<p>To reserve these resources on Cloudlab, open this profile page: </p>

<p>https://www.cloudlab.us/p/nyunetworks/education?refspec=refs/heads/spanning_tree_protocol</p>


<p>Click "next", then select the Cloudlab project that you are part of and a Cloudlab cluster with available resources. (This experiment is compatible with any of the Cloudlab clusters.) Then click "next", and "finish".</p>

<p>Wait until all of the sources have turned green and have a small check mark in the top right corner of the "topology view" tab, indicating that they are fully configured and ready to log in. Then, click on "list view" to get SSH login details for the bridges and hosts. Use these details to SSH into each.</p>

</div>

<br>



Next, we will set up the bridge nodes. Open a terminal for every bridge node, and SSH into each one using the details given in the GENI Portal. 

Follow the setup procedure in this section on _each_ bridge node. (It may be quickest to bring up four terminals, each logged in to another bridge node, then copy each command and paste it into all four terminals at once. That way, you will set up all the bridge nodes together.)

Flush the IP address on each experiment interface - since a bridge operates at Layer 2, bridge interfaces do not need an IP address:

```
sudo ip addr flush dev eth1  
sudo ip addr flush dev eth2  
```

Next, run


```
sudo apt-get update
sudo apt-get -y install bridge-utils nload
```

to install the bridge utilities, and also the `nload` utility for monitoring load on the network.

Then, create a new bridge interface named `br0` with the command

```
sudo brctl addbr br0
```

and add the two experiment interfaces to the bridge:

```
sudo brctl addif br0 eth1
sudo brctl addif br0 eth2
```

In the next part of this experiment, we will deliberately trigger a broadcast storm by sending a broadcast frame through this network of bridges.  However, there is some background network protocol traffic in the network that may trigger a broadcast storm automatically, even before we send our own broadcast frame! To make it less likely that a broadcast storm will be triggered by automatic network protocol traffic, we will turn off multicast frame flooding on our bridges.

```
sudo bridge link set dev eth1 mcast_flood off
sudo bridge link set dev eth2 mcast_flood off
```

Then, we can bring the bridge interface up:

```
sudo ifconfig br0 up
```

At this point, you should be able to list the bridge ports with

```
brctl show br0
```

The output should look something like this:

```
bridge name	bridge id		STP enabled	interfaces
br0		8000.02b0102cb433	no		eth1
                                       eth2
```

Note that the spanning tree algorithm is _not_ enabled.

### Create a broadcast storm

The spanning tree protocol creates a loop-free forwarding topology, so as to avoid a broadcast storm. In this part of the experiment, we will create a broadcast storm. Here's what this part of the experiment will look like:

<iframe width="560" height="315" src="https://www.youtube.com/embed/TsOf-Z9UNJc" frameborder="0" allowfullscreen></iframe>

To start, open an SSH terminal to _each_ bridge node, and run

```
sudo tcpdump -i br0 -c 50  "icmp" 
```

in each, to monitor traffic on the bridge interface (both traffic entering and traffic leaving, on all ports). Since we are expecting many frames to be flooded here, we'll stop capturing after 50 packets. (We won't save the packet capture to a file.)

Also, open another SSH terminal to any one bridge node, and on it, run

```
nload br0
```

to monitor traffic on the bridge interface.

Finally, open an SSH terminal to the host named "romeo", and run

```
ping -b -c 1 10.10.0.255
```

to send *one* broadcast packet on the LAN. (The `-b` flag is required when sending a ping to a broadcast address.) Note that this frame will use the broadcast address `ff:ff:ff:ff:ff:ff` as the destination MAC address.


Observe the `nload` output. Do you see a sudden increase in network load - much more than you would expect from a single packet?


Check the `tcpdump` processes running on the four bridge nodes. Can you see the many copies of the same ICMP packet? Look at the ID and sequence fields in the ICMP header, which are used to help match ICMP requests and responses - each ICMP "session" gets a unique ID, and the sequence number is incremented on each subsequent ICMP request in the same session. Are the packets you see in your `tcpdump` output different ICMP requests, or are they all copies of the same request? How can you tell?

After you have observed the broadcast storm, stop it by running 

```
sudo ifconfig br0 down
```

on one bridge node, to break the loop. Wait a few seconds, then bring the bridge back up on the same node with

```
sudo ifconfig br0 up
```


### Set up bridges to use spanning tree algorithm

Next, we will set up the bridges to use the spanning tree algorithm. We will see how a logical loop-free topology is created in the network, with at least one bridge port ending up in the "blocked" state. We can also identify the root bridge (with a path cost of 0), and the designated port for each network segment:

<iframe width="560" height="315" src="https://www.youtube.com/embed/TX5PUxCVAHM" frameborder="0" allowfullscreen></iframe>


On each _bridge_ node, bring down the bridge interface:

```
sudo ifconfig br0 down
```

Run

```
sudo brctl stp br0 on
```

to turn on the spanning tree algorithm.

Start `tcpdump` running on a host on each LAN to capture the BPDUs through which the spanning tree algorithm will be executed. On each of romeo (LAN 3-4), hamlet (LAN 1-2), othello (LAN 2-3), and petruchio (LAN 1-4), run

```
sudo tcpdump -i eth1 stp -w stp-$(hostname -s).pcap
```

Then, bring the bridge interface back up with 

```
sudo ifconfig br0 up
```

Finally, run

```
watch --interval 1 brctl showstp br0
```

on each bridge. This will run the command to show the state of the ports on each bridge (`brctl showstp br0`) repeatedly, every second, so that you can monitor all the bridge ports as the spanning tree algorithm is executed.

> **Note**: if your screen is not big enough, you may not be able to see all of the output. Since the output is updated continuously every second, you won't be able to scroll. You may want to reduce your zoom or font settings in your terminal, to see the complete output.

After some time, you should be able to find a bridge that has one port in the "blocking" state, like this:

<pre>
br0
 bridge id                  8000.02d83a9e5245
 designated root            8000.020327a37077
 root port                  1                       path cost               200
 max age                    20.00                   bridge max age          20.00
 hello time                 2.00                    bridge hello time       2.00
 forward delay              15.00                   bridge forward delay    15.00
 ageing time                300.00
 hello timer                0.00                    tcn timer               0.00
 topology change timer      0.00                    gc timer                226.26
 flags          TOPOLOGY_CHANGE 


eth1 (1)
 port id                    8001                    state            forwarding
 designated root            8000.020327a37077       path cost               100
 designated bridge          8000.026f041efece       message age timer     19.28
 designated port            8001                    forward delay timer    0.00
 designated cost            100                     hold timer             0.06
 flags          

eth2 (2)
 port id                    8002                    state              <b>blocking</b>
 designated root            8000.020327a37077       path cost               100
 designated bridge          8000.02b0102cb433       message age timer     19.27
 designated port            8001                    forward delay timer    0.00
 designated cost            100                     hold timer             0.05
 flags          
</pre>

You won't be able to copy the terminal contents while the `watch` process is still running. To save the output, stop the `brctl` and `tcpdump` instances with Ctrl+C. Then run 

```
brctl showstp br0
```

one last time on each bridge, and save the output. Transfer the packet captures to your own laptop with `scp`. Open the packet capture in Wireshark, and look at a BPDU; make sure you can identify the root ID, root path cost, bridge ID, and port ID in the BPDU.

Then, draw the spanning tree produced in your experiment, and justify your drawing using the BPDUs you collected and output of `brctl showstp br0`.

### Testing the loop-free topology

Let us verify that with the spanning tree algorithm in place to ensure a loop-free topology, a broadcast storm cannot occur. We should see that with the spanning tree protocol creating a logical loop-free topology, we can send a broadcast packet without creating a broadcast storm:

<iframe width="560" height="315" src="https://www.youtube.com/embed/K5A6VsfchiE" frameborder="0" allowfullscreen></iframe>

On each bridge node, run

```
sudo tcpdump -i br0 "icmp"
```

and then, on "romeo", run

```
ping -b -c 1 10.10.0.255
```

to generate a single broadcast packet.

How many times does the ICMP packet appear on each bridge? Does it reach every network segment? Does a broadcast storm occur?

### Reacting to changes in the topology

Finally, we will observe how the spanning tree protocol adapts to changes in the topology. After bringing down the root bridge, we will see a temporary loss in connectivity between two end hosts, then a change in bridge port state on some bridges and re-establishment of a link (following a different Layer 2 path):

<iframe width="560" height="315" src="https://www.youtube.com/embed/lxQRsTIpu8A" frameborder="0" allowfullscreen></iframe>


Choose two "host" nodes that are on opposite "sides" of the tree (on opposites sides of the root bridge. The root bridge is easily identified from the `brctl showstp br0` output as the one with a path cost of 0). 

From one of the two hosts, start to ping the other. For example, if you are using "othello" and "hamlet", on "othello" you might run

```
ping 10.10.0.102
```

On the second host - the one that is the target of the pings - use `tcpdump` to capture ICMP packets:

```
sudo tcpdump -i eth1 -w icmp-change-$(hostname -s).pcap
```

On the other three hosts, run

```
sudo tcpdump -i eth1 -w stp-change-$(hostname -s).pcap
```

Also, on each bridge, run

```
watch --interval 1 brctl showstp br0
```

Then, on the bridge node that is the _root_ bridge in the spanning tree, bring the bridge interface down with

```
sudo ifconfig br0 down
```

Watch the output of the `brctl showstp br0` command on the other bridges, and see how they reconfigure themselves to work around the change in the topology. How long does it take before the "ping" messages you are sending start to get a response again?

Once the ping messages are getting through again, stop the `tcpdump` instances, and use `brctl showstp br0` to get the bridge port state on all four bridges and save it to a file. Use `scp` to transfer these to your laptop.

Then, draw the new spanning tree, and justify it using the BPDUs you have captured and the output of the `brctl showstp br0` command.

## Notes

This exercise is based on Chapter 3 of [TCP/IP Essentials: A Lab-Based Approach](https://www.cambridge.org/core/books/tcpip-essentials/BF444CB283E9191B3A7BAD6E85EA8710).

### Exercise

Answer the following questions:

#### Exercise 1. Broadcast storm

Why does a broadcast storm occur specifically when there is a loop in the network? Even though a small number of broadcast packets are sent, the load in the network is high - why? 

Why does this occur specifically with broadcast frame? Why does this not occur when you send a unicast frame to a host that is in the network?

#### Exercise 2. Set up bridges to use spanning tree algorithm

In this question, you will show how the bridges in the loop form a spanning tree.

You will need screenshots of the final output of `brctl showstp br0` from each bridge (after the spanning tree is complete). These screenshots should show the terminal prompt (important - it must show which bridge the output is from!), the command, and the complete output. Open all four screenshots in your preferred image annotation program. You will progressively add annotations as you work on this exercise.

You will also need the BPDUs collected on each network segment (open these in Wireshark).


##### Elect the root bridge

The first step in the spanning tree algorithm is electing a root bridge, the bridge with the lowest bridge ID.

Annotate each of your `brctl` screenshots by drawing a circle or a box around the bridge ID, and another circle or box around the ID of the "designated root". Also, make a special marking on the screenshot from the root bridge itself (i.e. the one where the bridge ID and designated root are the same).

For example: 

<table style="width:100%">
  <tr>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br1-electroot.png"></img></td>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br2-electroot-1.png"></img></td>
  </tr>
  <tr>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br3-electroot.png"></img></td>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br4-electroot-2.png"></img></td>
  </tr>
</table>


Is the root bridge in your experiment bridge-1, bridge-2, bridge-3, or bridge-4?

To elect the root bridge, each bridge initially considers *itself* the root bridge. Then, bridges exchange spanning tree configuration BPDUs, including a "root identifier" field where they list the ID of the bridge they consider to be the root bridge.

From among the BPDUs you collected, find one where the root bridge in the "root identifier" field is *not* the same as the final root bridge you identified above. (You're likely to find this near the beginning of a packet capture.)

**Note**: in Wireshark, the first part of the bridge ID, which is a priority value, is shown in decimal digits in the Packet Details pane, while in the `brctl` output, it is in hex digits. You can see the ID in hex in Wireshark by highlighting the field in the Packet Details pane and then looking at the Packet Bytes pane.

Take a screenshot showing the Packet Details pane and Packet Bytes pane in Wireshark, with the Root Identifier in this BPDU highlighted. Also note which bridge this BPDU was sent from (use the Bridge ID field in this BPDU to identify the source). Was it sent from bridge-1, bridge-2, bridge-3, or bridge-4?

Upon receiving BPDUs from neighboring bridges, a bridge may see that its neighbor has a different root bridge than itself. If its neighbor's root bridge ID is smaller than the ID of the bridge it currently considers the root, it will update its root bridge to the new, smaller value.

Compare the root bridge ID in the BPDU you showed above, and the ID of the root bridge elected by all the bridges eventually. Give these two values in hex. Which one is greater? Why does the bridge whose BPDU you showed above eventually change its root bridge?

#####  Elect a root port on each non-root bridge

In the second step, each bridge (except the root bridge) computes the root path cost, i.e. cost of the path to the root bridge, through each port. Then, the root port is elected - the one with the lowest root path cost. This is the port that is "facing" the root bridge.


Find the `brctl` screenshots from your root bridge, and annotate it by drawing a circle or a box around the *root port*, which is listed near the top of the output. For the root bridge, the root port will be 0 (there is no root port).  Also draw a circle or a box around the *path cost* for the bridge overall, which is shown right next to the root port, near the top of the output. 

Next, find the `brctl` screenshots from the two bridges that have one port on the same network segment as the root bridge. This is the port that will be selected as the root port. Annotate these two screenshots by drawing a circle or a box around the *root port*, which is listed near the top of the output. Also draw a circle or a box around the *path cost* for the bridge overall, which is shown right next to the root port, near the top of the output. For the port that is the root port, note an "RP" designation in the relevant section in the bottom part of the screenshot.

Then, find the `brctl` screenshots from the bridge that is *farthest* from the root bridge. Annotate these two screenshots by drawing a circle or a box around the *root port*, which is listed near the top of the output.  Also draw a circle or a box around the *path cost* for the bridge overall, which is shown right next to the root port, near the top of the output. For the port that is the root port, note an "RP" designation in the relevant section in the bottom part of the screenshot.


For example: 

<table style="width:100%">
  <tr>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br1-rootport.png"></img></td>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br2-rootport.png"></img></td>
  </tr>
  <tr>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br3-rootport.png"></img></td>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br4-rootport.png"></img></td>
  </tr>
</table>

The bridges find out the path cost (to elect their root ports) by exchanging BPDUs.


From among the BPDUs you collected, find one where the root path cost is zero.  Take a screenshot showing the Packet Details pane and Packet Bytes pane in Wireshark, with the Root Path Cost in this BPDU highlighted. 

Also make a note of the bridge ID and root ID in this BPDU - is this BPDU sent by the root bridge, or a non-root bridge?

From among the BPDUs you collected, find one where the root path cost is *greater than* zero.  Upload a screenshot showing the Packet Details pane and Packet Bytes pane in Wireshark, with the Root Path Cost in this BPDU highlighted. 

Also make a note of the bridge ID and root ID in this BPDU - is this BPDU sent by the root bridge, or a non-root bridge?

##### Select a designated bridge and port on each network segment

In the third step of the spanning tree protocol, the bridge on each network segment with the lowest root path cost will be selected as the designated bridge, and the port that connects that bridge to the network segment is the designated port. 


Find the `brctl` screenshots from your root bridge, and annotate it by drawing a circle or a box around the *designated bridge* and *designated port* (on the designated bridge) for each bridge port. The root bridge will be the designated bridge on any network segment it is connected to.


Next, find the `brctl` screenshots from the two bridges that have one port on the same network segment as the root bridge. Draw a circle or a box around the *designated bridge* and *designated port* for each bridge port, and if this bridge is the designated bridge on the network segment (i.e. the designated bridge is the same as the bridge ID), note a "DP" designation in the relevant section.
 

For the bridge port on the same network segment as the root bridge, you should see that the root bridge is the designated bridge, since it has the lowest path cost. For the other port, this bridge and port should be the designated bridge and port, since it has a lower path cost than the other bridge connected to this network segment.

Then, find the `brctl` screenshots from the bridge that is *farthest* from the root bridge. Draw a circle or a box around the *designated bridge* and *designated port* for each bridge port, and if this bridge is the designated bridge on the network segment (i.e. the designated bridge is the same as the bridge ID), note a "DP" designation in the relevant section.

Is this bridge a designated bridge on any network segment (i.e. are any of its ports designated ports)? 


Example: 

<table style="width:100%">
  <tr>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br1-desport-1.png"></img></td>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br2-desport.png"></img></td>
  </tr>
  <tr>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br3-desport.png"></img></td>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br4-desport.png"></img></td>
  </tr>
</table>

#####  Set bridge ports' states

In the fourth step, on a bridge that is not the root, any bridge port that is neither a root port nor a designated port will be put in the blocked state.

Find the `brctl` screenshots from the two bridges that have one port on the same network segment on the root bridge. Annotate these by drawing a circle or a box around the *status* of each port. 

Then, find the `brctl` screenshots from the bridge that is *farthest* from the root bridge. Annotate it by drawing a circle or a box around the *status* of each port.

Example: 

<table style="width:100%">
  <tr>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br1-state.png"></img></td>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br2-state.png"></img></td>
  </tr>
  <tr>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br3-state.png"></img></td>
    <td><img width="100%" src="/blog/content/images/2021/01/stp-br4-state.png"></img></td>
  </tr>
</table>
#####  Draw the spanning tree

Finally, draw the spanning tree from this section. You can use [this template](https://viewer.diagrams.net/?highlight=0000ff&edit=_blank&layers=1&nav=1&title=spanning-tree-template.drawio#Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fffund%2Ftcp-ip-essentials%2Fmaster%2Flab3%2Fspanning-tree-template.drawio) to create the spanning tree - click the pencil icon and then fill in the values in the shaded boxes. Or, you can draw a spanning tree without the template:

* Put the root bridge at the top of your drawing. Label it "Root bridge". Then, draw each of the other bridges. On each bridge, write its hostname (e.g. "bridge-1", "bridge-2", etc.) Draw links connecting the bridges; label each network segment (e.g. "1-2", "2-3", etc.)
* Label each bridge with its bridge ID, and each port with its port ID (1 or 2).
* If a port is the root port for that bridge, underline its port ID.
* Next to each bridge port, draw a check mark if it is in the forwarding state. If a port is in the blocked state, then draw an X next to it.
* Next to each network segment (1-2, 2-3, 3-4, 1-4), write the designated bridge and the designated port on that bridge (1 or 2) for that network segment.
* Next to each bridge, write the root path cost for that bridge.

#### Exercise 3. Reacting to changes in the topology

Show the output of the `brctl showstp br0` command on each bridge at the end of the section where you practiced "Reacting to changes in the topology". These screenshots should show the terminal prompt (important - it must show which bridge the output is from!), the command, and the complete output.

Also draw the network from the section where you practiced "Reacting to changes in the topology" (i.e. the new spanning tree after you brought down the root bridge), following the same specifications:

* Put the root bridge at the top of your drawing. Draw a circle around the root bridge, and label it "Root". Then, draw each of the other bridges. On each bridge, write its hostname (e.g. "bridge-1", "bridge-2", etc.) Draw links connecting the bridges; label each network segment (e.g. "1-2", "2-3", etc.)
* Label each bridge with its bridge ID, and each port with its port ID (1 or 2).
* If a port is the root port for that bridge, underline its port ID.
* Next to each bridge port, draw a check mark if it is in the forwarding state. If a port is in the blocked state, then draw an X next to it.
* Next to each network segment (1-2, 2-3, 3-4, 1-4), write the designated bridge and the designated port on that bridge (1 or 2) for that network segment.
* Next to each bridge, write the root path cost for that bridge.

#### Exercise 4. Reaction time to changes

When you changed the topology, how much time elapsed between the last ping request arriving at the target _before_ you brought the root bridge down, and the first ping request arriving at the target _after_ you brought the root bridge down? (Use the packet capture from the network segment on which the target node was located.)

Show evidence from your packet captures to support your answer. For example, show a Wireshark screenshot, and annotate it to circle the latest time that a ping request arrives at the target before you brought the root bridge down, and the earliest time that a ping request arrives at the target after you brought the root bridge down.

