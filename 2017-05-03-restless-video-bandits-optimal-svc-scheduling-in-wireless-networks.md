This is an experimental evaluation of the Buffer State Aware scheduling heuristic that we have developed for SVC video delivery in wireless networks. Buffer State Aware scheduling achieves good performance relative to Proportional Fairness and Best Channel scheduling policies, while avoiding the complexity associated with joint optimization of wireless resource scheduling and video quality adaptation.

It should take about 120 minutes to run this experiment, but you will need to have reserved that time in advance. This experiment uses wireless resources, and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have reserved time on the sb4 testbed at [ORBIT](http://geni.orbit-lab.org), and you must run this experiment during your reserved time.  

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

Delivery of [SVC](https://en.wikipedia.org/wiki/Scalable_Video_Coding) video in a wireless network involves two optimizations:

* **Quality adaptation**: In SVC, each segment is encoded into a base layer containing the minimum
quality representation, and one or more enhancement layers for additional quality. The quality adaptation process involves determining the order by which different layers of different segments are requested by the user. The goal of this optimization is to maximize video quality (by getting as many enhancement  layers as possible) while avoiding rebuffering (video freezes) due to insufficient network capacity.
* **Scheduling**: Determine how to allocate the wireless resource (bandwidth and time) among many users. Usually this is a tradeoff between efficiency and fairness - the wireless resource can be used more efficiently (delivering more data per unit time and bandwidth) when allocated to a user with a strong signal quality, but we also want users with a weak signal to achieve at least a minimal level of service.

The above two tasks can either be implemented jointly by the base station:

![](/blog/content/images/2017/04/restless-joint-qa-scheduling.svg)

or independently of each other, with scheduling at the base station and QA at the end users:

![](/blog/content/images/2017/04/restless-separate.svg)

While jointly optimizing these tasks will attain better performance, the heavy crosslayer functionality required at the base station as well as the need for coordination between content provider and network provider makes it impractical and overly complex.

To address this issue, we have designed a scheduling policy that adapts itself to any arbitrary QA policy that is implemented on the end users. We used the concept of Restless Bandits (RB), a powerful tool for optimizing sequential decision making based on forward induction.  We devised a model for foresighted SVC scheduling for wireless networks that optimizes the scheduling policy given that users deploy different quality adaptation schemes. We proposed a criterion that simplifies the solution of the problem to that of the single user case. We then developed two simple heuristic algorithms that perform scheduling for two scenarios, one in which the base station is aware of the QA used by the end users, and the other in which the base station is blind to any of the users’ QA.

The experiment we demonstrate here is an evaluation of one of the heuristics we have developed, the Buffer State Aware heuristic for scheduling the wireless resource. It works as follows:

We assign a auxiliary buffer variable b to every user. At each time slot, we decrement this variable by &epsilon; to represent the draining the buffer due to playback. Whenever a user receives its previously requested segments, we increase the variable by &epsilon; times the sum number of segments it has received for all layers. 

At each time slot, we order all users in increasing order of their respective auxiliary buffer value and schedule the first M users (where M is the number of available subchannels, or in other words, the number of users that can be served at the same time). Among these M users, we schedule those with negative b according to their channel quality and prioritize those with better channel. Those with positive b are prioritized based on their base layer buffer occupancy and those with less base layers in the buffer are prioritized.

![](/blog/content/images/2017/04/restless-buffer-aware.svg)

Our experiment examines the performance of the Buffer State Aware heuristic relative to two standard policies for wireless resource scheduling: Proportional Fairness (PF) and Best Channel (BC). 

* In PF, in each time slot, the users with the highest index &nu; = c/&psi; are scheduled, where c is the current available rate and &psi; is the average rate the user has been allocated to date. 
* The BC scheme schedules the users simply by choosing the ones with the best current channel state.

Meanwhile, the end users in our experiment implement a Diagonal Buffer Policy (DBP) for QA. Results from existing research suggest that it is optimal to pre-fetch lower layers first, and fill higher layers after. In this scheme, which we call a diagonal policy, the user starts pre-fetching subsegments from the lowest layer until the difference between the subsegments between that layer and the one above it reaches a certain pre-fetch threshold, at which point it switches to the layer above, and this continues for all layers. This way, the difference between the buffer occupancy for each layer with the subsequent upper layer is kept at the fixed pre-fetch threshold. (We use a 2 second pre-fetch threshold in this experiment.)

## Results

The Buffer State Aware scheduling policy offers a higher sum reward per user than either the Proportional Fairness or Best Channel policies, over a range of network load (&rho;) conditions. Also, the Buffer State Aware policy leads to less rebuffering (fewer video freezes) than the other policies:

![](/blog/content/images/2017/04/restless-video-results.svg)

## Run my experiment

In this experiment, I used the sb4 testbed on ORBIT. To reserve time on this testbed, log in to [http://geni.orbit-lab.org](http://geni.orbit-lab.org), click on "Control Panel", click on "Scheduler", click on the grid squares corresponding to the "sb4" testbed and the date/time you want, and submit your reservation request. You may then complete the experiment at the reserved time.

### Set up the testbed

At your reserved time, SSH into 

```
sb4.orbit-lab.org
```

with your GENI wireless username and associated keys.

When you first log in to the sb4 console, you should [reset sb4's programmable attenuation matrix](http://www.orbit-lab.org/wiki/Hardware/bDomains/cSandboxes/dSB4) to zero attenuation between all pairs of nodes. From the sb4 console, run

```
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/setAll?att=0"

wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=1&port=1"
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=2&port=1"
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=3&port=1"
wget -qO- "http://internal2dmz.orbit-lab.org:5054/instr/selDevice?switch=4&port=1"
```

Next, load the disk image we have prepared onto the nine nodes you will use in your experiment:


```
omf load -i svcdemo.ndz -t system:topo:all
```

After the disk image is loaded onto all nine nodes, turn your nodes on:

```
omf tell -a on -t system:topo:all
```

and wait a few minutes for them to boot. 

After all nodes finish booting, from the sb4 console, log in to each one (as the "root" user).

On node1-1, run the [following script](https://github.com/ssah8524/SVC_scheduling/blob/demo/node_ap.sh) from that repository in order to turn the node into a WiFi AP:

```
bash SVC_scheduling/node_ap.sh
```

Wait for this to run and start the AP. When it's ready, it should say:

```
ap0: AP-ENABLED 
```
then hit Enter to get your terminal prompt back.

On nodes 2 through 9 run the [following script](https://github.com/ssah8524/SVC_scheduling/blob/demo/node_setup.sh) from that repository to connect them to the AP:

```
bash SVC_scheduling/node_setup.sh
```

When these have finished running, verify that there is connectivity between all the nodes. On any node, run

```
ping -c 1 192.168.0.100
for n in {2..9}; do
    ping -c 1 192.168.0.$n
done
```

### Run a video delivery experiment

After all users are connected to the AP, run the scheduler script that prepares the AP for the scheduling process. The command line arguments for the script are (in order):

* the number of users, 
* the number of subchannels on which it may schedule transmissions, 
* the length of a time slot (in seconds), 
* the total length of the video (in seconds), 
* the scheduling method (you can use "pf" for Proportional Fairness, "maxrate" for Best Channel, and "heuristic" for the Buffer Aware scheme), 
* and the port number to use for the socket of the first user. 

For example, we could run (on the AP):

```
cd SVC_scheduling
python schedule.py 8 4 1 300 pf 10002
```
The above example runs the experiment with 8 users, and 4 of them can be scheduled in every time slot. The length of the time slot is 1 second and the total length of the video is 300 seconds. The scheduling method is Proportional Fairness (pf), and the port numbers for the sockets assigned to each user should start from 10002. (The 8 users will connect to port 10002, 10003, 10004, 10005, 10006, 10007, 10008, 10009, respectively.)

Next, go to the user windows (node1-2 through node1-9) one by one and run:

```
cd SVC_scheduling
num=$(hostname -s | cut -f2 -d'-')
python client.py 192.168.0.100 1000$num
```

The command line arguments indicate the IP address of the AP and the port number of the process assigned to user x, as 1000x. For node1-2, x will be 2, for node1-3 it will be 3, and so forth, so that each node will connect to a different port. 

Once the specified number of users (8 in this example) run the client script, the streaming starts automatically. Watch the output on the scheduler node (node1-1); it should look similar to this:

<pre>
elapsed time: 1.00669002533
elapsed time: 10.4790098667
...
elapsed time: 290.67910862
<b>45.7220486698 11.25 0.0373279172168</b>
</pre>

At the end of the streaming process, the sum reward per user, the average re-buffering time, and the percent of time spent in rebuffering, will be printed on the screen. In this example, 

* 45.7220486698 is the mean reward, averaged over all the users. This is an estimate of the perceptual quality of the video.
* 11.25 is the mean rebuffering time in seconds per user.
* 0.0373279172168 is the percent of total time spent in rebuffering.


### Play back a video

The video files for all users are downloaded into the `user_files` folder on each of the nodes 2 through 9.  We can play back this video and observe the variations in video quality, although we won't be able to see instances of rebuffering. But,

* Video files from a current experiment will overwrite files from previous experiments in `user_files`. If you are playing back video, delete anything in `user_files` before starting the experiment, and after the experiment, save any video that you want to keep in another location.
* Rebuffering only occurs during "live" video play, not when playing back a stored video file.) * Only 21 distinct segments are used - the video then repeats, so segment 0 will be repeated after segment 20 and so on.

A utility script is provided to process the first 100 segments of video and:

* combine different layers into a single video file for every segment, 
* decode each video segment into a raw video files,
* encode each segment into an MP4 file, and
* combine the MP4 files for all segments into one.

To use it, run (on a client node):

```
cd ~/SVC_scheduling/user_files
bash ../mux.sh
```

Then, transfer the video file to your laptop:

<pre>
scp -o "StrictHostKeyChecking no"  -o "ProxyCommand ssh <b>USERNAME</b>@sb4.orbit-lab.org nc %h %p" root@<b>node1-2</b>:/root/SVC_scheduling/user_files/final_output.mp4 video.mp4
</pre>

where

* in place of <b>USERNAME</b>, put the username with which you access the ORBIT testbed,
* in place of <b>node1-2</b>, put the name of the node on which you have created the video file, and
* you may also need to specify a key (with `-i` argument) if you access the ORBIT testbed with a key that is not in the default location.

You should then be able to play back the "video.mp4" file with any standard video player.

### Find total reward and fraction of time in rebuffering for three policies and varying load

To generate the data for the figures in the [Results](#section), we need to run twelve experiments and fill in the following tables (fill in one cell in each table for each experiment):

---
##### Total reward per user (averaged across users)
<table id="table-1" class="table table-striped table-bordered col-4" data-columns="4"><thead>
<tr><th class="col-1">&rho;</th><th class="col-2">Buffer State Aware</th><th class="col-2">Proportional Fairness</th><th class="col-2">Best Channel</th></tr></thead>
<tbody>
<tr class="row-1"><td class="col-1">4</td><td class="col-2">...</td><td class="col-3">...</td><td class="col-4">...</td></tr>
<tr class="row-2"><td class="col-1">2.66</td><td class="col-2">...</td><td class="col-3">...</td><td class="col-4">...</td></tr>
<tr class="row-3"><td class="col-1">2</td><td class="col-2">...</td><td class="col-3">...</td><td class="col-4">...</td></tr>
<tr class="row-4"><td class="col-1">1.33</td><td class="col-2">...</td><td class="col-3">...</td><td class="col-4">...</td></tr>
</tbody>
</table>

---

##### Fraction of time spend in rebuffering (averaged across users)

<table id="table-1" class="table table-striped table-bordered col-4" data-columns="4"><thead>
<tr><th class="col-1">&rho;</th><th class="col-2">Buffer State Aware</th><th class="col-2">Proportional Fairness</th><th class="col-2">Best Channel</th></tr></thead>
<tbody>
<tr class="row-1"><td class="col-1">4</td><td class="col-2">...</td><td class="col-3">...</td><td class="col-4">...</td></tr>
<tr class="row-2"><td class="col-1">2.66</td><td class="col-2">...</td><td class="col-3">...</td><td class="col-4">...</td></tr>
<tr class="row-3"><td class="col-1">2</td><td class="col-2">...</td><td class="col-3">...</td><td class="col-4">...</td></tr>
<tr class="row-4"><td class="col-1">1.33</td><td class="col-2">...</td><td class="col-3">...</td><td class="col-4">...</td></tr>
</tbody>
</table>

---

The command to run on the AP node is given below for each experiment:


for &rho; = 4:
```
python schedule.py 8 2 1 300 heuristic 10002
python schedule.py 8 2 1 300 pf 10002
python schedule.py 8 2 1 300 maxrate 10002
```
for &rho; = 2.66:
```
python schedule.py 8 3 1 300 heuristic 10002
python schedule.py 8 3 1 300 pf 10002
python schedule.py 8 3 1 300 maxrate 10002
```
for &rho; = 2:
```
python schedule.py 8 4 1 300 heuristic 10002
python schedule.py 8 4 1 300 pf 10002
python schedule.py 8 4 1 300 maxrate 10002
```
for &rho; = 1.33:
```
python schedule.py 8 6 1 300 heuristic 10002
python schedule.py 8 6 1 300 pf 10002
python schedule.py 8 6 1 300 maxrate 10002
```

The client nodes always run the same thing:

```
python client.py 192.168.0.100 1000$(hostname -s | cut -f2 -d'-')
```


In the figures, the horizontal axis refers to the "network load" &rho;, which we define as the number of users divided by the number of subchannels. In other words, it shows the number of users who compete for each subchannel in one time slot. 

For the figures shown in the [Results](#section), we ran each of the twelve experiment multiple times, and show the mean values and 95% confidence interval in the figure. Each "run" of twelve experiments takes one hour.


## Notes

You can run this experiment on other ORBIT testbeds, such as the grid or the outdoor testbed. If you are not using consecutively numbered nodes starting at 2 for the client nodes, you will have to modify the instructions slightly so that you _don't_ use the last part of the hostname as the port argument (1000x) with which you run the `client.py` script.

To set up the disk image used in this experiment, we ran the [install.sh](https://github.com/ssah8524/SVC_scheduling/blob/demo/install.sh) script on the baseline image. You can follow a similar process to set up this experiment on a non-ORBIT testbed.