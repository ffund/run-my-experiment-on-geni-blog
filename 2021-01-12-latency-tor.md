Tor is an *anonymity network* that is used by privacy-conscious users around the world to browse the web anonymously. However, routing web traffic through the Tor network also adds extra latency to each packet. In this experiment, we will set up a private Tor network, with complete control over network conditions at each Tor node. Then, we'll retrieve several different websites with and without Tor, to better understand how the Tor latency affects the user experience, especially with respect to page load time.

This experiment will take approximately 30 minutes to setup, and another 30 minutes to test the network.

To reproduce this experiment, you will need an account on [CloudLab](https://cloudlab.us/), and you should have some basic familiarity with the platform. For more details, please refer to the [CloudLab user manual](https://docs.cloudlab.us/index.html).

## Background

Tor [[1]](#references) is an _anonymity network_, used by people around the world to browse the Internet anonymously. When using Tor, network traffic is encrypted with multiple layers of encryption and routed through a sequence of Tor relays, which facilitates an anonymous connection between the source user and destination host. The source address is known to first Tor relay along the path (and to eavesdroppers on networks between the source and the first Tor relay), and the destination address is known to the last Tor relay along the path (and to eavesdroppers on networks between the last Tor relay and the destination). However, none of the Tor relays, destination hosts, or eavesdroppers on a single network can easily identify _both_ source and destination address.

Many Tor users rely on the network to be able to speak freely and engage in activism while living under a repressive regime [[2]](#references), where they may be persecuted for these activities. For example, in the aftermath of the August 2020 Belarusian presidential election, where the incumbent declared victory in an election that was neither free nor fair according to international observers, both professional and citizen journalists were subject to arrest and prosecution. At the same time, the number of connections from Belarus to Tor bridges - which are used to facilitate anonymous connections from locations where the government or Internet service providers block ordinary Tor usage - increased almost tenfold, before settling at a level close to double the previous baseline. Belarusians used Tor and other privacy apps to help protect themselves while sharing information with the outside world about conditions inside the country [[3]](#references).

![](/blog/content/images/2021/01/tor-belarus-users-1.png)
<center><small><i>Figure 1: The number of Belarusians using a Tor bridge, which is used to facilitate anonymous connections from locations where the government or Internet service providers block ordinary Tor usage, increased almost tenfold in the days surrounding the August 2020 election. This number then settled on a value close to double the previous baseline. Data from [metrics.torproject.org](https://metrics.torproject.org/userstats-bridge-country.html?start=2020-01-01&end=2021-01-01&country=by).</i></small></center>


A major concern with Tor, however, is the quality of the user experience, especially due to its latency. Usability is a key requirement for the _security_ of Tor - as the designers of Tor explain [[1]](#references): "A hard-to-use system has fewer users - and because anonymity systems hide users among users, a system with fewer users provides less anonymity." Because Tor routes traffic through multiple hops in an overlay network, it significantly increases the latency of user traffic [[4]](#references). In user surveys, the cancellation rate of users browsing the web with Tor is much higher than without Tor [[5]](#references), indicating that the latency imposed by the Tor overlay exceeds many users' latency tolerance.

Several techniques have been proposed to improve the latency performance of, or example through modifications to the Tor transport protocol. However, progress is slow in part due to the difficulty associated with implementing and evaluating changes to Tor and its associated protocols. Experiments on the live Tor network are limited by lack of control over Tor relays and network links. With simulations, or private Tor testing networks on a single host, experimenters cannot investigate the user experience in an interactive fashion. 

To address this, we develop a CloudLab profile that instantiates a private Tor testing network for easy, reproducible experimental evaluations on Tor. We demonstrate the use of this CloudLab profile to evaluate web page load time over a Tor network under different network conditions.

## Results

To evaluate the performance of the Tor network, we used `tc` to establish rate limits, artificial delay, and queues on each router so as to emulate the throughput and latency of the Tor network according to published metrics [[6]](#references). We considered two network configurations. The _moderate_ configuration represents the median performance of the Tor network for U.S. users, with a round trip delay through the Tor circuit up to 380 ms and capacity of 6 Mbps. The _worst_ configuration represents the worst quartile of Tor connections in the U.S., with a round trip delay up to 520 ms and capacity of 3.7 Mbps. Then, we mirrored five selected web pages on the webserver node in the topology. Finally, we measured the time to retrieve each web page and its requisites (images, sounds, CSS and JS) with and without the Tor network.



![](/blog/content/images/2021/01/tor-results-1.png)

The results of the experiments are shown above. As expected, the page load time was much larger when using Tor than without Tor. Also, the effect size depends on both the network configuration and the characteristics of the web page (total size, number of objects). These results suggest that our experiment design may be used for a more thorough analysis of Tor latency, and to evaluate proposed solutions.

## Run my experiment


### Instantiate CloudLab profile

To run this experiment, you will need an account on CloudLab. Once you are logged in to your CloudLab account, you can open the Tor experiment profile using the following link: [https://www.cloudlab.us/p/CloudLab/tor-cloudlab](https://www.cloudlab.us/p/CloudLab/tor-cloudlab)

Click Instantiate to start an experiment using this profile.

The next step is to *parameterize* the experiment by selecting the number of clients, directory authority nodes, and other relay nodes. You may also specify the requested capacity for network links throughout the experiment topology:

![](/blog/content/images/2021/01/tor-parameters.png)

After you accept the default values or change them, click Next. On the following page, you will see a visualization of the experiment topology (with the number of nodes depending on the parameters you selected):

![](/blog/content/images/2021/01/tor-topology.png)

You will also have to select a CloudLab cluster on which to run your experiment. Select a cluster that has sufficient available resources for the experiment configuration. Then, click Next.

Finally, configure your experiment schedule, then click on Finish and wait for your experiment topology to be instantiated.

While your experiment is being prepared, you can monitor its status by clicking on List View in the CloudLab experiment interface:

![](/blog/content/images/2021/01/tor-startup.png)

After the nodes in your topology boot, they will run a setup script that:

* installs Tor
* prepares the cryptographic keys that are necessary to use Tor
* prepares the Tor configuration file, which includes fingerprints of keys from the directory authority nodes. 
* restarts Tor with the new configuration.

Your experiment is ready when all nodes show "Status" as "ready" and "Startup" as "Finished". At this point, you can log in to any node over SSH.


### Verify Tor connectivity

To verify that the Tor network has bootstrapped successfully on a node, you can run

```
sudo -u debian-tor nyx
```

and click the right arrow once to navigate to the Connections window in the Tor monitor. You should see some Tor circuits in this window, as in the following image:

![](/blog/content/images/2021/01/tor-circuit-example.png)

### Debugging potential Tor problems

If some hosts in the topology boot much faster than others, the Tor network may not bootstrap properly, because the fingerprints of the directory authority nodes were not yet available when the other nodes were configured. If you don't see any circuits, you can run

```
sudo cat /etc/tor/torrc | grep "DirAuthority" | wc -l
```

to see if that may have happened to you! This command will print the number of directory authorities listed in your Tor configuration. If it's less than the number of directory authorities that you configured, you can run

```
DIRS=$(cat /etc/hosts | grep dir | cut -d ' ' -f 3)
for d in $DIRS
do
   wget -qO- http://"$d"/fingerprint  | sudo tee -a /etc/tor/torrc
done
```

to update the directory authority fingerprints in the node's Tor configuration, then

```
sudo service tor restart
```

to restart Tor with the new configuration.


### Mirror websites

We will be saving homepages of 5 different websites on our webserver
The following are the 5 websites:

1.  NYU Tandon School of Engineering
2.  LinkedIn
3.  Reddit
4. Twitter
5.  YouTube

For each website, we will be storing the webpage in its own directory inside the /var/www/html/ directory of the webserver. 
On the webserver terminal, run

```
cd /var/www/html/  

sudo wget -e robots=off --wait 1 -H -p -k http://engineering.nyu.edu/  
sudo wget -e robots=off --wait 1 -H -p -k http://linkedin.com/ 
sudo wget -e robots=off --wait 1 -H -p -k http://reddit.com/ 
sudo wget -e robots=off --wait 1 -H -p -k http://twitter.com/ 
sudo wget -e robots=off --wait 1 -H -p -k http://youtube.com/  
```

Now to test if all of the websites are on the webserver, install proxychains using this command:
```
sudo apt-get -y install proxychains  
```

Then run
```
time wget -p http://10.10.253.200/reddit.com/
#Which is not over Tor

time proxychains wget -p http://10.10.253.200/reddit.com/
#Which is over Tor
```
as a way to test if the Tor server is working

### Set up link emulation for *moderate* configuration

Next, we will set up link emulation according to the network characteristics of the median Tor user in the U.S. 

**![](https://lh3.googleusercontent.com/TZRkdoTpP-abrnq0Jre-7MgMu5rbMzgahEYxN6aHS_6C_e0ZKbljxEmlGYKmMbkNiW4RvhrdMG9syyzfndTzKXCGHvFAAbHwA1Kn76Cwo5rnGy_U9rVB7bY30mXbUqAJmpu-3hX7)**

The configuration was based off of metrics found on Tor's site. 

> The throughput determined the Total Delay 
> The Latencies determined the Data Rate

**The Medium configuration:**

**Total delay**: 380ms 
**Base delay**: 200ms  (25ms)  
**Queuing delay**: 280ms (60ms per queue) 
**Rate(Capacity/bandwidth)**: 6Mbps (Medium network quality)  
**Queue size**: 0.36mbit (megabits)

On the Client, Every relay (and directory server) and Webserver run:
```
sudo tc qdisc del dev eth1 root
sudo tc qdisc add dev eth1 parent root netem delay 25ms limit 50000
```

On Router 2 run:
```
sudo tc qdisc del dev eth1 root
sudo tc qdisc replace dev eth1 root handle 1: htb default 3
sudo tc class add dev eth1 parent 1: classid 1:3 htb rate 6mbps
sudo tc qdisc add dev eth1 parent 1:3 bfifo limit 0.36mbit
```
(Make sure to run this on every interface on the Router 2 by changing the name)
> Ex eth1, eth2, etc.


### Measure page load times

To access each website both without Tor run
```
time wget -p http://10.10.253.200/reddit.com/
#Which is not over Tor

time proxychains wget -p http://10.10.253.200/reddit.com/
#Which is over Tor
```

Repeat this with all of the websites, substituting "reddit.com" with the other website URLs

```
Ex. time wget -p http://10.10.253.200/engineering.nyu.edu/
```
and record the the time in seconds 

Note: the only row we want to note is the "real" section

### Set up link emulation for *worst* configuration

Next, we'll repeat for the worst percentile of U.S. Tor connections:

**The Worst configuration:**

**Total delay:** 520ms
**Base delay:** 264ms  (33ms )
**Queuing delay:** 256ms: (85.3ms per queue)
**Rate(Capacity/bandwidth):** 3.695Mbps (low network quality)
**Queue size:** 0.3152 Mb (megabits)

On the Client, Every relay (and directory authority) and Webserver run:
```
sudo tc qdisc del dev eth1 root
sudo tc qdisc add dev eth1 parent root netem delay 33ms limit 50000
```

On Router 2 run:
```
sudo tc qdisc del dev eth1 root
sudo tc qdisc replace dev eth1 root handle 1: htb default 3
sudo tc class add dev eth1 parent 1: classid 1:3 htb rate 3.695mbps
sudo tc qdisc add dev eth1 parent 1:3 bfifo limit 0.3152mbit
```

Then, measure page load times again.

## Notes


Tor users often experience intolerable latency when using the network, but the challenges of experimental evaluation on Tor makes it difficult for researchers to work on this problem. In this work, we develop a procedure for easy, reproducible experimental evaluations on Tor. 
 

### Acknowledgement

This post describes joint work with Ashutosh Srivastava, Fraida Fund, and Shivendra Panwar.

This research is supported by the New York Center for Advanced Technology in Telecommunications (CATT), by the Center for K12 STEM Education at NYU Tandon, and by the Pinkerton Foundation.


### References

[1] R. Dingledine,  N.  Mathewson,  and  P.  Syverson,  “Tor:  The  second-generation  onion  router,”  Naval  Research  Lab  Washington  DC,  Tech.Rep., 2004. [PDF](https://apps.dtic.mil/dtic/tr/fulltext/u2/a465464.pdf)

[2] E. Jardine, “Tor, what is it good for? political repression and the use ofonline anonymity-granting technologies,” New media & society, vol. 20,no. 2, pp. 435–452, 2018. [URL](https://doi.org/10.1177/1461444816639976)

[3] A. Hamacher, "Tor and Psiphon activity surges in protest-stricken Belarus," 
Aug 12, 2020. [URL](https://decrypt.co/38443/tor-and-psiphon-activity-surges-in-protest-stricken-belarus)

[4] Dhungel, P., et al. “Waiting for Anonymity: Understanding Delays in the Tor Overlay.” 2010 IEEE Tenth International Conference on Peer-to-Peer Computing (P2P), 2010, doi:10.1109/p2p.2010.5569995.

[5] Müller, Sebastian, et al. “Distributed Performance Measurement and Usability Assessment of the Tor Anonymization Network.” Future Internet, vol. 4, no. 2, 2012, pp. 488–513., doi:10.3390/fi4020488.

[6] “Performance.” Tor Metrics, metrics.torproject.org/onionperf-throughput.html.
