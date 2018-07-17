<div class="st-alert">
<b>Note</b>: This post is preserved for reference; however, the WiMAX base station at WITest is not currently operational, so this experiment will not run as described here. It may be possible to run this experiment with some modifications on another testbed.
</div>


This experiment shows how adaptive modulation and coding profiles are assigned to wireless clients in a (WiMAX) cellular network. 

It should take about 120 minutes to run this experiment, but you will need to have [reserved that time](http://witestlab.poly.edu/respond/sites/witest/tutorial/make-a-reservation) in advance. This experiment uses wireless resources (specifically, on the [WITest](http://witestlab.poly.edu/) testbed), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have [reserved time on the WITest testbed](http://witestlab.poly.edu/respond/sites/witest/tutorial/make-a-reservation), and you must run this experiment during your reserved time.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

Wireless systems are a time-varying environment. If the clients are mobile, then the quality of the link changes as client move away from and towards the base station or access point. Even if the clients don't move, fading causes the wireless signal to vary in time.

If the wireless system settings are tuned to particular link characteristics of the wireless link, then these settings should also change when the link changes. Also, when the client connects initially, the correct settings should be used based on the quality of the link.

In this experiment, the wireless setting we'll adapt to channel conditions is the modulation and coding scheme (MCS) used to transmit data over the wireless link. In wireless communication, we want to transmit as much data as possible, as fast as possible, over as little bandwidth as possible. The quantity we are trying to maximize is called spectral efficiency, and is measured in bits per second per Hz (i.e., the number of bits that can be transmitted in one second over one Hz of bandwidth).

However, it's often not possible to change one quantity without affecting the others. For example, changing the symbol rate increases the rate at which we can send data, but it also increases the bandwidth required.

In a digital communication system, digital information is processed using modulation and coding before it is transmitted. The choice of modulation and coding techniques involves a tradeoff between spectral efficiency and resilience to error, which we'll discuss further in this section.

### Modulation

Before transmitting data, every bit is translated into one of a set of waveforms, also called symbols. This process is called modulation.

One way to reduce the time required to transmit a given quantity of data without increasing bandwidth is to increase the number of bits carried on one symbol. Carrying more than one bit per symbol reduces the time required to transmit a given quantity of data over a limited bandwidth.

The following images shows two different modulation schemes that carry more than one bit on a symbol. This kind of image is called a constellation diagram. The position of the symbol on the plot indicates the phase and amplitude of the carrier signal for that symbol. Each point is also labeled with the sequence of bits represented by that signal.

![](/blog/content/images/2016/03/constellation.svg)

In the constellation diagram, the number of points (symbols) is called the alphabet size. The alphabet size of a modulation scheme describes how many symbols are used to carry data, which in turn determines how many bits can be carried on one symbol.If a modulation alphabet has

$$ M = 2^n $$

symbols, then each symbol can carry _n_ bits of data. With a symbol rate of _R_ symbols per second, the modulation scheme can then transmit data at

$$ R \; log\_{2}(M) = R \; log\_{2}(2^n) $$

bits/second.

In practice, we can't use an arbitrarily large alphabet size because large alphabets are susceptible to noise and interference on the wireless link. Modulation schemes with a smaller alphabet size have symbols that are farther apart (given the same maximum power), and are therefore easier to differentiate from one another in the presence of noise. 

### Coding

In addition to modulation, data is also encoded at the physical layer using an error-correction code. This allows bits that are received in error (due to interference or noise on the link) to be corrected at the receiver. However, it also adds some redundant bits to each transmission, which means that the rate of the message data is reduced.

The total efficiency of a combined modulation and coding scheme (MCS) is equal to the spectral efficiency of the modulation scheme, multiplied by the code rate.

The code rate is the ratio of the size of the original message to the size of the encoded message. For example, in a 1/2 code, 2 coded bits (one redundant bit and one data bit) are used to send every 1 bit of data. In a 5/6 code, 6 coded bits are used to send every 5 bits (one redundant bit and five data bits).

A high code rate is more efficient, since we do not need to send as many encoded bits to get the same message through. However, a code with a high rate cannot correct as many errors as a code with a low rate.

### Adaptive modulation and coding in cellular systems

In cellular systems that use adaptive modulation and coding, the base station collects measurements of link characteristics (such as link quality, or bit error rate) from every mobile device at regular intervals. Then, it assigns a suitable MCS to each device; one that has the desired balance of spectral efficiency and error resilience for the link state observed by that device.

Consider the following simplified example: A mobile device that starts off near the base station with a high-quality signal is assigned 64QAM 5/6 for its MCS. This is an MCS with a very high spectral efficiency, but low resilience to error, so it can only be used on a link with a low probability of error. As the mobile device moves away from the base station, its signal quality degrades, probability of error increases, and its MCS is changed to 16QAM 3/4, which is better able to cope with channel errors. Near the edge of the cell, where signal quality is very poor, the device might be assigned QPSK 1/2, which has a low spectral efficiency but works effectively even over a noisy channel.

In the WiMAX cellular system used in this experiment, the following modulation and coding schemes are supported for the downlink path (for transmissions from the base station to the mobile device). Each MCS is identified by a profile number, listed in parentheses:

<table id="table-1" class="table table-striped table-bordered col-2" data-columns="2"><thead>
<tr><th class="col-1">MCS (Profile Number)</th><th class="col-2">Code Rate x Log2(Alphabet Size) = Bits/second per Hz </th></tr></thead>
<tbody>
<tr class="row-1"><td class="col-1">QPSK 1/2 (13)</td><td class="col-2">1/2 log2 (4) = 1</td></tr>
<tr class="row-2"><td class="col-1">QPSK 3/4 (15)</td><td class="col-2">3/4 log2 (4)  = 1.5 </td></tr>
<tr class="row-3"><td class="col-1">16QAM 1/2 (16)</td><td class="col-2">1/2 log2 (16)  = 2 </td></tr>
<tr class="row-4"><td class="col-1">16QAM 3/4 (17)</td><td class="col-2">3/4 log2 (16)  = 3 </td></tr>
<tr class="row-5"><td class="col-1">64QAM 1/2 (18)</td><td class="col-2">1/2 log2 (64)  = 3 </td></tr>
<tr class="row-6"><td class="col-1">64QAM 2/3 (19)</td><td class="col-2">2/3 log2 (64) = 4 </td></tr>
<tr class="row-7"><td class="col-1">64QAM 3/4 (20)</td><td class="col-2">3/4 log2 (64) = 4.5 </td></tr>
<tr class="row-8"><td class="col-1">64QAM 5/6 (21)</td><td class="col-2">5/6 log2 (64) = 5 </td></tr></tbody>
</table>

Note that using the highest MCS profile, we can receive data 5 times as fast as in the lowest MCS profile, without increasing bandwidth!


On the WiMAX testbed used in this experiment, every mobile device sends its CINR to the base station at regular intervals. CINR is a kind of signal to noise ratio; it is the ratio (measured in dB) of the received carrier signal power to the combined power of the noise and interference seen at that device. A high CINR indicates that the carrier power is much greater than the noise power; a low CINR indicates that the noise power is almost as high as the carrier power.

Given the CINR of a device, the base station consults a lookup table to decide what MCS to assign to that device. The lookup table lists a CINR threshold for every MCS profile. A MCS is assigned to a device if its CINR is (approximately) greater than or equal to the threshold for that MCS, but lower than the threshold for the next MCS. For example, if the threshold for profile QPSK 3/4 is 14 dB and the threshold for 16QAM 1/2 is 17 dB, a mobile device with a CINR of 15 dB will use QPSK 3/4; if its CINR jumps to 17 dB, it will adapt and begin using 16QAM 1/2.  (This is only an approximation, since the minimum time interval between MCS changes may be different from the CINR reporting interval.) 


## Results

Over approximately 60 seconds, we measured the downlink CINR and adaptive modulation and coding profile for four nodes as follows (mouse over any point to see the modulation and coding profile at that point):


##### Figure 1

<iframe width="650" height="500" frameborder="0" scrolling="no" src="https://plot.ly/~ffund/5.embed"></iframe>


Clients with a better signal quality are generally assigned a more aggressive modulation and coding scheme.

For the uplink direction, link adaptation was turned off and in our experiment, all nodes were assigned a modulation and coding profile of QPSK 1/2 regardless of their link quality:

##### Figure 2

<iframe width="650" height="500" frameborder="0" scrolling="no" src="https://plot.ly/~ffund/8.embed"></iframe>

We also ran some experiments to see how adaptive modulation and coding improves throughput, compared to using a single static modulation and coding scheme for all wireless devices. 

We measured downlink TCP throughput for each of four wireless nodes under three different circumstances: with adaptive modulation and coding, with a static modulation and coding profile of QPSK 1/2, and with a static modulation and coding profile of 16QAM 3/4.  

##### Figure 3

<iframe width="650" height="500" frameborder="0" scrolling="no" src="https://plot.ly/~ffund/10.embed"></iframe>

The results show that this implementation of adaptive modulation and coding generally improves throughput (blue boxes in image above) compared to using a single very conservative MCS (orange) where nodes cannot achieve such high throughput due to low spectral efficiency, or compared to using a single very aggressive MCS (green) under which nodes with moderate or poor link quality have a very high bit error rate that degrades TCP performance. 

## Run my experiment

This experiment has several parts:

* [Check MCS settings on the WiMAX base station](#checkmcssettingsonthewimaxbasestation)
* [Set up resources](#setupresources)
* [Measure CINR and MCS at each node](#measurecinrandmcsateachnode)
* [Measure throughput of different MCS profiles](#measurethroughputofdifferentmcsprofiles)

The first part, [Check MCS settings on the WiMAX base station](#checkmcssettingsonthewimaxbasestation), runs entirely on the WITest terminal and does not require any of the "nodes". To save time, you may choose to start by loading the disk image onto your nodes in the [Set up resources](#setupresources) section, and complete the [Check MCS settings on the WiMAX base station](#checkmcssettingsonthewimaxbasestation) in a second terminal while that is running.

### Check MCS settings on the WiMAX base station

SSH into witestlab.poly.edu at the beginning of your reserved time. (If you were already logged in at the beginning of your reserved time, log out and then log back in again.)

We will query the current base station settings using an HTTP interface, which will return information in XML stanzas.

First, find out whether adaptive modulation and coding is enabled on the downlink path (from the base station to the mobile device) by running the following command on the WITest console:

```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/get?la_dl" | xml_pp
```

The result should be a "1", indicating that link adaptation on the downlink ("la\_dl") is enabled:

```xml
<STATUS>
  <BaseStation>
    <la_dl>
      <la_dl>1</la_dl>
    </la_dl>
  </BaseStation>
</STATUS>
```

A value of 1 means that link adaptation is enabled; a value of 0 means it is disabled.

Next, check which modulation and coding profiles are enabled for the downlink path:

```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/dlprofile" | xml_pp | grep "value"
```

Note that only a subset of the modulation and coding profiles defined in [the table above](#table-1) may actually be enabled. For example, you may find that only QPSK 1/2, QPSK 3/4, 16QAM 1/2, and 16QAM 3/4 are enabled, as in the following output:

```xml
      <value>No Modulation (255)</value>
      <value>No Modulation (255)</value>
      <value>No Modulation (255)</value>
      <value>No Modulation (255)</value>
      <value>No Modulation (255)</value>
      <value>No Modulation (255)</value>
      <value>No Modulation (255)</value>
      <value>No Modulation (255)</value>
      <value>QPSK (CTC) 1/2 (13)</value>
      <value>QPSK (CTC) 3/4 (15)</value>
      <value>16-QAM (CTC) 1/2 (16)</value>
      <value>16-QAM (CTC) 3/4 (17)</value>
```

Finally, find out what minimum downlink CINR is necessary to for the base station to assign each modulation and coding profile to a node:

```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/get?mpc" | xml_pp | grep "dl_cn_list\."
```

The representative output shown below (together with the list of enabled MCS profiles above) indicates that clients with a DL CINR less than 14 will use QPSK 1/2, clients with a DL CINR of 14-17 will use QPSK 3/4, clients with a DL CINR of 18-21 will use 16QAM 1/2, and clients with a DL CINR of 22 or higher will use 16QAM 3/4. (Note that since 64QAM profiles were not enabled, according to the output of the previous commands, even clients with a very high DL CINR will still be assigned 16QAM 3/4.)

```xml
      <dl_cn_list.0>8 (QPSK 1/2 x4)</dl_cn_list.0>
      <dl_cn_list.1>9 (QPSK 1/2 x2)</dl_cn_list.1>
      <dl_cn_list.2>10 (QPSK 1/2)</dl_cn_list.2>
      <dl_cn_list.3>14 (QPSK 3/4)</dl_cn_list.3>
      <dl_cn_list.4>18 (16QAM 1/2)</dl_cn_list.4>
      <dl_cn_list.5>22 (16QAM 3/4)</dl_cn_list.5>
      <dl_cn_list.6>23 (64QAM 1/2)</dl_cn_list.6>
      <dl_cn_list.7>25 (64QAM 2/3)</dl_cn_list.7>
      <dl_cn_list.8>27 (64QAM 3/4)</dl_cn_list.8>
      <dl_cn_list.9>28 (64QAM 5/6)</dl_cn_list.9>
```

These values can be configured by the experimenter on this testbed, but for our experiment we will leave them as is.


Next, repeat these steps for the uplink (UL) path, from the mobile device to the base station. Find out if link adaptation is enabled:

```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/get?la_ul" | xml_pp
```

And find out which MCS profiles are enabled for UL traffic:

```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/ulprofile" | xml_pp | grep "value"
```

The uplink and downlink paths are handled separately, since their behavior is very different. Notably, the power at which the base station transmits to the wireless device is very high compared to the power at which the mobile device transmits to the base station, and so the DL CINR is generally much higher than the UL CINR. You can find out the transmit power of the base station (in dBm) by running the following on the WITest console: 

```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/get?bs_tx_power" | xml_pp
```


### Set up resources

The other parts of this  experiment require four wireless "nodes". The instructions here assume you are using node7, node2, node3, and node4. These nodes were chosen because they are physically distributed in such a way that they have a range of DL CINR values, which makes them interesting for this experiment. If any of those nodes are [not available](http://witestlab.poly.edu/site/activity/node-status) at the time of your reservation, or if any of the nodes fail to connect to the cellularreplace them with other nodes of those numbered 1-13, and modify the instructions accordingly. 


Load the "wmxlab-mcs" disk image onto your nodes with the following command:

```
omf load -i wmxlab-mcs.ndz -t omf.witest.node7,omf.witest.node2,omf.witest.node3,omf.witest.node4
```

When the disk load process finishes successfully, turn the nodes on:

```
omf tell -a on -t omf.witest.node7,omf.witest.node2,omf.witest.node3,omf.witest.node4
```


Use the [node status page](http://witestlab.poly.edu/respond/sites/witest/activity/node-status) to find out when your nodes are ready to log in (when they turn green). This may take a few minutes.

Then, open four terminals, and in each, SSH into the witestlab.poly.edu console and from there, into the console of each of your nodes:

```
# run this on WITest console
ssh root@nodeX
# where X is the node number, e.g. node4
```

Finally, on each node, run

```
wimaxcu connect network 51
```

to connect to the WiMAX network. Verify that each node connects successfully.

### Measure CINR and MCS at each node

Now we're going to run an experiment that reports the WiMAX signal quality and assigned modulation and coding scheme for each of our wireless nodes.

For convenience, we are going to use a tool called [OMF](https://mytestbed.net/projects/omf). This allows us to use an "experiment script" (written in a Ruby-like language) to define and execute our experiment, instead of manually configuring and running applications on each node ourselves. The experiment script is provided for you.

On the WITest console, save the following OMF experiment script into a file called "mcs.rb" (e.g. using nano or vim) and edit the fourth line to list the nodes that your experiment will run on:

```ruby
defProperty('prefix', "omf.witest.", "Prefix for node names")

# Edit this array to list the nodes that your experiment will run on
nodes = ["node7", "node2", "node3", "node4"]

# Code to set up configuration and apps to run on each node
nodes.each do |n|
  defGroup("#{n}", "#{property.prefix}#{n}") do |node|
    # Connect to WiMAX network with network ID 51
    node.net.x0.profile = '51'
    # Run application to query and report MCS, with the following arguments
    # Details of this application are in the "application wrapper" at
    # /usr/share/omf-expctl-5.3/repository/test/app/mcs_app.rb    
    node.addApplication("test:app:mcs_app") do |app|
      app.setProperty('times', 15)
      app.setProperty('url',"http://10.0.0.200:5053/result/queryDatabase")
      app.measure('dl')
      app.measure('ul')
    end
  end
end       

# Start experiment when all nodes are online
onEvent(:ALL_UP_AND_INSTALLED) do |event|
  wait 10
  # Start all the applications on all nodes
  allGroups.startApplications
  wait 60
  # Stop all the applications on all nodes
  allGroups.stopApplications
  Experiment.done
end
```

Once you have saved this file, run (from the same directory on the WITest console where "mcs.rb" is saved)

```
omf exec mcs.rb
```

to start the experiment.

Don't worry if you see lots of error messages like

```
ERROR NodeHandler:   The resource 'omf.witest.node3' reports that an error occured 
ERROR NodeHandler:   while running the application 'test:app:mcs_app#3'
ERROR NodeHandler:   The error message is ' INFO oml4r: Collection URI is tcp:omfserver-witest.poly.edu:3003'
ERROR NodeHandler: 
ERROR NodeHandler:   The resource 'omf.witest.node4' reports that an error occured 
ERROR NodeHandler:   while running the application 'test:app:mcs_app#4'
ERROR NodeHandler:   The error message is ' WARN oml4r: opts[:omlServer] and ENV['OML_SERVER'] are getting deprecated; please use opts[:collect] or ENV['OML_COLLECT'] instead'
ERROR NodeHandler: 
ERROR NodeHandler:   The resource 'omf.witest.node7' reports that an error occured 
ERROR NodeHandler:   while running the application 'test:app:mcs_app#1'
ERROR NodeHandler:   The error message is ' INFO oml4r: Collection URI is tcp:omfserver-witest.poly.edu:3003'
ERROR NodeHandler: 
ERROR NodeHandler:   The resource 'omf.witest.node4' reports that an error occured 
ERROR NodeHandler:   while running the application 'test:app:mcs_app#4'
ERROR NodeHandler:   The error message is ' INFO oml4r: Collection URI is tcp:omfserver-witest.poly.edu:3003'
```

in the experiment output.  These are not fatal errors; *anything* that appears on STDERR of the node will be reported as an "error." These particular "errors" are just informational messages that appear on STDOUT whenever the applications defined in this experiment run, and are no cause for concern.  

When your experiment finishes, the last line of output on your terminal will be something like


```
 INFO run: Experiment ffund-default_slice-2016-03-27t19.34.52-04.00 finished after 1:21
```

This messages includes your *experiment ID*, which you will need in order to retrieve your experiment results. In the output above, the experiment ID is "ffund-default_slice-2016-03-27t19.34.52-04.00". You will have a different experiment ID each time you run the OMF experiment.

Your experiment data is saved in an sqlite3 database named according to your experiment ID, and saved in the /var/lib/oml2 directory on the WITest console.

To get your downlink MCS data in CSV format, run

```
sqlite3 -csv /var/lib/oml2/EXPERIMENT-ID.sq3 "SELECT * FROM mcs_dl;"
```

substituting your own experiment ID in the command above.

To get your uplink MCS data in CSV format, run

```
sqlite3 -csv /var/lib/oml2/EXPERIMENT-ID.sq3 "SELECT * FROM mcs_ul;"
```

Each of these commands will produce comma-separated lines of output, with the following fields (in order):

**Sample ID, Unique ID of node, Sequence of sample from this node, Timestamp on client, Timestamp on measurement server, Modulation and coding profile, CINR (dB), Modulation and coding profile number**

You can use a tool such as [plotly](https://plot.ly/) to create [Figure 1](#figure1) and [Figure 2](#figure2) as shown in the [Results](#results) section.  In those plots, we put 

 * the timestamp on the x-axis (either timestamp), 
 * the modulation and coding profile number on the y-axis, 
 * the unique ID of node as the "group by" variable, 
 * and the modulation and coding profile text as the text to show on mouseover, 

like this:

![](/blog/content/images/2016/03/plotly-screenshot1.png)


### Measure throughput of different MCS profiles

Adaptive modulation and coding helps us to achieve a reasonably high data rate (by using a modulation and coding scheme with a high spectral efficiency), while still keeping our bit error rate low.

In this part of the experiment, we'll measure the TCP throughput to each of our four nodes with adaptive modulation and coding, and then with two different static modulation and coding schemes: one that is very conservative and one that is more aggressive. We expect to see a tradeoff. On the one hand, a more aggressive MCS allows a higher data rate, which translates to higher throughput; on the other hand, it also increases bit error rate, which degrades TCP throughput.

Open four terminals. In each terminal, SSH to the WITest console, then onto one of your four nodes.


On the terminal of each node, check if you are still connected to the WiMAX network:

```
wimaxcu status link
```

and reconnect if it says "Network is not connected.":
```
wimaxcu connect network 51
n=$(hostname)  
n=${n:4}  
ifconfig wmx0 10.43.4.$n netmask 255.255.255.0 
```

Then, start an iperf receiver on each node with
```
iperf -s -i 1
```

Send a four-second TCP flow to each node, e.g. if you are using node2, node3, node4 and node7:


```
# to node2
iperf -c 10.43.4.2 -t 4
# to node3
iperf -c 10.43.4.3 -t 4
# to node4
iperf -c 10.43.4.4 -t 4
# to node7
iperf -c 10.43.4.7 -t 4
```
and record the throughput. 

TCP throughput is highly variable, so you may want to perform additional replications in order to get a meaningful average. (For the image shown in the [Results](#results) section, I performed five replications for each experiment.)


Next, we'll disable all DL modulation and coding profiles except for QPSK 1/2, so that all nodes use a static DL MCS of QPSK 1/2:

```
# Set first MCS profile to 13 (QPSK 1/2)
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/dlprofile?dlprof1=13"
# disable all others
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/dlprofile?dlprof2=255&dlprof3=255&dlprof4=255&dlprof5=255&dlprof6=255&dlprof7=255&dlprof8=255"
```

Wait about 30 seconds for the new setting to take effect. Then repeat the "iperf" commands and record the results.

Finally, we'll disable all DL modulation and coding profiles except for 16QAM 3/4, so that all nodes use 16QAM 3/4 as their static DL MCS:

```
# Set first MCS profile to 17 (16QAM 3/4)
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/dlprofile?dlprof1=17"
# disable all others
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/dlprofile?dlprof2=255&dlprof3=255&dlprof4=255&dlprof5=255&dlprof6=255&dlprof7=255&dlprof8=255"
```

Wait about 30 seconds for the new setting to take effect. Then repeat the "iperf" commands and record the results.


To restore the original adaptive modulation and coding settings with four profiles enabled, run


```
wget -qO- "http://wimaxrf:5052/wimaxrf/bs/dlprofile?dlprof1=13&dlprof2=15&dlprof3=16&dlprof4=17&dlprof5=255&dlprof6=255&dlprof7=255&dlprof8=255"
```

Use the data from this experiment to create a plot comparing the TCP throughput for each of the three scenarios (like [Figure 3](#figure3) in the [Results](#results) section, which was created in [plot.ly](http://plot.ly/)).
