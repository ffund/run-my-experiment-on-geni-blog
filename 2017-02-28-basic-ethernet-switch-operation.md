In this experimental demonstration of the basic operation of a layer 2 switch/bridge, we will see:

* how to set up a layer 2 bridge on Linux,
* how a bridge or switch learns MAC addresses and updates its forwarding table, and how it forwards, filters, or floods a frame depending on the forwarding table,
* how a bridge or switch reduces collisions by separating each port into a separate collision domain.

It should take about 60-120 minutes to run this experiment.



You can run this experiment on CloudLab. Refer to the testbed-specific prerequisites listed below.



<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">
<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Prerequisites</h4>

To reproduce this experiment on Cloudlab, you will need an account on <a href="https://cloudlab.us/">Cloudlab</a>, you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._join-project%29">joined a project</a>, and you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._ssh-access%29">set up SSH access</a>.

</div>
<br>


<div style="border-color:#47aae1; border-style:solid; padding: 15px;">  
<h4 style="color:#47aae1;">FABRIC-specific instructions: Prerequisites</h4>  
To run this experiment on <a href="https://fabric-testbed.net/">FABRIC</a>, you should have a FABRIC account and be part of a FABRIC project. 
</div>  
<br>


<div style="border-color:#9ad61a; border-style:solid; padding: 15px;">  
<h4 style="color:#9ad61a;">Chameleon-specific comments</h4>
<p>This experiment is not supported on the Chameleon testbed. It requires true "broadcast" Ethernet service, which is not supported on Chameleon.</p>
</div>
<br>


* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

An Ethernet switch or bridge relays Ethernet frames between devices connected to different _ports_. It thereby links multiple hosts (or smaller network segments each with multiple hosts) into a single connected network.

When a frame arrives at a switch or a bridge, the source address of the frame is noted. 

* If this address is not already in the forwarding table, a new entry is added, including the address and the port on which the frame was received, and an ageing timer is started for that entry. 
* If the address is already in the forwarding table, the ageing timer for that entry is restarted. If the port on which the frame was received is different (e.g. if the host has moved to a different port) then the entry is updated to reflect the new port.
 
The ageing timer is used to remove old entries from the forwarding table. When the "age" value for an entry in the table passes a given threshold, that entry is removed.

Then, the forwarding table is used to determine whether to forward, filter, or flood the frame, depending on its destination address:

* When the switch receives a frame that is destined for an address that is not in the table, the switch will _flood_ the frame out of all ports other than the port on which it arrived.
* If the destination address is in the table, and is known to be reachable on the same port that the frame was received on, the switch will _filter_ (drop) the frame. 
* Otherwise, the switch will _forward_ the frame out of the port corresponding to its destination address in the forwarding table.

Switches and bridges improve performance by partitioning the _collision domain_, the section of network within which frames collide (destructively) with one another when they are sent simultaneously. Only one host in a collision domain may transmit at a time, in order to avoid collisions. Thus, the total network capacity is shared among all hosts in a collision domain. With a learning switch or bridge, each port on a switch becomes its own collision domain, and so each network segment can separately support the full network capacity. 


## Results

In this experiment, we verify the operation of an Ethernet bridge with four hosts connected to four bridge ports.

The following video shows how entries are added to the forwarding table when a frame is received at the bridge. First, the MAC address in the frame's source field is added to the forwarding table. If the destination MAC address of a frame is not in the forwarding table, the frame is flooded out all ports (except the source port); once the destination MAC address is added to the forwarding table, further frames for that address are only forwarded out of the port where the destination is located:

<iframe width="560" height="315" src="https://www.youtube.com/embed/xHH59_S-lX4" frameborder="0" allowfullscreen></iframe>

We also see how a learning switch or bridge reduces the number of hosts in each collision domain, increasing the overall network capacity. In the following experiment, each network segment has approximately 1000 Kbps capacity. When learning is enabled on the bridge, every network segment can support 1000 Kbps capacity. When learning is disabled (i.e., the bridge acts like a "basic" hub that does not separate the network into collision domains), the 1000 Kbps capacity is shared among all network segments:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Rjjnij3kunI" frameborder="0" allowfullscreen></iframe>

