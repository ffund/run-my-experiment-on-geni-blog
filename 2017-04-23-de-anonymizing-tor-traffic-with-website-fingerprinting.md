In this experiment, we will be setting up a private Tor network on GENI, and will be testing a website fingerprinting attack on the network. We will be getting the homepages of five different sites onto our webserver, and from there will be creating fingerprints of each site. After that is complete, we will have the client randomly choose one of the five sites, while we listen on the client's network. Using the traffic that we see, we will create another fingerprint, and will compare it with our website fingerprints to see if we can find a correlation with the site that the client visited.

This experiment will take approximately 30 minutes to setup, and another 30 minutes to test the network.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

When someone uses Tor, an eavesdropper on the same network as the user cannot identify the websites that they are visiting. A website fingerprinting attack is one way in which to compromise the anonymity provided by Tor, by matching users and the websites that they visit. The goal of a fingerprinting attack is to deanonymize a user and figure out the website that they are visiting, even if the have anonymous routing and encryption of data.

The general procedure of website fingerprinting is as follows:

1. The attacker first listens to and records packet traces from several websites that may be possible websites that the user visits. The attacker uses a traffic analyzer tool (for example, tcpdump) to capture packet traces of IP layer packets. These packets are known as _training instances_
2. Using information captured such as the length of the packet, the time sent and received, etc. the attacker creates a profile of the website, also known as a _fingerprint_ [[1](#references)].
3. Next the attacker listens on the target user's network and similarly captures packet that is going in and out from the user. This data is not going to be identical to the fingerprint that the attacker creates in step 2, due to a difference in user, possible packet fragmentation, and website updates. These packets are known as _test instances_.
4. The attacker tries to compromise the target user by comparing the website fingerprint and user data by looking at them with the eye, or uses statistical methods to probabilistically come to a conclusion.

### Data sets used for fingerprinting

There are two kinds of general datasets that are involved in fingerprinting:

* __Closed-world dataset__- the situation where the attacker already knows the set of possible webpages that the victim user uses. This scenario is a bit unrealistic because the probability of compromising the user increases drastically by only having to gain packet traces from a few websites.

* __Open-world dataset__- the more realistic situation where the attacker does not know what kinds of pages the user goes on, which makes the ranges of webpages basically infinite.

For our experiment, we are going to be following the closed-world dataset because of the difficulty of simulating an open-world dataset experiment.


### Characteristics of Packet Traces

For each website included in the dataset, the attacker first collects around 20 instances of traffic. Within this traffic, there are many characteristics that the attacker must look out for to create a better more detailed fingerprint of the website.

For this experiment, we use the packet trace features described by Panchenko, et al. in their paper, _Website Fingerprinting in Onion Routing Based Anonymization Networks_ [[1](#references)]:

* __Size and Direction__- The size of the packet and whether the packet is ingoing or outgoing is recorded. Incoming packets are marked as positive, while those that are outgoing are marked as negative. Keeping track of the order in which the packets arrived is important as well.

* __Filtering ACK Packets__- Acknowledgement packets are sent back and forth almost constantly, and do not provide any useful information to the attacker. Therefore we first filter out packets of these size.

* __Size Markers__- These are markers to be placed whenever the direction of traffic changes from ingoing to outgoing and vice-versa. These markers must work in conjunction with filtering out the ACK packets. These markers note how much bytes went a certain direction before going the other.

* __HTML Markers__- Whenever a request for a webpage is made, the initial process is to request for the HTML document. Being that every site has an HTML document of a different size, we can use this information to make the packet trace more unique. We place an HTML marker after the HTML document is fully received.

* __Total Transmitted Bytes__- This involves adding up separately the total number of bytes sent and received at the end of the packet trace.

* __Number Markers__- These markers are similar to size markers, but instead mark how many packets came a certain direction before switching to the other direction.

* __Occurring Packet Sizes__- This marker keeps track of the number of unique packet sizes there were in the packet trace, and appended at the end of trace.

* __Percentage of Incoming Packets__- This step involves finding the percentage of packets that were incoming compared to outgoing

* __Total Number of Packets__- Similar to adding up the total number of bytes sent and received, the total number of packets is also counted.

## Results

In this experiment, we create fingerprints of the website homepages that we have stored on the webserver. We then create another fingerprint of the client's traffic when the client randomly chooses one of the five websites to visit. Using the fingerprints of the website homepages, we compare it with the fingerprint of the client's traffic to see if we can figure out which website the client visited.

When we create a fingerprint of a website, using the make-fingerprint.py python script that I provide, the script takes in a packet trace and filters out unneeded packets and places markers of different kinds in the trace, defined in the __Characteristics of Packet Traces__ section above. After this process is done, something similar to the following is outputted onto the display:

```
('S', 600), ('N', 1), ('+', 609), ('S', 600), ('N', 1), ('-', 609), ('S', 600), ('N', 1), ('+', 1123), ('S', 1200), ('N', 1), ('-', 2962), ('-', 283), ('S', 3600), ('N', 2), ('+', 609), ('S', 600), ('N', 1), ('-', 1123), ('S', 1200), ('N', 1), ('+', 609), ('S', 600), ('N', 1), ('-', 1637), ('S', 1800), ('N', 1), ('+', 609), ('+', 609), ('S', 1200), ('N', 2), ('-', 609), ('S', 600), ('N', 1), ('+', 609), ('S', 600), ('N', 1), ('-', 609), ('S', 600), ('N', 1), ('+', 609), ('S', 600), ('N', 1), ('-', 609), ('S', 600), ('TS+', 70000), ('TS-', 830000), ('OP+', 2), ('OP-', 20), ('PP-', '0.20'), ('NP+', 105), ('NP-', 480)]
```

This is a list of tuples, containing the type of data (whether a data packet or a marker) and the data itself. During this process two plots are created as visuals of the fingerprint for the comparison part of our experiment later. The plots are saved as a PNG image file, and should be stored somewhere for later. As an additional part of the fingerprint, a table is created, containing information about the other markers that are appended at the end of the list. 

![](/blog/content/images/2017/04/engineering-fingerprint-grid.png)
<center><i><small>Figure 1: Representative plot of a website "fingerprint"</small></i></center>

Figure 1 is an example of the fingerprint visual that is created when we visited the NYU Engineering homepage.

The first pane shows the size in bytes of each data packet transmitted from the client (positive) and received at the client (negative). The second pane shows the size markers that are placed in the traffic whenever the direction of traffic changes. These markers note how many bytes went a certain direction before going the other. The third plot shows the number markers, which are placed in the traffic whenever the direction of traffic changes. Number markers count how many packets went a certain direction before switching to the other direction.

![](/blog/content/images/2017/04/engineering-fingerprint-table.svg)
<center><i><small>Figure 2: Representative table showing relevant markers for a website "fingerprint"</small></i></center>

We also produce a table for each site, which puts together the rest of the markers that are placed at the ends of a packet instance, and not at regular intervals like the size markers and number markers. These markers include the HTML marker, the Total Transmitted Bytes markers, the Occurring Packet Sizes markers, the Percentage of Incoming Packets marker, and the Total Number of Packets markers.

![](/blog/content/images/2017/04/youtube-fingerprint-grid.png)
<center><i><small>Figure 3: Generated fingerprint for youtube.com</i></small></center>
![](/blog/content/images/2017/04/reddit-fingerprint-grid.png)
<center><i><small>Figure 4: Generated fingerprint for reddit.com</i></small></center>
![](/blog/content/images/2017/04/engineering-fingerprint-grid.png)
<center><i><small>Figure 5: Generated fingerprint for engineering.nyu.edu</i></small></center>
![](/blog/content/images/2017/04/facebook-fingerprint-grid-1.png)
<center><i><small>Figure 6: Generated fingerprint for facebook.com</i></small></center>
![](/blog/content/images/2017/04/mets-fingerprint-grid.png)
<center><i><small>Figure 7: Generated fingerprint for mlb.com/mets</i></small></center>

After generating these fingerprints, we can compare a fingerprint of an unknown site and identify which site it likely represents. For example, looking at this fingerprint of traffic between the client an an "unknown" site:

![](/blog/content/images/2017/04/unknown-fingerprint-grid.png)
<center><i><small>Figure 8: Generated fingerprint for an unknown site</i></small></center>

and comparing the client traffic fingerprint graphs to the other graphs, we can see a strong correlation between the client traffic fingerprint and the New York Mets fingerprint, which is the correct answer, as the client did visit the New York Mets homepage at mlb.com/mets.

## Run my experiment

For this experiment, the following steps will be taken:

1. Reserve resources and set up new a new private Tor network
2. Get the homepages of a number of sites onto the webserver
3. Create a fingerprint for each site
4. Hit a random site on the client, and see if there is a match with any of the fingerprints.

For each website's fingerprint as well as the client's traffic that we capture, we will be creating plots as visuals to compare the fingerprints.

First, reserve resources for the experiment. We will use the following topology:

![](/blog/content/images/2017/04/basic-tor-topology.svg)
<center><i><small>Figure 9: Topology of the private Tor network</small></i></center>

In the GENI Portal, create a new slice. Load the RSpec from the following URL: [https://raw.githubusercontent.com/tfukui95/tor-experiment/master/basic-tor-topology.xml](https://raw.githubusercontent.com/tfukui95/tor-experiment/master/basic-tor-topology.xml). 

After the topology is loaded onto your canvas, click "Site", and choose an InstaGENI site to reserve our resources from. Then press Reserve Resources. When all VMs are green in the slice page, meaning that the whole topology is ready to be used, we will continue on to our next step of installing Tor onto each VM.

### Setting up a new Private Tor Network

In my other blog called [Anonymous Routing of Network Traffic Using Tor](https://witestlab.poly.edu/blog/anonymous-routing-of-network-traffic-using-tor/), I provide a step by step guide to setting up a private Tor network. In this experiment, we will set up the VMs using the script files on my [Github page](https://github.com/tfukui95/tor-experiment), without explaining each step in detail. 

The order in which the VMs are set up is very important. Making sure that the directory server is set up first before any of the other nodes is vital for the private Tor network to be set up properly, as the Tor configuration files for the rest of the nodes are first created in the directory server. Once the directory server is set up, the order of setup of the other VMs is flexible. 

To setup the directory server, go to the directory server VM and run

```
wget https://raw.githubusercontent.com/tfukui95/tor-experiment/master/wfp-directoryserver.sh
bash wfp-directoryserver.sh
```
You will have to enter a password when prompted, then hit Enter, re-enter the password, and hit Enter again to finish the setup:

```
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```

After that is finished, let us setup the webserver, relays, and client VMs. On the
webserver run

```
wget https://raw.githubusercontent.com/tfukui95/tor-experiment/master/wfp-webserver.sh
bash wfp-webserver.sh
```

On each of the relays run

```
wget https://raw.githubusercontent.com/tfukui95/tor-experiment/master/wfp-router.sh
bash wfp-router.sh
```

Lastly on the client VM run

```
wget https://raw.githubusercontent.com/tfukui95/tor-experiment/master/wfp-client.sh
bash wfp-client.sh
```

Now that all the VMs are setup, reconfirm that the private Tor network is up and running by using Tor Arm on each of the nodes:

```
sudo -u debian-tor arm
```


### Setting up Websites on the Webserver

Now we will be setting up the webserver for our experiment. We will be saving homepages of 5 different website homepages onto our webserver, so that we can create fingerprints of these websites later. The following are the 5 websites:

1. NYU Tandon School of Engineering
2. Facebook
3. Youtube
4. Reddit
5. Official New York Mets Website

For each website, we will be storing the webpage in its own directory inside the /var/www/html/ directory of the webserver. We will first download the NYU Tandon School of Engineering homepage. On the webserver terminal, run 

```
cd /var/www/html/
sudo wget -e robots=off --wait 1 -H -p -k http://engineering.nyu.edu/
```

Now do the same for the other four websites.

```
sudo wget -e robots=off --wait 1 -H -p -k http://facebook.com/
sudo wget -e robots=off --wait 1 -H -p -k http://youtube.com/
sudo wget -e robots=off --wait 1 -H -p -k http://reddit.com/
sudo wget -e robots=off --wait 1 -H -p -k http://www.mlb.com/mets
```

Now we have all 5 websites set up on our webserver's html directory.

### Creating Website Fingerprints

Now we will be creating the fingerprints of each website by accessing the website's homepage that is stored on the webserver from the client node. First we must install __proxychains__, which we will be using with the wget function to use the Tor network to access the websites. 
```
sudo apt-get -y install proxychains
```

Another utility that we need is __tshark__, which is a network protocol analyzer similar to tcpdump. Log on to the router1 node, through which the client connects to the rest of the network, and run

```
sudo apt-get update
sudo apt-get -y install tshark
```

Also, in order to accurately identify the size of packets going through the network, we need to turn off [Large Send Offload](https://en.wikipedia.org/wiki/Large_send_offload). On the router1 node, run 

```
sudo ethtool -K eth1 gro off
sudo ethtool -K eth1 gso off
sudo ethtool -K eth1 tso off

sudo ethtool -K eth2 gro off
sudo ethtool -K eth2 gso off
sudo ethtool -K eth2 tso off
```

When we create the fingerprints of the websites, we will be creating plots to visualize our fingerprints to better compare with the client traffic that we will eventually capture. We will be using __seaborn__, a python visualization library that is based off the more common matplotlib. On the router1 terminal, run

```
sudo apt-get -y install python-pip python-dev libfreetype6-dev liblapack-dev libxft-dev gfortran
sudo pip install stem seaborn pandas

```

Next, in order to make our fingerprints, we must run our website packet captures through a filter, defined by a python script that we have written. This script takes in the packet trace and filters out unneeded packets and places markers of different kinds in the trace, defined in the __Characteristics of Packet Traces__ section above. This script will create the fingerprint out of the packet trace. The python script filter can be accessed on my [Github page](https://github.com/tfukui95/tor-experiment). On the router1 terminal, run

```
wget https://raw.githubusercontent.com/tfukui95/tor-experiment/master/make-fingerprint.py
```

to save the script. 

To run the experiment, we will need two terminals. One terminal on the client will be to access the websites that are located on the webserver, and then the other terminal on router1 will be to listen and capture the packet trace. After capturing the traffic we will place the trace through the filter using the python script in order to create the fingerprint.

Let us first test to make sure that everything is working. On router1 terminals, start listening with tshark by running

<pre>
sudo tshark -i <b>eth1</b> -n -f "host 192.168.3.100 and tcp and port 5000" -T fields -e frame.len -e ip.src -e ip.dst -E separator=,
</pre>

but in place of <b>eth1</b>, use the name of the interface on router1 that is connected to the client. (Use ifconfig to find the name of the interface that has IP address 192.168.3.1.)

On the client terminal access the webserver's default index.php page

```
sudo proxychains wget -qO- http://192.168.2.200
```

On the client terminal listening on the network using tshark, you should see a comma separated list, where the rows contain the packet size, the source of the packet, and the destination of the packet, respectively.

Now we are ready to build the fingerprints of each of the websites. Let us first start with the first website that we stored on our webserver, which is the NYU Engineering site. In order to be able to run the packet trace through the filter, we must save it as a CSV file. 

On the router1 terminal using tshark (and substituting the name of the network interface connected to the client), run

<pre>
sudo tshark -i <b>eth1</b> -n -f "host 192.168.3.100 and tcp and port 5000" -T fields -e frame.len -e ip.src -e ip.dst -E separator=, >engineering.csv
</pre>

On the client terminal, run

```
proxychains wget -p http://192.168.2.200/engineering.nyu.edu/
```

Now use Ctrl+C to stop the tshark from listening on the network, which will save the captured traffic into engineering.csv. In order to generate the fingerprint visualization with the python script, on the router1 terminal, run

```
python make-fingerprint.py --filename engineering.csv
```

A list of tuples, containing the type of data (whether a data packet or a marker) and the data itself, will be outputted onto the display. During this process several plots are created as visuals of the fingerprint for the  comparison part of our experiment later. The plots are saved as a PNG image file, and should be stored somewhere for later. As an additional part of the fingerprint, a table is created, containing information about the other markers that are appended at the end of the list. The name of each of the images depends on the website, for example, where <sitename> can refer to facebook. The file <sitename>-fingerprint-plot-grid.png contains the visualization (as shown in the [Results](#results) section), and the table shown in that section is saved as <sitename>-fingerprint-table.pdf.

Now we must do the same for the other four sites. For Facebook:

```
# On router1 terminal
# Substitute interface name as necessary
sudo tshark -i eth1 -n -f "host 192.168.3.100 and tcp and port 5000" -T fields -e frame.len -e ip.src -e ip.dst -E separator=, >facebook.csv

# On client terminal
proxychains wget -p http://192.168.2.200/facebook.com/

# After stopping tshark, on router1 terminal
python make-fingerprint.py --filename facebook.csv
```

Next for the fingerprint for Youtube:

```
# On router1 terminal
# Substitute interface name as necessary
sudo tshark -i eth1 -n -f "host 192.168.3.100 and tcp and port 5000" -T fields -e frame.len -e ip.src -e ip.dst -E separator=, >youtube.csv

# On client terminal
proxychains wget -p http://192.168.2.200/youtube.com/

# After stopping tshark, on router1 terminal
python make-fingerprint.py --filename youtube.csv

```

Next for the fingerprint for Reddit:

```
# On router1 terminal
# Substitute interface name as necessary
sudo tshark -i eth1 -n -f "host 192.168.3.100 and tcp and port 5000" -T fields -e frame.len -e ip.src -e ip.dst -E separator=, >reddit.csv

# On client terminal
proxychains wget -p http://192.168.2.200/reddit.com/

# After stopping tshark, on router1 terminal
python make-fingerprint.py --filename reddit.csv

```

Lastly, for the fingerprint for the Mets homepage:

```
# On router1 terminal
# Substitute interface name as necessary
sudo tshark -i eth1 -n -f "host 192.168.3.100 and tcp and port 5000" -T fields -e frame.len -e ip.src -e ip.dst -E separator=, >mets.csv

# On client terminal
proxychains wget -p http://192.168.2.200/www.mlb.com/mets

# After stopping tshark, on router1 terminal
python make-fingerprint.py --filename mets.csv

```

After we have made all of the fingerprints, we can use SCP (Secure Copy Protocol) to save all of our images from our remote Linux terminal to our local machine. Go to your GENI slice page, and click Details to get more information of your client VM. Open up a local terminal, and go to whichever folder you want to save your plots to. The SCP command for saving the image to your local machine is the following:

```
scp -P <port_number> <user>@<site_address>:~/*.png . 
```

Where *.png is a wildcard to indicate all of the png files that we have saved on our client terminal. (This command is run on your local Bash terminal.) Therefore for example in my case, I would need to run:

```
scp -P 32570 tef243@pc2.instageni.maxgigapop.net:~/*.png .
```

### Testing the Website Fingerprints

Now that we have our fingerprints for each of the sites, we will be snooping on the client of our private Tor network and see if we can figure out what site the client is visiting. We will follow this process:

1. Start listening on the client's network
2. Have the client randomly choose one of the 5 sites
3. Capture the traffic that we can see, and create a similar fingerprint plot
as the one that we made for the websites.
4. See if we can determine which site the client visited

Now let us start our experiment. On the router1 terminal, run (with appropriate substitution of interface name, if not eth1)

<pre>
sudo tshark -i <b>eth1</b> -n -f "host 192.168.3.100 and tcp and port 5000" -T fields -e frame.len -e ip.src -e ip.dst -E separator=, >wfpAttack.csv
</pre>

There is a script file on my [Github page](https://github.com/tfukui95/tor-experiment) called randomSite.sh, that will have the client randomly visit one of the five websites. Save the script file onto the client VM and then run it

```
wget https://raw.githubusercontent.com/tfukui95/tor-experiment/master/randomSite.sh
bash randomSite.sh
```

The process to create the fingerprint of the user's traffic is the same as before, when we made the fingerprints for the websites. After stopping tshark, on the router1 terminal, run

```
python make-fingerprint.py --filename wfpAttack.csv
```

Use SCP to copy the images to your local machine. Open up the plots, and compare it with the other fingerprints. See whether it is clear or not which site the client randomly chose to visit. We notice that for certain sites it is much easier to distinguish from the rest, for example for the NYU Engineering homepage, we can see that towards the end of the retrieval of the webpage we see a lot of constant back and forth between client and server compared to the first half where there are a lot of packets sent in bunches from the webserver. Other sites are not as easy to distinguish, but looking closely, we can see that every site's fingerprint has some kind of unique part to it which differentiates it from the rest of them.

Go back to the client terminal in which you ran the shell script, which should be waiting for a keyboard input saying "Press any key to see which site the client visited:". After you arrive upon a guess as to which site the client visited, press any key on the keyboard to see whether your guess was correct.

This is the end of the experiment portion of the thesis. To expand on this experiment, feel free to increase the number of sites, clients, marker types, etc. to make the experiment even more interesting and realistic. Using Wireshark to gain a better understanding of the packet traces is also recommended. Wireshark can be downloaded from the company's [homepage](https://www.wireshark.org/). Besides downloading and installing the software, there is often no configuration necessary.

Wireshark's default file extension is a pcap file, which the Tcpdump function can save its traffic to by the following:

```
sudo tcpdump -s 1514 -i any 'port <port_num>' -U -w - | tee <name_file>.pcap | tcpdump -nnxxXSs 1514 -r -
```

This will tell the function to listen on a certain port number, output the traffic onto the display, and to save it to a pcap file with the name that you specify. 

## References

[1] A. Panchenko, L. Niessen, A. Zinnen, and T. Engel, “Website Fingerprinting in Onion Routing Based Anonymization Networks,” in Proceedings of the 10th Annual ACM Workshop on Privacy in the Electronic Society, New York, NY, USA, 2011, pp. 103–114.