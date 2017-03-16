In this experiment, we will be setting up a private Tor network on GENI, and will be testing this network by listening on the traffic going through the network. We will be examining the traffic to see the functions of Tor, specifically what information can and cannot be seen when passing traffic through the Tor network.

This experiment will take approximately 30 minutes to setup, and another 20 minutes to test the network.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

Today we live in a world where the Internet is accessible from almost anything that we have in our possession. However as the Internet continues to become more expansive and accessible, the attacks on people using the Internet by spying on and stealing their information also increases. Tor is an anonymity network that is volunteer-based, where the users of the Tor network can become part of the network to increase its size and randomness. The Tor network is a system of relays that passes around a client's traffic before sending the traffic to the client's actual destination. While a client's traffic traverses through the Tor network, the origin of the traffic becomes no longer known, and the final destination is only known at the exit point of the Tor network. Also while in the Tor network, client traffic is encrypted by Tor. In this way, Tor protects users from attackers who spy on a network and try to steal a user's information.

Nowadays many sites are sent through protocols such as HTTPS in order to protect a user's privacy. HTTPS (Hypertext Transfer Protocol over TLS) is a secure communication protocol used widely in the Internet, which provides authentication of accessed websites and provides privacy of the data that is exchanged between the client, web server, and the website [[1](#references)]. HTTPS provides end-to-end encryption of data so that in case an attacker is spying on the network, that attacker is not able to decrypt the information and figure out what is being transmitted. Tor works in conjunction with HTTPS, therefore the two methods combined provides end-to-end encryption of data, and protection by cloaking client and destination communication, as well as the locations of both ends.

### The Origin and History of Tor

