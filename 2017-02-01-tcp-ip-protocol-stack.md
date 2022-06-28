In this experimental demonstration of the TCP/IP protocol architecture, we will examine network addresses and connections at

* the network access (a.k.a. data link) layer,
* the Internet (IP) layer, 
* the transport layer (logical host-to-host), and
* the application layer.


It should take about 60 minutes to run this experiment.


To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.


## Background

TCP/IP, the protocol stack that is used in communication over the Internet and most other computer networks, has a five-layer architecture. From the bottom up, these layers and their primary responsibilities are:

1. **Physical layer**: responsible for transmitting bits as a physical signal over some medium, e.g. sending electrical signals over a cable, or transmitting an electromagnetic wave  over a wireless link.
2. **Link layer** (also called the data link layer or network access layer): responsible for transferring data between devices that are on the same network. This includes (local) addressing, arbitrating access to a shared medium, and checking for (and sometimes correcting) errors that occurred during physical transmission.
3. **Network layer** (also called Internet layer): responsible for transferring data between different networks. This includes (global) addressing and routing.
4. **Transport layer**: responsible for end-to-end communication. The two most common protocols at this layer are UDP, which provides a basic datagram service with no reliability guarantees, and TCP, which provides flow control, connection establishment, and reliable data delivery.
5. **Application layer**: defines protocols that are used by processes on the end hosts. For example: [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol), [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol), [SSH](https://en.wikipedia.org/wiki/Secure_Shell). 


![](/blog/content/images/2017/03/tcpip-layers.svg)
<small><i>Image from William Stallings, "Data and Computer Communications"</i></small>

At each layer, addresses or other identifiers are used in that layer's packet header in order to designate a particular networking device, an end host, or a process or service. For example, consider the image above, where Application X on Host A connects to Application X on Host B: 

* Using the interface between the application layer and transport layer (the [socket API](https://en.wikipedia.org/wiki/Network_socket)), Host A will establish a connection to Application X on Host B using Host B's **IP address** (network layer address) and the **TCP port number** (transport layer identifier) that Application X is using. 
* When packets are received at host B, the transport layer will look at the destination port number in the TCP header to identify which application to  deliver it to. 
* At the network layer, Host A, Host B, and Router J will determine which link to send packets on according to the destination **IP address** in the packet's IP header. (For example, Router J is connected to two networks; it uses the destination IP address of a packet to determine whether to send it out on Network 1 or Network 2.)
* The **MAC address** identifies the "local" destination of a packet (i.e. the destination that is on the same network). For example, Host A, which is on Network 1, sends data to Host B through Router J, which is also on Network 1. The destination MAC address in the link layer header of packets from Host A to Host B will be Router J's MAC address.

In this experiment, we will establish a connection like the one in the image above, and examine the addresses and identifiers used at each layer of the protocol stack.

## Run my experiment

First, reserve your resources.

In the GENI Portal, create a new slice, then click "Add Resources". Scroll down to where it says "Choose RSpec" and select the "URL" option, the load the RSpec from the URL: [https://git.io/JUWPU](https://git.io/JUWPU)

This should load the following topology onto your canvas:

![](/blog/content/images/2017/01/protocol-stack-topology-1.png)

In addition to setting up three VMs and connecting them with links, this RSpec also sets up an IP (Internet layer) address for each VM on each network that it is connected to.

![](/blog/content/images/2017/01/protocol-stack-addresses.svg)

Click on "Site 1" and choose an InstaGENI site to bind to, then reserve your resources. Wait for your nodes to boot up (they will turn green in the canvas display on your slice page in the GENI portal when they are ready) and then log in to each node.



### View network interfaces

A network interface is the point of interconnection between a computer and a network. It may represent a hardware network interface card (NIC), such as a WiFi network card, or a virtualized NIC that is realized in software.


On your "client" node, run `ifconfig` to see the network interfaces on this host:

```
ffund01@client:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:91:71:c7:6b:c4  
          inet addr:172.17.3.1  Bcast:172.31.255.255  Mask:255.240.0.0
          inet6 addr: fe80::91:71ff:fec7:6bc4/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6297 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4079 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:6318245 (6.3 MB)  TX bytes:369007 (369.0 KB)

eth1      Link encap:Ethernet  HWaddr 02:03:c7:4c:a3:66  
          inet addr:10.0.10.2  Bcast:10.0.10.255  Mask:255.255.255.0
          inet6 addr: fe80::3:c7ff:fe4c:a366/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:94 errors:0 dropped:0 overruns:0 frame:0
          TX packets:140 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:7095 (7.0 KB)  TX bytes:10596 (10.5 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:32 errors:0 dropped:0 overruns:0 frame:0
          TX packets:32 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:2368 (2.3 KB)  TX bytes:2368 (2.3 KB)

```

We note two Ethernet interfaces (named `eth0` and `eth1`) and a loopback interface (named `lo`). The loopback interface is a virtual network interface that the computer uses for processes on the same host to communicate with one another using network protocols. The two Ethernet interfaces represent two points of attachment to networks: one to the public Internet (which you need in order to SSH in to your system from outside!), and one to the private link that you've set up between the VMs in your topology.  

In addition to the name of each network interface, we can also see the MAC address (at the network access/data link layer) in the `ifconfig` output:

```
HWaddr 02:03:c7:4c:a3:66
```

and the IP (Internet layer) address:

```
inet addr:10.0.10.2
```

Since we know from the image above that the IP address of the "client" on the link in our topology is 10.0.10.2, we can tell that `eth1` is the interface through which we connect to the "private" experiment network. For the rest of this experiment, we will focus on the "private" networks that we have configured, and the interfaces connected to these (such as `eth1` in the example above).

If we run `ifconfig` on the "router", we will see an additional network interface, since we set up this node to connect to _two_ private links:

```
ffund01@router:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:79:61:b2:00:24  
          inet addr:172.17.3.2  Bcast:172.31.255.255  Mask:255.240.0.0
          inet6 addr: fe80::79:61ff:feb2:24/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1214 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1229 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:130786 (130.7 KB)  TX bytes:125836 (125.8 KB)

eth1      Link encap:Ethernet  HWaddr 02:9a:88:da:35:0b  
          inet addr:10.0.20.1  Bcast:10.0.20.255  Mask:255.255.255.0
          inet6 addr: fe80::9a:88ff:feda:350b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:49 errors:0 dropped:0 overruns:0 frame:0
          TX packets:101 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:3155 (3.1 KB)  TX bytes:7626 (7.6 KB)

eth2      Link encap:Ethernet  HWaddr 02:40:b6:e7:60:26  
          inet addr:10.0.10.1  Bcast:10.0.10.255  Mask:255.255.255.0
          inet6 addr: fe80::40:b6ff:fee7:6026/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:132 errors:0 dropped:0 overruns:0 frame:0
          TX packets:82 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:7808 (7.8 KB)  TX bytes:7475 (7.4 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
``` 

Since we already knew which IP address is assigned to each interface, we can now also identify the interface name and MAC address for each from the `ifconfig` output:

![](/blog/content/images/2017/01/protocol-stack-addresses-completed-1.svg)


### Address tables at the network access (data link) layer

As traffic moves through the network, hosts become aware of the MAC addresses of neighbors on the same network. For each network interface (point of attachment to a network), it will store a table of MAC addresses and IP addresses of hosts that are attached to that same network.

First, generate some traffic to ensure that all the local address tables are updated. On the "client" node, run

```
ping -c 3 10.0.20.2
```

to send some traffic to the "server" (via the router).

Then, on each node, run 

```
arp -n -i IFACE
```

where instead of `IFACE`, substitute the name of the interface for which you want to see the address table (e.g. `eth1`).

Note that the "client" knows the MAC address of one interface of the "router", which is on the same physical network, but not the "server", which is on a different network:

```
ffund01@client:~$ arp -n -i eth1
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.10.1                ether   02:40:b6:e7:60:26   C                     eth1
```

On the "router", we can see the MAC address of the "client" on one interface and the MAC address of the "server" on another interface:

```
ffund01@router:~$ arp -n -i eth1
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.20.2                ether   02:32:77:16:a8:98   C                     eth1
ffund01@router:~$ arp -n -i eth2
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.10.2                ether   02:03:c7:4c:a3:66   C                     eth2
```

Like the "client", the "server" knows the MAC address of one of the "router" interfaces that is on the same network as itself, but not any MAC addresses of interfaces on the other network:

```
ffund01@server:~$ arp -n -i eth1 
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.20.1                ether   02:9a:88:da:35:0b   C                     eth1
```

### Moving between networks at the Internet (IP) layer

The Internet layer is concerned with moving data between _different_ networks. Since our "client" and "server" are connected to two different networks, traffic between them is relayed through the "router".

To see how traffic is routed from the "client" to the "server", we will use the `traceroute` command. On the "client", run

```
traceroute -n 10.0.20.2
```

and observe the results. You should see an intermediate hop through the router, like this (you may have to run it twice):

```
ffund01@client:~$ traceroute -n 10.0.20.2
traceroute to 10.0.20.2 (10.0.20.2), 30 hops max, 60 byte packets
 1  10.0.10.1  0.603 ms  0.563 ms  0.521 ms
 2  10.0.20.2  0.908 ms  0.834 ms  0.761 ms
```

Similarly, on the "server", we can see how traffic goes through the "router" to reach the "client":

```
ffund01@server:~$ traceroute -n 10.0.10.2
traceroute to 10.0.10.2 (10.0.10.2), 30 hops max, 60 byte packets
 1  10.0.20.1  0.562 ms  0.517 ms  0.473 ms
 2  10.0.10.2  0.991 ms  0.938 ms  0.906 ms
```

If we stop the "router" from forwarding traffic between networks, then traffic from the "client" will no longer reach the "server". Try it - on the "router" run the following to stop forwarding: 

```
sudo sysctl -w net.ipv4.ip_forward=0
```

and then run the `traceroute` on the "client" again. 

This time, you should see only `*`s, denoting no route to the target:

```
ffund01@client:~$ traceroute -n 10.0.20.2
traceroute to 10.0.20.2 (10.0.20.2), 30 hops max, 60 byte packets
 1  * * *
 2  * * *
 3  * * *
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  * * *
 9  * * *
10  * * *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
```

Without the router forwarding traffic at the Internet layer, hosts can communicate within their own network, but not across networks. Try the following at the client:

```
# contact router, in same network - should work
ping -c 3 10.0.10.1 
# contact server, in another network - will not work
ping -c 3 10.0.20.2 
```

Restore packet forwarding on the router by running the following command on the "router" node:


```
sudo sysctl -w net.ipv4.ip_forward=1
```

and verify that now traffic can move across networks again:

```
# both in-network and out-of-network traffic should work
ping -c 3 10.0.10.1 
ping -c 3 10.0.20.2 
```


### Transport (host to host) and application layers

Finally, we will initiate a network connection at the application layer.

On the "server", we will use the `netcat` application to receive incoming connection on TCP port 4444:

```
netcat -l 4444
```

and on the "client", we will initiate an application-layer connection to the "server" on that same port:

```
netcat 10.0.20.2 4444
``` 

While it will initially appear as if nothing has happened, you can type anything into either window, hit "Enter", and see it appear in the other.

To see the transport-layer port used on each host, we can use `netstat`. (We already know that the "server" is using TCP port 4444, because that's what we instructed it to use; but the "client" will be using a random TCP port number).

While the `netcat` link is still active, open two more terminal windows and SSH into the client and server, respectively. In each, run 

```
netstat -n | grep 4444
```

to see the `netstat` line corresponding to the connection on port 4444.

The output below shows that in my network, the connection is from TCP port 32846 on 10.0.10.2, to TCP port 4444 on 10.0.20.2:

![](/blog/content/images/2017/01/protocol-stack-application-1.gif)


When you've finished this experiment, please delete your resources on the GENI Portal to free them up for other experimenters.

## Exercise 

As you know, that data (the "Hello" and "Hi" messages in the image above) will have traversed multiple layers of the network stack, each of which has its own address or identifier. As an exercise, you may create an image in which you identify the relevant address at each layer for the data communication in _your own_ execution of the experiment shown above.

Using the output of the commands you executed in this experiment, can you fill in the blank lines in the image below with the address or port at each layer of the TCP/IP stack for your `netcat` data communication?

![](/blog/content/images/2017/02/Lab-0--Protocol-stack.svg)

To make an editable copy of the image above in Google Drawings, click on the following link while signed in to a Google or Google Apps account:

[https://docs.google.com/drawings/d/14L3I7HF69N1SN8kzlW7qjN47MzKiwHxemYqoSFbxk6o/edit](https://docs.google.com/drawings/d/14L3I7HF69N1SN8kzlW7qjN47MzKiwHxemYqoSFbxk6o/edit)

and click on File > Make a copy. You can then add text boxes to fill in the missing values directly on the lines.


