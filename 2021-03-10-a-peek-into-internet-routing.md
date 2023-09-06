In this experiment, we will see how traffic traverses the Internet across different autonomous systems. We'll use `traceroute` to sample some Internet paths. Then, we'll try to find clues about the routers along the path from the router hostnames. We'll also examine how traffic moves from one autonomous system to another, by using a looking glass utility to see BGP routes. It should take less than one hour to run this experiment.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

## Background


### Peering and transit in the Internet


An *Internet service provider* (often abbreviated ISP) is a company that provides a telecommunication service that allows its customers to connect to the Internet.

An Internet service provider can provide two kinds of access:

* It can serve end users (residential and business users). This kind of service is called *last mile access*.
* Or it will serve *other* Internet service providers, connecting them to other ISPs. This kind of service is called *transit*.


For the Internet to work, it must be possible to exchange messages between customers of different Internet service providers. For example, if you are a Verizon customer, you want to be able to reach the entire Internet, not only websites set up by other Verizon customers! So Internet service providers must have some kind of agreement between themselves to exchange messages to/from each other’s customers.

On the Internet, ISPs organize their networks into one or more *autonomous systems* (ASes). Each AS is a group of networks that are managed as one entity. 
To exchange traffic between one AS and another, they need some prior *agreement* about where their networks connect to exchange traffic, and whether they have to pay to exchange traffic! There are two main kind of agreements between ISPs involving the exchange of traffic: peering, and paid transit.

When ISPs *peer*, they exchange traffic between themselves without charge. This usually works for ISPs that are similar in size, and are geographically close to one another, because it is mutually beneficial for them to exchange traffic.

Peering can involve a direct connection between the networks (*direct peering*), or an exchange of traffic at a facility called an Internet Exchange Point (IXP) (*public peering*). ISPs that peer through an IXP pay a fee to the IXP, depending on the capacity that they want to have available there.

