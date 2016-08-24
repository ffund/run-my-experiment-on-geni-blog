
This experiment compares the queueing delay for the PIE and RED queue disciplines, using an experiment on the GENI testbed. It is an attempt to reproduce the simulation results in the published paper:

> Rong Pan P. Natarajan, C. Piglione,M.S. Prabhu, V. Subramanian, F. Baker, and B. VerSteeg. "PIE: A lightweight control scheme to address the bufferbloat problem." In 2013 IEEE 14th International Conference on High Performance Switching and Routing (HPSR). 8-11 July 2013. [URL](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=6602305)

It should take about 60 minutes to run this experiment.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

Many active queue management schemes are designed to control the bufferbloat problem. It is necessary to compare their performance under some situations in order to see how well they achieve specific goals, like low latency and high link utilization. PIE (Proportional Integral controller Enhanced) is a new scheme that is meant to help get very low queueing delay compared with the RED (Random Early Detection).

## Results

Below is the original figure from the paper that we are trying to reproduce. In this experiment, the traffic intensity increases at 50 seconds (by increasing the number of TCP flows), increases again at 100 seconds, decreases at 150 seconds, and decreases again at 200 seconds. The authors aim to show that the queueing delay of PIE remains close to the target delay of 20 ms as the traffic intensity varies, while the queueing delay of RED increases with traffic intensity: 



