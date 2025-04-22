Most Internet-connected homes use a home network gateway to connect a local area network (LAN) in the home, to a wide area network (WAN) such as the Internet. In this experiment, we will look at some of the services that are always or often included on these gateways:

* routing between the LAN and WAN, 
* DHCP, to provide hosts in the home network with IP addresses,
* DNS, to respond to name resolution queries from hosts in the home network,
* NAT (Network Address Translation), to map one public IPv4 address to internal (private) IP addresses assigned to hosts on the home network.

We will also take a quick look at the HTTP protocol, so that by the end of this experiment, you should understand all of the protocol exchanges involved in: turning on a computer that is connected to a home gateway, opening a browser, and visiting a website!

It should take about 120-180 minutes to run this experiment. 

This experiment runs on CloudLab! 

<!--Whenever there are testbed-specific instructions, follow the ones for the testbed *you* are using.-->


<div class="cloudlab-specific">
<h4 class="cloudlab-specific"> Cloudlab-specific instructions: Prerequisites</h4>

To reproduce this experiment on Cloudlab, you will need an account on <a href="https://cloudlab.us/">Cloudlab</a>, you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._join-project%29">joined a project</a>, and you will need to have <a href="https://docs.cloudlab.us/users.html#%28part._ssh-access%29">set up SSH access</a>. You may want to try <a href="https://teaching-on-testbeds.github.io/hello-cloudlab/">Hello, CloudLab</a> if you have not yet completed those steps.

</div>
<br>

<div style="border-color:#47aae1; border-style:solid; padding: 15px;">  
<h4 style="color:#47aae1;">FABRIC-specific comments</h4>  
<p>This experiment is not supported on the FABRIC testbed. It requires a public IP address, and FABRIC's security policy does not permit FABNetExt services in slices managed by students.</p>
</div>  
<br>

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

A home or residential gateway is a device that sits between a local area network (LAN) in a home, and a wide area network (WAN) such as the Internet.

![](/blog/content/images/2017/03/gateway-typical.svg)

At a minimum, it routes traffic between the LAN and the WAN. In many cases, it provides _additional_ services that are useful for a home LAN, including three that are the subject of this experiment:

* DHCP, to provide hosts in the home network with IP addresses,
* DNS, to respond to name resolution queries from hosts in the home network,
* NAT (Network Address Translation), to map one public IPv4 address to internal (private) IP addresses assigned to hosts on the home network.

#### DHCP

The Dynamic Host Configuration Protocol (DHCP) is a protocol for dynamically distributing IP addresses to hosts in a network. This eliminates the need for a user or network administrator to configure IP address settings manually on every host. It also allows a "pool" of IP addresses to be shared among many hosts; if a host leaves the network, the IP address it was using can be reassigned to someone else just entering the network.

The process for getting an IP address with DHCP involves four phases known as DORA - discovery, offer, request, and acknowledgement - as shown in the following figure:

![](/blog/content/images/2017/03/dhcp-transaction.svg)

1. First, the client broadcasts a DHCP DISCOVER message.
2. Local DHCP server(s) receive this message, and can respond with a DHCP OFFER that is unicast to the client. This OFFER message "offers" an available IP address to the client. It may also include other "options", such as the address of a gateway or nameserver that the client should use. The client may receive multiple offers, if there are multiple DHCP servers on the LAN.
3. The client responds to _one_ offer with a REQUEST for the IP address that was offered to the client. This REQUEST message is broadcast on the LAN, so if another DHCP server also sent an OFFER, it will receive this REQUEST message and consider it as notification that the client has declined that server's offer.  
4. The DHCP server whose OFFER was followed up with a REQUEST will acknowledge the client with an ACK. 

After these four stages, the client is configured with an IP address. As long as it is still using that IP address, it will renew it at regular intervals (when the "lease" expires), using a REQUEST+ACK. If the client leaves the network, it can send a RELEASE message to alert the DHCP server that it is relinquishing the address, and it may be assigned to someone else.

A typical home gateway will come with a DHCP server built-in, and allow you to select the range of IP addresses it distributes, as well as other DHCP options, e.g.:

![](/blog/content/images/2017/03/dhcp-settings.png)

### DNS

For web browsing and similar use cases, hosts on the Internet are referred to by human-readable names, rather than by their IP addresses. The Domain Name System (DNS) is a system that stores records mapping those names to the corresponding IP address, and allows those records to be queried (name resolution).

Many gateways also act as a DNS server, i.e. they can initiate and sequence the queries that ultimately lead to the resolution of a hostname into an IP address. 

A DNS server may follow an iterative query procedure, in which it queries a chain of DNS servers. Each server in the chain refers server that initiates the query to the next server in the chain, until it reaches one that can fully resolve the requested name.

For example, consider a DNS server that wants to resolve the name

```
witestlab.poly.edu.
```

