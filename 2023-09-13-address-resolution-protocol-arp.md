In this experiment, we will examine how ARP is used in IPv4 networks - 

* to resolve IPv4 addresses of neighbors into link layer MAC addresses
* and to keep track of the neighbor reachability.

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

In this experiment, we will examine how ARP is used in IPv4 networks - 

* to resolve IPv4 addresses of neighbors into link layer MAC addresses
* and to keep track of the neighbor reachability.

where a "neighbor" is a network interface on the same IPv4 network. 

Although the details of neighbor reachability detection are somewhat implementation-specific, we will explore the behavior of the Linux/Unix implementation, which works as shown in the following state diagram:

![](/blog/content/images/2023/09/arp-state-diagram-5.svg)

* **Address resolution**: When a host needs to send an IPv4 datagram to a destination address that is not in its ARP table ("NONE" state), it broadcasts an ARP request for that IPv4 address (goes to "INCOMPLETE" state). If a host or router with that IPv4 address is present on the network, it will send an ARP reply, the state goes to "REACHABLE", and the MAC address is added to the ARP table entry. (We will explore this in [Exercise - Basic ARP request and response](#exercisebasicarprequestandresponse).)
* **Neighbor reachability verification**: After some time passes, an existing "REACHABLE" ARP table entry will go to "STALE" state. The host can still send frames to an address in this state, but when it does, it will try to verify reachability. It goes to "DELAY" state, in which it waits for an indication that the host is still reachable (for example: bidirectional TCP communication in this state can provide an upper layer protocol "hint" that the neighbor is reachable). Then, it will try to verify reachability by sending a unicast ARP request (and goes to "PROBE" state).  If a host or router with that IPv4 address and MAC address is present on the network, it will send an ARP reply, reachability is verified, and the state goes to "REACHABLE". (We will explore this in [Exercise - Verifying neighbor reachability](#exerciseverifyingneighborreachability).)
* **Neighbor *un*reachability**: If an ARP request has been sent in the "INCOMPLETE" or "PROBE" state and no ARP reply is received (for example, if no host with this address exists on the IPv4 network), then the entry goes to "FAILED" state and the IPv4 address is considered unreachable.  (We will explore this in [Exercise - ARP for a non-existent host](#exercisearpforanonexistenthost).)

## Run my experiment

First, reserve your resources.

For this experiment, we will use a topology with three connected workstations on a single network segment, with IP addresses configured as follows:

* romeo: 10.0.0.100
* juliet: 10.0.0.101
* hamlet: 10.0.0.102

each with a netmask of 255.255.255.0.

<div style="border-color:#5e8a90; border-style:solid; padding: 15px;">
<h4 style="color:#5e8a90;"> Cloudlab-specific instructions: Reserve resources</h4>


<p>To reserve these resources on Cloudlab, open this profile page:</p>

<p><a href="https://www.cloudlab.us/p/nyunetworks/education?refspec=refs/heads/arp">https://www.cloudlab.us/p/nyunetworks/education?refspec=refs/heads/arp</a></p>

<p>Click "next", then select the Cloudlab project that you are part of and a Cloudlab cluster with available resources. (This experiment is compatible with any of the Cloudlab clusters.) Then click "next", and "finish".</p>

<p>Wait until all of the sources have turned green and have a small check mark in the top right corner of the "topology view" tab, indicating that they are fully configured and ready to log in. Then, click on "list view" to get SSH login details for the two end hosts and the router. Use these details to SSH into each one. (Or, you can right-click and use the "shell" menu option to open a terminal session in the browser.)</p>

<p>When you have logged in to each node, continue to the next section.</p>
</div>
<br>

<div style="border-color:#47aae1; border-style:solid; padding: 15px;">
<h4 style="color:#47aae1;">FABRIC-specific instructions: Reserve resources</h4>
<p>To run this experiment on <a href="https://fabric-testbed.net/">FABRIC</a>, open the JupyterHub environment on FABRIC, open a shell, and run </p>

<pre>
git clone https://github.com/teaching-on-testbeds/fabric-education arp
cd arp
git checkout arp
</pre>
<p>Then open the notebook titled "start_here.ipynb".</p>
<p>Follow along inside the notebook to reserve resources, configure them, and get the login details for each host in the experiment.</p>
<p>When you have logged in to each node, continue to the next section.</p>
</div>
<br>



<div style="border-color:#9ad61a; border-style:solid; padding: 15px;">
<h4 style="color:#9ad61a;">Chameleon-specific instructions: Reserve resources</h4>
<p>To run this experiment on <a href="https://chameleoncloud.org/">Chameleon</a>, open the JupyterHub environment on Chameleon, open a shell, and run </p>

<pre>
cd work
git clone https://github.com/teaching-on-testbeds/chameleon-education arp
cd arp
git checkout arp
</pre>
<p>Then open the notebook titled "start_here.ipynb".</p>
<p>Follow along inside the notebook to reserve resources, configure them, and get the login details for each host in the experiment.</p>
<p>When you have logged in to each node, continue to the next section.</p>
</div>
<br>





### Exercise - Basic ARP request and response

On each host, run

```
ip addr
```

to find out the name of the experiment interface (which has an IPv4 address in the 10.0.0.0/24 subnet.)

Then, run

```
ip neigh show 
```

to see the entire ARP table, and make a note of any entries for the experiment interface (if there are any).

Observe that if there *are* any ARP entries, all the IP addresses displayed are on the same network segment. 

*If* the "juliet" host (10.0.0.101) is already listed in an ARP table, then delete it (but in the following command, put the name of the experiment interface in place of <b>XXX</b>) - 

<pre>
sudo ip neigh del 10.0.0.101 dev <b>XXX</b>
</pre>

Then, run 

```
ip neigh show
```

again. For each host, save the lines of the ARP table for the experiment interface (if any).

On "romeo", run

<pre>
sudo tcpdump -i $(ip route get 10.0.0.102 | grep -oP "(?<=dev )[^ ]+") -w $(hostname -s)-arp.pcap
</pre>

to capture network traffic on the experiment interface. Leave this running. Then, open a second SSH session to "romeo", and in that session, run

```
ping -c 1 10.0.0.101
```

to send an ICMP echo request to 10.0.0.101 ("juliet").

After this ends, terminate `tcpdump` with Ctrl+C. 

Run 

```
ip neigh show
```

on each host, again. For the new ARP tables, save the lines of the ARP table for the experiment interface (if any).

> **Note**: You'll see that a new line is added to juliet's ARP table with romeo's address, even though "juliet" did not send an ARP request to resolve romeo's address! When "juliet" receives and responds to an ARP request for its own address, it will also update its ARP table to include the IP address and MAC address of the host that _sent_ the ARP request. 

The `tcpdump` application will have saved a new file named "romeo-arp.pcap" in your home directory on the "romeo" node. You can "play back" a summary of the capture file in the terminal using


```
tcpdump -enX -r $(hostname -s)-arp.pcap
```


Next, run

<pre>
sudo tcpdump -i $(ip route get 10.0.0.102 | grep -oP "(?<=dev )[^ ]+") -w $(hostname -s)-no-arp.pcap
</pre>

on "romeo", and in a second terminal on "romeo", run

```
ping -c 1 10.0.0.101
```

again. Terminate `tcpdump` with Ctrl+C. Then "play back" a summary of the capture file in the terminal using

```
tcpdump -enX -r $(hostname -s)-no-arp.pcap
```

You can use `scp` to transfer both packet capture files to your laptop. Then, you can open them in Wireshark for further analysis.

> **Note**: In your packet capture, depending on the timing of your experiment, you may observe an ARP request and reply *after* the ICMP echo request and response are exchanged. And if you look closely at this unexpected ARP request, you may notice that the destination MAC address is not the broadcast address (as with a "regular" ARP request, when the sender needs to resolve an IP address and does not know the associated MAC address) - this ARP request has a unicast destination MAC address. This type of ARP request is an ARP *poll*, which we examine in the next section.

### Exercise - Verifying neighbor reachability

> **Note**: Most operating systems will have some way to verify neighbor reachability in IPv4 networks, but the details are implementation specific! In this section, we will consider the example of Linux/Unix based hosts.

You may have noticed in the previous exercise, that the ARP table entry is initially marked "REACHABLE".  After some time has passed, though, the output of 

```
ip neigh show
```

on "romeo" will show that the entry for "juliet" has become "STALE". 

When an ARP table entry is "STALE", it will be resolved again if it needs to be used. However, the ARP request that is used to verify reachability of a "STALE" entry is a little bit different from the "regular" ARP request! 

* With a "regular" ARP request, when the sender needs to resolve an IP address and does not know the associated MAC address, the destination address in the ARP request frame will be the broadcast address. 
* With an ARP *poll*, there is a "known" MAC address also associated with the IP address, and we just want to verify that this IP address is still reachable at this MAC address.  So we send the ARP request to the  last known unicast destination MAC address. 

For this part of the experiment, you will need **three** terminal sessions on the "romeo" host.

First verify from the output of 

```
ip neigh show
```

on "romeo" that the entry for "juliet" has become "STALE". In one terminal on "romeo", run

<pre>
watch -n 0.1 ip neigh show dev $(ip route get 10.0.0.102 | grep -oP "(?<=dev )[^ ]+") 
</pre>

to continuously monitor the state of the ARP table for the experiment interface.


Then, in a second terminal on "romeo", run

<pre>
sudo tcpdump -i $(ip route get 10.0.0.102 | grep -oP "(?<=dev )[^ ]+") -w $(hostname -s)-poll-arp.pcap
</pre>

and in a third terminal on "romeo", run

```
ping -c 1 10.0.0.101
```

again.  Wait until you see the state of the ARP table entry return to "REACHABLE". (You may first see it enter "DELAY" and/or "PROBE" state.)

Terminate `tcpdump` with Ctrl+C. Then "play back" a summary of the capture file in the terminal using

```
tcpdump -enX -r $(hostname -s)-poll-arp.pcap
```

will special attention to the address in the Ethernet header of the ARP request.
 
### Exercise - ARP for a non-existent host

For this experiment, you will need *four* terminal windows on the "romeo" host.

In one terminal on "romeo", run

<pre>
watch -n 0.1 ip neigh show dev $(ip route get 10.0.0.102 | grep -oP "(?<=dev )[^ ]+")
</pre>

to continuously monitor the state of the ARP table for the experiment interface.

In a second terminal on the "romeo" host, capture network traffic on the experiment interface with:

<pre>
sudo tcpdump -i $(ip route get 10.0.0.102 | grep -oP "(?<=dev )[^ ]+") -w $(hostname -s)-eth-nonexistent.pcap
</pre>

In a third terminal window on "romeo", run

```
sudo tcpdump -i lo -w $(hostname -s)-lo-nonexistent.pcap icmp
```

to capture ICMP traffic on the loopback interface (i.e. ICMP messages sent from romeo to itself).

Then, in a fourth terminal window on "romeo", run

```
ping -c 1 10.0.0.200
```

Note that **there is no host with this IP address** in your network configuration. Watch the state of the ARP table entry for the address 10.0.0.200.


Wait for it to finish. Terminate both `tcpdump` processes and the process that is monitoring the ARP table with Ctrl+C. 


The message "Destination Host Unreachable" in the `ping` output reflects that an ICMP message of type `Destination Unreachable` with code `Host Unreachable` was received! This message was sent by the host *to itself* when it failed to resolve the IP address (i.e. due to ARP timeout). 

"Play back" a summary of the loopback capture file in the terminal using

```
tcpdump -enX -r $(hostname -s)-lo-nonexistent.pcap
```

Observe this message in the loopback interface capture.

Also, "play back" a summary of the Ethernet capture file in the terminal using

```
tcpdump -enX -r $(hostname -s)-eth-nonexistent.pcap
```

You can also use `scp` to transfer the packet captures to your laptop, and open them in Wireshark to see these packets in more detail.


## Exercises

#### Basic ARP request and response 

1. Show the summary `tcpdump` output for both packet captures. In the first case, an ARP request was sent and a reply was received before the ICMP echo request was sent. In the second case, no ARP request was sent before the ICMP echo request. Why? Show evidence from the output of the `ip neigh` commands to support your answer.


2. From the first saved `tcpdump` output, answer the following questions:

  * What is the target IP address in the ARP request?
  * At the MAC layer, what is the destination Ethernet address of the frame carrying the ARP request? Why?
  * What is the frame type field in the Ethernet frame?
  * Of the three hosts on your network segment, which host sends the ARP reply? Why?


3. When an ARP request and ARP reply appear on a network segment, which hosts on the network segment will add the *target of the ARP request* to their ARP table? Which hosts on the network segment will add the *sender of the ARP request* to their ARP table? Explain in general, as well as for the specific case of this network (make sure to also answer: in this network, did the "hamlet" host learn from any of the ARP exchanges?). Use the ARP tables you captured to support your answer.

#### Verifying neighbor reachability

1. From the saved `tcpdump` output in this section, answer the following questions:

  * What is the target IP address in the ARP request?
  * At the MAC layer, what is the destination Ethernet address of the frame carrying the ARP request? Why?
  * How is this ARP request different from the one sent in the previous section - both in its content, and in its purpose?

#### ARP for a non-existent host

1. Show the summary `tcpdump` output from the Ethernet interface in this section, and use it to answer the following questions: 

 * In the previous exercise, after sending an ARP request and receiving a reply, "romeo" sends an ICMP echo request. In this exercise, is an ICMP echo request ever sent? Why or why not?
 * From the `tcpdump` output, describe how the ARP timeout and retransmission were performed. How many attempts were made to resolve a non-existing IP address? How much time separates each attempt?

2. Show the ICMP message you captured on the loopback interface, and answer these questions:

 * What is the value of the ICMP **type** field and the ICMP **code** field in the ICMP message?
 * Why does this message appear on the loopback interface, and not on the Ethernet segment?


#### ARP state machine

1. Considering the ARP state machine discussed above, 

![](/blog/content/images/2023/09/arp-state-diagram-5.svg)

* Show the message that is sent on the transition from "NONE" to "INCOMPLETE" state, and the corresponding message that is received on the transition to "REACHABLE" state.
* Show the message that is sent on the transition from "DELAY" to "PROBE" state, and the corresponding message that is received on the transition to "REACHABLE" state.
* For an entry that ends in the "FAILED" state, show the message(s) sent on the transition from "NONE" to "INCOMPLETE". Then show the *loopback interface* message that is transferred on the transition to "FAILED" state.