For example, one of the major IXPs in the U.S. is the Deutscher Commercial Internet Exchange in New York (DE-CIX New York).  You can see a list of ASes that peer there, at [this link](https://www.de-cix.net/en/locations/united-states/new-york/connected-networks). Note that some ASes have an “open” peering policy (they will peer with any other AS that is interested), while others have more specific policies about what kinds of ASes they are willing to peer with for free. The policies may be based on network size, technical capabilities, or other characteristics of the would-be peer. Also at that link, you can see the AS (*autonomous system*) ID of each ISP (its ASN), which is used to identify it to other ISPs.

The DE-CIX IXP carries hundreds of gigabits of data per second on a regular basis:

![DE-CIX traffic](/blog/content/images/2023/03/de-cix.png)

You  can see the traffic statistics [here](https://www.de-cix.net/en/locations/new-york/statistics).

For networks that an ISP cannot reach through its peering agreements, it will arrange paid transit through another Internet service provider, where it will pay according to the volume of traffic exchanged. 

**Example**: New York University is itself an Internet service provider. It operates an autonomous system, with ASN 12 (AS12). At [this link](https://bgp.he.net/AS12), you can see how it connects to other autonomous systems to exchange traffic.

### Traceroute basics

In this experiment, we'll use a traceroute tool to see how network traffic is routed within and between ASes. Traceroute is a popular tool used to troubleshoot routing problems on the Internet. Interpreting traceroute results from the public Internet takes some skill, however! In this experiment, we'll share some advanced tips for understanding traceroute results.

First, let's review the basic operation of traceroute. To understand traceroute, it's important to know a router receives a packet, it decrements the TTL value by 1; if the TTL hits 0, the router drops the packet and returns an ICMP TTL Exceeded to the source. Traceroute uses this to learn about routers along the path from a source to a destination address! 


When you launch a traceroute, you send one or more probe packets to the destination address with a TTL value of 1. What happens to that probe packet?

* The first router along the path will decrement the TTL and, since the TTL is now 0, it will drop the probe packet and send an ICMP TTL Exceeded to the source. 
* The source receives the ICMP TTL Exceeded, and notes its source address - this is the address of the first hop router. It can also use the time difference between the probe transmission and receipt of the ICMP TTL Exceeded to compute the round trip delay between the source and the first hop router. 

Next, traceroute sends one or more probe packets to the destination address with a TTL value of 2. For these packets,

* The first router along the path will decrement the TTL to 1.
* The second router along the path will decrement the TTL to 0, drop the probe packet, and send an ICMP TTL Exceeded to the source.
* The source receives the ICMP TTL Exceeded, and notes its source address - this is the address of the second hop router.

This process is repeated, with the TTL incremented each time, until the probe is received at the destination. (The destination will send back an ICMP Port Unreachable.)

With that in mind, we can use `traceroute` along with some other tools to understand how traffic is routed in the public Internet! In addition to the IP address and round trip delay measurements from each hop along the path, the `traceroute` tool we are using will also:

* use a [reverse DNS lookup](https://en.wikipedia.org/wiki/Reverse_DNS_lookup) for each IP address, to get its hostname.  You can often discover important information from the router hostname, including the geographical location of the router, the type of router, the capacities of the router interface, and the role of the router in the network.
* look up each IP address in the [Internet Routing Registry (IRR)](https://www.radb.net/), to get the AS number. With this information along with some online reference tools, we'll be able to learn more about the route between a particular source and destination address.

Ultimately, each line of traceroute output will include the following details:

<img src="/blog/content/images/2020/12/mtr-output.svg" width=160%>


Together with other tools, this information will help us understand the paths our traffic takes through the Internet.


### Interpreting router hostnames in traceroute output

Service providers often develop naming conventions for their router hostnames to make it easier to understand the status of the network and debug problems.

For example, you can read this page about the naming conventions used by the Wikimedia Foundation for their infrastructure: [Infrastructure naming conventions](https://wikitech.wikimedia.org/wiki/Infrastructure_naming_conventions).

There, they explain that their routers are named using 

* a name prefix that defines the role of the device (e.g. "cr" is "core router"), 
* and a cluster name that includes the name of the data center vendor and the airport code for the nearest airport to the data center (e.g. "ulsfo" is the United Layer cluster near SFO, in San Francisco).

Therefore, if we see a traceroute with the following router name in it: xe-0-1-1.**cr4-ulsfo**.wikimedia.org, we understand that this path goes through core router 4 at Wikimedia's United Layer SFO data center. 

Core Internet routers have many network interfaces, each of which has its own IP address and hostname. The hostname will often also include information about the router interface that it is assigned to. In the example above, **xe-0-1-1**.cr4-ulsfo.wikimedia.org follows an [interface naming convention](https://www.oreilly.com/library/view/junos-enterprise-routing/9781449309633/ch04s02.html) for routers running the Juniper operating system Junos, where 

* the first part indicates the type of interface (e.g. "xe" is  10 Gbps Ethernet),
* the next three numbers give the location of the interface in the router: the FPC slot number, the PIC slot number, and the port number.

So, now we understand that if a traceroute path goes through xe-0-1-1.cr4-ulsfo.wikimedia.org, it goes through core router 4 at Wikimedia's United Layer SFO data center, and specifically through the 10 Gbps Ethernet interface in FPC slot 0, PIC slot 1, and port 1.

Here are some more examples of publicly available router naming conventions:

* [Sprint router naming](https://www.sprint.net/faq/naming-conventions)
* [Torwardex router naming](http://www.towardex.com/router-name.html)

Now, we'll share some more tips for interpreting router hostnames.

#### Inferring geographical location

One of the most common naming conventions for routers is the use of geographical location in the router hostname! You'll often see airport codes or abbreviated city names within a router hostname. Once you geolocate the routers along the path, you can identify the geographical route a packet takes en route to its destination.

Here are some examples of router hostnames including geographical locations in the U.S.

<table>
<thead>
  <tr>
    <th>City</th>
    <th>Airport code(s)</th>
    <th>Other</th>
    <th width=400px>Notes and examples</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Washington, DC area<br>Ashburn, VA<br>McLean, VA</td>
    <td>iad, dca</td>
    <td>wash<br>ash, ashb, asbnva<br>mcln</td>
    <td>ae120.access-a.sech-<b>iad</b>.netarch.akamai.com<br>ae-4.4079.rtsw.<b>wash</b>.net.internet2.edu<br>umd-i2.<b>mcln</b>.demarc.maxgigapop.net<br>eeq-exchange.tr01-<b>asbnva</b>01.transitrail.net<br>akamai-ic317114-<b>ash</b>-b1.ip.twelve99-cust.net<br>be-1211-cr11.<b>ashburn.va</b>.ibone.comcast.net<br><br><i>A major IXP is located in Ashburn, VA.</i></td>
  </tr>
  <tr>
    <td>New York City, NY</td>
    <td>jfk, lga</td>
    <td>nyc, newy, nycmny</td>
    <td>ae-5.4079.rtsw.<b>newy</b>32aoa.net.internet2.edu<br><b>jfk1</b>.decixny.fastly.net<br><b>nyk</b>-bb2-link.ip.twelve99.net<br>
ae-0.r24.<b>nycmny</b>01.us.bb.gin.ntt.net<br></td>
  </tr>
  <tr>
    <td>Seattle, WA</td>
    <td>sea</td>
    <td>seat, sttlwa</td>
    <td>et-4-3-0.3505.rtsw.<b>seat</b>.net.internet2.edu<br>canet-2-is-jmb-778.<b>sttlwa</b>.pacificwave.net<br>be-200-pe12.<b>seattle.wa</b>.ibone.comcast.net<br>ae27.cs1.<b>sea</b>1.us.eth.zayo.com</td>
  </tr>
  <tr>
    <td>Chicago, IL</td>
    <td>ord, mdw</td>
    <td>chic</td>
    <td>lag9.cra02.<b>mdw</b>1.llnw.net<br>po110.bs-a.sech-<b>ord</b>.netarch.akamai.com<br>ae-3.4079.rtsw.<b>chic</b>.net.internet2.edu<br>ae-0.4079.rtsw3.<b>eqch</b>.net.internet2.edu</td>
  </tr>
  <tr>
    <td>Los Angeles, CA</td>
    <td>lax</td>
    <td>los, losa, la</td>
    <td><b>los</b>-edge-08.inet.qwest.net<br>ae11.er1.<b>lax</b>10.us.zip.zayo.com<br>ae-1.4079.rtsw.<b>losa</b>.net.internet2.edu<br>8-1-1-90.ear1.<b>LosAngeles1</b>.Level3.net<br>ggr3.<b>la2ca</b>.ip.att.net<br></td>
  </tr>
  <tr>
    <td>Dallas-Fort Worth, TX</td>
    <td>dfw</td>
    <td>dal, dlstx</td>
    <td>ae17.cs1.<b>dfw</b>2.us.eth.zayo.com<br>lag60.fr4.<b>dal</b>.llnw.net<br>gar2-p360.<b>dlstx</b>.ip.att.net<br></td>
  </tr>
  <tr>
    <td>Atlanta, GA</td>
    <td>atl</td>
    <td>atla</td>
    <td>ae7.cs1.<b>atl</b>10.us.eth.zayo.com<br>et-4-3-0.189.rtsw.<b>atla</b>.net.internet2.edu<br>CenturyLink-level3-<b>Atlanta</b>2.Level3.net</td>
  </tr>
  <tr>
    <td>Salt Lake City, UT</td>
    <td>slc</td>
    <td>salt</td>
    <td>ae-1.4079.rtsw.<b>salt</b>.net.internet2.edu<br>ae4.mpr1.<b>slc</b>2.us.zip.zayo.com</td>
  </tr>
  <tr>
    <td>Denver, CO</td>
    <td>den</td>
    <td>denv</td>
    <td>ae5.cs1.<b>den</b>5.us.eth.zayo.com<br>ae-3.4079.rtsw.<b>denv</b>.net.internet2.edu</td>
  </tr>
  <tr>
    <td>Phoenix, AZ</td>
    <td>phx</td>
    <td>phn, phoe</td>
    <td>et-7-0-0.4079.rtsw.<b>phoe</b>.net.internet2.edu<br>ae1.pe05.<b>phn</b>d01-az.us.windstream.net<br><b>phn</b>4-edge-03.inet.qwest.net<br>lag15.fr4.<b>phx</b>4.llnw.net<br></td>
  </tr>
</tbody>
</table>

Of course, routers may be located in any city, not only those in the table above - here is an example of a route that goes through London (UK), Paris (France), Geneva (Switzerland, a.k.a. CH), Milan (Italy), and Athens (Greece):
 
```
12  ae6.mx1.lon2.uk.geant.net (62.40.98.37)  87.687 ms
13  ae5.mx1.par.fr.geant.net (62.40.98.179)  94.053 ms
14  ae5.mx1.gen.ch.geant.net (62.40.98.182)  101.531 ms
15  ae6.mx1.mil2.it.geant.net (62.40.98.81)  108.380 ms
16  ae3.mx2.ath.gr.geant.net (62.40.98.151)  130.760 ms
```


Somtimes, the location may be specified even more exactly than just a city - for example, **de-cix-new-york**.as13335.net is located at the [DE-CIX exchange point](https://www.de-cix.net/) in New York, and **equinix.sjc**.datapipe.net is at the Equinix exchange point in San Jose. (Routers located at an IXP often have the ASN in their name - this makes it easier to debug peering problems, when you can see at a glance what AS the router belongs to.)



#### Inferring interface capabilities, link info, router role

The hostname often also includes some information about the interface to which it is assigned, especially about the interface type or capabilities, or information about the link it is on. The following table shows some common examples you may see in this experiment:

<table>
<thead>
  <tr>
    <th>Interface type</th>
    <th>Naming convention</th>
    <th>Examples</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Gigabit Ethernet</td>
    <td>Cisco: gi<br>Juniper: ge</td>
    <td><b>gi</b>3-13.ptx-core-r1.net.umd.edu<br>
<b>ge</b>-9-2-9.ar2.ord6.us.scnet.net</td>
  </tr>
  <tr>
    <td>10 Gigabit Ethernet</td>
    <td>Cisco: te<br>Juniper: xe<br>Other: tenge, 10g</td>
    <td><b>te</b>0-7-1-3.rcr21.syr01.atlas.cogentco.com<br>
<b>xe</b>-0-0-0-667-kolanut-re0.uhnet.net<br>
<b>tenge</b>0-1.kettering5.mich.net<br>
huji-gp1-<b>10g</b>.ilan.net.il</td>
  </tr>
  <tr>
    <td>100 Gigabit Ethernet</td>
    <td>Cisco: hu<br>Juniper: et<br>Other: 100g</td>
    <td>dcr02-<b>hu</b>-0-0-0-0.bos01.twdx.net<br>
<b>et</b>-0-2-0-61-ohelo-re0.uhnet.net<br>
i2-to-sox-<b>100g</b>.sox.net<br>
hpr-svl-hpr3--stan-<b>100ge</b>.cenic.net</td>
  </tr>
  <tr>
    <td>Bundled Ethernet<br>(Aggregated Ethernet)</td>
    <td>Cisco: be<br>Juniper: ae</td>
    <td><b>be</b>3172.ccr21.alb02.atlas.cogentco.com<br>
<b>ae</b>-4.4079.rtsw.wash.net.internet2.edu</td>
  </tr>
</tbody>
</table>


You may also see routers named for the router model number - for example, a router named 1211F-**MX960**-02-xe-0-0.dortmund.unity-media.net is most likely a Juniper MX960 router, and **mx480**-xe-1-1-0.electronicbox.net is probably a Juniper MX480.

Router hostnames can also be helpful for guessing the role of the router. Core routers will often have names that include **cr**, **core**, for example: 100ge8-1.**core**1.sjc2.he.net, **cr**01.h.as24679.net. Routers that are at the edge of an AS network, that peer with other ASes, might have names including: **border**, **edge**, **peer**, or related terms. Here are some examples: **boundary**a-rtr.stanford.edu, polypri-tmr**borderrtr**.net.nyu.edu, los-**brdr**-02.inet.qwest.net, phn4-**edge**-03.inet.qwest.net.

Another example of a router role in the hostname - this is how NYU's [DMZ](https://en.wikipedia.org/wiki/DMZ_(computing)) gateway appears in traceroute results: nyugwa-ptp-**dmzgw**b-vl3082.net.nyu.edu. You can also see a [VLAN](https://en.wikipedia.org/wiki/Virtual_LAN) ID in this router hostname, and some of the other examples.

Routers that are used to peer with another AS might also indicate the ASes in the hostname, for example: **i2-to-sox**-100g.sox.net, **TWDX-level3**-100G.Boston1.Level3.net, **Sprint-level3**-100G.LosAngeles1.Level3.net. (You might notice that these are often 100 Gbps interfaces - since ASes peer at only a few locations, they need plenty of capacity there!)


### Learning about ASes  in traceroute output

In addition to router hostnames, the traceroute tool we are using will also give us AS numbers, where available. We can use this information to learn more about the ASes involved in the path, and the routes between them.


#### Find out AS details

We can find out more details about any AS using the `whois` Linux command line utility. For example, if you try 

```
whois AS12
```

at the Linux command line, you'll find key information about the AS, including its name and registration history, and information about the organization that this AS is registered to:

```
ASNumber:       12
ASName:         NYU-DOMAIN
ASHandle:       AS12
RegDate:        1984-07-05
Updated:        2011-10-10    
Ref:            https://rdap.arin.net/registry/autnum/12


OrgName:        New York University
OrgId:          NYU
Address:        726 Broadway, 8th Floor - ITS
City:           New York
StateProv:      NY
PostalCode:     10003
Country:        US
RegDate:        1984-07-05
Updated:        2017-11-27
Ref:            https://rdap.arin.net/registry/entity/NYU
```

You can find out even more information by visiting the Hurricane Electric BGP toolkit website at https://bgp.he.net/ .

If we put an ASN into the search box and click "Search", it will show us more details about that autonomous system. For example, AS237 is [Merit Networks](), a non-profit "research and education network" owned by Michigan universities:

![](/blog/content/images/2020/10/as237.png)

We can click on the "Prefixes v4" tab to see the IP address blocks that originate with this AS, and the "Peers v4" tab to see some other ASes that this AS peers with. (Note: sometimes this information may be incomplete, or out of date.)



#### Using public route servers and looking glass sites

While the `whois` utility and the BGP toolkit site listed above allow us to query various databases for information about an AS, other tools are available that allow us to get more detail about the routing configuration at and between ASes.

ASes use two types of routing protocols to determine where to send traffic: 

* an Interior Gateway Protocol (IGP), which is used for routing traffic within an AS. Examples of IGPs include: RIP, IGRP, EIGRP, OSPF, and IS-IS. Each AS can make its own decisions regarding what IGP to use and how to configure it.
* an Exterior Gateway Protocol (EGP), which is used for routing traffic between different ASes. The EGP used on the Internet is BGP.



Since BGP is used between different ASes, under different administrative control, it can be tricky to debug routes. A network engineer for an AS can view and configure the BGP configuration at border routers for that AS, but it's difficult to fix problems if the engineer can't see what other ASes are doing! To help make BGP work smoothly, many ASes offer tools that allow the public or engineers from other ISPs visibility into their routes.

Public route servers are one tool to give us some visibility into an AS's routes. A route server is a router or other device running routing software, which is open for the public to log in (usually using SSH or telnet) and see the routing tables and other BGP details.

Here are some examples of some public route servers. You can use the `telnet` command provided in the table to log in to each server from the Linux terminal, and give the username and password (if there is one). For each route server, a sample command is provided - you can run this sample command to see the routes from that server to nyu.edu:

<table>
<thead>
  <tr>
    <th>AS</th>
    <th>Access</th>
    <th>Username</th>
    <th>Password</th>
    <th>Sample command</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>3582 University of Oregon</td>
    <td>telnet route-views.routeviews.org</td>
    <td>rviews</td>
    <td></td>
    <td>show ip bgp 216.165.47.10</td>
  </tr>
  <tr>
    <td>6939 Hurricane Electric</td>
    <td>telnet route-server.he.net</td>
    <td></td>
    <td></td>
    <td>show ip bgp 216.165.47.10</td>
  </tr>
  <tr>
    <td>7018 AT&amp;T</td>
    <td>telnet route-server.ip.att.net</td>
    <td>rviews</td>
    <td>rviews</td>
    <td>show route 216.165.47.10</td>
  </tr>
  <tr>
    <td>14609 Equinix</td>
    <td>telnet route-views.eqix.routeviews.org</td>
    <td></td>
    <td></td>
    <td>show ip bgp 216.165.47.10</td>
  </tr>
  <tr>
    <td>7474 Optus Australia</td>
    <td>telnet route-views.optus.net.au</td>
    <td></td>
    <td></td>
    <td>show ip bgp 216.165.47.10</td>
  </tr>
  <tr>
    <td>27552 Towardex</td>
    <td>telnet route-server.twdx.net</td>
    <td>public</td>
    <td>public</td>
    <td>show route 216.165.47.10</td>
  </tr>
  <tr>
    <td>286 KPN</td>
    <td>telnet route-server.eurorings.net</td>
    <td>rs</td>
    <td>loveAS286</td>
    <td>show bgp 216.165.47.10</td>
  </tr>
  <tr>
    <td>3741 Internet Solutions</td>
    <td>telnet public-route-server.is.co.za</td>
    <td>rviews</td>
    <td>rviews</td>
    <td>show ip bgp 216.165.47.10</td>
  </tr>
</tbody>
</table>

Here is an example of part of the command output from the Equinix server:

<pre>
route-views.eqix.routeviews.org> show ip bgp 216.165.47.10
BGP routing table entry for 216.165.0.0/18
Paths: (<b>17 available, best #1</b>, table default)
  Not advertised to any peer
  <b>293 3754 12, (aggregated by 12 128.122.254.10)
    206.126.236.137 from 206.126.236.137 (134.55.200.107)
      Origin IGP, metric 1, valid, external, atomic-aggregate, best (AS Path)</b>
      Last update: Fri Feb 26 00:05:50 2021
  57695 3356 12 12 12 12, (aggregated by 12 128.122.254.19)
    206.126.236.214 from 206.126.236.214 (45.11.106.1)
      Origin IGP, metric 0, valid, external, atomic-aggregate
      Community: 3356:3 3356:22 3356:100 3356:123 3356:575 3356:903 3356:2010 57695:13000 57695:50272
      Extended Community: SoO:0:0
      Last update: Thu Feb 25 17:19:23 2021
  25885 10913 174 3257 12 12 12
    206.126.236.234 from 206.126.236.234 (74.123.206.254)
      Origin IGP, valid, external
      Last update: Tue Mar  2 21:12:47 2021
  6830 3257 12 12 12, (aggregated by 12 128.122.254.18)
    206.126.236.117 from 206.126.236.117 (84.116.128.70)
      Origin incomplete, valid, external, atomic-aggregate
      Community: 3257:3257 6830:17000 6830:17442 6830:35307 17152:1
      Last update: Thu Feb 25 03:18:46 2021
  3257 12 12 12, (aggregated by 12 128.122.254.18)
    206.126.236.19 from 206.126.236.19 (213.200.87.157)
      Origin incomplete, metric 80, valid, external, atomic-aggregate
      Community: 3257:4000 3257:8025 3257:8114 3257:8174 3257:50002 3257:50120 3257:51100 3257:51101
      Last update: Thu Feb 25 03:18:42 2021
-- More --
</pre>

This route server knows of 17 AS paths from its own AS (AS14609) to nyu.edu, which is in AS12. The path that is considered the "best" is the first one in this table, which goes to AS293 (ESNet) → AS3754 (NYSERNet) → AS12 (NYU).

Note that the route servers in the table above are by different hardware vendors and run different OS versions, so they won't all use exactly the same command syntax. To learn the syntax, at the router terminal interface, you can run

```
?
```

to find out what commands are available. You can also run a command followed by a `?` to learn how to use it - for example, 

```
show ?
```

Some other ISPs don't have public router servers that are accessed by telnet, but host equivalent tools online. These are often known as "Looking Glass" websites. 

For example, here is Sprint's looking glass: https://www.sprint.net/tools/looking-glass

In the following screenshot, I show how to use the Sprint Looking Glass to see BGP routes from a Sprint router in Buffalo, NY, to nyu.edu:

![](/blog/content/images/2021/03/sprint-looking-glass-2.png)

Many ISPs have similar Looking Glass tools. For this lab, the Looking Glass from Internet2 will be especially handy: https://routerproxy.net.internet2.edu/routerproxy/

Internet2 is a transit ISP that connects many  of the research and education networks that GENI sites are hosted on, so in our traceroute experiments on GENI, many paths will go through Internet2. We can use the [Internet 2 Looking Glass](https://routerproxy.net.internet2.edu/routerproxy/) to gain more insight into the paths we observe.


### Limitations and anomalies

Although we can learn a lot about the path a packet takes by using traceroute and public looking glasses, it is important to understand some common "anomalies" we may observe:

* The target host that we select may be hosted by a content distribution network in the "cloud". In this case, the hostname may resolve to different IP addresses in consecutive name resolution requests (for load balancing), and will resolve to IP addresses at different data center locations based on the geographical location of the request.
* Traceroute will not always identify every router IP address. For example, if the router is configured not to send ICMP TTL Exceeded messages, traceroute will recognize that there is a router there, but won't be able to tell its address.
* Traceroute will not always identify every hop. Some routers may do [zero-TTL forwarding](https://conferences.sigcomm.org/imc/2006/papers/p15-augustin.pdf) (forwards packets with a TTL of zero, instead of dropping them and sending an ICMP TTL Exceeded). In this case, traceroute may not even realize there is a hop at that point, let alone know its address! Also, some networks that use [MPLS](https://en.wikipedia.org/wiki/Multiprotocol_Label_Switching) may configure routers to not decrement the IP header TTL within the MPLS network (since MPLS headers have their own TTL), which would make the entire MPLS network invisible to traceroute.
* A router's IP address, hostname,  and AS number may not reflect its owner, especially at boundaries between ASes. When there is a boundary between two ASes, the router interfaces on either side of the boundary need to have addresses in the same subnet. (Typically they’ll use a /30 subnet, which provides 2 usable IP addresses, or a /31 subnet which is allowed only on point to point links, and also provides 2 IP addresses.) But, each AS controls a different IP address space. In this scenario, one AS will provide the addresses for both routers, out of its own pool of IP addresses. That can make it difficult to identify who controls the router that uses the "borrowed address"!
* When a router responds to a traceroute by sending an ICMP TTL Exceeded, it usually responds from the ingress interface (the interface on which the packet with zero TTL was received), and the source address in the ICMP is the address of the ingress interface. But, some routers may instead use a different address as the source address in the ICMP: for example, they may use the address of the interface through which they will forward packets back to the source, or they may use the address of the interface through which they will forward packets to that destination. (For more about this, you can look at [this paper](https://www.caida.org/publications/papers/2020/vrfinder/vrfinder.pdf).)
* A packet may be routed along different paths in consecutive examples (e.g. for load balancing), and the forward and reverse paths the packet takes may not be the same (asymmetric routing).


## Run my experiment

In the GENI Portal, create a new slice, then click "Add Resources". Scroll down to where it says "Choose RSpec" and select the "URL" option, the load the RSpec from the URL: [https://git.io/JqOTw](https://git.io/JqOTw)

This will load the following topology in your canvas:

![](/blog/content/images/2020/10/multi-site-topology.png)

(The hosts in this topology are named after computer scientists and engineers who were instrumental in the development of the Internet: [Radia Perlman](https://en.wikipedia.org/wiki/Radia_Perlman), [Sally Floyd](https://en.wikipedia.org/wiki/Sally_Floyd), [Anita Borg](https://en.wikipedia.org/wiki/Anita_Borg), and [Bob Kahn](https://en.wikipedia.org/wiki/Bob_Kahn).)

Note that this topology includes hosts at four sites. Click on each site (Site 1, Site 2, Site 3, and Site 4), and select a *different* InstaGENI site for each one. To make it more interesting, you may choose four sites that are geographically distributed - for example, one on the East coast of the U.S., one on the West coast, and two in the center of the U.S.

Then, click Reserve Resources, and wait for your resource reservation to finish. This may take a little longer than you're used to, since the Portal has to negotiate the resource reservation with multiple InstaGENI sites.

When the resource reservation has finished, go to the slice page, and wait until all of the nodes turn green on your canvas, indicating that they are ready to log in. Then, click on "Details" to get login information for each of the four hosts.

Once logged in to the hosts, you will need to install some required software. `mtr` is a traceroute utility that allows us to see each router along a path between two end hosts, and it has another useful feature: it allows us to look up the ASN of each of those routers. We will use this feature to see how network traffic moves between one AS and the next. We will install install `whois`, which we'll use to get some basic information about each AS.

Install `mtr` and the `whois` utility on each of the four hosts, with

```
sudo apt update
sudo apt -y install mtr whois
```

Now, choose a hostname that is available over the public Internet. In the following example, I'm going to use nyu.edu. For your experiment, you should choose a *different* hostname as the target of your `mtr` traceroute, and in all of the command that follow, you should replace nyu.edu with the address of the hostname you selected.


For the Looking Glass section of the lab, you will also need to find out the IP address of the hostname you selected - you can use `dig`, e.g.

```
dig +short nyu.edu
```

In any commands that follow that use the nyu.edu IP address, you should substitute the address you found for *your* selected hostname.

### Run an `mtr` traceroute

First, we will use `mtr` to identify all of the routers in the path between the host on GENI and our selected hostname. Run

```
mtr --aslookup --show-ips -w nyu.edu
```

Note that:

* `--aslookup` tells `mtr` to try and look up the ASN associated with each router along the path (based on its IP address).
* `--show-ips` tells `mtr` to show the IP address of each router along the path, in addition to its hostname.
* `-w` tells `mtr` to show its final report in wide format, including all available details.

The complete manual for `mtr` is [here](https://linux.die.net/man/8/mtr).

Here is some sample output, from a host at Kettering InstaGENI:

<pre>
Start: 2020-10-01T21:13:42-0400
HOST: perlman.lab4-ff524-as.ch-geni-net.geni.kettering.edu             Loss%   Snt   Last   Avg  Best  Wrst StDev
  1. AS237    pc1.geni.kettering.edu (192.245.254.16)                   0.0%    10    0.3   0.3   0.2   0.3   0.0
  2. AS237    192.245.254.254                                           0.0%    10    0.6   0.6   0.5   0.7   0.1
  3. AS237    valkyrie.kettering.edu (192.138.137.1)                    0.0%    10    0.6   0.6   0.5   0.7   0.1
  4. AS237    tenge0-1.kettering5.mich.net (207.74.114.25)              0.0%    10    0.8   0.8   0.8   0.9   0.0
  5. AS237    198.108.94.96                                             0.0%    10    0.8   3.1   0.7  20.2   6.1
  6. AS237    irbx928.flnt-m-gisd.mich.net (207.72.234.70)              0.0%    10    1.0   2.4   0.9  14.9   4.4
  7. AS237    ae7x74.sfld-cor-123net.mich.net (198.108.23.81)           0.0%    10    3.0   3.1   2.6   4.3   0.5
  8. AS237    v0x1004.rtr.wash.net.internet2.edu (192.122.183.10)       0.0%    10    6.4   6.3   6.3   6.5   0.1
  9. AS3754   buf-9208-I2-CLEV.nysernet.net (199.109.11.33)             0.0%    10   10.6   9.8   9.7  10.6   0.3
 10. AS???    syr-9208-buf-9208.nysernet.net (199.109.7.193)            0.0%    10   13.4  14.0  12.8  21.6   2.7
 11. AS???    nyc111-9204-syr-9208.nysernet.net (199.109.7.94)          0.0%    10   22.2  22.4  21.9  25.0   0.9
 12. AS???    nyc-9208-nyc111-9204.nysernet.net (199.109.7.165)         0.0%    10   22.1  22.2  22.0  23.0   0.3
 13. AS3754   nyu-nyc-9208.nysernet.net (199.109.5.6)                   0.0%    10   22.2  22.3  22.1  23.0   0.3
 14. AS12     dmzgwa-p2p-extgwa-e4-1.net.nyu.edu (128.122.254.65)       0.0%    10   22.5  22.6  22.5  23.0   0.1
 15. AS12     nyugwa-ptp-dmzgwa-vl3081.net.nyu.edu (128.122.254.108)    0.0%    10   22.4  24.1  22.3  39.2   5.3
 16. AS12     nyufw-outside-ngfw-vl3080.net.nyu.edu (128.122.254.116)  70.0%    10   22.8  22.7  22.7  22.8   0.1
 17. AS???    ???                                                      100.0    10    0.0   0.0   0.0   0.0   0.0
 18. AS12     wsqdcgwa-vl901.net.nyu.edu (128.122.1.6)                 80.0%    10   23.3  23.5  23.3  23.7   0.3
 19. AS???    ???                                                      100.0    10    0.0   0.0   0.0   0.0   0.0
</pre>

Repeat this command on all four hosts in your topology, and save the output.


### Understanding the anomalies

First, let's note some anomalous lines of output in *my* output, so you won't be concerned when you see something similar in *your* output.

There may be a few lines with the ASN missing, where `mtr` was not able to identify the AS number for that IP address. (This will occur most often with routers that are inside the AS, not boundary routers. Routers inside the AS may use IP addresses that are part of prefixes not announced outside of the AS network.) 

A missing ASN is OK, it's not an indication that something was wrong. For example:

<pre>
 10. AS???    syr-9208-buf-9208.nysernet.net (199.109.7.193)            0.0%    10   13.4  14.0  12.8  21.6   2.7
</pre>

Note that if you look up the address above in the [IRR database](https://www.radb.net/query?advanced_query=1&keywords=199.109.7.193), there is no record of any AS announcing a prefix including that address. That's why no AS number is returned by `mtr` - it uses the IRR database. The IRR database can be missing information, or have outdated information, so it's not unsual for `mtr` to be missing the ASN for some lines of output.

Even when the `mtr` output does not include the ASN in the designated AS field, you may still be able to identify the AS from the router hostname. For example, if you see router hostnames that end in `internet2.edu`, you may infer that these belong to [AS11537](https://bgp.he.net/AS11537).

There may also be a few lines with some or all details missing, or with very high packet loss, when there's a router that doesn't send the expected ICMP response. That's also OK! From the output above, we see some lines like this:
<pre>
 17. AS???    ???                                                      100.0    10    0.0   0.0   0.0   0.0   0.0
</pre>

This can happen whenever a router does not send back an ICMP response - for example,

* if the router is configured to *not* send ICMP TTL exceeded messages.
* if the probe packet did not reach the router.

Similarly, the last line (which normally shows the response from the end host) may be missing if the end host is configured not to send ICMP Destination Unreachable: Port Unreachable messages.

For similar reasons, a large packet loss ratio or extreme delay at one hop is also often not a cause for concern - this arises when ICMP generation at particular a router is slow or rate-limited, for example. 

(This is also why you may sometimes see more latency to a router in the middle of the path, than to a later router. The round trip time reported includes the time it takes for the router to generate the ICMP TTL Exceeded message, which is a low-priority action and can be very slow on some routers.)

### Identifying autonomous systems

Now, we will learn more about the ASes observed in the `mtr` output. In the example above, I go through three ASes to get from my VM at Kettering to nyu.edu: 237, 3754, and 12.  

Choose one of your `mtr` outputs. For each AS in this output, get more details about the AS using the `whois` Linux command line utility. 

For example, for AS237:

```
whois AS237
```

Identify the name of the organization that each ASN belongs to, and its location. 

You can find out even more about each AS by visiting the Hurricane Electric BGP toolkit website at https://bgp.he.net/ - put an ASN into the search box and click "Search". You can click on the "Prefixes v4" tab to see the IP address blocks that originate with this AS, and the "Peers v4" tab to see some other ASes that this AS peers with.

### Infer geographical path

Next, look at the geographic path of the traffic. In my example above,

* We can see that the path starts at Kettering University, at routers in the kettering.edu domain. Kettering University is located in Flint, MI.
* After a few hops within the Kettering network, we end up at irbx928.**flnt**-m-gisd.mich.net, which is also in Flint, MI, but not in the Kettering network.
* The next hop, ae7x74.**sfld**-cor-123net.mich.net, seems to be in Southfield, MI.
* From there, we move to v0x1004.rtr.**wash**.net.internet2.edu in Washington, DC. This appears to be where Merit connects with NYSERNet via Internet2 (note the internet2.edu domain!). Our next stop will be a router that belongs to NYSERNet, AS3754.
* Our next hop is in Buffalo, NY: **buf**-9208-I2-CLEV.nysernet.net. This appears to be a router that connects NYSERNet to Internet2 (note the I2 in the router name.)
* Next, we go to NYC via Syracuse, NY: **syr**-9208-**buf**-9208.nysernet.net to **nyc**111-9204-**syr**-9208.nysernet.net
* Our next hop in NYC is at a point where NYU connects to NYSERNet: nyu-nyc-9208.nysernet.net
* The remaining hops are in AS12, NYU's network.

For *one* of the `mtr` outputs you collected, annotate the output to include any details you can infer from the router hostnames.

### Use a looking glass

> **Note**: This section was updated in March 2022 to reflect changes in Internet2 following its upgrade to its Next Generation Infrastructure (NGI) network.

If you have a path that goes through Internet2's research and education network (AS11537), you can use Internet2's [looking glass](https://routerproxy.net.internet2.edu/routerproxy/) to gain some additional insight into how this traffic is routed. 

(If you don't have a path that goes through this network, you can generate one by using a target that is on a university network - a traceroute from a GENI site to a target on another university networks is likely to involve a path through Internet2.)

For this example, I'll walk through the following traceroute:

![](/blog/content/images/2022/03/lg-ref-mtr-1.png)

Note that hops 6, 7, 8, 9, 10, 11, 12 have `internet2.edu` in the router suffix. These routers are in Internet2's Research and Education network (AS11537), even though `mtr` was not able to resolve the AS.

In your traceroute, find the first hop and last hop in Internet2's network, and follow along with the procedure substituting your own first hop and last hop routers.

In my example, the first router that I can attribute to Internet2 is hop 6:

```
fourhundredge-0-0-0-2.4079.core2.salt.net.internet2.edu (163.253.1.115)
```

so it seems that the path enters Internet2's network in Salt Lake City, at the router named **core2.salt.net.internet2.edu**. 

Find your "first" Internet2 router in the list of NGI routers on the [looking glass](https://routerproxy.net.internet2.edu/routerproxy/), and select it, like this:

![](/blog/content/images/2022/03/router-select.png)

Our first step will be to find out the details of the router interface. After selecting the router, scroll down and choose the "show interfaces" action, then click "Submit":

![](/blog/content/images/2022/03/lg-show-interfaces.png)

The output will be very long, since each router has many interfaces! Find the interface which has the IP address that appeared in the `mtr` output for this hop - in my example, **163.253.1.115**:

![](/blog/content/images/2022/03/lg-iface-ingress.png)

Notice that the interface uses a `31` prefix length. This means that there are only two endpoints on this link - the other will have address 163.253.1.114 (in my example). 
Identify the other address on the same link for *your* ingress router. 

If this interface is the point where traffic enters the Internet2 network (the _ingress router_), we would expect the other endpoint to be a BGP neighbor in a different autonomous system.  Let's see if we can find it! Change the looking glass action from "show interfaces" to "show bgp vrf RE neighbors", and specify the address of the *other* endpoint (for *your* experiment) in the input field. Then click "Submit":

![](/blog/content/images/2022/03/lg-bgp-not-found.png)

You may find, as I have in the example above, that the other endpoint of the link is *not* in the list of BGP neighbors. Let's see if we can find out more about this other endpoint. On a terminal at one of the hosts, run

<pre>
dig +noall +answer -x <b>163.253.1.114</b>
</pre>

to do a "reverse" DNS lookup on the address of the *other* endpoint. Substitute the IP address *you* derived for the other endpoint, in place of the bold address above.

Here is what the output looks like in my example:

<pre>
114.1.253.163.in-addr.arpa. 3550 IN	PTR	fourhundredge-0-0-0-8.4079.core1.losa.net.internet2.edu.
</pre>

Note the hostname, which indicates that this is actually another routers in Internet2's network, at Los Angeles!

Although the `mtr` output implies that the first hop in Internet2's network is hop 6 (at Salt Lake City) in my example, it appears that the network interface at Salt Lake City is linked to another Internet2 router in Los Angeles. So the router in Salt Lake City is not actually the true "ingress router". 

Now that we have done this extra bit of network reconnaissance, we can also note that the description in the "show interfaces" output:

![](/blog/content/images/2022/03/lg-interface-desc.png)

also offers a clue that this interface does not connect to another AS, but instead is one end of a backbone link to the Internet2 router **core1.losa.net.internet2.edu**.

Let's repeat this process at core1.losa.net.internet2.edu and see if *that* is the true ingress router. Change the router selection to the router that you have identified in the previous steps:

![](/blog/content/images/2022/03/lg-router-select-2.png)

then "show interfaces" again:

![](/blog/content/images/2022/03/lg-show-interfaces.png)

In the output, find the other endpoint of the link from the previous router you "visited":

![](/blog/content/images/2022/03/lg-ingress-endpoint.png)

This address won't appear in the `mtr` output, though. In a traceroute, the router that *receives* the packet with a TTL of 1 sends back an ICMP time exceeded, but this interface is the one that would have *forwarded* the packet (if its TTL was higher), not *received* the packet.

In our `mtr` output, hop 5 had address **137.164.26.201** (which resolved to CENIC, AS2153). Look for this address - the "previous" address in the `mtr` output - in the "show interfaces" output:

![](/blog/content/images/2022/03/lg-show-ingress-true.png)

Note that this interface description indicates that it connects CENIC (AS2153) to Internet2's Research and Education network.

The other endpoint of *this* interface will be **137.164.26.200**, and we can find it as a BGP neighbor:

![](/blog/content/images/2022/03/lg-ingress-bgp.png)

Now we understand that **core1.losa.net.internet2.edu** (and not **core2.salt.net.internet2.edu**) is the Internet2 ingress router for this path, and that Internet2 peers with CENIC in Los Angeles! This interface on Internet2's router has an address from a prefix that belongs to CENIC, not to Internet2, so in the `mtr` output it was not immediately identifiable as an Internet2 routers.

You can use the "show bgp vrf RE summary" action in the looking glass to see who else peers with Internet2 at this point:

![](/blog/content/images/2022/03/lg-ingress-peer-summary.png)

Next, change the router selection to the router that is the *egress router* - the last hop in Internet2's research and education network, before traffic is transferred to the next AS along the path. In my example, this was hop 12 - **core2.clev.net.internet2.edu**.

![](/blog/content/images/2022/03/lg-router-select-3.png)

If you execute the "show bgp vrf RE summary" action at this router, you should see the next hop from the `mtr` output listed as a BGP neighbor. The Internet2 egress router peers with the next AS along the path here.

![](/blog/content/images/2022/03/lg-egress-summary.png)

Also at the egress router, use the "show bgp vrf RE" action and specify the target address of your traceroute in the input field. This action will show you all of the routes that BGP on this router knows about for this destination address:

![](/blog/content/images/2022/03/lg-as-path.png)

In particular, make a note of the AS path - this is a list of all of the ASes that will be traversed to reach this destination address.

In my example, we can see that after Internet2, packets will be transferred to AS3754 (NYSERnet) and AS12 (New York University).

### References

<iframe width="560" height="315" src="https://www.youtube.com/embed/L0RUI5kHzEQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

[PDF slides](https://archive.nanog.org/meetings/nanog47/presentations/Sunday/RAS_Traceroute_N47_Sun.pdf), [Lecture notes](https://major.io/wp-content/uploads/2012/06/RAS_Traceroute_Book_Format.pdf) by Richard A Steenbergen.