## Run my experiment

First, you will reserve a topology that includes a bridge with four ports, and one host connected to each port.

Follow the instructions for the testbed you are using (GENI or Cloudlab) to reserve the resources and log in to each of the hosts in this experiment. 



<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">

<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Reserve resources</h4>

<p>To reserve resources on Cloudlab, open this profile page: </p>

<p>https://www.cloudlab.us/p/nyunetworks/education?refspec=refs/heads/learning_switch_22</p>

<p>Click "next", then select the Cloudlab project that you are part of and a Cloudlab cluster with available resources. (This experiment is compatible with any of the Cloudlab clusters.) Then click "next", and "finish".</p>

<p>Wait until all of the sources have turned green and have a small check mark in the top right corner of the "topology view" tab, indicating that they are fully configured and ready to log in. Then, click on "list view" to get SSH login details for the client, router, and server hosts, and SSH into each.</p>

</div>
<br>


<div style="border-color:#47aae1; border-style:solid; padding: 15px;">  
<h4 style="color:#47aae1;">FABRIC-specific instructions: Reserve resources</h4>  
<p>To run this experiment on <a href="https://fabric-testbed.net/">FABRIC</a>, open the JupyterHub environment on FABRIC, open a shell, and run </p>

<pre>
git clone https://github.com/teaching-on-testbeds/fabric-education bridge
cd bridge
git checkout bridge
</pre>
<p>Then open the notebook titled "start_here.ipynb".</p>  
<p>Follow along inside the notebook to reserve resources and get the login details for each host and router in the experiment.</p>  
<p>When you have logged in to each node, continue to the next section.</p>  
</div>  
<br>





### Set up the bridge

Use SSH to log in to the "bridge" node in your topology, and install the bridge software with:

```
sudo apt update
sudo apt -y install bridge-utils
```


<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">
<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Bridge interfaces</h4>

<p>If you run this experiment on CloudLab, the names of the bridge interfaces will be: <code>eth1</code>, <code>eth2</code>, <code>eth3</code>, <code>eth4</code>, as indicated in the instructions. </p>
</div>
<br>


<div style="border-color:#47aae1; border-style:solid; padding: 15px;">  
<h4 style="color:#47aae1;">FABRIC-specific instructions: Bridge interfaces</h4>

<p>If you run this experiment on FABRIC, the names of the bridge interfaces will be: <code>enp7s0</code>, <code>enp8s0</code>, <code>enp9s0</code>, <code>enp10s0</code>. In the instructions, you must replace all references to <code>eth1</code>, <code>eth2</code>, <code>eth3</code>, <code>eth4</code> with the names <code>enp7s0</code>, <code>enp8s0</code>, <code>enp9s0</code>, <code>enp10s0</code>. </p>
  
</div>  
<br>

Since a bridge operates at Layer 2, bridge interfaces should not have an IP address. On the bridge, flush the IP address from each of the four experiment interfaces (but be careful not to bring down the control interface):

```
sudo ip addr flush dev eth1
sudo ip addr flush dev eth2
sudo ip addr flush dev eth3
sudo ip addr flush dev eth4
```

Then, create a new bridge interface named `br0` with the command

```
sudo brctl addbr br0
```

And add the four interfaces to the bridge:

```
sudo brctl addif br0 eth1
sudo brctl addif br0 eth2
sudo brctl addif br0 eth3
sudo brctl addif br0 eth4
```

Finally, we will bring "up" all of the interfaces:

```
sudo ip link set eth1 up
sudo ip link set eth2 up
sudo ip link set eth3 up
sudo ip link set eth4 up
sudo ip link set br0 up
```

At this point, you should be able to list the bridge ports with

```
brctl show br0
```

The output should look something like this:

```
bridge name     bridge id               STP enabled     interfaces
br0             8000.021e6b573974       no              eth1
                                                        eth2
                                                        eth3
                                                        eth4
```

You can also see the MAC addresses that the bridge is aware of, with 

```
brctl showmacs br0
```

For now, since no traffic has passed through the bridge, it will only know the four MAC addresses corresponding to the four local interfaces that are on this host (i.e. on the bridge):