Onion routing research began in 1995 by David Goldschlag, Michael Reed, and Paul Syverson, with one goal in mind, which was to separate identification from routing [[2](#references)]. Authenticating one's identity can be done through the data that is passed in the data stream, and does not necessarily have to be through one's location. The goal that these three men had in mind was not to create a complete form of anonymity when browsing the Internet, but anonymous routing.

Today there are over 6000 Tor relays inside the Tor network, serving over 1.5 million users [[3](#references)]. In the present day, there is a variety of Tor users with different goals that they wish to achieve from using Tor. One group of users are law enforcement and intelligence agency personnel, who must stay anonymous when browsing the Internet to investigate a certain case. Their actions cannot be traced, therefore anonymous routing is the best solution. Another group of users stems from those who want to voice their opinion on a certain topic without revealing their true identity. Tor provides a means for people to discuss and post on the Internet without a risk for exposure of their location and identity. Then there are other more ordinary people who simply want to make sure that there information is protected at a level on step higher than just HTTPS. Simply limiting the amount of information that can be seen by a person spying on the network is the main goal that people wish to achieve from using Tor.

### Overview of Tor's Security

Now let us examine what happens when we only use Tor to access a website (including a login):


![](/blog/content/images/2017/02/tor-https-2-1.png)
<small> Image Source: Electronic Frontier Foundation (EFF): [Tor and HTTPS](https://www.eff.org/pages/tor-and-https), under the [Creative Commons Attribution License](https://www.eff.org/copyright) </small>

The above network shows a number of users/nodes with different roles:

* user: the person using the network
* hacker: a malicious user who has gained access to the user's network
* NSA: a U.S. government agency
* ISP: the user's or site's Internet service provider
* site: the website, application, or other online resource that the user is accessing


We can clearly see that when using Tor, the user, the website, and those that have data sharing with the website still see all the information. The hacker, the nodes sharing data with the user's ISP, and the NSA agent with access to the user's ISP, can only see the location of the user sending traffic. They do not know the username, the data the user is sending, nor the site that the user is accessing. This is because Tor encrypts the information before it even leaves the user's computer. Tor also uses the first relay in the Tor circuit in the unencrypted "destination" field of packets sent through this part of the network, so that the site the user is accessing is also protected. The last Tor relay (the "exit relay") knows the username information, the data being sent, and the site to access because it is the exit relay's job to send the data to the site. At this point the traffic exits the Tor network and is no longer encrypted by Tor (although it may still be encrypted by HTTPS, if using HTTPS together with Tor). Mostly everything can be seen by the anyone that is eavesdropping on the network, except for one important piece of information: the user location. After having been bounced around the Tor network, the traffic no longer contains the original user's location, which makes it difficult to match which user is accessing the site. 


## Results

In this experiment, we verify that the client's location is known to the server it access when not using Tor, but is hidden from the server when it uses Tor. We also verify that the traffic that is being communicated between the client and server are encrypted and not encrypted depending on the part of the network that we listen on.

Without using Tor, when accessing the webserver that returns the client's IP address, we see:

```
Remote address: 192.168.5.100
Forwarded for:
```
192.168.5.100 is an IP address assigned to the client, indicating that its location is not protected.

When accessing the same webserver using Tor, we see that it returns the IP address of the exit relay, e.g.

```
Remote address: 192.168.2.1
Forwarded for:
```
indicating that the client's location is unknown to the server.

The following is an example to see that the location and traffic between a client and server can be seen by a person spying on the network, when the Tor network is not used. 

<iframe width="710" height="480" src="https://www.youtube.com/embed/4sOm61x4rAI" frameborder="0" allowfullscreen></iframe>

This next example is a similar experiment to the previous example, but this time we are using the Tor network to send our client's traffic to the server. In this case, depending on the part of the network which we listen to using tcpdump, we can only see certain pieces of information about the client and server's location, and the data being communicated between the two. An important conclusion to arrive upon here is that no one node in the network can see all the information, which makes it difficult for an attacker to see who is communicating with who.

<iframe width="710" height="480" src="https://www.youtube.com/embed/c4Ujb1IimQg" frameborder="0" allowfullscreen></iframe>

In this last example, we use a function of Tor called Tor Arm, which is a monitor interface that displays useful information that a Tor relay can see and knows. Using Tor Arm, we monitor which Tor relays are being used to create a circuit in the Tor network to pass traffic as a client downloads a large file from the server. In the beginning we use the curl function to access the web server in order to see which Tor relay is the exit relay that accesses the web server.

<iframe width="710" height="480" src="https://www.youtube.com/embed/VWoUo4lUgdg" frameborder="0" allowfullscreen></iframe>

## Run my experiment

First, reserve resources for the experiment. We will use the following topology:

![](/blog/content/images/2017/02/ToyTopology.png)

In the GENI Portal, create a new slice. Load the RSpec from the following URL: [https://raw.githubusercontent.com/tfukui95/tor-experiment/master/Toy\_Exp\_Using\_Tor\_request\_rspec.xml](https://raw.githubusercontent.com/tfukui95/tor-experiment/master/Toy_Exp_Using_Tor_request_rspec.xml)

 
After the topology is loaded onto your canvas, click "Site", and choose an InstaGENI site to reserve our resources from. Then press Reserve Resources. When all VMs are green in the slice page, meaning that the whole topology is ready to be used, we will continue on to our next step of installing Tor onto each VM.

### Installing the Tor Software

To start, we install Tor on all of the nodes *except* the web server using the
following steps:

```
sudo sh -c 'echo "deb http://deb.torproject.org/torproject.org trusty main" >> /etc/apt/sources.list'
sudo sh -c 'echo "deb-src http://deb.torproject.org/torproject.org trusty main" >> /etc/apt/sources.list'

sudo gpg --keyserver keys.gnupg.net --recv 886DDD89
sudo gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -

sudo apt-get update
sudo apt-get -y install tor deb.torproject.org-keyring vim curl tor-arm
```

An important point to note here, is that the code to get the repository from
Torproject differs depending on the version of Ubuntu that you are using. In my
case, I am using Ubuntu 14.04. If you are using an older or newer version,
look online for the correct code for your version.

### Setting up the Web Server

On the node that is designated as the web server, set up Apache:

```
sudo apt-get update
sudo apt-get -y install apache2 php5 libapache2-mod-php5
```

Then, we will set up a simple PHP script that returns the client's IP addresses
as the homepage of the web server"

```
sudo rm /var/www/html/index.html
echo '<?php' | sudo tee -a /var/www/html/index.php
echo 'echo "Remote address: " . $_SERVER['REMOTE_ADDR'] . "\n";' | sudo tee -a /var/www/html/index.php
echo 'echo "Forwarded for:  " . $_SERVER['HTTP_X_FORWARDED_FOR'] . "\n";' | sudo tee -a /var/www/html/index.php
echo '?>' | sudo tee -a /var/www/html/index.php
```

Finally, restart the Apache server:

```
sudo /etc/init.d/apache2 restart
```

In order to monitor when someone tries to access the webserver, run:

```
sudo tail -f /var/log/apache2/access.log
```

### Setting up the Directory Authority

Directory authorities help Tor clients learn the addresses of relays that make up the Tor network. Specifically, via the Tor documentation [[4](#references)]:

> How do clients know what the relays are, and how do they know that they have the right keys for them? Each relay has a long-term public signing key called the "identity key". Each directory authority additionally has a "directory signing key". The directory authorities provide a signed list of all the known relays, and in that list are a set of certificates from each relay (self-signed by their identity key) specifying their keys, locations, exit policies, and so on. So unless the adversary can control a majority of the directory authorities (as of 2012 there are 8 directory authorities), he can't trick the Tor client into using other Tor relays.
>
> How do clients know what the directory authorities are? The Tor software comes with a built-in list of location and public key for each directory authority. So the only way to trick users into using a fake Tor network is to give them a specially modified version of the software.

First, stop any currently running Tor process:

```
sudo /etc/init.d/tor stop
```

In previous attempts, we had trouble using the DataDirectory, because it was
owned by the debian-tor user and group. So we are going to run some commands as
debian-tor.

First, run

```
sudo -u debian-tor mkdir /var/lib/tor/keys
sudo -u debian-tor tor-gencert --create-identity-key -m 12 -a 192.168.1.4:7000 -i /var/lib/tor/keys/authority_identity_key -s /var/lib/tor/keys/authority_signing_key -c /var/lib/tor/keys/authority_certificate
```

and enter a password when prompted. Finally, run

```
sudo -u debian-tor tor --list-fingerprint --orport 1 --dirserver "x 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" --datadirectory /var/lib/tor/
```

The output should say something like:

```
Nov 23 12:27:31.540 [notice] Your Tor server's identity key fingerprint is
'Unnamed 84F349212E57E0E33A324849E290331596BB6217' Unnamed 84F3 4921 2E57 E0E3
3A32 4849 E290 3315 96BB 6217
```

Now we'll create a configuration file for the directory authority. First, get
the two fingerprints:

```
finger1=$(sudo cat /var/lib/tor/keys/authority_certificate  | grep fingerprint | cut -f 2 -d ' ')
finger2=$(sudo cat /var/lib/tor/fingerprint | cut -f 2 -d ' ')
```

Use echo to verify that the finger1 and finger2 variables now contain the fingerprints:

```
echo $finger1
echo $finger2
```

Also, get the hostname, which we will use as the Tor "nickname", with

```
HOSTNAME=$(hostname -s)
```

Then, write the config file with

```
sudo bash -c "cat >/etc/tor/torrc <<EOL
TestingTorNetwork 1
DataDirectory /var/lib/tor
RunAsDaemon 1
ConnLimit 60
Nickname $HOSTNAME
ShutdownWaitLength 0
PidFile /var/lib/tor/pid
Log notice file /var/log/tor/notice.log
Log info file /var/log/tor/info.log
Log debug file /var/log/tor/debug.log
ProtocolWarnings 1
SafeLogging 0
DisableDebuggerAttachment 0
DirAuthority $HOSTNAME orport=5000 no-v2 hs v3ident=$finger1 192.168.1.4:7000 $finger2
SocksPort 0
OrPort 5000
ControlPort 9051
Address 192.168.1.4
DirPort 7000
# An exit policy that allows exiting to IPv4 LAN
ExitPolicy accept 192.168.1.0/24:*

AuthoritativeDirectory 1
V3AuthoritativeDirectory 1
ContactInfo auth0@test.test
ExitPolicy reject *:*
TestingV3AuthInitialVotingInterval 300
TestingV3AuthInitialVoteDelay 20
TestingV3AuthInitialDistDelay 20
EOL"
```


Note: See [[5](#references)] for background on writing a multi-line file with variables, and
[[6](#references)] for background on using cat to write a multi-line file to a protected file.

Use

```
sudo cat /etc/tor/torrc
```

to make sure that the correct variables are written to the config file.

Since the router and client config files also need the directory server's
fingerprints in them, we'll generate them on the directory server (which
knows its own fingerprints). We'll download them to the individual router
and client nodes and customize them later.

First, install apache2, in order to write the router config file and store it
on the web

```
sudo apt-get update
sudo apt-get -y install apache2 php5 libapache2-mod-php5
```

Next write the router config file with

```
sudo bash -c "cat >/var/www/html/router.conf <<EOL
TestingTorNetwork 1
DataDirectory /var/lib/tor
RunAsDaemon 1
ConnLimit 60
ShutdownWaitLength 0
PidFile /var/lib/tor/pid
Log notice file /var/log/tor/notice.log
Log info file /var/log/tor/info.log
Log debug file /var/log/tor/debug.log
ProtocolWarnings 1
SafeLogging 0
DisableDebuggerAttachment 0
DirAuthority $HOSTNAME orport=5000 no-v2 hs v3ident=$finger1 192.168.1.4:7000 $finger2

SocksPort 0
OrPort 5000
ControlPort 9051

# An exit policy that allows exiting to IPv4 LAN
ExitPolicy accept 192.168.1.0/24:*
ExitPolicy accept 192.168.2.0/24:*
ExitPolicy accept 192.168.3.0/24:*
ExitPolicy accept 192.168.4.0/24:*
EOL"
```

This config file created on the directory authority creates a generic config
file for all routers, which can then be copied over to a router. The file is
saved in `/var/www/html/router.conf` and can be seen by

```
sudo cat /var/www/html/router.conf
```

Then, write the client config file with

```
sudo bash -c "cat >/var/www/html/client.conf <<EOL
TestingTorNetwork 1
DataDirectory /var/lib/tor
RunAsDaemon 1
ConnLimit 60
ShutdownWaitLength 0
PidFile /var/lib/tor/pid
Log notice file /var/log/tor/notice.log
Log info file /var/log/tor/info.log
Log debug file /var/log/tor/debug.log
ProtocolWarnings 1
SafeLogging 0
DisableDebuggerAttachment 0
DirAuthority $HOSTNAME orport=5000 no-v2 hs v3ident=$finger1 192.168.1.4:7000 $finger2

SocksPort 9050
ControlPort 9051
EOL"
```

This config file created on the directory authority creates a generic config
file for a client. The file is saved in `/var/www/html/client.conf` and can be seen by

```
cat /var/www/html/client.conf
```


Finally, start Tor with

```
sudo /etc/init.d/tor start
```

From running

```
sudo cat /var/log/tor/debug.log | grep "Trusted"
```

to search the log file, we see a line

```
Nov 09 11:03:02.000 [debug] parse_dir_authority_line(): Trusted 100 dirserver
at 192.168.1.4:7000 (CA36BEB3CDA5028BDD7B1E1F743929A81E26A5AA)
```

which is promising - it seems to indicated that we are using our directory
authority at 192.168.1.4 (the current host).

### Setting up a Router

First, stop any currently running Tor process:

```
sudo /etc/init.d/tor stop
```

Generate fingerprints as the debian-tor user with:

```
sudo -u debian-tor tor --list-fingerprint --orport 1 --dirserver "x 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" --datadirectory /var/lib/tor/
```

The output should say something like:

```
Nov 23 12:34:00.137 [notice] Your Tor server's identity key fingerprint is
'Unnamed D2EB9948027BF5795FCA85182869FBFAA7C15B4C' Unnamed D2EB 9948 027B F579
5FCA 8518 2869 FBFA A7C1 5B4C
```

Now, download the generic router config file that we created on the
directory server:

```
sudo wget -O /etc/tor/torrc http://directoryserver/router.conf
```

Now, we'll add some extra config settings that are different on each router node: the nickname and the address(es).

Add the nickname and address(es) with

```
HOSTNAME=$(hostname -s)
echo "Nickname $HOSTNAME" | sudo tee -a /etc/tor/torrc
ADDRESS=$(hostname -I | tr " " "\n" | grep "192.168")
for A in $ADDRESS; do
  echo "Address $A" | sudo tee -a /etc/tor/torrc
done
```

Now, if you look at the contents of the config file on the router:
```
sudo cat /etc/tor/torrc
```

you should see a couple of lines like

```
Nickname router1
Address 192.168.1.2
```

at the end.

Finally, start the Tor service on the router node with

```
sudo /etc/init.d/tor restart
```

On the directory server, check if it has been made aware of the newly
added router by searching for its nickname in the log file. For example,
if the router's nickname is `relay1`, check that it has been recognized
on the directory server with

```
sudo cat /var/log/tor/debug.log | grep "relay1"
```

You should see some output like

```
Nov 23 13:06:29.000 [debug] router_parse_list_from_string(): Read router
'$D2EB9948027BF5795FCA85182869FBFAA7C15B4C~router1 at 192.168.1.2', purpose 'general'
Nov 23 13:06:29.000 [debug] dirserv_single_reachability_test(): Testing reachability
of router1 at 192.168.1.2:5000.
Nov 23 13:06:29.000 [info] dirserv_add_descriptor(): Added descriptor from 'router1'
(source: 192.168.1.2): Descriptor accepted.
Nov 23 13:06:29.000 [info] dirserv_orconn_tls_done(): Found router
$D2EB9948027BF5795FCA85182869FBFAA7C15B4C~router1 at 192.168.1.2 to be reachable
at 192.168.1.2:5000. Yay.
```

Repeat all of the above commands for all of the router nodes that you create.

### Setting up a Client

First, stop any currently running Tor process:

```
sudo /etc/init.d/tor stop
```

Then, generate fingerprints as the debian-tor user with:

```
sudo -u debian-tor tor --list-fingerprint --orport 1 --dirserver "x 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" --datadirectory /var/lib/tor/
```

The output should say something like:

```
Nov 23 13:25:30.877 [notice] Your Tor server's identity key fingerprint is
'Unnamed 4BD9274359B639B5E812913A9B1962BD84BABFFF' Unnamed 4BD9 2743 59B6 39B5
E812 913A 9B19 62BD 84BA BFFF
```

Download the client config file (that we created on the directory
server) to the default Tor config file location with:

```
sudo wget -O /etc/tor/torrc http://directoryserver/client.conf
```

Add the nickname and address(es) with

```
HOSTNAME=$(hostname -s)
echo "Nickname $HOSTNAME" | sudo tee -a /etc/tor/torrc
ADDRESS=$(hostname -I | tr " " "\n" | grep "192.168")
for A in $ADDRESS; do
  echo "Address $A" | sudo tee -a /etc/tor/torrc
done
```

Finally, start the Tor service on the client node with

```
sudo /etc/init.d/tor restart
```

## Testing the Private Tor Network

In order to clearly see the benefits of using the Tor network, we must first
run a test to see what information can be seen by a hacker when not using Tor.

### Testing the Network Without Using Tor

In order to access the webserver without going through the Tor network, we
simply run

```
curl http://webserver/
```

and verify that the server returns the client's IP address. You should see
something like

```
Remote address: 192.168.5.100
Forwarded for:
```

From this we can see how that the webserver is telling us that the VM that
accessed the webserver is the client, which shows that the client and webserver
are directly communicating with each other.

Next we will be using Tcpdump to watch the traffic on the network. We want
Tcpdump to both display the output to screen while also saving the output to a
file. The following is the format

```
sudo tcpdump -s 1514 -i any 'port <port_num>' -U -w - | tee <name_file>.pcap | tcpdump -nnxxXSs 1514 -r -
```

where "port\_num" is the specific port number to access, and "name\_file" is the name that you want to save the file as.

Since we are not using the Tor network, we will be listening on port 80 which
is the most common HTTP port. Have two client terminals opened, and on one, start listening through port 80 by running

```
sudo tcpdump -s 1514 -i any 'port 80' -U -w - | tee clientnotor.pcap | tcpdump -nnxxXSs 1514 -r -
```

On the web server terminal also listen through port 80 by running

```
sudo tcpdump -s 1514 -i any 'port 80' -U -w - | tee servernotor.pcap | tcpdump -nnxxXSs 1514 -r -
```

On the other client terminal, access the webserver by running

```
curl http://webserver/
```

On both terminals that are listening on the network, we should see something like

```
IP 192.168.5.100.38826 > 192.168.5.200.80
IP 192.168.5.200.80 > 192.168.5.100.38826
```

which shows how we can see that the client and webserver are communicating
directly with each other. On the right-hand side of the output we should see
something like

```
MGET./.HTTPS/1.1..User-Agent:.curl/7.35.0..Host:.webserver..Accept:. */*
```

which is a request to access the webserver, and something like

```
Content-Length:.47..Content-Type:.text/html....Remote.address:.192.168.5.100.Forwarded.for:...
```

which is the output of the curl command that we ran earlier. This shows how we can see exactly what traffic they are passing to each other. We can conclude how communicating only through HTTP allows a person spying on this network to see who is communicating with who, and what specifically is being communicated. This form is communication is very risky and prone to getting your information stolen.

### Testing the Network Using Tor

Now we will test the same curl experiment of accessing the web server, now through the Tor network. Then we will compare the results with when we do not use Tor.

First we need to find the exit relay that is being used in the Tor network, which serves as the relay that sends the data packet to the webserver. In order to do this we run

```
curl -x socks5://127.0.0.1:9050/ http://webserver/
```

and verify that when using the Tor network (through the SOCKS proxy), the server does not know the client's IP address; it returns the IP address of the exit relay. You should see something like

```
Remote address: 192.168.2.1
Forwarded for:
```

Clearly the returned address is not the client's IP address, but the IP
address of a Tor relay that we set up. This relay is the exit relay of the
circuit that is being used.

We can see who and when someone accesses the webserver by running

```
sudo su
tail -f /var/log/apache2/access.log
```

and then running the curl command again.

Our next step is to figure out which Tor circuit is being used to access the
webserver. This can be done using Tor Arm (anonymizing relay monitor), a program
which serves as a terminal status monitor of Tor [[7](#references)]. Arm provides useful
statistics such as bandwidth, cpu, and memory usage, as well as known
connections, and the Tor configuration file.

First we run Arm on each of the Tor relays including the client and the directory
server.

```
sudo -u debian-tor arm
```

We can see a running display of bandwidth, cpu, and memory usage. On the client's
Arm window, if we flip to the second page we can see the Tor circuits that are
available to pass traffic. In order to figure out which circuit is being used,
we need to pass traffic through the network and observe which Tor relays are
also passing that same traffic. To do this, we will generate a large file on the
webserver, to be downloaded by the client to allow enough time for us to observe
each relay's Arm window to see the passing traffic.

Now we will generate a large file from the webserver. On the webserver run

```
sudo truncate -s 2G /var/www/html/large
```

Now from the client terminal we will download a large file from the webserver

```
curl -x socks5://127.0.0.1:9050/ -s http://webserver/large > /dev/null
```

There will be no output on the screen for the client as it is downloading.
However on the other client terminal you should observe that the client is
downloading a file. Other than the Arm window of the client, you should notice a
similar pattern in three other relay terminals. These three relays are the
relays being used in the Tor network to access the website and through which
the file is being downloaded. In order to see which circuit specifically is
being used, move to the second page of the Arm window of the client to see a
list of connections that the client knows. Since we found out which relay is
serving as the exit node before, we should be able to see the circuit that lists
that information. Make sure that the other two relays are also listed as the
guard and middle relays. This circuit is the one that is being used to pass
the client's traffic to the webserver, and vice-versa.


### Using Tcpdump to Watch Traffic

We will again be using tcpdump to listen to the network at different locations.
We will require three terminals for the clients for this part. On one, run

```
sudo tcpdump -s 1514 -i any 'port 9050' -U -w - | tee client9050.pcap | tcpdump -nnxxXSs 1514 -r -
```

to watch the traffic that is going through the SOCKS proxy port 9050, and then
save what it sees in a pcap file called client9050.pcap (see [[8](#references)]). The SOCKS proxy is the
proxy that is used for the client and the onion proxy to communicate with each
other. This communication is a loop-back interface, in other words the
communication occurs in the local ethernet. On another client terminal run

```
sudo tcpdump -s 1514 -i any 'port 5000' -U -w - | tee client5000.pcap | tcpdump -nnxxXSs 1514 -r -
```

to watch the traffic through port 5000, which is the port to listen on the Tor
network. This would mean that we would be listening on any traffic that is
being transferred from the client to the Tor network. Specifically, this
communication would be between the client and the entry relay that is being
used when the traffic enters the Tor network.

Next on each relay that is being used in the circuit, run

```
sudo tcpdump -s 1514 -i any 'port 5000' -U -w - | tee $(hostname -s).pcap | tcpdump -nnxxXSs 1514 -r -
```

where **$(hostname -s)** will be converted to the name of the specific relay
that you are working with. For example, if using tcpdump on relay1, the file
that tcpdump will write to will be relay1.pcap. This tcpdump function will
watch the traffic that is going through the OR port 5000, and then save what it
sees in a pcap file.

We must perform one extra step for the exit relay, and that is to open another
terminal and have the relay listen on port 80, which is the most commonly used
port for HTTP. On the new terminal run

```
sudo tcpdump -s 1514 -i any 'port 80' -U -w - | tee exitrelay.pcap | tcpdump -nnxxXSs 1514 -r -
```

to start listening on the network through port 80.

Lastly we must also set up the web server to have it listen for traffic as well.
On the web server terminal run

```
sudo tcpdump -s 1514 -i any 'port 80' -U -w - | tee webserver.pcap | tcpdump -nnxxXSs 1514 -r -
```

to start listening for traffic on port 80.

Now at this point we should have seven terminals listening on a network. These
terminals are the two client terminals, terminals for the three ORs being used
in the circuit, an extra exit relay terminal, and the web server terminal.

Next on the third client terminal, we run

```
curl -x socks5://127.0.0.1:9050/ http://webserver/
```

to access the webserver. Since we are using the Tor network to access the site,
the three ORs must have seen some kind of traffic passing through it. Now let
us take a look at what the client and each OR saw. Stop the tcpdump process
with Ctrl^C. When we stop tcpdump, a file is created with the traffic that was
seen passing through. If you want to access the saved file to see the traffic
on Wireshark's interface, follow the steps in the following section. Otherwise,
as we listen on the network, the traffic that we can see will be outputted on
the display of each terminal.

### Summary of Experiment

From using tcpdump when using and not using Tor, there is a clear difference in the information that we can and cannot see at each node. When not using Tor, and using the direct link between the client and web server, all information can be seen. Clearly we know that the client and the web server knows who it is communicating with, and the packet contents that are being sent. However, what is dangerous is that any attacker that is listening on the same network can see all the information that both ends of the network can also see. This leads to easy spying, stealing of information and privacy, and may lead to a virus since the attacker knows your location and what site you visit.

When we communicate while using the Tor network:

* the client address can be seen in the beginning, where the entry relay is the last node which knows the client address. The reason why the change occurs here is because this is the point where the packet enters the Tor network. 
* Inside the Tor network, including the entry and exit relays, each relay only knows who sent it the packet and who to send the packet to next. Therefore the entry relay gets the packet from the client, and knows to send it to the middle relay, but once it gets to the middle relay, the middle relay only knows that it got the packet from the entry relay and does not know anything about the client who originally sent the packet. Therefore from that point the
client address is no longer known.
* The web server address is not known towards the beginning, until it reaches the exit relay and while listening through port 80. The reason why the web server address is not known until this point is due to the encryption that is done by the onion proxy at the beginning. When the packet reaches the exit relay, the exit relay decrypts the last layer of encryption and at that point, the location of the webserver is first known. Since the exit relay is responsible for sending the packet to the web server, who can't decrypt any kind of Tor encryption, the web server address as well as the packet contents are unencrypted.
* The nodes which can see the unencrypted packet data are the nodes at the beginning and the end of the circuit. In order words, the nodes that are inside the Tor network cannot see the packet contents, because the information is encrypted while traveling inside the Tor network, and the ends of the circuit cannot decrypt Tor encryption so the information must be unencrypted at the ends.

One of the most important conclusions to arrive upon from this experiment is that when using the Tor network, there is no one node that knows the whole mapping of the circuit. Therefore even if one node, say for example one of the Tor relays gets taken control over, the relay only knows who gave it the packet and who it needs to send it to, and nothing else. As long as no too many nodes get taken over, anonymity will definitely stay intact. This is the power of Tor, in being able to cloak client and web server communication as well as their locations and actual packet contents while the packet traverses through the Tor network.

## Notes

Before restarting Tor, you must kill the current Tor process first, and then restart Tor.

```
sudo pkill -9 tor
sudo /etc/init.d/tor restart
```

It is good practice to restart Tor whenever you haven't used the Tor network for a few hours or so, in order to recreate the circuits.


To run the Tor monitor, use

```
sudo -u debian-tor arm
```

Use the left and right arrow keys to switch between different screens.
Use `q` to quit.

There are other methods in which we can see the information that is being
passed along the circuit of the Tor network. However these methods require
additional installations of programs, and will be listed in this section as an
additional resource.

One such method to get more information about circuits available and about which exit relay is used for each connection, is to use a couple of Python utility
scripts.

Another method is an addition to tcpdump, which allows for a better and more organized window to analyze the packets that are being passed along the Tor network. This method requires Wireshark and winSCP. Wireshark is a software that can read in a file written by tcpdump, and displays the traffic very neatly to
allow better observation of the data. WinSCP is a program for Windows users,
that is going to be used to access the tcpdump files that are saved on the VMs,
and to copy it to our local Desktop, so that we can open the file with Wireshark.

### Using Python Utility Scripts

First, install some prerequisites:

```
sudo apt-get update
sudo apt-get -y install python-pip
sudo pip install stem
```

Then, you can download the utility scripts with

```
wget https://raw.githubusercontent.com/tfukui95/tor-experiment/master/utilities/exit-relay.py
wget https://raw.githubusercontent.com/tfukui95/tor-experiment/master/utilities/list-circuits.py
```

To see what circuits your Tor client is currently aware of, run

```
sudo python list-circuits.py
```

To see what exit relay is associated with an outgoing connection, you'll
need two terminals open to the client node. On one, run

```
sudo python exit-relay.py
```

to start monitoring. On another, make an outgoing connection with

```
curl -x socks5://127.0.0.1:9050/ http://webserver/                              
```

and in the first terminal, look for a message like

```
Exit relay for our connection to 192.168.2.1:80
  address: 192.168.1.3:5000
  fingerprint: B1A2C989985CD3C95C0D6C17B0A64A38007F90FB
  nickname: router3
  locale: ??
```

### Setting up Wireshark and WinSCP

Wireshark can be downloaded from the company's [homepage](https://www.wireshark.org/).
Besides downloading and installing the software, there is no often configuration
necessary.

WinSCP can also be downloaded from the company's [homepage](https://winscp.net/eng/download.php).
After installation, we open the application, and immediately a login window
comes up. To login to a specific VM, we must have certain information available,
which can be found on GENI, in the Details page of your slice. Choose **SCP** for
File Protocol. The Host name is the information after the @ symbol used for
logging into each VM, followed by the port number. Your username is the same as
the username before the @ symbol. For the password, we must use our ssh key as
an authentication method. Click "Advanced", followed by "Authentication" under
SSH. Under Authentication Parameters, click Private Key File, and browse to
your location of your ssh key. Once you add the key, you will be asked to convert
your key to PuTTY format because winSCP only supports PuTTY. Go ahead and convert
the key, and now you can save your login info of your VM, to allow faster login
the next time.

## References

[1] "What is HTTPS?" [https://www.instantssl.com/ssl-certificate-products/https.html](https://www.instantssl.com/ssl-certificate-products/https.html)  

  [2] "A Peel of Onion" Paul Syverson, [http://dl.acm.org/citation.cfm?id=2076750](http://dl.acm.org/citation.cfm?id=2076750)

  [3] "Tor Metrics" Tor Project, [https://metrics.torproject.org/](https://metrics.torproject.org/)

[4] "Tor FAQ - Key Management" [https://www.torproject.org/docs/faq#KeyManagement](https://www.torproject.org/docs/faq#KeyManagement)  

[5] "How do you write multiple line configuration file using BASH, and use variables on multiline?" YumYumYum, Stack Overflow,  [http://stackoverflow.com/questions/7875540/how-do-you-write-multiple-line-configuration-file-using-bash-and-use-variables](http://stackoverflow.com/questions/7875540/how-do-you-write-multiple-line-configuration-file-using-bash-and-use-variables)  

[6] "sudo cat << EOF > File doesn't work, sudo su does" iamauser, Stack Overflow, [http://stackoverflow.com/questions/18836853/sudo-cat-eof-file-doesnt-work-sudo-su-does](http://stackoverflow.com/questions/18836853/sudo-cat-eof-file-doesnt-work-sudo-su-does)  

[7] "Arm (Project Page)" [https://www.torproject.org/projects/arm.html.en](https://www.torproject.org/projects/arm.html.en)  

[8] "How can I have tcpdump write to file and standard output the appropriate data." Stack Overflow, [http://stackoverflow.com/questions/25603831/how-can-i-have-tcpdump-write-to-file-and-standard-output-the-appropriate-data](http://stackoverflow.com/questions/25603831/how-can-i-have-tcpdump-write-to-file-and-standard-output-the-appropriate-data)  