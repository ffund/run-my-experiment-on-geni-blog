In these exercises, you will observe how routing principles apply when packets are forwarded by a router from one network segment to the next.

It should take about 60-120 minutes to run this experiment.

You can run this experiment on GENI or on CloudLab. Refer to the testbed-specific prerequisites listed below.


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

## Background

In this experiment, you will add configure static routes on the hosts and routers in the experiment, in order to observe how they are applied.

To understand IPv4 routes, though, we first need to understand IPv4 addressing!

### IPv4 address and subnet mask or prefix length

When a network interface is configured with an IPv4 address, it will also be configured to be part of a "subnet". The interface will be assigned:

* an IPv4 address, that should be the destination address of any packet sent to this interface. This address has 32 bits, and it is usually written in dotted decimal notation with four "octects" separated by a `.` - for example, the dotted decimal address `10.9.8.7` is:

```
00001010.00001001.00001000.00000111
```

* and a subnet mask (or equivalently, a prefix length). A subnet mask is also 32 bits, but it is always organized with 1 bits on the left and 0 bits on the right. The number of 1 bits is called the prefix length. For example, the subnet mask `255.255.255.0` is equivalent to the prefix length **24**, because it has 24 1 bits:

```
11111111.11111111.11111111.00000000
```

When we AND the IPv4 address and the subnet mask, we get a special "address" called the *network address*. All devices in the same network segment (not separated by a router) will have the same network address. When configuring routes in a routing table, we'll usually use a network address (so that the route will apply to every address in the network segment!) rather than a host-specific address.

For the device above with IPv4 address `10.9.8.7` and subnet mask `255.255.255.0`, the network address would be `10.9.8.0`.

### Routing tables

When a host sends a packet, or when a router forwards a packet, it first looks up the destination address of the packet in a routing table, to decide how it should be forwarded.

Every rule in the routing table has:

* a destination address
* a subnet mask or prefix length
* and either the G flag is set to 1 and there is a gateway address, or the G flag is set to 0 and there is no gateway address.

The first two parts of the rule, the address and subnet mask, determine which rules "match" a particular packet's destination address. For example, consider a packet with destination address `10.9.8.7` and the routing table 

<pre>
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.0        10.20.30.40     255.255.0.0     UG    600    0        0 eth0
10.0.0.0        10.20.30.1      255.0.0.0       UG    600    0        0 eth0
10.20.30.0      0.0.0.0         255.255.255.0   U    600    0        0 eth0
</pre>

To decide whether a routing table rule "matches" the packet's destination address, we compute the AND of the packet's destination address and the subnet mask (from the `Genmask` column) in the routing rule. Then, we compare the result to the *rule's* destination address.

In this example, for the packet destination address `10.9.8.7`,

1. ❌ `10.9.8.7 &  255.255.0.0 = 10.9.0.0` which does not match `10.0.0.0`
2. ✅ `10.9.8.7 &  255.0.0.0 = 10.0.0.0` which *does* match `10.0.0.0` 
3. ❌ `10.9.8.7 &  255.255.255.0 = 10.9.8.0` which does not match `10.20.30.0` 

so the second rule matches.

The gateway part of the rule determines how the packet will be forwarded, once we have decided which rule will apply. Recall that the network layer of the TCP/IP protocol stack is responsible for forwarding a packet across networks, but the link layer which is immediately below it is only responsible for delivering a packet within a network. Therefore, to pass a packet down to the link layer, the network layer must determine which is the "next hop" in *the same network* that should be the link layer destination for the packet.