![Imgur](http://i.imgur.com/XVhepRl.png)

> Fig. 8. PIE vs. RED Performance Comparison Under Varying Traffic Intensity: 0s-50s, traffic load is 10 TCP flows; 50s-100s, traffic load is 30 TCP flows;100s-150s, traffic load is increased to 50 TCP flows; traffic load is then reduced to 30 and 10 at 200s and 250s respectively. Due to static configured parameters, the queueing delay increases under RED as the traffic intensifies. The autotuning feature of PIE, however, allows the scheme to control the queueing latency quickly and effectively.

The authors specify the following parameters for their experiment, which we seek to mimic as closely as possible in our replication:

> The simulation setup consists of a bottleneck link at 10Mbps with a RTT of 100ms. Unless otherwise stated the buffer size is 200KB. We use both TCP and UDP traffic for our evaluations. All TCP traffic sources are implemented as TCP New Reno with SACK running an FTP application. While UDP traffic is
implemented using Constant Bit Rate (CBR) sources. Both UDP and TCP packets are configured to have a fixed size of 1000B. Unless otherwise stated the PIE parameters are configured to their default values; i.e., _delay\_ref_ = 20ms, _T<sub>update</sub>_ = 30ms, _&alpha;_ = 0.125Hz, _&beta;_ = 1.25Hz, _dq\_threshold_ = 10KB, _max\_burst_ = 100ms.

When we run the equivalent experiment on GENI, with varying traffic intensity and using the same parameters exactly as described in the paper, we find the following queueing delay for PIE:


![pie_10M_no](http://i.imgur.com/Y0FEGWp.png)

and the following queuing delay for RED:

![red_10M_no](http://i.imgur.com/Icc3FZh.png)

In our experiment, the queueing delay for PIE is quite close to the original result, which shows PIE delay remaining around 20 ms throughout the experiment. As in the original experiment, RED imposes more delay than PIE for these particular settings. 

However, in our experiment, we do not see an obvious difference in the delay when the number of TCP flows changes. Instead, my plot shows a "flat" trend of the queueing delay when traffic intensity changes, which is not the same as the original RED result.

We hypothesize that the reason for the difference between the original simulation and our testbed experiment is that in the testbed experiment, even a small number of flows  saturates the link. Therefore, if we increase the number of flows, the actual traffic intensity does not change in a meaningful way. 

We therefore tried a modified version of this experiment, in which we limit the rate of each TCP flow so that the link can only become saturated when all 50 TCP flows are active (during the 100-150 second time interval).


Below are the results of this modified experiment:
	
![pie_10M_25000](http://i.imgur.com/ifOAj4K.png)

![red_10M_25000](http://i.imgur.com/EdCLTtl.png)

We notice that the queueing delay for the RED queue varies a lot when traffic intensity changes, which is the feature we expected before. 
	
To summarize, when the data rate before entering the queue (in router) is less than the bandwidth of the link connected to the outgoing interface, we could see the remarkable change of queueing delay as traffic intensity changes, just like this modified version. However, with the specific parameters detailed in the original paper, the bottleneck link is completely saturated and the incoming rate is greater than the outgoing rate, so the queueing delay will not change that much even if the number of TCP flows keeps increasing.


## Run my experiment

_Note: This section with the "short version" of the experiment was written by [Fraida Fund](https://witestlab.poly.edu/blog/author/ffund/), based on the full experiment developed by Bowen Yan and detailed in the next section._

 
We have created a "short" version of this experiment, with which you can reproduce our results with minimal time and effort.

First, you will need to reserve resources on GENI. Create a new slice on [GENI](https://www.geni.net) and click the "Add Resources" button. Then select the "URL" option in the "Choose RSpec" section, and load the [RSpec](https://gist.github.com/ffund/24e5bf500524ed87d0c18ab73d6a649e) from the following URL:

[https://git.io/v66vK](https://git.io/v66vK)

Click on "Site 1" and choose an InstaGENI aggregate, then click "Reserve Resources". This will reserve the following topology:

![](/blog/content/images/2016/08/pie-v-red-topology.png)

Wait until all your nodes turn green, indicating that they are ready. After that, wait _another_ 15 minutes (approximately) so that the setup scripts that run automatically when the nodes boot up will finish. Then log in to each client node, and the router node.

First we will run the PIE experiment. Prepare each of these commands in your terminal. When you're ready, run them in quick succession (i.e. so that each script starts running at approximately the same time).

<table>
<thead>
<tr>
<th>Node</th>
<th>Command</th>
</tr>
</thead>
<tbody>
<tr>
<td>router</td>
<td><code>pie-v-red-router pie</code></td>
</tr>
<tr>
<td>client1</td>
<td><code>pie-v-red-client server1 250 0</code></td>
</tr>
<tr>
<td>client2</td>
<td><code>pie-v-red-client server2 150 50</code></td>
</tr>
<tr>
<td>client3</td>
<td><code>pie-v-red-client server3 50 100</code></td>
</tr>
</tbody>
</table>

The experiment runs for about 250 seconds. While the experiment runs, the router terminal will show an ASCII art display of the current load on the interface connected to the clients. Use Ctrl+C to quit the monitor when the experiment ends. At the end of the experiment, each of the client node terminals will show an ASCII art plot of the measured queueing delay over 2500 samples (one ever 0.1 seconds):



The horizontal axis on these plots is the sequence number, and the vertical axis is the measured queueing delay in milliseconds. In addition to the ASCII art plot, the data files `ping_delay.csv` and `originalping.csv` and the image file `ping_delay.png` will have been created in the home directory of each of the client nodes, and a data file `router_pie.txt` will have been created in the home directory of the router. If you need to, you may transfer these files off the nodes with `scp` for additional analysis.  (If you don't copy or move the files, they will be overwritten the next time the experiment runs.)

To run the RED experiment, prepare the commands in the following table and then run them all at the same time:

<table>
<thead>
<tr>
<th>Node</th>
<th>Command</th>
</tr>
</thead>
<tbody>
<tr>
<td>router</td>
<td><code>pie-v-red-router red</code></td>
</tr>
<tr>
<td>client1</td>
<td><code>pie-v-red-client server1 250 0</code></td>
</tr>
<tr>
<td>client2</td>
<td><code>pie-v-red-client server2 150 50</code></td>
</tr>
<tr>
<td>client3</td>
<td><code>pie-v-red-client server3 50 100</code></td>
</tr>
</tbody>
</table>

The plots for RED show a much higher queueing delay, as in the original experiment we are trying to reproduce, but we don't see any variation with traffic intensity:

![](/blog/content/images/2016/08/red-nolimit.png)

From the network monitor display on the router node, it is clear that the 10 Mbps link is saturated for the entire duration of the experiment, even when there is only a small number of flows, so there is no meaningful variation in traffic intensity.

To try the modified experiment, where each traffic flow is rate limited so as to force variation in traffic intensity over the duration of the experiment, run the following command on each _server_ node:

```
sudo cp /etc/vsftpd-limit.conf /etc/vsftpd.conf
sudo service vsftpd restart
```

and then repeat the steps to run the PIE and RED experiments.

With the rate limiting of the individual TCP flows, we can see on the router node that the bottleneck link is _not_ always saturated, rather its load varies according to the number of TCP flows. In the image below, we can see the difference in traffic intensity when the second "set" of TCP flows turns on midway through:

![](/blog/content/images/2016/08/router-variation.png)

Furthermore, we can now confirm that with variation in traffic intensity, the RED queue has varying queueing delay:

![](/blog/content/images/2016/08/red-limited.png)

but the delay of the PIE queue remains constant near the target of 20ms:

![](/blog/content/images/2016/08/pie-limited.png)

To restore the original configuration with no rate limiting of individual FTP traffic flows, run the following on each of the server nodes:

```
sudo cp /etc/vsftpd-nolimit.conf /etc/vsftpd.conf
sudo service vsftpd restart
```

To create the image in the header of this blog post, I saved the `ping_delay.csv` of the RED experiment from a single node as `ping_delay_red.csv`, and of the PIE experiment as `ping_delay_pie.csv`. Then I ran [this gnuplot script](https://gist.github.com/ffund/07799036bc914298ae42ce37db9be2e8) to create the following image:

![](/blog/content/images/2016/08/ping_delay-3.svg)

### Run my experiment - extended version

First, you will need to get my materials. Open a terminal and run
	
	git clone https://bitbucket.org/by626/pievsred.git
	
to get my scripts and some .txt files as examples. Go to the directory of **./pievsred** then continue with the  following steps.


#### Reserve resources
 
Next, you will reserve resources on GENI. Create a new slice on [GENI](https://www.geni.net) and click the "Add Resources" button. Then select the "URL" option in the "Choose RSpec" section, and load the RSpec from the following URL:

[https://bitbucket.org/by626/pievsred/raw/master/final_by626\_request\_rspec.xml](https://bitbucket.org/by626/pievsred/raw/master/final_by626_request_rspec.xml)


Click on "Site 1" and choose an InstaGENI aggregate, then click "Reserve Resources". This will reserve the following topology:

![](/blog/content/images/2016/08/pie-v-red-topology.png)

Wait until all your nodes turn green, indicating that they are ready to log in.


#### Get VMs' information and upload scripts to VMs

Get the details of each VM such as IP address, host name and port number, from the GENI Portal. You should also log into each server and the router to find the interface corresponding to the IP address. Create 8 files with suffix .txt to store those informations. The file names and contents are:

* `USERNAME.txt`: only one line (your username)
* `CIP.txt`: each line has one client's IP address
* `SIP.txt`: each line has one server's IP address
* `CPORT.txt`: each line has one client's port number
* `SPORT.txt`: each line has one server's port number
* `CNAME.txt`: each line has one client's hostname
* `SNAME.txt`: each line has one server's hostname
* `ROUTER.txt`: the first line is router's hostname and the second line is router's port number

You should be careful that the same line in `CIP.txt`, `CPORT.txt`, `CNAME.txt` corresponds to the **same** client and the same line in `SIP.txt`, `SPORT.txt`, `SNAME.txt` corresponds to the **same** server. (In my situation,  the information for server 3 are all at the third line on `SIP.txt`, `SPORT.txt` and `SNAME.txt`) 

#### Prepare for the experiment

First, update your `iproute2` package. Go to [this link](https://launchpad.net/ubuntu/+source/iproute2/3.14.0-1) and choose the corresponding version for your own situation. You can type `uname -m` on terminal to check the machine hardware. If you see "x86_64" you need to choose *amd64* under "Builds", or you'll choose *i386* wihch refers to the 32-bit edition.

After you click the *amd64* or *i386*, you could find the Build files on the new page. Right click that file and choose *Copy Link Location* (or the one which has the same meaning with "copy link location"). Then on terminal (on router), type

	wget <Link>
	
where < Link > is the link you just copied.

Type
		
	sudo dpkg -i iproute2_3.14.0*

Next, run

	sudo apt-get update
	sudo apt-get install linux-generic-lts-vivid
	
to get the new kernel.

Finally, run

	sudo /sbin/reboot
	
to reboot the system.

##### Upload scripts to all VMs

On your local machine, run 

	./upload.sh
This will upload each bash script to the corresponding remote VM.

##### Set up `vsftpd`

This step should be completed on each of the three servers. 

Log into one server, run

	sudo apt-get update
	sudo apt-get install vsftpd

to install vsftpd. After you finish that, on your local machine, run 

	scp -P <port> ./vsftpd.conf <username>@<hostname>:./

where < port > is server's port number and <hostname> is server's hostname. Then on server, run

	sudo cp vsftpd.conf /etc/
to copy the config file for vsftpd to the right location so that anonymous ftp will be allowed. 

Then on your local machine, run 

	scp -P <port> ./test.rmvb <username>@<hostname>:./

to copy the files to the server( each file is 253MB so this may take several minutes). Here the < port > and the < hostname > also mean the port number and hostname of that server. Then on server, run

	./cp.sh

This will create twenty files and put them into the default FTP directory to let clients get those files. (This will also take several miniutes. After it's done you can run `ls /srv/ftp/` to check that all the files are in that location.)

On the other two servers, do the same thing.


### Experiment starts!

#### Get RTT before setting the queue discipline

In order to set the bottleneck RTT, run
	
	./bottleneck_rtt.sh

on your local machine to set delay=100ms on 3 clients.

Then still on your local machine, run

	./pingtest.sh

and after 30 seconds, you will see the average RTT measured by each server. The value will also be automatically written to a .txt file with the name of each client's IP address. (Don't worry if you see *"rm: cannot remove ‘rtt.csv’: No such file or directory"*. The file will be created later.)

#### Some configurations

To get the configuration the same with what the paper said, on your local machine run

	./modify.sh
	
The output means that you've already set the TCP New Reno(The “Reno” congestion control provided by the Linux kernel is actually the  New Reno algorithm) with SACK on each server.

#### Set queue discipline on router

On router, run

	./tc.sh <qdisc> <interface>
	
where < qdisc > should be **pie**, which is the queue descipline we are testing and < interface > is the interface connected to the clients. The output tells you the current settings of the queue and you could ignore "RTNETLINK answers: No such file or directory" if you see that on the first line. Now you should see a new line *qdisc pie 10:*.

![Imgur](http://i.imgur.com/x3c1LyC.png)

#### Start traffic
On your local machine, run

	./all.sh <qdisc> <interface>
	
where < interface > is the same interface where you set the queue and < qdisc > is the queue discipline(pie or red). This will first start running vsftpd on each server to let three clients retrieve files from them. It will last around 5 minutes and you should do nothing during this whole process. When you first run it you probabily see the error like 
```
/usr/bin/xauth:  file /users/by626/.Xauthority does not exist
```

If you see that, just wait for it to finish and run `all.sh` again. Similarly, you can also ignore the message like 

```
Warning: time of day goes back (-1225680us), taking countermeasures.
```
	
#### Get data from VMs

After you see "servers stopped" which means you've finished your test, run

	./getdata.sh <qdisc>

on your local machine and < qdisc > should be the queue discipline the same as step 3 and 4 in section B. In both situations, you will get 3 files on your local machine.

#### RED experiment

Change < qdisc > into **red** and then repeat step 3-4 in section B.

#### Visualize the results

Some of the files you just got is used to plot the results in MATLAB so make sure those files (including **rtt.txt** you got in step 1 of section B) and the matlab scripts are at the same location.

I have two MATLAB scripts, pie.m and red.m. They will show you the delay from ping (minus the delay before setting the queue). For `ylim([0 300])`, you may have some other settings based on your situation.


#### Modified experiment with rate limiting

The "flat" result in RED queue is not the same with the orginal result. We could not see the remarkable difference between queueing delay when traffic intensity changes. Maybe the reason is that even a small number of flows already saturate the link and therefore, if we increase the number of flows, there will not be an actual increase in traffic intensity. Therefore, I add a modified version of this experiment. On each server, run 

	sudo vi /etc/vsftpd.conf

and uncomment the last line **anon_max_rate = 25000**, which will limit the maximum data transfer rate to 25kB/s. If we have 50 sources limited by it, the maximum data rate is 10Mb/s, which is just the bottleneck link bandwidth. Below are the results of this modifed experiment.

## Release resources

After finish the experiment and if you do not need to check them anymore please release all the resources.


## Notes


### External sources

The `queuemonitor.sh` bash script is from [this lab assignment](http://witestlab.poly.edu/~ffund/el7353/2-testbed-mm1.html).
	
I referred to these Ubuntu manuals: [ftp](http://manpages.ubuntu.com/manpages/trusty/man1/tnftp.1.html), [pie](http://manpages.ubuntu.com/manpages/xenial/en/man8/tc-pie.8.html), [red](http://manpages.ubuntu.com/manpages/xenial/en/man8/tc-pie.8.html)
	
Other reference documents: [htb manual](http://luxik.cdi.cz/~devik/qos/htb/manual/userg.htm), [Linux Advanced Routing & Traffic Control HOWTO](http://www.lartc.org/howto/)


### Software versions

I ran this experiment on an InstaGENI aggregate, on VM with the following kernel version: Linux ubuntu 3.19.0-58-generic #64~14.04.1-Ubuntu (64 bit)

The visualization scripts were test on Matlab R2015a(8.5.0)