```
port no	mac addr		is local?	ageing timer
  4	02:1e:6b:57:39:74	yes		   0.00
  1	02:34:7e:04:7b:4e	yes		   0.00
  2	02:97:e4:d3:aa:95	yes		   0.00
  3	02:cb:96:8d:5b:1f	yes		   0.00
```

but it won't know about any other nodes' addresses. (At this point, you can use `ip addr` to find the MAC address of each interface, then match with the output of the command above to determine the "port number" (1, 2, 3, 4) associated with each interface.)

> **Note**: On some versions of the Ubuntu operating system, you'll see each MAC address appear twice in the `brctl showmacs br0` output! This is OK.

Verify that all of the end hosts in the topology can reach one another through the bridge. On romeo, use `ping` to contact each of the other hosts and verify that a response is returned - 

```
ping -c 1 10.0.0.101
ping -c 1 10.0.0.102
ping -c 1 10.0.0.103
```

Immediately afterwards, the bridge should be aware of every nodes' MAC address, and which port they are located on. The output of 

```
brctl showmacs br0
```

should show something like this:

```
port no	mac addr		is local?	ageing timer
  4	02:1e:6b:57:39:74	yes		   0.00
  1	02:34:7e:04:7b:4e	yes		   0.00
  1	02:5d:96:11:77:19	no		   1.86
  2	02:97:e4:d3:aa:95	yes		   0.00
  2	02:a0:6e:e0:58:93	no		   1.89
  3	02:cb:96:8d:5b:1f	yes		   0.00
  3	02:cd:b5:11:64:e2	no		   1.89
  4	02:da:4f:51:ac:c9	no		   1.86
```

Now, you can use `ip addr` on each of the four hosts to find out its MAC address on the experiment interface, and determine which bridge port (1,2,3,4) each one is connected to.

As time passes without further traffic passing through the bridge, the values in the "ageing timer" column will increase. Once an entry ages past 300 seconds, it will be removed from the forwarding table. So, after 5 minutes with no traffic, the output of

```
brctl showmacs br0
```

again shows only the MAC addresses local to the bridge.

### Learning MAC addresses

Now, we will watch as MAC addresses are added to the forwarding table on the bridge. Start by verifying that the forwarding table contains _only_ local addresses:

```
brctl showmacs br0
```
and if not, wait for the non-local addresses to age out.

Then, run

```
bridge monitor fdb
```

on the bridge. This will show us new entries live, as they are added to or removed from the forwarding table.

On romeo, juliet, hamlet, and ophelia, we will use `tcpdump` to watch traffic on the interface connected to the bridge. Run

```
sudo tcpdump -n -e -i eth1
```

This will run `tcpdump` on the "eth1" interface (`-i eth1`), with the Ethernet headers showing (`-e`) and with IP addresses showing as numeric values, rather than hostnames (`-n`).

Then, in a second terminal on romeo, we will send some traffic to juliet:

```
ping -c 5 10.0.0.101
```

<!--
The results of this experiment are shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/xHH59_S-lX4" frameborder="0" allowfullscreen></iframe>
-->

In the output, we can see:

* As a frame from romeo arrives at the switch, an entry for that host is added to the forwarding table on the bridge, indicated which switch port that MAC address is connected to:

```
02:cd:b5:11:64:e2 dev eth3 master br0  
```

* The first frame from romeo is sent out of _all_ of the other bridge ports, because it is not yet known which bridge port the destination address (02:a0:6e:e0:58:93) is connected to. In the `tcpdump` output, we see an entry similar to the following on all of the nodes:

```
17:02:37.531348 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 2736, seq 1, length 64
```

* juliet sends a reply to the ping request. When this reply reaches the bridge, an entry is added to the forwarding table for juliet, indicated which switch port that MAC address is connected to:

```
02:a0:6e:e0:58:93 dev eth2 master br0 
```

* Since the location of romeo is already known, the reply from juliet (shown below) is only sent out the bridge port where romeo is located. Therefore, the reply does _not_ appear in the `tcpdump` output on hamlet or ophelia. We do see it in the `tcpdump` output on juliet, and if we were running `tcpdump` on romeo we would see it there, too:

