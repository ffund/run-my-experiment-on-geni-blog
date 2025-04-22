In this experimental demonstration of the TCP/IP protocol architecture, we will examine network addresses and connections at

* the network access (a.k.a. data link) layer,
* the Internet (IP) layer, 
* the transport layer (logical host-to-host), and
* the application layer.


It should take about 60 minutes to run this experiment.

You can run this experiment on CloudLab, FABRIC, or Chameleon. Refer to the testbed-specific prerequisites listed below.

<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">
<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Prerequisites</h4>

To reproduce this experiment on Cloudlab, you will need an account on <a href="https://cloudlab.us/">Cloudlab</a>, you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._join-project%29">joined a project</a>, and you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._ssh-access%29">set up SSH access</a>, as described in <a href="https://teaching-on-testbeds.github.io/blog/hello-cloudlab">Hello, CloudLab</a>.

</div>
<br>

<div style="border-color:#47aae1; border-style:solid; padding: 15px;">  
<h4 style="color:#47aae1;">FABRIC-specific instructions: Prerequisites</h4>
To run this experiment on <a href="https://fabric-testbed.net/">FABRIC</a>, you should have a FABRIC account with keys configured, and be part of a FABRIC project, , as described in <a href="https://teaching-on-testbeds.github.io/blog/hello-fabric">Hello, FABRIC</a>. 

</div>  
<br>

<div style="border-color:#9ad61a; border-style:solid; padding: 15px;">  
<h4 style="color:#9ad61a;">Chameleon-specific instructions: Prerequisites</h4>
To run this experiment on <a href="https://chameleoncloud.org/">Chameleon</a>, you should have a Chameleon account with keys configured on KVM@TACC, and be part of a Chameleon project, as described in <a href="https://teaching-on-testbeds.github.io/blog/hello-chameleon">Hello, Chameleon</a>. 

</div>  
<p><br></p>

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
 
