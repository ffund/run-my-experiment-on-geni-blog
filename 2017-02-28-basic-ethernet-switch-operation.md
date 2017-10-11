In this experimental demonstration of the basic operation of a layer 2 switch/bridge, we will see:

* how to set up a layer 2 bridge on Linux,
* how a bridge or switch learns MAC addresses and updates its forwarding table, and how it forwards, filters, or floods a frame depending on the forwarding table,
* how a bridge or switch reduces collisions by separating each port into a separate collision domain.

It should take about 60-120 minutes to run this experiment.


To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

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

Switches and bridges improve performance by partitioning the _collision domain_, the section of network within which frames collide (destructively) with one another when they are sent simultaneously. Only one host in a collision domain may transmit at a time, in order to avoid collisions. Thus, the total network capacity is shared among all hosts in a collision domain. With a switch or bridge, each port on a switch becomes its own collision domain, and so each network segment can separately support the full network capacity. 


## Results

In this experiment, we verify the operation of an Ethernet bridge with four hosts connected to four bridge ports.

The following video shows how entries are added to the forwarding table when a frame is received at the bridge. First, the MAC address in the frame's source field is added to the forwarding table. If the destination MAC address of a frame is not in the forwarding table, the frame is flooded out all ports (except the source port); once the destination MAC address is added to the forwarding table, further frames for that address are only forwarded out of the port where the destination is located:

<iframe width="560" height="315" src="https://www.youtube.com/embed/xHH59_S-lX4" frameborder="0" allowfullscreen></iframe>

The next video shows what happens when a host connected to a bridge tries to send traffic to an IP address that is not reachable, and whose MAC address is not known to the sending host. The host will generate ARP requests with the broadcast MAC address (ff:ff:ff:ff:ff:ff) as their destination; these will be flooded out of all bridge ports (except the source port):

<iframe width="560" height="315" src="https://www.youtube.com/embed/FMYmWIcRUV4" frameborder="0" allowfullscreen></iframe>

Sometimes, we may have a scenario where a host tries to send traffic to an IP address that is not reachable, but whose MAC address _is_ known to the sending host. In this case, the frame has a "real" MAC address in the destination field (not the broadcast MAC address), but that MAC address is not present in the forwarding table. The following video shows how those frames are flooded on all bridge ports (except the source port):

<iframe width="560" height="315" src="https://www.youtube.com/embed/EPpUnVKxc7E" frameborder="0" allowfullscreen></iframe>

Finally, we see how a learning switch or bridge reduces the number of hosts in each collision domain, increasing the overall network capacity. In the following experiment, each network segment has approximately 1000 Kbps capacity. When learning is enabled on the bridge, every network segment can support 1000 Kbps capacity. When learning is disabled, the 1000 Kbps capacity is shared among all network segments:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Rjjnij3kunI" frameborder="0" allowfullscreen></iframe>

## Run my experiment

First, you will reserve a topology on GENI that includes a bridge with four ports, and one host connected to each port.