```
17:02:37.636918 02:a0:6e:e0:58:93 > 02:cd:b5:11:64:e2, ethertype IPv4 (0x0800), length 98: 10.0.0.2 > 10.0.0.1: ICMP echo reply, id 2736, seq 1, length 64
```

* Since the location of juliet is known, further ping requests (shown below) are only sent out the bridge port where juliet is located. They therefore appear in the `tcpdump` output on juliet, but not on the other nodes:

```
17:02:38.638746 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 2736, seq 2, length 64
17:02:38.638776 02:a0:6e:e0:58:93 > 02:cd:b5:11:64:e2, ethertype IPv4 (0x0800), length 98: 10.0.0.2 > 10.0.0.1: ICMP echo reply, id 2736, seq 2, length 64
17:02:39.640698 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 2736, seq 3, length 64
17:02:39.640728 02:a0:6e:e0:58:93 > 02:cd:b5:11:64:e2, ethertype IPv4 (0x0800), length 98: 10.0.0.2 > 10.0.0.1: ICMP echo reply, id 2736, seq 3, length 64
17:02:40.642796 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 2736, seq 4, length 64
17:02:40.642833 02:a0:6e:e0:58:93 > 02:cd:b5:11:64:e2, ethertype IPv4 (0x0800), length 98: 10.0.0.2 > 10.0.0.1: ICMP echo reply, id 2736, seq 4, length 64
17:02:41.644840 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 2736, seq 5, length 64
17:02:41.644866 02:a0:6e:e0:58:93 > 02:cd:b5:11:64:e2, ethertype IPv4 (0x0800), length 98: 10.0.0.2 > 10.0.0.1: ICMP echo reply, id 2736, seq 5, length 64
```


### Effect of a smaller collision domain

Finally, we will observe how a switch or a bridge improves network performance by reducing the number of hosts in each collision domain. Only one host in a collision domain may transmit at any one time, and the other hosts listen to the network in order to avoid collisions. Thus, the total network capacity is shared among all hosts on the collision domain.

To measure this effect, we will use `iperf3`, a tool that estimates network capacity by trying to send data as quickly as possible over a link, and measuring the total data rate.

To install `iperf3`, on each of romeo, juliet, hamlet, and ophelia, run

```
sudo apt update
sudo apt -y install iperf3
```

Then, run the `iperf3` receiver application on juliet and ophelia:

```
iperf3 -s -1
```

Finally, we will run the `iperf3` transmitter to send traffic to each of those. Try to start the two transmitters quickly, one after the other.

On romeo, run

```
iperf3 -c 10.0.0.101 -t 90
```

and on hamlet, quickly run

```
iperf3 -c 10.0.0.103 -t 90
```

After a couple of minutes, you should see messages on the two receiver instances, indicating the total data rate in kilobits per second, e.g.

```
[  4]  0.0-97.3 sec  9.88 MBytes   851 Kbits/sec
```

on one, and

```
[  4]  0.0-97.3 sec  9.88 MBytes   851 Kbits/sec
```

on the other.

Now, we will turn off MAC address learning on the bridge (by setting the ageing timeout to 0, so that all forwarding table entries expire immediately):

```
sudo brctl setageing br0 0
```

Without MAC address learning, all frames will be flooded on all ports, and all nodes will be in a single collision domain. The 1000 Kbps capacity will be shared between them.

Run the `iperf3` receiver application again on juliet and ophelia:

```
iperf3 -s -1
```

and start the transmitters again, one after the other. On romeo, run

```
iperf3 -c 10.0.0.101 -t 90
```

and on hamlet, quickly run

```
iperf3 -c 10.0.0.103 -t 90
```

Now we can see from the measured data rate on each of the two receivers, that the 1000 Kbps link capacity is shared between the two traffic flows.
```
[  5]  0.0-150.8 sec  7.50 MBytes   417 Kbits/sec
```

on one, and

```
[  5]  0.0-130.8 sec  8.88 MBytes   569 Kbits/sec
```

on the other.

To restore MAC address learning, on the bridge run

```
sudo brctl setageing br0 300
```