(Note: all "full" DNS domain names have a dot at the end, although DNS resolvers will still work if you omit the dot. [This answer](http://webmasters.stackexchange.com/a/73946) explains why.) 

Suppose the DNS server does not know the IP address associated with this name, and also does not know the address of a DNS server that _does_ know the address associated with this name. The following figure describes the iterative procedure it will follow: 

![](/blog/content/images/2017/03/dns-iterative-messages.svg)


* It will start to query other name servers iteratively, starting with the name servers responsible for the rightmost part of the name, the `.`. (All DNS servers know the address of some name servers responsible for `.`, called _root_ domain servers.) 
* A root server won't know the address of `witestlab.poly.edu.`, but it will respond with a list of name servers that are responsible for `.edu.`, which is a _top level_ domain.
* A `.edu.` name server also won't know the address of `witestlab.poly.edu.`, but it will respond with a list of name servers that are responsible for `poly.edu.` (a _second level_ domain).
* Finally, a name server that is responsible for `poly.edu.` is likely to know the address of `witestlab.poly.edu.`, a _subdomain_ and will respond with an address record.

Thus, any DNS server that knows the address of some root servers can eventually find out the IP address associated with any name.

The hierarchy of the name servers in this example (and examples of other name servers at similar levels) is shown in the following figure:

![](/blog/content/images/2017/03/dns-iterative-2.svg)


### NAT

Another common feature of residential gateways is NAT, Network Address Translation. This is a method of method of remapping one IP address space into another by modifying source and destination IP addresses in packet headers as they pass through the gateway. 

Publicly routable IPv4 addresses, which are scarce, are not generally assigned to every host in a home network. Instead, the Internet Service Provider will provide _one_ IP address, which is assigned to the Internet-facing interface of the gateway. Within the LAN, hosts will use a private IP address space, such as the 192.168.0.0/16 space. However, those addresses are not routable on the Internet. Thus, a common NAT use case is to map between private addresses used on a home LAN and the public address associated with the router, so that traffic can be routed between the LAN and the Internet.

The following figure shows how a NAT gateway in this scenario will rewrite the headers of packets that traverse it:

![](/blog/content/images/2017/03/gateway-nat-2.svg)

1. A host on the LAN initiates a connection to a host outside the LAN, sending a packet through the NAT gateway. The packet has as its source address the private IP address of the host. 
2. The NAT gateway will add a new entry to a translation table, in which it notes the internal IP address and TCP port number associated with the connection (from the packet header). Then, it assigns this connection a port number. (In the diagram, it says that the WAN port will be randomly selected. This is sometimes, but not always, the case. If the source port number in the packet header is available at the NAT, and not already in use for another connection, it may use that port number. Otherwise, it will assign it a new one from a list of available port numbers.)
3. The NAT gateway rewrites the source IP address and possibly the port number in the packet header according to the translation table, then forwards it on the WAN.
4. When a response arrives at the NAT gateway from outside the LAN, the NAT gateway looks at the destination port in the packet header. Then, it finds the entry in the translation table that has this port number in the "WAN Port" column.
5. The NAT gateway rewrites the destination IP address and possibly the port in the packet header according to the "LAN IP" and "LAN Port" in the relevant entry of the translation table, then forwards it on the LAN.

An entry is added to the translation table only when a packet associated with a new connection arrives from the LAN. If a packet arrives at the NAT from the WAN whose destination port number does not match any "WAN Port" entry in the translation table, it will be dropped, because the NAT does not know how to translate it. Therefore, a device behind a NAT cannot accept connections from the WAN that it did not initiate. 

A workaround for this problem, [port forwarding](https://en.wikipedia.org/wiki/Port_forwarding), enables a home user to _proactively_ map a port to a particular host on the home network. 

For example, consider the home gateway configuration in the following image. Suppose the gateway has 128.238.66.220 as a public IP address. Hosts on the WAN can initiate a connection to 128.238.66.220 on port 443. The NAT will look up the internal IP address in the port forwarding table, rewrite the destination in the packet header from IP 128.238.66.220 and port 443, to IP 192.168.1.2 and port 443, and then forward the packet on the LAN. Similarly, a packet arriving from the WAN with destination IP 128.238.66.220 and port 3389 will be translated to IP 192.168.1.145 and port 3389.

![](/blog/content/images/2017/03/nat-port-forwarding-1.png)

## Results

In this experiment, we set up a gateway with clients connected to it, and observe the behavior of DHCP, DNS, and NAT services.

### DHCP

We see a client requesting and receiving an IP address from DHCP:

<pre>
15:15:26.554336 02:eb:60:57:10:ad > ff:ff:ff:ff:ff:ff, ethertype IPv4 (0x0800), length 342: (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
    0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from 02:eb:60:57:10:ad, length 300, xid 0xc0902a0f, Flags [none]
	  Client-Ethernet-Address 02:eb:60:57:10:ad
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: <b>Discover</b>
	    Hostname Option 12, length 10: "<hostname>"
	    Parameter-Request Option 55, length 13: 
	      Subnet-Mask, BR, Time-Zone, Default-Gateway
	      Domain-Name, Domain-Name-Server, Option 119, Hostname
	      Netbios-Name-Server, Netbios-Scope, MTU, Classless-Static-Route
	      NTP
</pre>

 
<pre>
15:15:29.058621 02:e6:fb:b7:f3:a5 > 02:eb:60:57:10:ad, ethertype IPv4 (0x0800), length 342: (tos 0xc0, ttl 64, id 37575, offset 0, flags [none], proto UDP (17), length 328)
    192.168.100.1.67 > 192.168.100.157.68: BOOTP/DHCP, Reply, length 300, xid 0xc0902a0f, Flags [none]
	  Your-IP 192.168.100.157
	  Server-IP 192.168.100.1
	  Client-Ethernet-Address 02:eb:60:57:10:ad
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: <b>Offer</b>
	    Server-ID Option 54, length 4: 192.168.100.1
	    Lease-Time Option 51, length 4: 14400
	    RN Option 58, length 4: 7200
	    RB Option 59, length 4: 12600
	    Subnet-Mask Option 1, length 4: 255.255.255.0
	    BR Option 28, length 4: 192.168.100.255
	    Domain-Name-Server Option 6, length 4: 192.168.100.1
	    Default-Gateway Option 3, length 4: 192.168.100.1
</pre>


<pre>
15:15:29.060208 02:eb:60:57:10:ad > ff:ff:ff:ff:ff:ff, ethertype IPv4 (0x0800), length 342: (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
    0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from 02:eb:60:57:10:ad, length 300, xid 0xc0902a0f, Flags [none]
	  Client-Ethernet-Address 02:eb:60:57:10:ad
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: <b>Request</b>
	    Server-ID Option 54, length 4: 192.168.100.1
	    Requested-IP Option 50, length 4: 192.168.100.157
	    Hostname Option 12, length 10: "<hostname>"
	    Parameter-Request Option 55, length 13: 
	      Subnet-Mask, BR, Time-Zone, Default-Gateway
	      Domain-Name, Domain-Name-Server, Option 119, Hostname
	      Netbios-Name-Server, Netbios-Scope, MTU, Classless-Static-Route
	      NTP
</pre>


<pre>
15:15:29.090184 02:e6:fb:b7:f3:a5 > 02:eb:60:57:10:ad, ethertype IPv4 (0x0800), length 342: (tos 0xc0, ttl 64, id 37576, offset 0, flags [none], proto UDP (17), length 328)
    192.168.100.1.67 > 192.168.100.157.68: BOOTP/DHCP, Reply, length 300, xid 0xc0902a0f, Flags [none]
	  Your-IP 192.168.100.157
	  Server-IP 192.168.100.1
	  Client-Ethernet-Address 02:eb:60:57:10:ad
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: <b>ACK</b>
	    Server-ID Option 54, length 4: 192.168.100.1
	    Lease-Time Option 51, length 4: 14400
	    RN Option 58, length 4: 7200
	    RB Option 59, length 4: 12600
	    Subnet-Mask Option 1, length 4: 255.255.255.0
	    BR Option 28, length 4: 192.168.100.255
	    Domain-Name-Server Option 6, length 4: 192.168.100.1
	    Default-Gateway Option 3, length 4: 192.168.100.1
</pre>

### DNS

We see an iterative name resolution with DNS, for the address "website.ffund00-179676.cl-education.emulab.net" (in this example).  
First, the DNS server returns a list of root name servers:

<pre>
.			187987	IN	NS	e.root-servers.net.
.			187987	IN	NS	d.root-servers.net.
.			187987	IN	NS	k.root-servers.net.
.			187987	IN	NS	i.root-servers.net.
.			187987	IN	NS	b.root-servers.net.
.			187987	IN	NS	m.root-servers.net.
.			187987	IN	NS	h.root-servers.net.
<b>.			187987	IN	NS	f.root-servers.net.</b>
.			187987	IN	NS	a.root-servers.net.
.			187987	IN	NS	j.root-servers.net.
.			187987	IN	NS	c.root-servers.net.
.			187987	IN	NS	l.root-servers.net.
.			187987	IN	NS	g.root-servers.net.
.			187987	IN	RRSIG	NS 8 0 518400 20231129220000 20231116210000 46780 . wFuw1L2ORsgOuQwkD1Lb4Pq/H+Zm6M8K2nWBvq7yFOhTd29zbE7OYBBU bYGhuNb4Cqfm2WN4w6cs//h9/nxzmLorQ7bzP0Q+qH4ebrm0WNMf0ppY b9a0igsAqn/i6FczD6f0YEKaQo9A5pnNtL55lLrxts5CU8p/M5A2Ti/f IOZ4y4TjWQ3K3xqPMNoyoi9qLNJ78mC+rhCEEACADY9Sir5xB8u4hC5P zSuMUg5uq3mYlT/wNQFRGxA/ueDFsX9OXzdY7xQgdm3idb6RRlkWwJfn azwlevwtL3umNW1Pw0BQOl0Fnb00V76QWlb2fIWfww7lZGCJ55PIwYSQ JMultw==
;; Received 1125 bytes from 128.110.100.4#53(128.110.100.4) in 0 ms
</pre>

Then it queries one of the root name servers ("f.root-servers.net" in this example), and _that_ returns a list of `.net.` servers:

<pre>
net.			172800	IN	NS	e.gtld-servers.net.
net.			172800	IN	NS	b.gtld-servers.net.
net.			172800	IN	NS	a.gtld-servers.net.
net.			172800	IN	NS	d.gtld-servers.net.
<b>net.			172800	IN	NS	i.gtld-servers.net.</b>
net.			172800	IN	NS	f.gtld-servers.net.
net.			172800	IN	NS	j.gtld-servers.net.
net.			172800	IN	NS	k.gtld-servers.net.
net.			172800	IN	NS	c.gtld-servers.net.
net.			172800	IN	NS	g.gtld-servers.net.
net.			172800	IN	NS	h.gtld-servers.net.
net.			172800	IN	NS	l.gtld-servers.net.
net.			172800	IN	NS	m.gtld-servers.net.
net.			86400	IN	DS	37331 13 2 2F0BEC2D6F79DFBD1D08FD21A3AF92D0E39A4B9EF1E3F4111FFF2824 90DA453B
net.			86400	IN	RRSIG	DS 8 1 86400 20231203170000 20231120160000 46780 . Gv2eB2bmNp4W7mQOBs79DZw0RPh99qDcThZqK14r2sN+Z9T55VpMVEuA YOMlWUewm3zPDQMriLrT7Xg9P/zl0PBtATi196QfPuoWsKsQ54EYAtq0 hHgZojY0Qukr51C+CYpGbv09A/ZTVHyRurijwfIbU9SpgcFLJ5NOdpM+ UUYCb5YwkwJaLnRN0sePLa4/tdU5FRlVEbgojSwyf88hUSEmAYT+hzhc nNgDSEgV8Nd3lcKilO7YjjIA4V+oLAFzkJmuI4h+sqChz5L0Z+o4TrPp 4gSmTgT2LFxG3/TX1648HxU62CIVS4ORtojaCMHB0MojMzFrbXnvegFE Bidpsw==
;; Received 1203 bytes from 192.5.5.241#53(<b>f.root-servers.net</b>) in 20 ms
</pre>

Next, the "i.gtld-servers.net." name server returns a list of name servers for the `emulab.net.` domain:

<pre>
emulab.net.		172800	IN	NS	ns.emulab.net.
emulab.net.		172800	IN	NS	ns2.emulab.net.
emulab.net.		172800	IN	NS	ns5.emulab.net.
A1RT98BS5QGC9NFI51S9HCI47ULJG6JH.net. 86400 IN NSEC3 1 1 0 - A1RTLNPGULOGN7B9A62SHJE1U3TTP8DR NS SOA RRSIG DNSKEY NSEC3PARAM
A1RT98BS5QGC9NFI51S9HCI47ULJG6JH.net. 86400 IN RRSIG NSEC3 13 2 86400 20231126081804 20231119070804 44222 net. XeOPnYJQbkl01ke0LVIYy0iETCK82ntHX/tkLmYeX/iADbhxXSEK5G4L zIIUgBHmfEYZemnAZR8POMj74+rwVQ==
T5EELC66E4B0POQ7GVSMR0BBHLMDDNN8.net. 86400 IN NSEC3 1 1 0 - T5EHIB9IP2MOALEC60ELDJQM1D8HO89N NS DS RRSIG
T5EELC66E4B0POQ7GVSMR0BBHLMDDNN8.net. 86400 IN RRSIG NSEC3 13 2 86400 20231126081935 20231119070935 44222 net. 72Fi0sXAjusZPXTDykLZOTcfH1sKcdh79bMI19OgOqVu5q8LT2OhlITL GFFs1GnZHsXW1wGRxO93pgTTov4OCg==
;; Received 533 bytes from 192.43.172.30#53(<b>i.gtld-servers.net</b>) in 16 ms
</pre>

Finally, from "ns.emulab.net" we get the address of `website.ffund00-179676.cl-education.emulab.net.`:

<pre>
<b>website.ffund00-179676.cl-education.emulab.net.	1 IN CNAME pcvm603-18.emulab.net.
pcvm603-18.emulab.net.	30	IN	A	155.98.37.85</b>
emulab.net.		30	IN	NS	ns2.emulab.net.
emulab.net.		30	IN	NS	ns5.emulab.net.
emulab.net.		30	IN	NS	ns.emulab.net.
;; Received 245 bytes from 155.98.32.70#53(<b>ns.emulab.net</b>) in 4 ms
</pre>



### Use NAT



### NAT

We see NAT rewriting packet headers. Here are the packets in the LAN:

<pre>
16:53:13.663719 IP <b>192.168.100.157</b>.57962 > 155.98.37.85.80: Flags [S], seq 3245904370, win 29200, options [mss 1460,sackOK,TS val 1688452 ecr 0,nop,wscale 7], length 0
16:53:14.231939 IP 155.98.37.85.80 > <b>192.168.100.157</b>.57962: Flags [S.], seq 3992044697, ack 3245904371, win 28960, options [mss 1460,sackOK,TS val 1691466 ecr 1688452,nop,wscale 7], length 0
16:53:14.232738 IP <b>192.168.100.157</b>.57962 > 155.98.37.85.80: Flags [.], ack 1, win 229, options [nop,nop,TS val 1688595 ecr 1691466], length 0
</pre>


And here are the same packets on the WAN:

<pre>
16:53:14.032389 IP <b>128.110.96.132</b>.57962 > 155.98.37.85.80: Flags [S], seq 3245904370, win 29200, options [mss 1460,sackOK,TS val 1688452 ecr 0,nop,wscale 7], length 0
16:53:14.032440 IP 155.98.37.85.80 > <b>128.110.96.132</b>.57962: Flags [S.], seq 3992044697, ack 3245904371, win 28960, options [mss 1460,sackOK,TS val 1691466 ecr 1688452,nop,wscale 7], length 0
16:53:14.299903 IP <b>128.110.96.132</b>.57962 > 155.98.37.85.80: Flags [.], ack 1, win 229, options [nop,nop,TS val 1688595 ecr 1691466], length 0
</pre>

## Run my experiment

First, you will need to reserve resources for your experiment.

<div class="cloudlab-specific">
<h4 class="cloudlab-specific"> Cloudlab-specific instructions: Reserve resources</h4>

<p>For this experiment, we will use the CloudLab profile available at the following link: <a href="https://www.cloudlab.us/p/cl-education/home-gateway">https://www.cloudlab.us/p/cl-education/home-gateway</a>.</p>

<p>If you visit this link, you’ll see a brief description of the profile. Click “Next”. On the following page, you’ll see a diagram of your experiment topology, and on the left you’ll be asked to select the “Cluster” on which you want your experiment to run. For this experiment, you'll be asked to select <b>two</b> clusters, and they should be two <b>different</b> ones! The website host will be on one cluster, and the rest of the hosts will be on the other.</p>

<p>This experiment can run on any two of the following clusters: Utah, Wisconsin, Clemson, Emulab, APT. However, since CloudLab is a shared resource, on some occasions the cluster you select might not have enough available resources to support your experiment. The status indicator next to each cluster tells you roughly how heavily utilized it is at the moment - green indicates that there are not many users, orange means heavy load, and red means that it is almost fully utilized. You are more likely to be successful if you choose a cluster with a green indicator.</p>

<p>Make your selections for the rest of the options, then continue to request resources. Once you have successfully instantiated a profile, it will still take some time before your resources are ready for you to log in.</p>

<p>As your resources come online, you’ll see their progress on the CloudLab experiment page. Once each host in your experiment is "green" and has a "✓" icon in the top right corner, it is ready for you to log in!</p>

<p>Use a terminal application installed on your laptop or PC (not the terminal in the CloudLab web portal) to log in to each node in your topology.</p>

</div>
<br>



### Reset configuration on the clients

This experiment mimics clients connecting to a residential gateway and using the network services provided by that gateway (DHCP, DNS, NAT, for example). However, the clients in our experiment are _already_ set up to use a university gateway for those services. (Otherwise, they would not have a functional network connection and we would not be able to get in to them over SSH!) To run this experiment, we will tell the clients _not_ to use the university gateway for anything except our SSH session. 



<div class="cloudlab-specific">
<h4 class="cloudlab-specific"> CloudLab-specific instructions: Removing route via university gateway</h4>

<p>On CloudLab, once you remove the route via university gateway, you will only be able to log in to these client nodes either:</p>

<ul>
<li>using the shell in the CloudLab web portal.</li>
<li>or using the same network connection that you used to remove the university network settings on the clients.</li>
</ul>

<p>You won't be able to access them using the terminal application on your laptop or PC anymore, unless you are on the same network connection.</p>


<p>When you're ready, on each of the two "client" nodes (and <i>only</i> on the client nodes), run</p>

<pre>
bash /local/repository/remove-default.sh
</pre>

<p>to download and run a script that removes most of the network settings on the client that involve the university gateway.</p>

<p>The output will look something like:</p>

<pre>
dhclient: no process found
127.0.0.1     localhost
sudo: unable to resolve host client-1
127.0.0.1     client-1
RTNETLINK answers: File exists
RTNETLINK answers: File exists
RTNETLINK answers: File exists
RTNETLINK answers: File exists
</pre>

</div>
<br>


### Set up gateway

Now we'll set up the "gateway" node:

First, make sure packet forwarding is enabled, so that it can route traffic between the LAN and WAN. On the "gateway" node, run:

```
sudo sysctl -w net.ipv4.ip_forward=1
```

Next, we will install `dnsmasq`, a lightweight DNS and DHCP server. On the "gateway" node, run:

<pre>
sudo apt update
sudo apt -y install dnsmasq dnsmasq-base
</pre>

Don't worry if you see an error message in the output that says:

```
failed to create listening socket for port 53: Address already in use
```

we'll fix that in a moment with a new configuration file!

We will now configure `dnsmasq`.

On "gateway", run

```
sudo wget -O /etc/dnsmasq.conf https://git.io/JThhR
```

to download a [config file](https://gist.github.com/ffund/ae43447629c3c5b9da9eb54962f93adf) into "/etc/dnsmasq.conf". The contents of the config file are:

```
interface=eth1
dhcp-range=192.168.100.100,192.168.100.199,4h
dhcp-option=3,192.168.100.1
dhcp-option=6,192.168.100.1
bind-interfaces
```

This tells `dnsmasq` 

* to accept incoming requests on the experiment interface, `eth1`, 
* to give out IP addresses in the range 192.168.100.100-192.168.100.199, and
* to also notify clients to use it (192.168.100.1) as a default gateway (DHCP option 3),
* to also notify clients to use it (192.168.100.1) as the nameserver for DNS queries (DHCP option 6).
* to only bind to the interface on which it is listening for incoming connections (`eth1`)


Restart the `dnsmasq` service to load the new configuration:

```
sudo service dnsmasq restart
```

Now that both the gateway and the clients are set up, we will observe what happens when clients use some typical residential gateway services.

### Observe a DHCP request and response

We will use `tcpdump` to capture traffic involving DHCP on the LAN. On "gateway", run

```
sudo tcpdump -i eth1 -n -e -v "udp port 67 or udp port 68"
```
where

* `-i eth1` sets the interface to listen on 
* `-n` says to show numerical addresses, rather than trying to resolve them to hostnames in the output,
* `-e` says to show MAC addresses, 
* `-v` says to show verbose output (including details about the DHCP packets), and
* `"udp port 67 or udp port 68"` is a filter that says to only show traffic using the DHCP ports (UDP port 67 for the DHCP server, UDP port 68 for the DHCP client).

Now, from one of the client nodes, run:

```
sudo dhclient eth1
```

to initiate a request for an address from DHCP. 

Meanwhile, in the `tcpdump` output, you should see each of the DHCP messages involved in an initial address assignment. 

First, the client sends a "discover" request to try and find DHCP servers on the LAN. The client does not have an IP address yet, so this message has "0.0.0.0" (an invalid address) as the source IP address. Also, the client does not know the address of the DHCP server, so it uses the broadcast IP address "255.255.255.255" and the broadcast MAC address "ff:ff:ff:ff:ff:ff" in the destination fields of the Layer 2 and Layer 3 headers:

<pre>
15:15:26.554336 02:eb:60:57:10:ad > ff:ff:ff:ff:ff:ff ethertype IPv4 (0x0800), length 342: (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
    0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from 02:eb:60:57:10:ad, length 300, xid 0xc0902a0f, Flags [none]
	  Client-Ethernet-Address 02:eb:60:57:10:ad
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: <b>Discover</b>
	    Hostname Option 12, length 10: "<hostname>"
	    Parameter-Request Option 55, length 13: 
	      Subnet-Mask, BR, Time-Zone, Default-Gateway
	      Domain-Name, Domain-Name-Server, Option 119, Hostname
	      Netbios-Name-Server, Netbios-Scope, MTU, Classless-Static-Route
	      NTP
</pre>


Next, a DHCP server that receives this request (in this case: our gateway at 192.168.100.1), responds with an offer. In the example below, it offers the IP address "192.168.100.157":
 
<pre>
15:15:29.058621 02:e6:fb:b7:f3:a5 > 02:eb:60:57:10:ad, ethertype IPv4 (0x0800), length 342: (tos 0xc0, ttl 64, id 37575, offset 0, flags [none], proto UDP (17), length 328)
    192.168.100.1.67 > 192.168.100.157.68: BOOTP/DHCP, Reply, length 300, xid 0xc0902a0f, Flags [none]
	  <b>Your-IP 192.168.100.157
	  Server-IP 192.168.100.1</b>
	  Client-Ethernet-Address 02:eb:60:57:10:ad
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: <b>Offer</b>
	    Server-ID Option 54, length 4: 192.168.100.1
	    Lease-Time Option 51, length 4: 14400
	    RN Option 58, length 4: 7200
	    RB Option 59, length 4: 12600
	    Subnet-Mask Option 1, length 4: 255.255.255.0
	    BR Option 28, length 4: 192.168.100.255
	    <b>Domain-Name-Server Option 6, length 4: 192.168.100.1
	    Default-Gateway Option 3, length 4: 192.168.100.1</b>
</pre>

The server also includes other DHCP options in the message -in this case, it informs the client of the default gateway to use (option 3) and the name server to use (option 6).

The client responds with a request for the IP address that was just offered. In case multiple DHCP servers respond to the "Discover" message with an "Offer", the client will put in a "Request" for only one of them, from one DHCP server. The other DHCP servers will notice the request (since it is sent to the broadcast address, they will all receive it) and understand that the client does not accept their offer:

<pre>
15:15:29.060208 02:eb:60:57:10:ad > ff:ff:ff:ff:ff:ff, ethertype IPv4 (0x0800), length 342: (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
    0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from 02:eb:60:57:10:ad, length 300, xid 0xc0902a0f, Flags [none]
	  Client-Ethernet-Address 02:eb:60:57:10:ad
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: <b>Request</b>
	    <b>Server-ID Option 54, length 4: 192.168.100.1
	    Requested-IP Option 50, length 4: 192.168.100.157</b>
	    Hostname Option 12, length 10: "<hostname>"
	    Parameter-Request Option 55, length 13: 
	      Subnet-Mask, BR, Time-Zone, Default-Gateway
	      Domain-Name, Domain-Name-Server, Option 119, Hostname
	      Netbios-Name-Server, Netbios-Scope, MTU, Classless-Static-Route
	      NTP
</pre>

Finally, the server acknowledges the request, completing the process:

<pre>
15:15:29.090184 02:e6:fb:b7:f3:a5 > 02:eb:60:57:10:ad, ethertype IPv4 (0x0800), length 342: (tos 0xc0, ttl 64, id 37576, offset 0, flags [none], proto UDP (17), length 328)
    192.168.100.1.67 > 192.168.100.157.68: BOOTP/DHCP, Reply, length 300, xid 0xc0902a0f, Flags [none]
	  Your-IP 192.168.100.157
	  Server-IP 192.168.100.1
	  Client-Ethernet-Address 02:eb:60:57:10:ad
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: <b>ACK</b>
	    Server-ID Option 54, length 4: 192.168.100.1
	    Lease-Time Option 51, length 4: 14400
	    RN Option 58, length 4: 7200
	    RB Option 59, length 4: 12600
	    Subnet-Mask Option 1, length 4: 255.255.255.0
	    BR Option 28, length 4: 192.168.100.255
	    Domain-Name-Server Option 6, length 4: 192.168.100.1
	    Default-Gateway Option 3, length 4: 192.168.100.1
</pre>


If you run 

```
ip addr show dev eth1
```

on the client, you should observe that it is now using the IP address assigned to it by DHCP, e.g.:

<pre>
eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:eb:60:57:10:adbrd ff:ff:ff:ff:ff:ff
    inet 192.168.100.157/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
</pre>

You can also see that the client has been configured to use the gateway as the name server, as specified in the DHCP options. On the client, run

```
resolvectl status
```

to see that the DNS server is listed for the experiment interface. 

You may also see some secondary nameservers from the host testbed site on the control interface. To make sure that "our" nameserver is preferred, check the contents of `/etc/resolv.conf`:

```
cat /etc/resolv.conf
```

and make sure that "our" nameserver is listed first! If not, edit this file with

```
sudo nano /etc/resolv.conf
```

so that "our" nameserver is listed first.


Finally, we can see that the client uses the gateway as the default gateway for routing purposes. On the client, run

```
ip route show dev eth1
```

to see routing rules. We should see two rules associated with the interface connected to the LAN:

```
default via 192.168.100.1 
192.168.100.0/24 proto kernel scope link src 192.168.100.157 
```

The first rule says to use "192.168.100.1" as the next hop for any traffic whose destination address does not meet any more specific rule, i.e. as the default rule. The second rule says to send traffic for the 192.168.100.0/24 subnet out of the <code>eth1</code> interface, which is connected to the LAN.

On the second client node, use 

```
sudo dhclient eth1
```

to get an IP address on this node as well, and edit the `/etc/resolv.conf` file if necessary to specify that this interface's nameserver should be preferred.

---

### Observe a DNS query and response

Next, we'll observe a DNS query and response. We can use a DNS lookup utility called [`dig`](https://linux.die.net/man/1/dig).

On the "gateway", run `tcpdump` to monitor traffic on UDP port 53, the DNS port:

```
sudo tcpdump -i eth1 -n -v "udp port 53"
```

On a client, we are going to look up the IP address associated with the "website" node in our topology. 

First, get the *hostname* of the "website" node. On "website", run

```
hostname
```

and note the result. Make sure it is a *fully qualified domain name* - a hostname followed by a `.` and then a domain (with one or more subdomains) ending in a top-level domain such as `.us`, `.net`, `.org`, or similar. If it is not, use 

```
hostname -A
```

to get an alternative hostname that is a *fully qualified domain name*.


Then, on a client node, run

<pre>
dig <b>website.ffund00-179676.cl-education.emulab.net</b>
</pre>

substituting <i>your</i> hostname in place of mine in the command above.

The output should look something like this:

<pre>
; <<>> DiG 9.18.12-0ubuntu0.22.04.3-Ubuntu <<>> website.ffund00-179676.cl-education.emulab.net
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44699
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 092d59deb35bf169564c4259655bb886537259acb84dd8f6 (good)
<b>;; QUESTION SECTION:
;website.ffund00-179676.cl-education.emulab.net.	IN A</b>

<b>;; ANSWER SECTION:
website.ffund00-179676.cl-education.emulab.net.	1 IN CNAME pcvm603-18.emulab.net.
pcvm603-18.emulab.net.	30	IN	A	155.98.37.85</b>

;; Query time: 3 msec
;; SERVER: 192.168.100.1#53(192.168.100.1) (UDP)
;; WHEN: Mon Nov 20 12:50:30 MST 2022
;; MSG SIZE  rcvd: 154
</pre>



In particular, note:

* the QUESTION section describes the query: we are seeking an address ("A" [record type](https://en.wikipedia.org/wiki/List_of_DNS_record_types)) for the name `website.ffund00-179676.cl-education.emulab.net.`.
* the ANSWER section gives the address of the name that was the subject of the query. In this instance, it informs us (through a CNAME [record type](https://en.wikipedia.org/wiki/List_of_DNS_record_types)) that the name `website.ffund00-179676.cl-education.emulab.net.` is an alias for the host with "canonical name" `pcvm603-18.emulab.net.`, and then returns the address ("A" record) for `pcvm603-18.emulab.net.`.

We can ask the client to also return some information about the nameservers that are "authoritative" for this domain - re-run your query with `+auth`, as in

<pre>
dig <b>website.ffund00-179676.cl-education.emulab.net</b> +auth
</pre>

and note some additional sections added to the answer *if available*, for example:

<pre>
;; AUTHORITY SECTION:
emulab.net.		28	IN	NS	ns2.emulab.net.
emulab.net.		28	IN	NS	ns.emulab.net.
emulab.net.		28	IN	NS	ns5.emulab.net.

;; ADDITIONAL SECTION:
ns.emulab.net.		35329	IN	A	155.98.32.70
ns2.emulab.net.		35329	IN	A	155.98.60.2
ns5.emulab.net.		35329	IN	A	155.99.144.4
</pre>

* the AUTHORITY section lists the DNS servers that are authoritative for answering queries about the name, and the ADDITIONAL section lists their addresses. Specifically, it lists "ns.emulab.net." at 155.98.32.70 and several others as authoritative for names ending in `emulab.net`. (An authoritative nameserver is one that holds the actual records for a particular domain or address; if queried, it marks its response for that address as an authoritative response. Other non-authoritative nameservers may relay the information from a cache or from another nameserver.)

You should also see the DNS traffic in your `tcpdump` output on the gateway. First the query from the client to the gateway, seeking the address of "website.ffund00-179676.cl-education.emulab.net.":

<pre>
15:56:51.703460 IP (tos 0x0, ttl 64, id 12987, offset 0, flags [none], proto UDP (17), length 115)
    192.168.100.135.47145 > 192.168.100.1.53: 17368+ [1au] <b>A? website.ffund00-179676.cl-education.emulab.net.</b> (87)
</pre>

and then the response from the gateway, that gives the address as "155.98.37.85" in my example:

<pre>
15:56:51.704291 IP (tos 0x0, ttl 64, id 20344, offset 0, flags [DF], proto UDP (17), length 182)
    192.168.100.1.53 > 192.168.100.135.47145: 17368* 2/0/1 website.ffund00-179676.cl-education.emulab.net. CNAME pcvm603-18.emulab.net., pcvm603-18.emulab.net. A <b>155.98.37.85</b> (154)
</pre>


With the addition of the `+trace` option, `dig` will show you each successive hierarchical step that the query takes, until it reaches the name server that is authoritative for the name. 

Let's try it! We can't run an iterative query on a client node in our experiment, because at this point, the client nodes don't have access to the Internet to query DNS servers. (The client won't have Internet access until the next section, when we set up NAT.) Therefore, we will try this on the gateway node.

**On the "gateway"** (*not* on a client node), run:

<pre>
dig +trace <b>website.ffund00-179676.cl-education.emulab.net</b>
</pre>

again, substituting your own "website" node's hostname. This time, you'll see much more output.


First, the DNS server returns a list of root name servers:

<pre>
.			187987	IN	NS	e.root-servers.net.
.			187987	IN	NS	d.root-servers.net.
.			187987	IN	NS	k.root-servers.net.
.			187987	IN	NS	i.root-servers.net.
.			187987	IN	NS	b.root-servers.net.
.			187987	IN	NS	m.root-servers.net.
.			187987	IN	NS	h.root-servers.net.
<b>.			187987	IN	NS	f.root-servers.net.</b>
.			187987	IN	NS	a.root-servers.net.
.			187987	IN	NS	j.root-servers.net.
.			187987	IN	NS	c.root-servers.net.
.			187987	IN	NS	l.root-servers.net.
.			187987	IN	NS	g.root-servers.net.
.			187987	IN	RRSIG	NS 8 0 518400 20231129220000 20231116210000 46780 . wFuw1L2ORsgOuQwkD1Lb4Pq/H+Zm6M8K2nWBvq7yFOhTd29zbE7OYBBU bYGhuNb4Cqfm2WN4w6cs//h9/nxzmLorQ7bzP0Q+qH4ebrm0WNMf0ppY b9a0igsAqn/i6FczD6f0YEKaQo9A5pnNtL55lLrxts5CU8p/M5A2Ti/f IOZ4y4TjWQ3K3xqPMNoyoi9qLNJ78mC+rhCEEACADY9Sir5xB8u4hC5P zSuMUg5uq3mYlT/wNQFRGxA/ueDFsX9OXzdY7xQgdm3idb6RRlkWwJfn azwlevwtL3umNW1Pw0BQOl0Fnb00V76QWlb2fIWfww7lZGCJ55PIwYSQ JMultw==
;; Received 1125 bytes from 128.110.100.4#53(128.110.100.4) in 0 ms
</pre>

Then it queries one of the root name servers ("f.root-servers.net" in this example), and _that_ returns a list of `.net.` servers:

<pre>
net.			172800	IN	NS	e.gtld-servers.net.
net.			172800	IN	NS	b.gtld-servers.net.
net.			172800	IN	NS	a.gtld-servers.net.
net.			172800	IN	NS	d.gtld-servers.net.
<b>net.			172800	IN	NS	i.gtld-servers.net.</b>
net.			172800	IN	NS	f.gtld-servers.net.
net.			172800	IN	NS	j.gtld-servers.net.
net.			172800	IN	NS	k.gtld-servers.net.
net.			172800	IN	NS	c.gtld-servers.net.
net.			172800	IN	NS	g.gtld-servers.net.
net.			172800	IN	NS	h.gtld-servers.net.
net.			172800	IN	NS	l.gtld-servers.net.
net.			172800	IN	NS	m.gtld-servers.net.
net.			86400	IN	DS	37331 13 2 2F0BEC2D6F79DFBD1D08FD21A3AF92D0E39A4B9EF1E3F4111FFF2824 90DA453B
net.			86400	IN	RRSIG	DS 8 1 86400 20231203170000 20231120160000 46780 . Gv2eB2bmNp4W7mQOBs79DZw0RPh99qDcThZqK14r2sN+Z9T55VpMVEuA YOMlWUewm3zPDQMriLrT7Xg9P/zl0PBtATi196QfPuoWsKsQ54EYAtq0 hHgZojY0Qukr51C+CYpGbv09A/ZTVHyRurijwfIbU9SpgcFLJ5NOdpM+ UUYCb5YwkwJaLnRN0sePLa4/tdU5FRlVEbgojSwyf88hUSEmAYT+hzhc nNgDSEgV8Nd3lcKilO7YjjIA4V+oLAFzkJmuI4h+sqChz5L0Z+o4TrPp 4gSmTgT2LFxG3/TX1648HxU62CIVS4ORtojaCMHB0MojMzFrbXnvegFE Bidpsw==
;; Received 1203 bytes from 192.5.5.241#53(<b>f.root-servers.net</b>) in 20 ms
</pre>

Next, the "i.gtld-servers.net." name server returns a list of name servers for the `emulab.net.` domain:

<pre>
emulab.net.		172800	IN	NS	ns.emulab.net.
emulab.net.		172800	IN	NS	ns2.emulab.net.
emulab.net.		172800	IN	NS	ns5.emulab.net.
A1RT98BS5QGC9NFI51S9HCI47ULJG6JH.net. 86400 IN NSEC3 1 1 0 - A1RTLNPGULOGN7B9A62SHJE1U3TTP8DR NS SOA RRSIG DNSKEY NSEC3PARAM
A1RT98BS5QGC9NFI51S9HCI47ULJG6JH.net. 86400 IN RRSIG NSEC3 13 2 86400 20231126081804 20231119070804 44222 net. XeOPnYJQbkl01ke0LVIYy0iETCK82ntHX/tkLmYeX/iADbhxXSEK5G4L zIIUgBHmfEYZemnAZR8POMj74+rwVQ==
T5EELC66E4B0POQ7GVSMR0BBHLMDDNN8.net. 86400 IN NSEC3 1 1 0 - T5EHIB9IP2MOALEC60ELDJQM1D8HO89N NS DS RRSIG
T5EELC66E4B0POQ7GVSMR0BBHLMDDNN8.net. 86400 IN RRSIG NSEC3 13 2 86400 20231126081935 20231119070935 44222 net. 72Fi0sXAjusZPXTDykLZOTcfH1sKcdh79bMI19OgOqVu5q8LT2OhlITL GFFs1GnZHsXW1wGRxO93pgTTov4OCg==
;; Received 533 bytes from 192.43.172.30#53(<b>i.gtld-servers.net</b>) in 16 ms
</pre>

Finally, from "ns.emulab.net" we get the address of `website.ffund00-179676.cl-education.emulab.net.`:

<pre>
<b>website.ffund00-179676.cl-education.emulab.net.	1 IN CNAME pcvm603-18.emulab.net.
pcvm603-18.emulab.net.	30	IN	A	155.98.37.85</b>
emulab.net.		30	IN	NS	ns2.emulab.net.
emulab.net.		30	IN	NS	ns5.emulab.net.
emulab.net.		30	IN	NS	ns.emulab.net.
;; Received 245 bytes from 155.98.32.70#53(<b>ns.emulab.net</b>) in 4 ms
</pre>



### Use NAT

Next, we will set up our gateway to use NAT. On the gateway, run


<pre>
sudo iptables -A FORWARD -o eth0 -i eth1 -s 192.168.100.0/24 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -F POSTROUTING
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
</pre>

Here,

* The first rule tracks connections involving the 192.168.100.0/24 network, and makes sure that packets initiating a new connection are forwarded from the LAN to the WAN. 
* The second rule allows forwarding of packets that are part of an established connection.
* The third and fourth rules actually do the network address translation. They will rewrite the source IP address in the Layer 3 header of packets forwarded out on the WAN interface. Also, when packets are received from the WAN, it identifies the connection that they belong to, rewrites the destination IP address in the Layer 3 headers, and forwards them on the LAN.

Find out the public IP address of the NAT node. On the "gateway" device, run 

```
wget -qO- http://ipinfo.io/
```
and find out the public IP address associated with yours, e.g. 

<pre>
{
  "ip": "128.110.96.132",
  "hostname": "apt132.apt.emulab.net",
  "city": "Salt Lake City",
  "region": "Utah",
  "country": "US",
  "loc": "40.7608,-111.8911",
  "org": "AS17055 University of Utah",
  "postal": "84101",
  "timezone": "America/Denver",
  "readme": "https://ipinfo.io/missingauth"
}
</pre>

Note that because this "gateway" node itself may be behind a NAT, this "public" IP address is not necessarily the one you will see in the output of `ip addr` for the control interface.

With NAT, to the rest of the Internet, all of the hosts in our "home" network will appear as if they are coming from that IP address. On the client nodes, run

```
wget -qO- http://ipinfo.io/
```

Note that this connection will go to the Internet via the gateway, using interface `eth1` which has a private IP address in the range 192.168.100.0/24. Despite this, you should see that the client, too, appears to be coming from the IP address belonging to the NAT:

<pre>
{
  "ip": "128.110.96.132",
  "hostname": "apt132.apt.emulab.net",
  "city": "Salt Lake City",
  "region": "Utah",
  "country": "US",
  "loc": "40.7608,-111.8911",
  "org": "AS17055 University of Utah",
  "postal": "84101",
  "timezone": "America/Denver",
  "readme": "https://ipinfo.io/missingauth"
}
</pre>


To further explore our NAT functionality, we'll configure a web server, then access it from one of our client hosts and observe what IP address the client appears to be connecting from.

SSH into the "website" node and install the [Apache](https://en.wikipedia.org/wiki/Apache_HTTP_Server) web server application on it:

```
sudo apt update
sudo apt -y install apache2
```

We'll monitor traffic to and from the web server (which operates on TCP port 80) using tcpdump. We will monitor it in two places, on the LAN and on the WAN.

On the "gateway" node, monitor traffic to and from the LAN:

<pre>
sudo tcpdump -i <b>eth1</b> -n "tcp port 80"
</pre>

and on the "website" node, monitor traffic to and from the WAN:

<pre>
sudo tcpdump -i <b>eth0</b> -n "tcp port 80"
</pre>


<div class="cloudlab-specific">
<h4 class="cloudlab-specific"> CloudLab-specific instructions: Open website in a GUI web browser</h4>


<p>Firefox is a graphical web browser, so to use it, you will need a VNC session. Open a VNC session on the client, and run</p>

<pre>
firefox
</pre>

<p>You may already see a lot of traffic in your <code>tcpdump</code> window on the gateway - the browser will start to load various resources and assets immediately when it is opened, even before you visit "your" website.</p>

<p>Then, inside your Firefox session, put the URL http://<b>website.ffund00-179676.cl-education.emulab.net</b>/ in the address bar (substituting the hostname of <i>your</i> "website" node in the URL) and hit Enter. You should see the website load in the browser.</p>

<p>Then, close the browser and the VNC session.</p>

<p>As an alternative, in case your VNC session doesn't work out (it can be a little bit flaky), that's OK: we have also installed a terminal-based web browser that you can use inside your shell session. To try the terminal-based option, in a terminal session on the client run </p>

<pre>
lynx http://<b>website.ffund00-179676.cl-education.emulab.net</b>/
</pre>

<p>(substituting the hostname of <i>your</i> "website" node in the URL above). You should see the website load in the terminal.</p>


</div>
<br>



Meanwhile, in the `tcpdump` output you can see how the IP addresses are rewritten in the packet headers. On the LAN (the `tcpdump` running on the "gateway" interface facing the LAN), the Layer 3 packet header shows the connection between 192.168.100.157 (port 57962) and 155.98.37.85 (port 80):


<pre>
16:53:13.663719 IP <b>192.168.100.157</b>.57962 > 155.98.37.85.80: Flags [S], seq 3245904370, win 29200, options [mss 1460,sackOK,TS val 1688452 ecr 0,nop,wscale 7], length 0
16:53:14.231939 IP 155.98.37.85.80 > <b>192.168.100.157</b>.57962: Flags [S.], seq 3992044697, ack 3245904371, win 28960, options [mss 1460,sackOK,TS val 1691466 ecr 1688452,nop,wscale 7], length 0
16:53:14.232738 IP <b>192.168.100.157</b>.57962 > 155.98.37.85.80: Flags [.], ack 1, win 229, options [nop,nop,TS val 1688595 ecr 1691466], length 0
</pre>

However, for the _same_ packets, the `tcpdump` running on the "website" node (on the WAN) shows the connection as being between 128.110.96.132 (port 57962) and 155.98.37.85 (port 80):


<pre>
16:53:14.032389 IP <b>128.110.96.132</b>.57962 > 155.98.37.85.80: Flags [S], seq 3245904370, win 29200, options [mss 1460,sackOK,TS val 1688452 ecr 0,nop,wscale 7], length 0
16:53:14.032440 IP 155.98.37.85.80 > <b>128.110.96.132</b>.57962: Flags [S.], seq 3992044697, ack 3245904371, win 28960, options [mss 1460,sackOK,TS val 1691466 ecr 1688452,nop,wscale 7], length 0
16:53:14.299903 IP <b>128.110.96.132</b>.57962 > 155.98.37.85.80: Flags [.], ack 1, win 229, options [nop,nop,TS val 1688595 ecr 1691466], length 0
</pre>

confirming that the packet headers are rewritten in the NAT gateway.

**Note**: your "gateway" node may itself be located behind a NAT! So the WAN-facing IP address on the gateway may not be the same as the address you observe on the "website" node.

### Extra Section: HTTP exchange

In the section above, we used the HTTP protocol to retrieve a web page from our "website" node. To get a closer look, we will use `telnet` to manually write and send an HTTP request, and observe the response from the HTTP server.

On the "gateway" node, run

```
sudo tcpdump -i eth1 -w http-gateway.pcap 'tcp port 80'
```

While this is running, run

<pre>
telnet <b>website.ffund00-179676.cl-education.emulab.net</b> 80
</pre>

on a "client" node, but substitute the hostname of your own "website" host. You should see the following indication of a successful connection:

<pre>
Trying 155.98.37.85...
Connected to pcvm603-18.emulab.net.
Escape character is '^]'.
</pre>

(but with a different address and hostname.)

At the console, type the following HTTP request line by line:

<pre>
GET /index.html HTTP/1.0
From: guest@client
User-Agent: HTTPTool/1.0

</pre>

Note that you need to type "Enter" to input the last line, which is blank, and then "Enter" again to send it. You should see that the page `index.html` is returned in the `telnet` client!

Terminate `tcpdump` and transfer the packet capture to your laptop with `scp`. Analyze the captured HTTP packets. Identify the HTTP response header, and the HTML file sent from the HTTP server.



## Notes

Updated November 2023: Added CloudLab instructions, add HTTP section

Updated November 2020: Thanks to Devesh Yadav and Professor Violet Syrotiuk at Arizona State University for corrections.


### Exercise

#### DHCP

* Show the "Discover" packet sent by the client to try and find DHCP servers on the LAN. What are the source and destination IP addresses in this request? Why are these addresses used?
* Show the "Offer" packet sent by the server. What IP address does the server offer in this example? What is the range of addresses that the server in our experiment may offer? (You can refer to the `dnsmasq` configuration file.)
* Show the "Request" packet sent by the client. What is the destination address in this request? Why?
* Show the DHCP ACK sent by the server to complete the process. 
* What command would you run at the client to verify each of the following? Take screenshots showing the command *and* the output for each of these, and annotate your screenshots by drawing a circle or a box around the configuration suggested by the server in the DHCP Offer/ACK.
  * That the `eth1` interface will use the newly acquired IP address, and the   `Subnet-Mask` suggested by the server?
  * That the client uses the `Domain-Name-Server` suggested by the server?
  * That the client uses the `Default-Gateway` suggested by the server?


#### DNS

* For the DNS resolution at the client with `+auth` (not the one with `+trace` at the gateway!) show the `dig` command and its output. Also show the DNS query and response from the `tcpdump` output. Answer the following questions using the `dig` output. No explanation is required - just copy and paste the relevant word from the `dig` output for each answer.
  * What is the hostname that you tried to resolve?
  * What is the DNS record *type* that your query relates to? ([Here is a list of DNS record types](https://en.wikipedia.org/wiki/List_of_DNS_record_types).)
  * What is the *address* for the hostname you asked to resolve?
  * If the result includes an "authority" section, give the name of the first "authoritative" server listed for this name, and the IP address of that "authoritative" server.
  * What is the IP address of the *server* that the DNS response is from?
* For the hierarchical DNS resolution with `+trace`, show the `dig` command and its output. Draw a diagram showing how the hostname was resolved iteratively, starting from the implied `.` at the end and moving toward the beginning.
 * At the top, show the nameservers for the root domain. Highlight the one that you queried for the top-level domain (as shown in the `dig +trace` output).
 * At the next level, show the nameservers for the top-level domain. Highlight the one that you queried for the second-level domain.
 * At the next level, show the nameservers for the second-level domain. Highlight the one that you queried for the subdomain.
 * Repeat until you have shown how the complete hostname is resolved.

#### NAT

* Show the three-way TCP handshake for a connection between client and website as seen by `tcpdump` at the website, and as seen by `tcpdump` at the gateway (on the LAN). Make sure you can see the IP addresses and port numbers used in the connection!
* Draw a diagram showing how NAT is used between client and website, similar to [this diagram](https://witestlab.poly.edu/blog/content/images/2017/03/gateway-nat-2.svg) but with the IP addresses, hostnames, and ports from *your* experiment.


#### Extra Section: HTTP

* Show the HTTP request and response headers (only the headers!).
* In the HTTP response header, identify these key elements:
 * the version of the HTTP protocol
 * the status code and the status message (for a successful HTTP request, the standard is "200 OK")
 * the header fields that indicate the type of file that is returned, and its length