In the GENI Portal, create a new slice, then click "Add Resources". Scroll down to where it says "Choose RSpec" and select the "URL" option, the load the RSpec from the URL: [https://git.io/vysIM](https://git.io/vysIM)

Your topology may have some red warning indicators on it, like this:

![](/blog/content/images/2017/03/bridge-reserve.svg)

This is not a problem - it's just a warning that some IP addresses are duplicated in the topology. In this case, that's intentional (all the bridge interfaces are assigned an IP address of "0.0.0.0" which results in them having no IPv4 address.) The "host" machines in the topology (node-1, node-2, node-3, node-4) each have the IP address 10.0.0.X, where the X is the node number: 10.0.0.1, 10.0.0.2, 10.0.0.3, and 10.0.0.4.

Click on "Site 1" and choose an InstaGENI site to bind to, then reserve your resources. Wait for your nodes to boot up (they will turn green in the canvas display on your slice page in the GENI portal when they are ready).


### Set up the bridge

Use SSH to log in to the "bridge" node in your topology, and install the bridge software with:

```
sudo apt-get update
sudo apt-get -y install bridge-utils
```

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
sudo ifconfig eth1 up
sudo ifconfig eth2 up
sudo ifconfig eth3 up
sudo ifconfig eth4 up
sudo ifconfig br0 up
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

but it won't know about any other nodes' addresses. (At this point, you can use `ifconfig` to find the MAC address of each interface, then match with the output of the command above to determine the "port number" (1, 2, 3, 4) associated with each interface.)

Verify that all of the end hosts in the topology can reach one another through the bridge. On each of node-1, node-2, node-3, and node-4, run

```
ping -c 1 10.0.0.1
ping -c 1 10.0.0.2
ping -c 1 10.0.0.3
ping -c 1 10.0.0.4
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

Now, you can use `ifconfig` on each of the nodes to find out its MAC address and determine which bridge port (1,2,3,4) each one is connected to.

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

On node-2, node-3, and node-4, we will use `tcpdump` to watch traffic on the interface connected to the bridge. Run

```
sudo tcpdump -n -e -i eth1
```

This will run `tcpdump` on the "eth1" interface (`-i eth1`), with the Ethernet headers showing (`-e`) and with IP addresses showing as numeric values, rather than hostnames (`-n`).

Then, on node-1, we will send some traffic to node-2:

```
ping -c 5 10.0.0.2
```

The results of this experiment are shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/xHH59_S-lX4" frameborder="0" allowfullscreen></iframe>


In the output, we can see:

* As a frame from node-1 arrives at the switch, an entry for that host is added to the forwarding table on the bridge, indicated which switch port that MAC address is connected to:

```
02:cd:b5:11:64:e2 dev eth3 vlan 0  
```

* The first frame from node-1 is sent out of _all_ of the other bridge ports, because it is not yet known which bridge port the destination address (02:a0:6e:e0:58:93) is connected to. In the `tcpdump` output, we see an entry similar to the following on all three nodes:

```
17:02:37.531348 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 2736, seq 1, length 64
```

* node-2 sends a reply to the ping request. When this reply reaches the bridge, an entry is added to the forwarding table for node-2, indicated which switch port that MAC address is connected to:

```
02:a0:6e:e0:58:93 dev eth2 vlan 0 
```

* Since the location of node-1 is already known, the reply from node-2 (shown below) is only sent out the bridge port where node-1 is located. Therefore, the reply does _not_ appear in the `tcpdump` output on node-3 or node-4. We do see it in the `tcpdump` output on node-2, and if we were running `tcpdump` on node-1 we would see it there, too:

```
17:02:37.636918 02:a0:6e:e0:58:93 > 02:cd:b5:11:64:e2, ethertype IPv4 (0x0800), length 98: 10.0.0.2 > 10.0.0.1: ICMP echo reply, id 2736, seq 1, length 64
```

* Since the location of node-2 is known, further ping requests (shown below) are only sent out the bridge port where node-2 is located. They therefore appear in the `tcpdump` output on node-2, but not on the other nodes:

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

### Destination address is not reachable

We will also observe what happens when frames are sent to a destination address that is not connected to the switch. 

First, we will repeat the experiment above, but with a destination address that is not connected to the bridge, and never has been. 

Verify that the bridge only knows local MAC addresses, and wait for non-local addresses (if there are any) to expire. Then, on the bridge, run 

```
bridge monitor fdb
```

on node-2, node-3, and node-4, run

```
sudo tcpdump -n -e -i eth1
```

and on node-1, run

```
ping -c 5 10.0.0.5
```

The result is shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FMYmWIcRUV4" frameborder="0" allowfullscreen></iframe>

Because node-1 does not know the MAC address associated with the IP address 10.0.0.5, it sends ARP requests to the broadcast MAC address, ff:ff:ff:ff:ff:ff, asking for the owner of the IP address 10.0.0.5 to reply with its MAC address. These frames with the _broadcast_ MAC address in the destination field are flooded out of all ports, as seen in the `tcpdump` output:

```
18:22:49.844039 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.5 tell 10.0.0.1, length 28
18:22:50.842626 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.5 tell 10.0.0.1, length 28
18:22:51.843635 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.5 tell 10.0.0.1, length 28
18:22:52.862170 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.5 tell 10.0.0.1, length 28
18:22:53.860613 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.5 tell 10.0.0.1, length 28
18:22:54.861167 02:cd:b5:11:64:e2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.0.0.5 tell 10.0.0.1, length 28
```

We will also observe what would happen to frames with a destination MAC address that is not connected to any switch port, but that is also not a broadcast MAC address. For example, what would happen if node-2 was disconnected from the switch?

Verify that the bridge only knows local MAC addresses, and wait for non-local addresses (if there are any) to expire. Then, on the bridge, run 

```
bridge monitor fdb
```

on node-3 and node-4, run

```
sudo tcpdump -n -e -i eth1
```

and on node-2, run

```
sudo ifconfig eth1 down
```

to bring down the interface on node-2 that is connected to the bridge. Finally, on node-1, run

```
ping -c 5 10.0.0.2
```

The results are shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/EPpUnVKxc7E" frameborder="0" allowfullscreen></iframe>

We can see that all five ping requests, which have as their destination the MAC address of node-2 (02:a0:6e:e0:58:93), were flooded out of the ports that node-3 and node-4 are connected to:

```
18:31:50.359797 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 3612, seq 1, length 64
18:31:51.367118 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 3612, seq 2, length 64
18:31:52.375218 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 3612, seq 3, length 64
18:31:53.383385 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 3612, seq 4, length 64
18:31:54.391493 02:cd:b5:11:64:e2 > 02:a0:6e:e0:58:93, ethertype IPv4 (0x0800), length 98: 10.0.0.1 > 10.0.0.2: ICMP echo request, id 3612, seq 5, length 64
```

Bring the interface on node-2 up again by running the following on node-2:

```
sudo ifconfig eth1 up
```

### Effect of a smaller collision domain

Finally, we will observe how a switch or a bridge improves network performance by reducing the number of hosts in each collision domain. Only one host in a collision domain may transmit at any one time, and the other hosts listen to the network in order to avoid collisions. Thus, the total network capacity is shared among all hosts on the collision domain.

To measure this effect, we will use `iperf`, a tool that estimates network capacity by trying to send data as quickly as possible over a link, and measuring the total data rate.

To install `iperf`, on each of node-1, node-2, node-3, and node-4, run

```
sudo apt-get update
sudo apt-get -y install iperf
```

Then, run the `iperf` receiver application on node-3 and node-4:

```
iperf -s
```

Finally, we will run the `iperf` transmitter to send traffic to each of those. Try to start the two transmitters quickly, one after the other.

On node-1, run

```
iperf -c 10.0.0.3 -t 90
```

and on node-2, quickly run

```
iperf -c 10.0.0.4 -t 90
```

After a couple of minutes, you should see messages on the two receiver instances, indicating the total data rate, e.g.

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

The `iperf` receivers should still be running on node-3 and node-4. We will start the transmitters again, one after the other. On node-1, run

```
iperf -c 10.0.0.3 -t 90
```

and on node-2, quickly run

```
iperf -c 10.0.0.4 -t 90
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

This experiment is shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Rjjnij3kunI" frameborder="0" allowfullscreen></iframe>

### Release your resources

When you finish this experiment, please delete your resources in the GENI Portal, to free them for other experimenters.

## Notes

### Exercise

As you complete the "Set up the bridge" section of the tutorial, fill in the blank table cells in [this image](https://docs.google.com/drawings/d/1m1CTV5YJdGkup80SDCZSgwcfX4zhwUW076rdukgVodw/edit?usp=sharing). Use the output of brctl showmacs br0 and ifconfig to find this information.

(To make an editable copy of the image above in Google Drawings, click on the link while signed in to a Google or Google Apps account and click on File > Make a copy. You can then fill in the missing values in the table cells.)

Note how you found each of those values - include the output of any relevant commands, either as a screenshot or copy and paste, and highlight the relevant part. 

As you run the "Learning MAC addresses" section of the experiment, make an ordered list of the following six events:

* the bridge learns the MAC address of node-1,
* the bridge learns the MAC address of node-2,
* node-1 sends the first ping request (with seq 1) to node-2, 
* node-2 sends the first ping reply (with seq 1) to node-1,
* node-2 receives the first ping request (with seq 1) from node-1,
* node-1 receives the first ping reply (with seq 1) from node-2.

(By "make an ordered list", I mean: note which event happened first, which happened second, etc.)

With each list entry, include either

* a frame from your `tcpdump` output or
* a line from the `bridge monitor fdb` output or  `brctl showmacs br0` output

that shows each event occurring. If a frame appears in the `tcpdump` output on multiple network segments, you can copy the frame from any of them. 

(If the output you include shows more than just that one event, make sure to highlight or otherwise mark the relevant part.)

Finally, run the "Effect of a smaller collision domain" section of the experiment. Then, run the following variation: With MAC learning turned ON, run an iperf receiver on node-4 (`iperf -s`) and on node-1, node-2, and node-3, simultaneously run iperf transmitters to send traffic to node-4 (`iperf -c 10.0.0.4 -t 90`).

What network capacity is "seen" by each of the three transmitters? Show the relevant output on the iperf receiver, and explain. Explain how and why this is different from the experiment shown in the first 75 seconds of [this video](https://www.youtube.com/watch?v=Rjjnij3kunI), where MAC learning is also turned on.