<!-- 

This experiment is shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Rjjnij3kunI" frameborder="0" allowfullscreen></iframe>

--> 

> **Note**: If you are answering the questions in the "Exercises" section, you will have to additionally run another variation on this experiment! Make sure to run it before deleting your resources.


### Transparent operation of a bridge 

Let us also note that the bridge is transparent - it does not modify frame headers.

On *both* romeo and juliet, run

<pre>
sudo tcpdump -i eth1 -w $(hostname -s)-bridge.pcap
</pre>

Then, in another terminal tab or window, send echo requests from romeo to juliet:

```
ping -c 3 10.0.0.101
```

After receiving the third echo reply, stop both `tcpdump` processes. You can play back a summary of your packet capture with

```
tcpdump -en -r $(hostname -s)-bridge.pcap
```

on each host, and you can use `scp` to transfer them to your laptop for further analysis. 

Pick a single ICMP request, and find the packet carrying that ICMP request in both packet captures; the one on romeo and the one on juliet. (To make sure it is the same packet, check the ICMP sequence number. It should be the same in both.)

In the packet capture on romeo, you will see what this packet looks like as it traverses the link from romeo to the bridge. In the packet capture on juliet, you will see what this packet looks like as it traverses the link from the bridge to juliet. Make a note of the address fields in the Ethernet header, in particular. Does anything change as the packet traverses the bridge?



## Notes

### Exercise