If the routing table rule that applies has G=0 and no gateway address, that means that the destination address is in the *same network* as the host or router, and the link layer can send it directly to its destination. (The link layer will use the destination's MAC address in the link layer header.)

If the routing table rule that applies has G=1 and a gateway address, that means that the destination address is *not* in the same network as the host or router, so it will be forwarded to a "gateway" that is the "next hop" *in the same network*. (The link layer will use the gateway's MAC address in the link layer header.)

### Longest prefix matching

It's possible for multiple routing table rules to match a particular packet's destination address. For example, consider a packet with destination address `10.9.8.7` and the routing table 

<pre>
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.20.30.40     0.0.0.0         UG    600    0        0 eth0
10.9.0.0        10.20.30.2      255.255.0.0     UG    600    0        0 eth0
10.0.0.0        10.20.30.1      255.0.0.0       UG    600    0        0 eth0
10.20.30.0      0.0.0.0         255.255.255.0   U    600    0        0 eth0
</pre>

In this example, for the packet destination address `10.9.8.7`,

1. ✅ `10.9.8.7 &  0.0.0.0 = 0.0.0.0` which *does* match `0.0.0.0`! (This rule, with netmask `0.0.0.0` is a special rule called a "default route" because it matches every possible destination address.)
2. ✅ `10.9.8.7 &  255.255.0.0 = 10.9.0.0` which *does* match `10.9.0.0`!
3. ✅ `10.9.8.7 &  255.0.0.0 = 10.0.0.0` which *does* match `10.0.0.0`!

When there are multiple matching rules, the one with the longest prefix length is the one that is applied to the packet. In this case, the prefix length of each rule is:

1. 0 for the first rule (subnet mask was `0.0.0.0`)
2. 16 for the second rule (subnet mask was `255.255.0.0`)
3. 8 for the third rule (subnet mask was `255.0.0.0`)

so the second rule, with prefix length 16, will be applied, and the packet will be forwarded via the gateway `10.20.30.2`.


### Configuring a static route

There are two Linux utilities that are widely used to add static routes: `route` and `ip`. The `route` utility is a bit simpler to use, and is older. The `ip` utility is newer and more powerful. I will show you how to use both of these to add a static route.


**Method A**: For the `route` command, the syntax is as follows, but you will have to **substitute the correct values** for all of the words in capital letters:

```
sudo route add -net NETADDR netmask NETMASK gw GW
```

where,

* in place of `NETADDR` you should put the network address of the subnet that you want to add a route for,
* in place of `NETMASK` you should put the netmask of the subnet that you want to add a route for,
* in place of `GW`, you should put the IP address of the next-hop router that is in the *same* subnet as the host that you're running this command on,

This rule says that: "All traffic for destinations in the network defined by `NETADDR` and `NETMASK` should be forwarded to `GW` (a "local" address)."

**Method B**: Note that instead of the `netmask` notation, you can alternatively use prefix length notation:


```
sudo route add -net NETADDR/PREFIX gw GW
```

where,

* in place of `NETADDR` you should put the network address of the subnet that you want to add a route for,
* in place of `PREFIX` you should put the prefix length of the subnet that you want to add a route for,
* in place of `GW`, you should put the IP address of the next-hop router that is in the *same* subnet as the host that you're running this command on,

This rule says that: "All traffic for destination addresses where the first `PREFIX` bits match `NETADDR` should be forwarded to `GW` (a "local" address)."

**Method C**: Alternatively, you can use the `ip` command as follows:

```
sudo ip route add NETADDR/PREFIX via GW
```

where,

* in place of `NETADDR` you should put the network address of the subnet that you want to add a route for,
* in place of `PREFIX` you should put the prefix length of the subnet that you want to add a route for,
* in place of `GW`, you should put the IP address of the next-hop router that is in the *same* subnet as the host that you're running this command on,

This rule says that: "All traffic for destination addresses where the first `PREFIX` bits match `NETADDR` should be forwarded to `GW` (a "local" address)."




## Run my experiment


### Reserve resources

For this experiment, we will use a topology with five hosts, three routers, and four network segments, with addresses on each network interface configured as follows:

![](https://raw.githubusercontent.com/ffund/tcp-ip-essentials/gh-pages/lab3/static-routing-topo.svg)

each with a netmask of 255.255.255.0.

Follow the instructions for the testbed you are using (GENI or Cloudlab) to reserve the resources and log in to each of the hosts in this experiment. 


<div style="border-color:#FB8C00; border-style:solid; padding: 15px;">

<h4 style="color:#FB8C00;"> GENI-specific instructions: Reserve resources</h4>


<p>To set up this topology in the GENI Portal, create a slice, click on "Add Resources", and load the RSpec from the following URL: </p>

<p><a href="https://raw.githubusercontent.com/ffund/tcp-ip-essentials/gh-pages/rspecs/static-routing.xml">https://raw.githubusercontent.com/ffund/tcp-ip-essentials/gh-pages/rspecs/static-routing.xml</a></p>

<p>Refer to the <a href="https://fedmon.fed4fire.eu/overview/instageni">monitor website</a> to identify an InstaGENI site that has many "free VMs" available. Then bind to an InstaGENI site and reserve your resources. Wait for them to become available for login ("turn green" on your canvas) and then SSH into each, using the details given in the GENI Portal.</p>

<p>Use <code>ifconfig</code> to view the network interface configuration on each host, and save the output for your own reference.</p>

</div>

<br>

<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">

<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Reserve resources</h4>

<p>To reserve resources on Cloudlab, open this profile page: </p>

<p>https://www.cloudlab.us/p/nyunetworks/education?refspec=refs/heads/static_routing</p>

<p>Click "next", then select the Cloudlab project that you are part of and a Cloudlab cluster with available resources. (This experiment is compatible with any of the Cloudlab clusters.) Then click "next", and "finish".</p>

<p>Wait until all of the sources have turned green and have a small check mark in the top right corner of the "topology view" tab, indicating that they are fully configured and ready to log in. Then, click on "list view" to get SSH login details for the client, router, and server hosts, and SSH into each.</p>

<p>Use <code>ifconfig</code> to view the network interface configuration on each host, and save the output for your own reference.</p>

</div>
<br>


### Exercises in a single segment network

You may have assumed that a routing table is only relevant when there are routers connecting multiple networks; but routing tables are also used at end hosts to determine how to send packets, even on a single segment!

Before sending any packet, a host _first_ checks its routing table, and determines whether the route that matches this destination address is a **directly connected** route.

A **directly connected** route is one where:

* the `G` flag is _not_ set (indicating that this route does not involve any gateway through which to forward)
* there is no gateway specified

When a network interface on a device is configured with an IP address and subnet mask, a directly connected route is *automatically* added to the routing table on that device. This automatic rule applies to all destination addresses in the same subnet as the network interface.

On "romeo", run

```
route -n
```

to see the current routing table. Save this output.

Then, on "romeo", use `ping` to send an ICMP echo request to the IP address 10.10.0.101 ("juliet"):


```
ping -c 1 10.10.0.101
```

and note the response. Refer back to the routing table you saw earlier. What rule in the routing table applies to this destination address?

Now, on "romeo", use `ping` to send an ICMP echo request to an IP address for which there is no route on in the routing table:

```
ping -c 1 10.10.99.101
```

and note the response.



### Exercise on static routes for forwarding between networks

Now, we will observe how routing principles apply when packets are forwarded by a router from one network segment to the next.

In this exercise, we will focus on routing traffic between two hosts in different network segments: romeo and othello. You will need to open:

* one terminal window on the romeo host
* one terminal window on router-1
* one terminal window on router-2
* one terminal window on the othello host

We will attempt to send a packet from the "romeo" host to the "othello" host, and a response in the reverse direction. However, this exchange will succeed only after we have configured routes in both directions along the path between "romeo" and "othello".

#### Part 1: No added rules

On romeo, run

```
ping -c 5 10.10.1.104
```

to send an ICMP echo request to othello.

#### Part 2: Add a route on romeo

Because the routing table on romeo has no entry for othello's subnet, you will get a "Network is unreachable" message in the previous step. Let's add a rule for othello's subnet.

Use any of the methods described in the [Background](#background) section to add a route on **romeo** that will forward traffic to the **green network** through **router-1**. Save the command.


On romeo, run

```
ping -c 5 10.10.1.104
```

#### Part 3: Add a route on router-1

After adding a rule on **romeo**, an ICMP echo request for 10.10.1.104 should be forwarded to **router-1**. However, **router-1** has no rule in its routing table that applies to the destination address, so it sends an ICMP Destination Net Unreachable message back to **romeo**.

Use any of the methods described in the [Background](#background) section to add a route on **router-1** that will forward traffic to the **green network** through **router-2**. Save the command.

On romeo, run

```
ping -c 5 10.10.1.104
```

to send an ICMP echo request to othello.  


#### Part 4: set up the reverse path

After following the previous steps, the ICMP echo request from romeo arrives at othello via the following path: **romeo** &rarr; **router-1** &rarr; **router-2** &rarr; **othello**.

(We did not have to add any route on router-2 - because it has a network interface that is configured to be in the same subnet as othello's, the directly connected route will match the destination address 10.10.1.104.)

However, othello does not respond because it has no route to romeo! You will need to set up the *reverse* path from othello to romeo in a similar way.

Use any of the methods described in the [Background](#background) section to add a route on **othello** that will forward traffic to the **red network** through **router-2**. Save the command. 

Use any of the methods described in the [Background](#background) section to add a route on **router-2** that will forward traffic to the **red network** through **router-1**. Save the command. 

(We did not have to add any route on router-1 - because it has a network interface that is configured to be in the same subnet as romeo's, the directly connected route will match the romeo's address.)


Then, get all the routing table rules. On romeo, othello, router-1 and router-2, run


```
route -n
```

and save these outputs.

On romeo, run

```
ping -c 5 10.10.1.104
```

to send an ICMP echo request to othello. Save the command and output.


### Exercise on longest prefix matching

In the previous exercise, you added a rule to the routing table on router-1 that matches all destination addresses in the green network (10.10.1.0/24), and that uses router-2 as the next hop. (This rule should still be in router-1's routing table!)

Now, we will add more routes so that *multiple* rules in router-1's routing table match the destination address 10.10.1.104, and we'll see which rule is actually applied.

First, let's make sure that router-3 can also forward packets to the destination address 10.10.1.104. Add a rule on router-3 that uses othello's interface on the **purple network** as the next hop toward the green network:

```
sudo route add -net 10.10.1.0/24 gw 10.10.2.104
```

#### Part 1: 10.10.0.0/16

Add the following rule on router-1:

```
sudo route add -net 10.10.0.0/16 gw 10.10.100.3
```

This rule says that for all destination addresses matching 10.10.0.0/16, use router-3 as the next hop. Note that this rule matches othello's address, 10.10.1.104!

On router-1, run

```
route -n
```

and save the output. Note that there are now *two* rules that match othello's address, 10.10.1.104. One rule says to use router-2 as the next hop and one rule says to use router-3 as the next hop.

We will use `tcpdump` to monitor the networks and see which rule is actually applied.

Start a `tcpdump` on both interfaces of router-2 and on both interfaces of router-3:

```
sudo tcpdump -env -i eth1 icmp
```

and

```
sudo tcpdump -env -i eth2 icmp
```

Make sure you know which `tcpdump` shows the interface on the **blue network** and which shows the interface on the **green network** or the **purple network**. (Use `ifconfig` to identify which interface of each router is on which network.)

Now, on romeo, ping othello:

```
ping -c 1 10.10.1.104
```

Stop the `tcpdump` processes and save the output.


#### Part 2: 10.10.1.96/28

Add the following rule on router-1:

```
sudo route add -net 10.10.1.96/28 gw 10.10.100.3
```

This rule says that for all destination addresses matching 10.10.1.96/28, use router-3 as the next hop. Note that this rule also matches othello's address, 10.10.1.104!

On router-1, run

```
route -n
```

and save the output. Note that there are now *three* rules that match othello's address, 10.10.1.104. 

Start a `tcpdump` on both interfaces of router-2 and on both interfaces of router-3:

```
sudo tcpdump -env -i eth1 icmp
```

and

```
sudo tcpdump -env -i eth2 icmp
```

Make sure you know which `tcpdump` shows the interface on the **blue network** and which shows the interface on the **green network** or the **purple network**.

Now, on romeo, ping othello:

```
ping -c 1 10.10.1.104
```

Stop the `tcpdump` processes and save the output.


## Questions

#### Exercises in a single segment network

* In the routing table on "romeo", show the rule that applies to traffic that is sent within the same subnet. (This rule is added to the routing table automatically when you configure the IP address and netmask on the network interface.) 

* What happened when you tried to send an IP packet to an address that does not match any rule in the routing table on "romeo"?

#### Exercise on static routes for forwarding between networks


For each of the four parts of this experiment,

* Show the route(s) that you added (if any). 
* Show the result of the `ping` command.

At the end of the four parts of this exercise, show the routing table on romeo, othello, router-1, and router-2. On which hosts and routers is there a rule that matches destination addresses in the **green network**?  On which hosts and routers is there a rule that matches destination addresses in the **red network**? 

#### Exercise on longest prefix matching


Use your `tcpdump` output to answer the following questions about Part 1 and Part 2 of this exercise:

* In your `tcpdump` output from the **green network**, can you see the ICMP echo *request* packet - was it forwarded to othello by router-2? Or do you see it on the **purple network**, because it was forwarded to othello by router-3? 
* Which of the rules in router-1's routing table matches the destination address 10.10.1.104 (there should be multiple rules that match)? Which *one* rule was applied in this case, and why?
