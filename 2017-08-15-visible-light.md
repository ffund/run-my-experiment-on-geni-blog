In this experiment, you will learn how to use the VLC - visible light communication - devices on the WITest testbed, and how to capture and visualize the light levels measured at the receiver.

It should take about 2 hours to run this experiment, but you will need to have reserved that time in advance. This experiment uses wireless resources, and you can only use wireless resources on GENI during a reservation. 


To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have reserved time on [WITest](https://witestlab.poly.edu) and you must run this experiment during your reserved time.  

## Background

With growing demand for wireless access and limited availability of traditional radio frequency spectrum, has come increasing interest in visible light as a communication medium. This has resulted in a great deal of recent research activity on visible light communication (VLC).

However, despite this increased research activity, it is not easy to start experimenting with VLC. A variety of platforms have been developed for experimentation with VLC, such as Shine \[1]\, EnLighting \[2]\, DarkLight \[3]\, and OpenVLC \[4]\. However, there is no commercially available VLC network interface card. To begin working with VLC, a research team has to set up a local VLC testbed, and possibly also build devices themselves, even if they are only interested in VLC protocols or applications and do not intend to innovate in hardware. This creates a barrier to entry for VLC research (compared to other wireless technologies, for which devices are generally available, and for which open-access testbeds already exist). 

This creates a barrier to entry for VLC research, and also serves as a barrier to reproducibility. Because there is no common hardware platform that everyone has access to, it is difficult to reproduce published results, to directly compare results from different experiments, or to build on other researchers' work. Furthermore, because VLC links are so sensitive to blocking and the position of the transmitter and receiver relative to one another, it can be difficult to reproduce a result even with the same hardware platform. Factors in the experiment environment, such as nearby reflective surfaces or the intensity of surrounding ambient light, can have a major impact on the resulting measurements. This, too, is a barrier to reproducibility. 

To address this problem, we have integrated a VLC testbed (using the OpenVLC platform) into WITest.  By enabling access to a common platform and common environment for experiments, we hope to lower the barrier to entry, encouraging further research on VLC protocols and applications and use of VLC experiments in education.

Three VLC transceivers are available through the WITest facility, each with a BeagleBone Black (BBB) board with the OpenVLC driver installed, an OpenVLC 1.1 cape, and a 5mm LED. Experimenters have full access to the BBB via SSH, and can modify the VLC driver or run VLC applications from there. 

With VLC links, the position of the transceivers relative to one another (distance and orientation) is an important factor in link quality. We have put each LED on a pan/tilt platform with two servo motors, which experimenters can control remotely to change the direction of the light beam. 

![](/blog/content/images/2017/09/vlc-testbed-1.jpg)

<small>A VLC transceiver on the WITest testbed, with the LED transmitter mounted on a pan/tilt platform.</small>

Experimenters can also visually monitor an experiment by streaming video from a webcam connected to another testbed node.

<iframe width="560" height="315" src="https://www.youtube.com/embed/w-WPe1xIxPM?rel=0" frameborder="0" allowfullscreen></iframe>

<small>Overhead view of the VLC devices from the webcam. You can see VLC3 (on the left), VLC2 (on the right), and VLC1 (middle, on the bottom). Experimenters can watch a live stream of their experiment, like the one shown above, via the webcam.</small>

As a shared, remote-access platform for VLC experiments, this testbed has several limitations with respect to the kinds of experiments that are supported. Experimenters cannot change the VLC hardware or the external environment (e.g. ambient light, physical layout of the room), and they have limited ability to change the position of the VLC transceivers. However, we anticipate that this facility may support experiments involving applications that use VLC links, or experiments on protocols for VLC communication (which can be implemented by modifying the [OpenVLC driver](https://github.com/openvlc/openvlc)). This facility can also support experiments involving hybrid networks with VLC and also conventional wireless technologies, such as WiFi or cellular networks, or links between software defined radio devices. 

## Results

In this experiment, we will send data between two VLC devices. We'll also record the light levels received at the receiver during "sensing" periods. In the following image, we plot the recorded light levels over time as black points. We also show when the device is in transmitting mode (in green) and when it is receiving a frame (in purple):

![](/blog/content/images/2017/08/vlc-light-levels.svg)

In the recorded light levels, we can pick out the pattern of the preamble. At the beginning of each frame, the transmitter [sends the pattern](https://github.com/openvlc/openvlc/blob/master/openvlc.c#L645) `10101010` three times. We can see this pattern as high/low light levels recorded at the receiver:

![](/blog/content/images/2017/08/vlc-preamble.svg)

## Run my experiment
 
In order to run our experiment, you first have to [reserve time on the WITest testbed](https://witestlab.poly.edu/site/activity/reservation-calendar).

### Set up the testbed

At the reserved time, open a terminal and log into the appropriate testbed console using SSH:

<pre>
ssh <b>USERNAME</b>@witestlab.poly.edu
</pre>

using your own GENI wireless username and password (typically, "geni-X" where X is your regular GENI username). 

Then continue by loading a disk image onto node24, which has a webcam attached to it that will allow us to visually monitor the experiment. On the WITest console, run:

<pre>
omf-5.4 load -i wifi-experiment.ndz -t omf.witest.node24
</pre>

After this is complete, turn it on:

<pre>
omf tell -a on -t omf.witest.node24
</pre>


and wait a few minutes for the node to boot. 

Next, turn on the VLC devices. All three devices are powered on and off at the same time. To power on the VLC devices, run:


<pre>
omf tell -a on -t omf.witest.vlc
</pre>

### Set up the webcam to watch your experiment

To monitor the experiment using the webcam, we will set up an SSH tunnel to node24, which has a webcam attached to it.

In a new terminal window, run

<pre>ssh -L 8081:node24:8081 <b>USERNAME</b>@witestlab.poly.edu
</pre>

(using your own GENI wireless username). This will set up a tunnel between TCP port 8081 on your local host, and TCP port 8081 on node24 on the testbed, where the webcam stream is. You'll have to keep this terminal window open in order to keep using the tunnel.

From the WITest console, log in to node24:

<pre>ssh root@node24</pre>

(If you are not able to log in after turning it on try resetting it, then waiting a few minutes for it to come up:

<pre>
omf tell -a reset -t omf.witest.node24
</pre>

and then try to log in again.)

From the console on node24, you need to install some software and set up a config file to use the webcam.

First, to install the [motion](https://linux.die.net/man/1/motion) software, run

<pre>apt-get update
apt-get -y install motion
</pre>

Then download the config file from [this gist](https://gist.github.com/ffund/e976a8fd00125318f90bfe81f16521ed):

<pre>
wget -O motion.conf https://git.io/v5xdO
</pre>

Finally, to start the software, run

<pre>
motion -c motion.conf
</pre>

Now, you can view a live stream of your experiment in your local browser at [http://localhost:8081](http://localhost:8081).

You may notice that the camera focus is lost when the LEDs are flashing. To fix the focus of the camera, you have to turn off the auto focus. On the node24 terminal, run

<pre>apt-get update
apt-get install uvcdynctrl
uvcdynctrl -d video0 -s "Focus, Auto" 0
uvcdynctrl -d video0 -s "Focus (absolute)" 0
</pre>

### Setup VLC devices

Now, we will set up the VLC devices.

Open two more terminals, and log into the WITest console.

Then, from the WITest console, log on to the "vlc2" device in one terminal:

<pre>ssh debian@vlc2</pre>

and the "vlc3" device in the other terminal:

<pre>
ssh debian@vlc3
</pre> 

Use "temppwd" as the password when prompted.

Next, we want to set up the VLC link. On both boards, we will navigate to the directory where the `openvlc` driver is saved, load the driver with the appropriate arguments, and set up an IP address for the VLC interface.

On vlc2 run:

<pre>cd openvlc
sudo insmod vlc.ko frq=50 mtu=1300 mac_or_app=1 show_msg=1
sudo ifconfig vlc0 192.168.0.2
</pre>

and on vlc3 run:
<pre>cd openvlc
sudo insmod vlc.ko frq=50 mtu=1300 mac_or_app=1 show_msg=1
sudo ifconfig vlc0 192.168.0.3
</pre>

When we load the VLC driver with the option `show_msg=1`, the light levels received during the sensing periods will be printed to the system log. We can use this to produce a figure like the one in the [Results](#Results) section.

Then we want to setup the low power LEDs as the transmitter and the photodiode as the receiver. On both boards run:

<pre>sudo echo 1 > /proc/vlc/rx
sudo echo 0 > /proc/vlc/tx
</pre>


### Test the visible light link

To test the visible light link, on vlc2 run:

<pre>
ping -c 10 192.168.0.3
</pre>

This will send 10 "ping" messages to vlc3. When in range, vlc2 sends an echo request to vlc3, and vlc3 sends an echo reply back. 

If there is no line of sight between the boards, no responses will be received, and you will have to attempt to use the pan/tilt platform to change the position of the transceivers in relation to one another.  To move the position of the transmitter, run

<pre>
sudo pan-tilt --pan <b>X</b> --tilt <b>Y</b>
</pre>

where **X** and **Y** are numeric values. Use the webcam view to adjust iteratively adjust the positions:

* If you see both LEDs light up when you send the ping, then the requests _are_ received at vlc3, and you don't have to move vlc2's transmitter. You should adjust the position of vlc3 until the replies are received at vlc2. (You can start with the "known good" positions listed below, then make slight adjustments as needed.)
* If you only see the LED of vlc2 light up, then you'll have to adjust its position until vlc3's LED also lights up, indicating that the requests are received at vlc3. Then, you may still need to adjust vlc3's position so that the replies will be received at vlc2. (You can start with the "known good" positions listed below, then make slight adjustments as needed.)

The following video shows the general procedure:

<iframe width="560" height="315" src="https://www.youtube.com/embed/o3gbA6hCnMg" frameborder="0" allowfullscreen></iframe>


We have observed successful two-way communication with these settings, so you should find these useful as a starting point (you may have to make slight adjustments):

* On vlc2: `sudo pan-tilt --pan 9 --tilt 6.5`
* On vlc3: `sudo pan-tilt --pan 8 --tilt 7.5`

Similarly, we have observed two-way communication between vlc1 and vlc2 with these settings:

* On vlc1: `sudo pan-tilt --pan 4.5 --tilt 4`
* On vlc2: `sudo pan-tilt --pan 11 --tilt 3`

And we have observed two-way communication between vlc1 and vlc3 with these settings:

* On vlc1: `sudo pan-tilt --pan 11.5 --tilt 4`
* On vlc3: `sudo pan-tilt --pan 6.75 --tilt 6.5`


### Inspect the received light levels

During operation, the VLC device is either in the sensing state (looking for a preamble), receiving a frame, or transmitting. While in the sensing state, it will print received light levels to the system log.

To record these values, on the BBB console run

<pre>tail -f /var/log/syslog |  tee outfile.txt</pre>

This will simultaneously display the system log, and also record it to a file. While this is running, you can transfer data between the VLC devices.

When you are satisfied, stop this with Ctrl+C. Then, run

```
cat outfile.txt | grep "photodiode" | awk '{gsub ( /\]/, "" ); gsub("Found a preamble!", ""); print $7 "," $10 "," $11 "," $12 "," $13 "," $14 "," $15}' > outfile.csv
```

to extract the received light levels in CSV format. In the "outfile.csv" file, each row will include a timestamp, then the values (light levels) of the six symbols in the receive buffer. You can plot these values to create a figure similar to the one in the [Results](#Results) section.


When you're finished, you can unload the VLC driver on each device with

```
sudo rmmod vlc.ko
```

## Notes

If a VLC device becomes unresponsive, you can power cycle it and set it up again; but you have to power cycle all three at once.

Run

<pre>
omf tell -a reset -t omf.witest.vlc
</pre>

to power cycle the VLC devices.

### References

\[1\] L. Klaver and M. Zuniga. Shine: A step towards distributed multi-hop visible light communication. In 2015 IEEE 12th International Conference on Mobile Ad Hoc and Sensor Systems, pages 235–243, Oct 2015.

\[2\] S. Schmid, T. Richner, S. Mangold, and T. R. Gross. EnLighting: An indoor visible light communication system based on networked light bulbs. In 2016 13th Annual IEEE International Conference on Sensing, Communication, and Networking, SECON, pages 1–9, June 2016. 


\[3\] Z. Tian, K. Wright, and X. Zhou. The DarkLight rises: Visible light communication in the dark. In Proceedings of the 22Nd Annual International Conference on Mobile Computing and Networking, MobiCom ’16, pages 2–15, 2016.

\[4\] Q. Wang, D. Giustiniano, and D. Puccinelli. OpenVLC: Software-deﬁned visible light embedded networks. In Proceedings of the 1st ACM MobiCom workshop on visible light communication systems, VLCS ’14.


### Acknowledgments

This research was supported by the [NYU Tandon School of Engineering Center for K12 STEM Education](http://engineering.nyu.edu/k12stem) via the [ARISE](http://engineering.nyu.edu/k12stem/arise/) program, and by the Pinkerton Foundation.