* Using the interface between the application layer and transport layer (the [socket API](https://en.wikipedia.org/wiki/Network_socket)), Host A will establish a connection to Application X on Host B using Host B's **IP address** (network layer address) and the **TCP port number** (transport layer identifier) that Application X is using.  (When packets are received at host B, the transport layer will look at the destination port number in the TCP header to identify which application to  deliver it to. )
* At the network layer, Host A, Host B, and Router J will determine which link to send packets on according to the destination **IP address** in the packet's IP header. (For example, Router J is connected to two networks; it uses the destination IP address of a packet to determine whether to send it out on Network 1 or Network 2.)
* The **MAC address** identifies the "local" destination of a packet (i.e. the destination that is on the same network). For example, Host A, which is on Network 1, sends data to Host B through Router J, which is also on Network 1. The destination MAC address in the link layer header of packets from Host A to Host B will be Router J's MAC address.

In this experiment, we will establish a connection like the one in the image above, and examine the addresses and identifiers used at each layer of the protocol stack.

It's also useful to understand where each part of the network protocol stack is implemented. On a host, the different layers of the TCP/IP protocol stack "live" in various places - 

![](/blog/content/images/2023/09/network-stack-hosts.svg)

* Most applications "live" in user space, and interact with lower layers through the socket API. 
* The transport layer is implemented inside the operating system kernel space, as is the network layer.
* The link layer is partly in the operating system kernel space, and partly in the firmware of the network interface card. The NIC is a specific hardware component that connects the host to a physical network. A device driver for this hardware component is the interface between the host operating system and the NIC.
* The physical layer is implemented inside the NIC. 

Also as part of this experiment, we will try to develop some insight into how these pieces fit together within a host.

## Run my experiment

First, reserve your resources.

For this experiment, we will reserve three VMs and connect them with links, as shown below. We will also assign an IP (Internet layer) address to each VM on each network that it is connected to.

![](/blog/content/images/2023/09/line-addresses-compat.svg)

Follow the testbed-specific instructions to reserve resources in this configuration.

<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">
<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Reserve resources</h4>

<p>To reserve these resources on Cloudlab, open this profile page:</p>

<p><a href="https://www.cloudlab.us/p/nyunetworks/education?refspec=refs/heads/line_tso_compat">https://www.cloudlab.us/p/nyunetworks/education?refspec=refs/heads/line_tso_compat</a></p>

<p>Click "next", then select the Cloudlab project that you are part of and a Cloudlab cluster with available resources. (This experiment is compatible with any of the Cloudlab clusters.) Then click "next", and "finish".</p>

<p>Wait until all of the sources have turned green and have a small check mark in the top right corner of the "topology view" tab, indicating that they are fully configured and ready to log in. Then, click on "list view" to get SSH login details for the two end hosts and the router. Use these details to SSH into each one. (Or, you can right-click and use the "shell" menu option to open a terminal session in the browser.)</p>

<p>When you have logged in to each node (romeo, juliet, and router), continue to the <a href="#viewnetworkinterfaces">View Network Interfaces</a> section.</p>


</div>
<br>


<div style="border-color:#47aae1; border-style:solid; padding: 15px;">
<h4 style="color:#47aae1;">FABRIC-specific instructions: Reserve resources</h4>
<p>To run this experiment on <a href="https://fabric-testbed.net/">FABRIC</a>, open the JupyterHub environment on FABRIC, open a shell, and run </p>

<pre>
git clone https://github.com/teaching-on-testbeds/fabric-education stack
cd stack
git checkout stack
</pre>
<p>Then open the notebook titled "start_here.ipynb".</p>
<p>Follow along inside the notebook to reserve resources, configure them, and get the login details for each host in the experiment.</p>
<p>When you have logged in to each node (romeo, juliet, and router), continue to the <a href="#viewnetworkinterfaces">View Network Interfaces</a> section.</p>
</div>
<br>



<div style="border-color:#9ad61a; border-style:solid; padding: 15px;">
<h4 style="color:#9ad61a;">Chameleon-specific instructions: Reserve resources</h4>
<p>To run this experiment on <a href="https://chameleoncloud.org/">Chameleon</a>, open the JupyterHub environment on Chameleon, open a shell, and run </p>

<pre>
cd work
git clone https://github.com/teaching-on-testbeds/chameleon-education stack
cd stack
git checkout stack
</pre>
<p>Then open the notebook titled "start_here.ipynb".</p>
<p>Follow along inside the notebook to reserve resources, configure them, and get the login details for each host in the experiment.</p>
<p>When you have logged in to each node (romeo, juliet, and router), continue to the <a href="#viewnetworkinterfaces">View Network Interfaces</a> section.</p>
</div>
<br>


### View network interfaces

A network interface is the point of interconnection between a computer and a network. It may represent a hardware network interface card (NIC), such as a WiFi network card, or a virtualized NIC that is realized in software.

To see the NICs on a host, and the addresses (at the link layer and at the network layer) associated with each NIC, we can use the `ip` utility. Let's try the `ip` utility - on the "romeo" node, run `ip addr` to see the addresses of each of the network interfaces on this host:

<pre>
1: lo: &langle;LOOPBACK,UP,LOWER_UP&rangle; mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: &langle;BROADCAST,MULTICAST,UP,LOWER_UP&rangle; mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:25:c9:f2:9f:f6 brd ff:ff:ff:ff:ff:ff
    inet 128.110.223.24/21 metric 1024 brd 128.110.223.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::25:c9ff:fef2:9ff6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: &langle;BROADCAST,MULTICAST,UP,LOWER_UP&rangle; mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:0b:7d:94:87:d2 brd ff:ff:ff:ff:ff:ff
    inet 10.0.1.100/24 brd 10.0.1.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::b:7dff:fe94:87d2/64 scope link 
       valid_lft forever preferred_lft forever
</pre>

This host has three network interfaces, numbered 1, 2, and 3. The loopback interface (named `lo`) is a virtual network interface that the computer uses for processes on the same host to communicate with one another using network protocols. The two Ethernet interfaces (named `eth0` and `eth1`) represent two points of attachment to physical networks: in this case, 

* the `eth0` interface is attached to the public Internet (which you need in order to SSH in to your system from outside!). This interface is known as the *control* interface,
* and the `eth1` interface is attached to the private link that you've set up between the VMs in your topology.  This is an *experiment* interface.

For the rest of this experiment, we will focus on the "private" networks that we have configured, and the interfaces connected to these (such as `eth1` in the example above).

For each network interface, we can also see the MAC address (at the network access/data link layer), e.g.:

```
link/ether 02:25:c9:f2:9f:f6
```

and the IPv4 (Internet layer) address:

```
inet 10.0.1.100/24
```

(this indicates that the IPv4 address is 10.0.1.100. The integer value following the `/` is a prefix length.) 

(We can use the IPv4 address to distinguish between the *control* interface and *experiment* interfaces - the *experiment* interface(s) will have been assigned a specific address by us, as part of the experiment setup!)


If we run `ip addr` on the "router", we will see an additional network interface, since we set up this node to connect to _two_ private links:

<pre>
1: lo: &langle;LOOPBACK,UP,LOWER_UP&rangle; mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: &langle;BROADCAST,MULTICAST,UP,LOWER_UP&rangle; mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:0b:cd:ad:73:a8 brd ff:ff:ff:ff:ff:ff
    inet 128.110.223.25/21 metric 1024 brd 128.110.223.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::b:cdff:fead:73a8/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: &langle;BROADCAST,MULTICAST,UP,LOWER_UP&rangle;mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:da:a6:d5:3d:28 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.10/24 brd 10.0.2.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::da:a6ff:fed5:3d28/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: &langle;BROADCAST,MULTICAST,UP,LOWER_UP&rangle; mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:14:65:31:01:5b brd ff:ff:ff:ff:ff:ff
    inet 10.0.1.10/24 brd 10.0.1.255 scope global eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::14:65ff:fe31:15b/64 scope link 
       valid_lft forever preferred_lft forever
</pre>


### Address tables at the network access (data link) layer

As traffic moves through the network, hosts become aware of the MAC addresses of neighbors *on the same network*. For each network interface (point of attachment to a network), it will store a table of MAC addresses and the corresponding IP addresses of hosts that are attached to that same network.

First, generate some traffic to ensure that all the local address tables are updated. On the "romeo" node, run

```
ping -c 3 10.0.2.100
```

to send some traffic to "juliet" (via the router).

Then, on romeo, run the following command, but **in place of `XXX` substitute the interface name you identified above for the experiment interface**:

```
ip neigh show dev XXX
```

For example:

```
ffund00@romeo:~$ ip neigh show dev eth1
10.0.1.10 lladdr 02:14:65:31:01:5b REACHABLE
```

Note that "romeo" knows the MAC address of one interface of the "router", which is on the same physical network as itself, but not the address of "juliet", which is on a different network. The MAC address is an identifier at the link layer, which is only concerned with communication within a physical network. 

On the router, run the `ip neigh show` command twice, specifying a different experiment interface each time.

Note that on the "router", we can see that it is aware of the MAC address of the "romeo" node on one interface and the MAC address of the "juliet" node on another interface:

```
ffund00@router:~$ ip neigh show dev eth1
10.0.2.100 lladdr 02:73:6d:88:f0:dc REACHABLE
ffund00@router:~$ ip neigh show dev eth2
10.0.1.100 lladdr 02:0b:7d:94:87:d2 REACHABLE
```

Like "romeo", "juliet" knows the MAC address of one of the "router" interface that is on the same network as itself, but not any MAC addresses of interfaces on the other network. On "juliet", run the `ip neigh show` command (with the appropriate interface name for *your* experiment) to verify this.

```
ffund00@juliet:~$ ip neigh show dev eth1
10.0.2.10 lladdr 02:da:a6:d5:3d:28 REACHABLE
```

### Moving between networks at the Internet (IP) layer

The Internet layer is concerned with moving data between _different_ networks. Since our "romeo" and "juliet" are connected to two different networks, traffic between them is relayed through the "router".

To see how traffic is routed from "romeo" to "juliet", we will use the `traceroute` command. On "romeo", run

```
traceroute -n 10.0.2.100
```

and observe the results. You should see an intermediate hop through the router, like this (you may have to run it twice):

```
ffund00@romeo:~$ traceroute -n 10.0.2.100
traceroute to 10.0.2.100 (10.0.2.100), 30 hops max, 60 byte packets
 1  10.0.1.10    0.603 ms  0.563 ms  0.521 ms
 2  10.0.2.100   0.908 ms  0.834 ms  0.761 ms
```

Similarly, on "juliet", we can see how traffic goes through the "router" to reach "romeo":

```
ffund01@server:~$ traceroute -n 10.0.1.100
traceroute to 10.0.1.100 (10.0.1.100), 30 hops max, 60 byte packets
 1  10.0.2.10  0.600 ms  0.578 ms  0.562 ms
 2  10.0.1.100  0.922 ms  0.881 ms  0.861 ms
```

If we stop the "router" from forwarding traffic between networks, then traffic from "romeo" will no longer reach "juliet". Let's try it. We use the `sysctl` utility on Linux to configure the host operating system settings, including the settings of the network layer (IPv4) or the transport layer.

On the "router" run the following to stop forwarding: 

```
sudo sysctl -w net.ipv4.ip_forward=0
```

and then run the `traceroute` on "romeo" again. 

This time, you should see only `*`s, denoting no route to the target:

```
ffund00@romeo:~$ traceroute -n 10.0.2.100
traceroute to 10.0.2.100 (10.0.2.100), 30 hops max, 60 byte packets
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

Without the router forwarding traffic at the Internet layer, hosts can communicate within their own network, but not across networks. Try the following at "romeo":

```
# contact router, in same network - should work
ping -c 3 10.0.1.10 
# contact juliet, in another network - will not work
ping -c 3 10.0.2.100 
```

Restore packet forwarding on the router by running the following command on the "router" node:


```
sudo sysctl -w net.ipv4.ip_forward=1
```

and verify that now traffic can move across networks again:

```
# both in-network and out-of-network traffic should work
ping -c 3 10.0.1.10
ping -c 3 10.0.2.100 
```


### Transport (host to host) and application layers

Finally, we will initiate a network connection at the application layer.

On "juliet", we will use the `netcat` application to receive incoming connections on TCP port 4444:

```
netcat -l 4444
```

and on "romeo", we will initiate an application-layer connection to "juliet" on that same port:

```
netcat 10.0.2.100 4444
``` 

While it will initially appear as if nothing has happened, you can type anything into either window, hit "Enter", and see it appear in the other.

To see the transport-layer port used on each host, we can use `ss`, which informs us about **s**ocket **s**tatistics. (We already know that "juliet" is using TCP port 4444, because that's what we instructed it to use; but "romeo" will be using a random TCP port number).

While the `netcat` link is still active, open two more terminal windows and SSH into "romeo" and "juliet", respectively. On romeo, to see information about the socket that is connected to destination address 10.0.2.100 ("juliet"'s address), run

```
ss -p dst 10.0.2.100
```

The output below shows that in my network, the connection is from TCP port 58488 on 10.0.1.100, to TCP port 4444 on 10.0.2.100:

<pre>
ffund00@romeo:~$ ss -p dst 10.0.2.100
Netid       State       Recv-Q       Send-Q    Local Address:Port    Peer Address:Port       Process       
tcp         ESTAB       0            0         10.0.1.100:58488      10.0.2.100:4444         users:(("netcat",pid=2866,fd=3)) 
</pre>

and that this socket is associated with the application named "netcat" which is running with process ID 2866 on "romeo", and the socket uses file descriptor number 3. Similarly, you can run on "juliet"

```
ss -p dst 10.0.1.100
```

and observe the output.

### Delete resources

When you've finished this experiment, don't forget to delete your resources to free them up for other experimenters!

## Exercise 

As you know, that data will have traversed multiple layers of the network stack, each of which has its own address or identifier. As an exercise, you may create an image in which you identify the relevant address at each layer for the data communication in _your own_ execution of the experiment shown above.

Using the output of the commands you executed in this experiment, can you fill in the blank lines in the image below with the address or port at each layer of the TCP/IP stack for your `netcat` data communication?

![](/blog/content/images/2022/09/Lab-0-Protocol-stack-2.svg)

To make an editable copy of the image above in Google Drawings, click on the following link while signed in to a Google or Google Apps account:

[https://docs.google.com/drawings/d/14L3I7HF69N1SN8kzlW7qjN47MzKiwHxemYqoSFbxk6o/edit](https://docs.google.com/drawings/d/14L3I7HF69N1SN8kzlW7qjN47MzKiwHxemYqoSFbxk6o/edit)

and click on File > Make a copy. You can then add text boxes to fill in the missing values directly on the lines.

Also show the `ip addr` and `ss` command outputs from *your* experiment, that shows how you found these addresses and identifiers.