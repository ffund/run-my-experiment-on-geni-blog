This experiment looks at the relationship between data transmission rate, bandwidth, and modulation scheme, as described by the Nyquist formula.

It should take about 60-120 minutes to run this experiment, but you will need to have reserved that time in advance. This experiment uses wireless resources (specifically, either the [WITest](http://witestlab.poly.edu) testbed or any one of the sb2, sb3, or sb7 sandboxes at [ORBIT](http://geni.orbit-lab.org)), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have reserved time on either [WITest](https://witestlab.poly.edu/respond/sites/witest/activity/reservation-calendar) or a sandbox at [ORBIT](http://geni.orbit-lab.org) (either sb2, sb3, or sb7), and you must run this experiment during your reserved time. 

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

The Nyquist formula gives the upper bound for the data rate of a transmission system by calculating the bit rate directly from the number of signal levels and the bandwidth of the system.

Specifically, in a noise-free channel, Nyquist tells us that we can transmit data at a rate of up to

$$ C = 2B\;log\_2\;M$$

bits per second, where _B_ is the bandwidth (in Hz) and _M_ is the number of signal levels.


Nyquist is only an upper bound, and on the baseband signal bandwidth - the occupied transmission bandwidth for a wireless signal will further depend on how the signal is modulated onto a carrier frequency for wireless transmission. In this experiment, we will use [PSK modulation](https://en.wikipedia.org/wiki/Phase-shift_keying), a digital modulation scheme in which the phase of a carrier signal is varied to represent different bits, or different groups of bits, and there are a discrete number of signal "levels" represented by different phase shifts. In our experiment, the modulated wireless signal at RF will occupy a transmission bandwidth that is **double** the Nyquist bandwidth (at baseband):

![](/blog/content/images/2017/02/Baseband_to_RF.svg)
<small><i>Image: Original bitmap version by Splash, SVG version by Qef. [CC-BY-SA-3.0](http://creativecommons.org/licenses/by-sa/3.0/), via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Baseband_to_RF.svg)</i></small> 

The basic relationships implied by Nyquist will still hold:

* Doubling the data rate (_C_) and keeping the number of signal levels (_M_) the same, will double the bandwidth used (_B_), and
* Squaring the number of signal levels (_M_) and keeping the data rate (_C_) the same, will halve the bandwidth used (_B_).

In this experiment, we will send a constant amount of data over a wireless channel, with varying data rates (_C_) and number of signal levels (_M_). We will observe the effect of these variations on two metrics:

1. The total time required to transfer the data, and 
2. The transmission bandwidth.

We expect to see that there are two ways to increase the speed of data transmission: using more bandwidth, or using more signal levels. 


## Results

When we send 5 megabytes (40 Mbits) of data at a rate of 0.5 Mbps, using BPSK (2 signal levels), we see that about 0.5 MHz of bandwidth is used, and the total transmission takes a little over 80 seconds (the "ticks" on the display are 250 kHz apart):

<iframe width="560" height="315" src="https://www.youtube.com/embed/IRYOEaa4rNs" frameborder="0" allowfullscreen></iframe>

However, if we change the bitrate to 2 Mbps, 2 MHz of bandwidth is used and the transmission takes about 20 seconds. To reduce the speed of data transfer by a factor of 4, we had to increase the bandwidth by a factor of 4:


<iframe width="560" height="315" src="https://www.youtube.com/embed/av2rvkXBVm0" frameborder="0" allowfullscreen></iframe>

Finally, on changing the constellation size to 4 points (squaring the number of signal levels relative to the first transmission), the transmission also takes about 20 seconds, but uses only 1 MHz, when transmitting at 2 Mbps:


<iframe width="560" height="315" src="https://www.youtube.com/embed/oxjWFkrwE9o" frameborder="0" allowfullscreen></iframe>


## Run my experiment

To run this experiment, you will need a reservation on either the WITest testbed, or the sb2, sb3, or sb7 testbed on ORBIT. You will have to make your reservation in advance.

* To reserve WITest, visit [http://witestlab.poly.edu](http://witestlab.poly.edu). Click on the profile icon in the top right, then on "Log in with GENI". Log in to your GENI Portal account and agree to send your information to WITest. Then, use the [reservation calendar](https://witestlab.poly.edu/site/activity/reservation-calendar) to reserve one or two (consecutive) hours for this experiment. For further information, refer to this [tutorial on the reservation system](https://witestlab.poly.edu/site/tutorial/make-a-reservation).
* To reserve an ORBIT sandbox, visit [http://geni.orbit-lab.org](http://geni.orbit-lab.org). Click on "Log in", then log in using your GENI Portal account and agree to send your information to ORBIT. Then, click on "Control Panel". Use the calendar interface to request time on sb2, sb3 or sb7.

### Set up testbed

At your reserved time, open a terminal and log in to the console of the testbed that you have reserved. For example, if you have reserved sandbox 3 on ORBIT, 

```
ssh GENI-WIRELESS-USERNAME@sb3.orbit-lab.org -i /PATH/TO/KEY
```

where `GENI-WIRELESS-USERNAME` is your wireless username assigned by GENI. This is usually your regular GENI username with a `geni-` prefix, e.g. `geni-ffund`. Also specify the path to the key you have uploaded to the GENI Portal as the `/PATH/TO/KEY`.

If you are using sandbox 7 or sandbox 2 on ORBIT, log in to sb7.orbit-lab.org or sb2.orbit-lab.org, instead. If you are using WITest, log in to witestlab.poly.edu.

Then, you must load a disk image onto the testbed nodes. From the testbed console, run:

* If you are using WITest (note that there is no space around the comma):
```
omf-5.4 load -i gr-nyquist.ndz -t omf.witest.node16,omf.witest.node22
```

* If you are using sb2, sb3, or sb7 on ORBIT:

```
omf load -i gr-nyquist.ndz -t system:topo:all
```

This disk image has the [GNU Radio](http://gnuradio.org/) software suite, and the [ShinySDR spectrum analyzer](https://github.com/kpreid/shinysdr/), both of which we'll use for this experiment, pre-installed. 

This process can take 5-10 minutes. Don't interrupt it in middle - you'll just have to start again, and it will only take longer. 

If it's been successful, then once the process finishes running completely you should see output similar to:

```
 INFO exp:  ----------------------------- 
 INFO exp:  Imaging Process Done 
 INFO exp:  2 nodes successfully imaged - Topology saved in '/tmp/omf-pxe_slice-2014-02-04t16.38.52-05.00-topo-success.rb'
 INFO exp:  ----------------------------- 
```

Sometimes, transient errors can cause the process to fail - if you haven't successfully imaged 2 nodes, wait a few minutes and try again. 


Then, turn on your nodes with the following command:

* If you are using WITest (note that there is no space around the comma):
```
omf tell -a on -t omf.witest.node16,omf.witest.node22
```

* If you are using sb2, sb3, or sb7 on ORBIT:

```
omf tell -a on -t system:topo:all
```



Wait a few minutes for your testbed nodes to turn on, then continue with the experiment.


### Prepare your receiver

Open a new terminal window, and run the following command to tunnel the ShinySDR ports between your laptop and the receiver node:

* If you are using WITest (note: this is all one line):

```
ssh -L 8100:node22:8100 -L 8101:node22:8101 GENI-WIRELESS-USERNAME@witestlab.poly.edu -i /PATH/TO/KEY
```


* If you are using sb2, sb3, or sb7 on ORBIT (note: this is all one line):

```
ssh -L 8100:node1-2:8100 -L 8101:node1-2:8101 GENI-WIRELESS-USERNAME@sb3.orbit-lab.org -i /PATH/TO/KEY
```

again, using the correct `GENI-WIRELESS-USERNAME`, `/PATH/TO/KEY`, and substituting sb3.orbit-lab.org for the hostname of the console of the testbed that you have reserved.

Then, in that terminal window (which should now be logged in to your testbed console), log on to the receiver node:

* If you are using WITest:

```
ssh root@node22
```

* If you are using sb2, sb3, or sb7 on ORBIT:

```
ssh root@node1-2
```

On the receiver node, run


```
cd ~/shinysdr

# Run shiny server
python -m shinysdr.main ~/.shiny
```

This last command should start the Shiny server, which, when it is running successfully, will say something like:

```
INFO:shinysdr:ShinySDR is ready.
INFO:shinysdr:Visit http://localhost:8100/ShinySDR/
```

In a [Google Chrome](https://www.google.com/chrome/browser/) browser window (should be a recent version of Chrome), open the URL that is shown in the Shiny server output. (http://localhost:8100/ShinySDR/).

Configure your Shiny window as follows:

<iframe width="560" height="315" src="https://www.youtube.com/embed/YT7hchUqN4U" frameborder="0" allowfullscreen></iframe>

* Click on the "hamburger" icon in the top left corner to open the menu, if it isn't already open.
* Un-select the "Frequency DB" display to hide that display if it is showing.
* Select the "Radio Config" display if it isn't showing.
* In the "Radio Config" section, set the "RF Source" to "OsmoSDR", the "Antenna" to "TX/RX", and the "Center Frequency" to 2,480,000,000 (2.48 GHz). 
* Note the "Gain" slider in the "Radio Config" section. You may have to increase the gain later if you aren't able to see the transmission in the ShinySDR window.
* Check the "Use DC Cancellation" box, but don't worry if it doesn't stay checked. This setting is not strictly required.
* Use the "Options" section to adjust the dynamic range and reference level of the visualization so that you can see the noise floor and there is about 60-80 dB of range.
* Click on the "hamburger" again to close the menu.

### Prepare your transmitter

In a second terminal window, log in to the node that will act as transmitter. First, log in to the testbed console:

* If you are using WITest:

```
ssh GENI-WIRELESS-USERNAME@witestlab.poly.edu -i /PATH/TO/KEY
```


* If you are using sb2, sb3, or sb7 on ORBIT:

```
ssh GENI-WIRELESS-USERNAME@sb3.orbit-lab.org -i /PATH/TO/KEY
```

again, using the correct `GENI-WIRELESS-USERNAME`, `/PATH/TO/KEY`, and substituting sb3.orbit-lab.org for the hostname of the console of the testbed that you have reserved.


Then, from there, log in to the transmitter node:

* If you are using WITest:
```
ssh root@node16
```

* If you are using sb2, sb3, or sb7 on ORBIT:
```
ssh root@node1-1
```

To generate a PSK signal, we will run the following command in the transmitter terminal window:

```
time /usr/local/share/gnuradio/examples/digital/narrowband/benchmark_tx.py -f 2480e6 -r 0.5e6 -M 5 -p 2 --excess-bw=0.05
```

where

* `-f` is used to set the frequency at which to transmit,
* `-r` specifies the bitrate at which to transmit, in bits per second,
* `-M` specifies the total amount of data to transmit, in megabytes,
* `-p` indicates how many constellation points (signal levels) to use (must be a power of 2),
* `--excess-bw` is a filter setting that determines how much "extra" leakage bandwidth the signal is allowed to use.

Later in this experiment, we will modify the values of the `-r` and `-p` arguments.

(If you aren't able to see any transmission in the ShinySDR window, see the [Notes](#notes) section for advice on increasing the gain.)

When we run this, we see that about 0.5 MHz of bandwidth is used (the "ticks" on the display are 250 kHz apart), and the total transmission takes a little over 80 seconds (look at the "real" time in the final output on the transmitter):

<iframe width="560" height="315" src="https://www.youtube.com/embed/IRYOEaa4rNs" frameborder="0" allowfullscreen></iframe>

However, if we change the bitrate to 2 Mbps, 2 MHz of bandwidth is used and the transmission takes about 20 seconds:

```
time /usr/local/share/gnuradio/examples/digital/narrowband/benchmark_tx.py -f 2480e6 -r 2e6 -M 5 -p 2 --excess-bw=0.05
```


<iframe width="560" height="315" src="https://www.youtube.com/embed/av2rvkXBVm0" frameborder="0" allowfullscreen></iframe>

Finally, changing the constellation size to 4 points, the transmission still takes about 20 seconds, but uses only half the bandwidth, 1 MHz, when transmitting at 2 Mbps:

```
time /usr/local/share/gnuradio/examples/digital/narrowband/benchmark_tx.py -f 2480e6 -r 2e6 -M 5 -p 4 --excess-bw=0.05
```

<iframe width="560" height="315" src="https://www.youtube.com/embed/oxjWFkrwE9o" frameborder="0" allowfullscreen></iframe>


## Notes

If you aren't able to see the transmission in the ShinySDR window, you may have to make some adjustments; some testbeds (such as sb2), may have more attenuation (signal loss) between the transmitter and receiver, and so the default gain settings are not sufficient to see the transmission. You can increase the gain on both the receiver and the transmitter:

* To increase the gain on the receiver, use the gain slider in the "Radio Config" window in ShinySDR.
* To increase the transmission amplitude on the transmitter, run the transmitter with `--tx-amplitude=0.8` at the end of the command, each time you run it. For example:

```
time /usr/local/share/gnuradio/examples/digital/narrowband/benchmark_tx.py -f 2480e6 -r 0.5e6 -M 5 -p 2 --excess-bw=0.05 --tx-amplitude=0.8
```
### Exercise

Using the same procedure as described above, measure the time to deliver 5 MB and the occupied transmission bandwidth for each of the following experiments (i.e. fill in the table):

<table>  <colgroup>   <col width="176">   <col>   <col>   <col>  </colgroup>  <tbody>   <tr>    <td>Number of signal levels</td>    <td>Bitrate (bps)</td>    <td>Time to deliver 5MB (s)</td>    <td>Occupied bandwidth (Hz)</td>   </tr>   <tr>    <td>2</td>    <td>500,000.00</td>    <td>&nbsp;</td>    <td>&nbsp;</td>   </tr>   <tr>    <td>2</td>    <td>1,000,000.00</td>    <td>&nbsp;</td>    <td>&nbsp;</td>   </tr>   <tr>    <td>2</td>    <td>2,000,000.00</td>    <td>&nbsp;</td>    <td>&nbsp;</td>   </tr>   <tr>    <td>4</td>    <td>500,000.00</td>    <td>&nbsp;</td>    <td>&nbsp;</td>   </tr>   <tr>    <td>4</td>    <td>2,000,000.00</td>    <td>&nbsp;</td>    <td>&nbsp;</td>   </tr>   <tr>    <td>4</td>    <td>1,000,000.00</td>    <td>&nbsp;</td>    <td>&nbsp;</td>   </tr>   <tr>    <td>16</td>    <td>500,000.00</td>    <td>&nbsp;</td>    <td>&nbsp;</td>   </tr>   <tr>    <td>16</td>    <td>2,000,000.00</td>    <td>&nbsp;</td>    <td>&nbsp;</td>   </tr>   <tr>    <td>16</td>    <td>1,000,000.00</td>    <td>&nbsp;</td>    <td> </td>   </tr>  </tbody> </table>

Create a scatter plot of your experiment data. Put time to deliver 5MB on the y-axis, occupied bandwidth on the x-axis, and have the color of each data point indicate the number of signal levels. Add lines connecting data points of the same color. For example:

![](/blog/content/images/2017/06/lab0-physical.png)