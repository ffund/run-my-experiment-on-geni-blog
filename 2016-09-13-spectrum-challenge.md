This tutorial shows how to pit software radio designs in a head-to-head competition to see which is more effective at delivering data when both operate on the same frequency.

It should take about 60 minutes to run this experiment, but you will need to have [reserved that time](http://witestlab.poly.edu/respond/sites/witest/activity/reservation-calendar) in advance. This experiment uses wireless resources (specifically, the [WITest](http://witestlab.poly.edu) facility), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have [reserved time on WITest](http://witestlab.poly.edu/respond/sites/witest/activity/reservation-calendar) in advance, and you must run this experiment during your reserved time.

If you are an instructor, refer to Appendix A for more information about getting your class set up with accounts on GENI.

## Background

A spectrum challenge is a competition in which participants design and implement a radio protocol that is effective even in difficult and uncertain RF environments. Recent spectrum challenges include:

* **2013 [DARPA Spectrum Challenge](http://archive.darpa.mil/spectrumchallenge/)**. Radio pairs designed by teams of academics, radio enthusiasts, and practitioners, competed head-to-head to win $200,000 in prize money in two events: a [competitive tournament](https://www.youtube.com/watch?v=eDDvxa_5_9g) and a [cooperative tournament](https://www.youtube.com/watch?v=LZ9Q0df1fkw). (An explanation of the data visualization in those videos is available [here](https://www.youtube.com/watch?v=JzBHFQKwI3o).)
* **2015 [IEEE DySPAN 5G Spectrum Sharing Challenge](http://dyspan2015.ieee-dyspan.org/content/5g-spectrum-sharing-challenge)**. This competition was "designed to demonstrate a radio protocol that can achieve high spectral efficiency in coexistence with legacy technology." Teams tuned the sharing performance of their radio protocol based on feedback about the legacy receiver's instantaneous throughput and packet loss performance.
* **2016 [Spectrum Sharing Radio Challenge](http://radiocontest.wireless.vt.edu/)**. Organized and hosted by Virginia Tech, this student competition involved multiple phases, each at a different layer of the network stack.

Meanwhile, more than $3 million in prize money is up for grabs in the ongoing [DARPA Spectrum Collaboration Challenge](https://spectrumcollaborationchallenge.com/).

This post describes how you can run a spectrum challenge yourself, using the open-access GENI wireless testbeds, [WITest (at NYU)](https://witestlab.poly.edu) and [ORBIT](http://orbit-lab.org).

A spectrum challenge can be a fun and exciting way to teach wireless communications, wireless signal processing, software defined radio, cognitive radio, and other related concepts. In Fall 2015, I taught "Selected Topics in Wireless Communications: Software Defined Radio Lab", a course in which graduate students were tasked with implementing software radio designs and competing against one another in a Spectrum Challenge-like tournament format. (I had previously developed this course for Thanasis Korakis, who taught it for the first time in 2104 at the University of Thessaly in Greece.)

If you just want to run the spectrum challenge, you can skip to [Run the Experiment](#runtheexperiment), but if you're an educator looking to do something similar, I share some experiences related to using this in a classroom setting in my [GrCon '16 presentation](#appendixbgrcon16presentation).



 
## Run the experiment



### Set up the challenge arena

We need to prepare the five host computers used in this experiment. 

![](/blog/content/images/2016/12/challenge-arena.jpg)

They are node16, node17, node18, node19, and node22. These five hosts have USRP devices set up in the following configuration:

![](/blog/content/images/2016/12/challenge-arena.svg)

"node16" and "node19" are designated as transmitters, and their TX outputs are also connected to the input of "node22", which acts as a monitor node. "node17" and "node18" are designated as receivers. 

You cannot access the nodes directly over the Internet - you have to first log in to the WITest console, and connect to the testbed nodes from there. To connect, use

```
ssh USERNAME@witestlab.poly.edu
```

where USERNAME is your WITest username. (If you access WITest using your GENI account, your wireless username is your GENI account username prefixed with a "geni-", e.g. mine is "geni-ffund".) You may also have to specify the location of your key for login.


To start, you'll need to load the `sdr-match.ndz` image onto five SDR nodes. This is a disk image that has UHD and GNU Radio preinstalled. In the WITest console terminal, run

```
omf-5.4 load -i sdr-match.ndz -t omf.witest.node16,omf.witest.node17,omf.witest.node18,omf.witest.node19,omf.witest.node22
```

It should take 10-15 minutes to load the disk image onto the nodes. 

One the disk load process is done, you can turn them on with 

```
omf tell -a on -t omf.witest.node16,omf.witest.node17,omf.witest.node18,omf.witest.node19,omf.witest.node22
```

then wait a few minutes for them to boot up.

### "Hello, world"

The "Hello, world" of the spectrum challenge involves sending data between the radio pairs on the testbed.

Open six terminals, and log in to the WITest console in five of them:

```
ssh GENI-WIRELESS-USER@witestlab.poly.edu
```

where GENI-WIRELESS-USER is your GENI wireless username. (Typically it starts with "geni-".) You may also have to specify the location of your key for login.

Then, from the WITest console, log in to each of the five nodes in the challenge configuration:

```
ssh root@node22
ssh root@node16
ssh root@node19
ssh root@node17
ssh root@node18
```

In the sixth terminal, from your _local_ shell, you'll open a couple of tunnels to help you see the visualization:

```
ssh -L 8100:node22:8100 -L 8101:node22:8101 GENI-WIRELESS-USERNAME@witestlab.poly.edu
```
(again, using your own GENI wireless username.)

![](/blog/content/images/2016/09/gr-login-config.png)


On node22, run

```
python -m shinysdr.main ~/.shinysdr_settings
```

to start the [ShinySDR](https://github.com/kpreid/shinysdr) web-based receiver. The output will show a couple of lines like

```
INFO:shinysdr:ShinySDR is ready.
INFO:shinysdr:Visit http://localhost:8100/nfZ2UMGc6olYz_7bJKq3aQ/
```

Visit the URL using your local Google Chrome web browser. Tune to 2.48 GHz and adjust the options so that you can see some noise at the bottom of the display:

<iframe width="560" height="315" src="https://www.youtube.com/embed/YT7hchUqN4U" frameborder="0" allowfullscreen></iframe>

Now we'll send some traffic. On one of the transmitters, run

```
tournament/benchmark_tx.py -f 2.48G -r 1e6 --tx-amplitude=0.5 -M 100 -m gmsk
```

and on one of the receivers, run

```
tournament/benchmark_rx.py -f 2.48G -r 1e6 -m gmsk
```

On the ShinySDR web interface, you can monitor the output of the transmitter. You can switch back and forth between the "RX2" and "TX/RX" inputs to see change the view from the output of one transmitter, to the output of the other (although there is some leakage between the two).

### Run a match

To run a tournament "match" between two designs, we will use OMF, a tool that helps us execute carefully timed commands on testbed nodes.

Running a "match" is slightly different from running the benchmark scripts directly from the terminal:

* The transmitter nodes get packets to send from a "packet source", located at 10.0.0.200:50000
* The receiver nodes need to deliver packets they receive to a "packet sink", located at 10.0.0.200:50001
* The score of a radio pair is equal to the number of packets provided by the packet source to its transmitter, and delivered to the packet sink by its receiver.

The Python source code for each radio pair should be saved in a Git repository. When you run a match, you can specify both the HTTPS URL of the repository and the branch, for each radio pair. You also won't have an opportunity to specify command line arguments, so they should be hardcoded. The match execution script will run

```
benchmark_tx.py --server --freq 2480e6
```

and 

```
benchmark_rx.py --server --freq 2480e6
```
where the `--server` flag indicates that packets should be sourced from and delivered to the server.

To run the match script with a simple on/off transmitter on both radio pairs, run

```
omf exec sdr-match -- --git1 https://bitbucket.org/ffund/spectrum-challenge.git --branch1 onoff --git2 https://bitbucket.org/ffund/spectrum-challenge.git --branch2 onoff
```

While the match is running, you can see the output of each transmitter with ShinySDR. (You can toggle between the TX/RX and RX2 inputs to see output from each of the radio pairs.)

At the end of the experiment, you will see the "score" (number of packets delivered) for each radio pair, e.g.:

```
  ************************************************
                                                  
          Pair 1:                                 
          Received 839 packets correctly         
                                                  
                                                  
          Pair 2:                                 
          Received 1282 packets correctly         
                                                  
  ************************************************
```

### Design your own

Fork the [spectrum-challenge](https://bitbucket.org/ffund/spectrum-challenge) repository, modify the benchmark transmitter and receiver, and supply your own Git URL to the "match script" to try your own design.

## Appendix A: Setting up accounts for classroom use

The following instructions are for instructors who intend to use GENI wireless resources in the classroom.

1. Log on to the [GENI Portal](http://portal.geni.net). You (and your students) should be able to log in with your university credentials, if you are at a US university. If you can't use your university account, you can [request a login](https://shib-idp.geni.net/geni/request.html) from the GENI Project Office.
2. Ask for your account to be made a [project lead](https://portal.geni.net/secure/modify.php) account, if it isn't already.
3. When you are a project lead, [create a project](https://portal.geni.net/secure/edit-project.php).
4. Upload an [SSH key](https://portal.geni.net/secure/profile.php#ssh) to the portal. Because of the way the wireless accounts are synced between the Portal and the wireless testbeds, as the project lead, you must have a key set up in the portal in order for your students to also get wireless accounts.
5. Instruct your students to log on to the portal and [join the project](https://portal.geni.net/secure/join-project.php) that you created. You will have to approve their join requests.
6. Instruct your students to supply SSH keys to the portal. 
7. In the portal, go [here](https://portal.geni.net/secure/wimax-enable.php) and enable wireless for your project, then click "Sync Project"

Your students will now all have accounts on the wireless testbeds, with the username "geni-USER" where USER is their regular GENI username. 

Now, your students should be able to SSH into witestlab.poly.edu (or into any of the [ORBIT testbeds](http://geni.orbit-lab.org)) with their SSH key that they set up in the portal and that "wireless username". (You may have to periodically sync the project again as more students join.)

## Appendix B: GrCon '16 Presentation

<iframe width="560" height="315" src="https://www.youtube.com/embed/feWPAnkM08Q" frameborder="0" allowfullscreen></iframe>