* As you complete the [Set up the bridge](#setupthebridge) section of the experiment, save the output of `ip addr` on each host and on the bridge, and also save the output of the `brctl showmacs br0` command when the bridge has seen the MAC addresses of all four hosts. 

  Annotate these to draw a box or a circle around the MAC addresses on the experiment interfaces in the `ip addr` output, and to write the name of the host (or bridge) that the MAC address belongs to next to each line of the `brctl` output.

    Then, use this information to fill in a table showing the bridge configuration. For each bridge port (1, 2, 3, 4), indicate:

    * which interface on the bridge - `eth1`, `eth2`, `eth3`, `eth4` - is on that port, and the MAC address of that interface.
   * which host (romeo, juliet, hamlet, or ophelia) is on that port, and its MAC address
<table id="table-1" class="table table-striped table-bordered col-2" data-columns="2"><thead>
<thead>
<tr>
<th>Bridge port</th>
<th>Bridge interface, MAC</th>
<th>Hostname, MAC</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td></td>
<td></td>
</tr>
<tr>
<td>2</td>
<td></td>
<td></td>
</tr>
<tr>
<td>3</td>
<td></td>
<td></td>
</tr>
<tr>
<td>4</td>
<td></td>
<td></td>
</tr>
</tbody></table>

 

* As you run the [Learning MAC addresses](#learningmacaddresses) section of the experiment, **use your knowledge of how a bridge works** to make an ordered list of the following six events:


  * the bridge learns the MAC address of romeo,
  * the bridge learns the MAC address of juliet,
  * romeo sends the first ping request (with seq 1) to juliet, 
  * juliet sends the first ping reply (with seq 1) to romeo,
  * juliet receives the first ping request (with seq 1) from romeo,
  * romeo receives the first ping reply (with seq 1) from juliet.

  (By "make an ordered list", I mean: note which event happened first, which happened second, etc.)

 With each list entry, include either

  * a frame from your `tcpdump` output or
  * a line from the `bridge monitor fdb` output or  `brctl showmacs br0` output

  that shows each event occurring. 

  (If the output you include shows more than just that one event, make sure to highlight or otherwise mark the relevant part.)

* Finally, run the [Effect of a smaller collision domain](#effectofasmallercollisiondomain) section of the experiment. Show the `iperf3` receiver output both with and without MAC learning.

  Then, run the following variation: With MAC learning turned ON, open three terminal sessions on ophelia, and run the following `iperf3` sessions - one in each terminal: `iperf3 -s -1 -p 5201`, `iperf3 -s -1 -p 5301`, `iperf3 -s -1 -p 5401`. Leave these running.

   Then, on romeo, juliet, and hamlet, (as close to simultaneously as you can), start iperf3 transmitters to send traffic to ophelia: `iperf3 -c 10.0.0.103 -t 90 -p 5201` on romeo, `iperf3 -c 10.0.0.103 -t 90 -p 5301` on juliet, and `iperf3 -c 10.0.0.103 -t 90 -p 5401` on hamlet.

  What network capacity is "seen" by each of the three transmitters? Show the relevant output on the iperf3 receiver, and explain. Explain how and why this is different from the previous experiment, where MAC learning is also turned on.

* For the [Transparent operation of a bridge](#transparentoperationofabridge) section of the experiment: Show the `iperf3` receiver output both with and without MAC learning.

  * What are the source and destination IP and MAC addresses of a packet that went from romeo to the bridge? Show the annotated screenshot from your `tcpdump` replay on romeo, with the source and destination IP address and source and destination MAC address clearly labeled. Does romeo put the bridge's MAC address or juliet's MAC address in the destination address field of the Ethernet header?
  * What are the source and destination IP and MAC addresses of the same packet when it goes from the bridge to juliet? Show the annotated screenshot *of the same packet* from your `tcpdump` replay on juliet, with the source and destination IP address and source and destination MAC address clearly labeled. Does the bridge modify the source or destination MAC address in the Ethernet header?


### Supplemental activities 
#### Destination address is not reachable

We can also observe what happens when frames are sent to a destination address that is not connected to the switch. 

First, we will repeat the experiment above, but with a destination address that is not connected to the bridge, and never has been. 

Verify that the bridge only knows local MAC addresses, and wait for non-local addresses (if there are any) to expire. Then, on the bridge, run 

```
bridge monitor fdb
```

on juliet, hamlet, and ophelia run

```
sudo tcpdump -n -e -i eth1
```

and on romeo, run

```
ping -c 5 10.0.0.200
```

<!--
The result is shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FMYmWIcRUV4" frameborder="0" allowfullscreen></iframe>

-->

Because romeo does not know the MAC address associated with the IP address 10.0.0.200, it sends ARP requests to the broadcast MAC address, ff:ff:ff:ff:ff:ff, asking for the owner of the IP address 10.0.0.200 to reply with its MAC address. These frames with the _broadcast_ MAC address in the destination field are flooded out of all ports, as seen in the `tcpdump` output:

```
18:22:49.844039 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.200 tell 10.0.0.100, length 28
18:22:50.842626 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.200 tell 10.0.0.100, length 28
18:22:51.843635 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.200 tell 10.0.0.100, length 28
18:22:52.862170 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.200 tell 10.0.0.100, length 28
```

We will also observe what would happen to frames with a destination MAC address that is not connected to any switch port, but that is also not a broadcast MAC address. For example, what would happen if juliet was disconnected from the switch?

Verify that the bridge only knows local MAC addresses, and wait for non-local addresses (if there are any) to expire. Then, on the bridge, run 

```
bridge monitor fdb
```

on hamlet and ophelia, run

```
sudo tcpdump -n -e -i eth1
```

and on juliet, run

```
sudo ip link set dev eth1 down
```

to bring down the interface on juliet that is connected to the bridge. Finally, on romeo, run

```
ping -c 5 10.0.0.101
```

<!--
The results are shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/EPpUnVKxc7E" frameborder="0" allowfullscreen></iframe>
--> 

We can see that all five ping requests, which have as their destination the MAC address of juliet (02:a0:6e:e0:58:93), were flooded out of the ports that hamlet and ophelia are connected to:

```
18:31:50.359797 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.100 > 10.0.0.101: ICMP echo request, id 3612, seq 1, length 64
18:31:51.367118 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.100 > 10.0.0.101: ICMP echo request, id 3612, seq 2, length 64
18:31:52.375218 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.100 > 10.0.0.101: ICMP echo request, id 3612, seq 3, length 64
18:31:53.383385 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.100 > 10.0.0.101: ICMP echo request, id 3612, seq 4, length 64
18:31:54.391493 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.100 > 10.0.0.101: ICMP echo request, id 3612, seq 5, length 64
```

Bring the interface on juliet up again by running the following:

```
sudo ip link set dev eth1 up
```


