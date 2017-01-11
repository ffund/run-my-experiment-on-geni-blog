This experiment came out of a software radio design challenge used in coursework at New York University and the University of Thessaly. (See [Report on university course based on DARPA Spectrum Challenge](https://lists.gnu.org/archive/html/discuss-gnuradio/2015-02/msg00100.html).) The goal of the project is to design a radio pair that delivers more data from transmitter to receiver than an "opposing" radio pair, when both radio pairs are operating in the same frequency band. In this particular implementation, we seek to generate interference from our receiver to impede our opponent's ability to receive its own communication. However, we have to balance the benefit caused to us by interfering with our opponent's transmission, against the harm caused by self-interference on the adjacent channel.


It should take about 60-120 minutes to run this experiment, but you will need to have [reserved that time](http://geni.orbit-lab.org) in advance. This experiment uses wireless resources (specifically, parts of the grid testbed on [ORBIT](http://geni.orbit-lab.org)), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have [reserved time on the grid testbed at ORBIT](http://geni.orbit-lab.org), and you must run this experiment during your reserved time.


##Background

This experiment came out of a software radio design challenge used in coursework at New York University and the University of Thessaly. (See [Report on university course based on DARPA Spectrum Challenge](https://lists.gnu.org/archive/html/discuss-gnuradio/2015-02/msg00100.html).) The goal of the project is to design a radio pair that delivers more data from transmitter to receiver than an "opposing" radio pair, when both radio pairs are operating in the same frequency band. In this particular implementation, we seek to generate interference from our receiver to impede our opponent's ability to receive its own communication. This experiment came out of a software radio design challenge used in coursework at New York University and the University of Thessaly. (See [Report on university course based on DARPA Spectrum Challenge](https://lists.gnu.org/archive/html/discuss-gnuradio/2015-02/msg00100.html).) The goal of the project is to design a radio pair that delivers more data from transmitter to receiver than an "opposing" radio pair, when both radio pairs are operating in the same frequency band. 


We use an open source software-defined radio package called GNU Radio and a basic radio hardware component called the Universal Software Radio Peripheral (USRP).

GNU is a project that was created to provide a Unix-like operating system environment that is comprised of free software. Building upon the GNU free software concept, GNU Radio is a free software implementation of actual radio hardware. It is includes a set of signal processing blocks that can be used to implement software defined radio transceivers together with external RF hardware.

This project requires each "team" to transmit between a pair of nodes, on the same frequency band. We aim to maximize the throughput of our own communication, while simultaneously competing against the transmission of an opposing  pair of nodes. 

Our design focuses on receiving the maximum possible amount of packets correctly in our own receiving node, while reducing the number of packets that received correctly at the opponent channel. This is achieved by interfering on the opponent's channel (jamming). However, we have to balance the benefit caused to us by interfering with our opponent's transmission, against the harm caused to our own receiver by self-interference on the adjacent channel.


## Results

First, we show how the throughput of the receiving channel, for a simultaneously transmitting and receiving Rx node, is affected as we vary the amplitude on its transmitting channel. We compare this behavior of the node’s receive channel for an adjoining and a non-adjoining transmitting channel to its receiving channel.

![](/blog/content/images/2016/03/Screen-Shot-2016-03-07-at-2-32-46-AM.png)
[Figure1](https://plot.ly/~Charalampos/4/?share_key=S9DDtChL4HLes7jMYNKuvN)

Then, for a non-adjoining transmitting channel, we interfere to an opponent node’s receiving channel. In this occasion, we examine how varying the amplitude of the interferer affects its own and the opponent’s node receiving channel.
![](/blog/content/images/2016/03/Screen-Shot-2016-03-07-at-2-23-56-AM.png)
[Figure2](https://plot.ly/~Charalampos/6/?share_key=BqpiRiYzj8cdQolD7CZuVu)

##Run my experiment
In order to run the experiment, we should have a reservation in ORBIT’s main grid. We need two pairs of USRP nodes to run this experiment. You can choose any USRP nodes to run on the experiment, but this time we will use the following pairs of nodes (1-1, 20-1) and (1-1, 20-2) respectively.
During your reservation, SSH into the testbed using your GENI wireless username and ssh key by typing:
`ssh username@console.grid.orbit-lab.org`

Once you are on the testbed console, load a baseline SDR disk image onto the nodes with the command:
```
omf load -i el9043-2015.ndz -t node1-1.grid.orbit-lab.org,node1-2.grid.orbit-lab.org,node20-1.grid.orbit-lab.org,node20-2.grid.orbit-lab.org
```

This might take some time. Next, you have to turn up the nodes:
```
omf tell -a on -t node1-1.grid.orbit-lab.org,node1-2.grid.orbit-lab.org,node20-1.grid.orbit-lab.org,node20-2.grid.orbit-lab.org
```

Wait for some time for nodes to turn on. Then, SSH in to the four nodes from separate consoles using the following commands:
```
ssh root@node1-1
ssh root@node1-2
ssh root@node20-1
ssh root@node20-2
```

Make sure that the USRP device is detected on each node by running:
[```uhd_find_devices```](http://www.orbit-lab.org/wiki/Tutorials/k0SDR/Tutorial00)

Load the experiment’s repository in each node:
```
cd el9043s2015
git remote add origin https://cmanolidis@bitbucket.org/cmanolidis/el9043s2015-cmanolidis.git
git pull origin master
```

Then, you have to compile GNU Radio code:
```
cd /root/el9043s2015/build
make
make install
```

The scripts that we use for this experiment are the following:
`benchmark_tx.py
benchmark_rx.py`

You can check the parameters that can be passed for both scripts by executing:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_tx.py -h
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -h
```

Now, we run some experiments with one transmitting node and two receiving nodes. The transmitting node is 20-1 and the receiver nodes are 1-1 and 1-2.
Νode 1-1 uses only its receiving channel while node 1-2 uses both receiving and transmitting channels simultaneously.

In all our experiments we will use GMSK modulation with 4 constellation points for the communication between Tx and the two Rx nodes while their communication channel’s central frequency is set at 1600Mhz, their channel bandwidth is 0.75Mhz (bitrate = 333325), their tx-amplitude is 1.00, their tx-gain is 87.00 and finally their rx-gain is 38.00. The node 1-2 in addition to 1-1 has a transmission channel with a bandwidth of 0.5625Mhz (bitrate = 249993.75) and tx-gain 35.0 while we will examine its behaviour for various transmitting amplitude values.

![](/blog/content/images/2016/03/Screen-Shot-2016-03-07-at-1-55-19-AM.png)

Before to run the first series of our experiments, we set the 1-1 node’s transmitting channel central frequency at 1600.5625Mhz next to it’s receiving channel (1600Mhz) like in the above picture and we run some experiments for several values on its transmitting amplitude. In order to run the experiment we first have to run the scripts on receiving nodes and then to the transmitter. We set he transmitting amplitude of node 1-2 at 0.03. On node’s 1-2 console we run the benchmark_rx script and wait for a few seconds:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1600.5625e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.030
```

Next, we do the same on 1-1 node, giving the following command: 
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0
```
 
Before starting the benchmark_tx script on 20-1 node, we check that the benchmark_rx scripts are running on 1-1 and 1-2 nodes. Having confirmed that, we are ready to start node’s 20-1 transmission:

``` 
~/target/share/gnuradio/examples/digital/narrowband/benchmark_tx.py -m gmsk -p 4 -r 333325 --tx-freq 1600.0e6 --tx-gain 87.0 --tx-amplitude 1.00 --packets 5000
```

Node 20-1 will send 5000 packets at the frequency that nodes 1-1 and 1-2 share. As nodes 1-1 and 1-2 receiving packets they start printing messages like this:
``` 
ok = False  pktno = 3976  n_rcvd = 3973  n_right = 2270
ok = False  pktno = 3977  n_rcvd = 3974  n_right = 2270
ok =  True  pktno = 3978  n_rcvd = 3975  n_right = 2271
ok =  True  pktno = 3979  n_rcvd = 3976  n_right = 2272
ok = False  pktno = 3980  n_rcvd = 3977  n_right = 2272
``` 
When node’s 20-1 transmission has finished, we get a message like this:
``` 
Packets sent: 5000
Transmission duration: 180.041602135sec
``` 
Then we have to stop the execution of benchmark_rx scripts on 1-1 and 1-2 nodes by pressing `ctrl` + `c`.

In the last line of each node (1-1 and 1-2) we can see the amount of packets that the node received correctly (n_right). We can use these values from Tx and Rx nodes scripts to calculate the channel’s throughput (note that default packet size is 1500B):

`Throughput in B/sec: packet size *
correctly received packets / transmition duration`

Using this formula we collect the throughput data for every series of our experiments.
Finaly for each series we will plot its throughput curve in relation to the transmitting amplitude that we used in the 1-2 node.

In the next experiments, in the nodes 20-1 and 1-1 we run the scripts with the same values as before. The only script’s that we run with different values is this of 1-2 node’s.
Following what was just described, we run the experiment for the following values of transmitting amplitude on 1-2 node.

* Transmitting amplitude value 0.035:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1600.5625e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.035
```
* Transmitting amplitude value 0.040:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1600.5625e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.040
```

* Transmitting amplitude value 0.045:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1600.5625e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.045
```

* Transmitting amplitude value 0.050:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1600.5625e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.050
```

* Transmitting amplitude value 0.055:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1600.5625e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.055
```

* Transmitting amplitude value 0.060:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1600.5625e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.060
```

We finally got throughput values for the following transmitting amplitude values 0.035, 0.040, 0.045, 0.050, 0.055, 0.060.

![](/blog/content/images/2016/03/Screen-Shot-2016-03-07-at-1-56-01-AM.png)

When we have obtained all the results, we start a new series of experiments but this time on node 1-2 we change the transmitting central frequency (--int-freq) to 1601.125Mhz. The transmitting signals are like the picture’s above.

As in the last series of experiments the only script that we use different values compared with the first experiment’s scripts values is the script on 1-2 node.
We run the experiment for the following values of transmitting amplitude on 1-2 node and collect the.

* Transmitting amplitude 0.075:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.075
```

* Transmitting amplitude 0.1:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.1
```

* Transmitting amplitude 0.125:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.125
```

* Transmitting amplitude 0.15:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.15
```

* Transmitting amplitude 0.175:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.175
```

* Transmitting amplitude 0.2
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.2
```

* Transmitting amplitude 0.225:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.225
```

* Transmitting amplitude 0.25:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.25
```

* Transmitting amplitude 0.3:
```
 ~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.3
```
 

When all experiments are finished we get results similar to Figure 1. We are now ready to start our last series of experiments.


* Transmitting amplitude 0.075:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.075
```

* Transmitting amplitude 0.1:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.1
```

Transmitting amplitude 0.125:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.125
```


* Transmitting amplitude 0.15:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.15
```


* Transmitting amplitude 0.175:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.175
```

* Transmitting amplitude 0.2
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.2
```

* Transmitting amplitude 0.225:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.225
```

* Transmitting amplitude 0.25:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.25
```

* Transmitting amplitude 0.3:
```
 ~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.3
``` 

When all experiments are finished we get results similar to *Figure1*. We are now ready to start our last series of experiments.


The previous experiments are used to discover how the transmitting channel of a node affects its receiving one. Now, let’s see what happens when we tune this channel and interfere to an opponent’s communication channel.
 
This time we use two pairs of nodes. Namely, pair (20-1, 1-2) and pair (1-1, 20-2). The first pair uses the same central frequency (1600Mhz) for the communication channel as before, and nodes 20-1 and 1-2 act as a transmitter and receiver respectively. The other pair has a new communication channel, with a central frequency of 1601.125Mhz, where 1-1 is the transmitter node and 20-2 is the receiver node. Both pairs have bandwidth of 0.75Mhz. You can see the transmitting channel of both pairs in th next picture (avoid the signal at  1598.9Mhz).

![](/blog/content/images/2016/03/Screen-Shot-2016-03-07-at-1-56-33-AM.png)

The node 1-2 uses its transmitting channel with a bandwidth of 0.5625Mhz, to interfere with the communication channel between 1-1 and 20-2 nodes at 1601.125Mhz. We examine how the interferer (transmitting channel) of node 1-2 is affecting it’s own receiving channel and the opponent Rx node’s (20-2) channel for the following amplitude values:

`--int-amplitude 0.1`, 
`--int-amplitude 0.1125`, 
`--int-amplitude 0.125`, 
`--int-amplitude 0.1375`, 
`--int-amplitude 0.15` 
`--int-amplitude 0.1625` 
`--int-amplitude 0.175` 
`--int-amplitude 0.1875` 
`--int-amplitude 0.2`

Let’s start our last series of experiments with first amplitude value being 0.1. As before, we first have to run the scripts on Rx nodes, before Tx nodes start transmitting. On node 1-2 we type the command:
``` 
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1600e6 --rx-gain 38.0 --interferer --int-freq 1601.125e6 --int-gain 35.0 --int-bitrate 249993.75 --int-amplitude 0.1
```

* On node 20-2:
``` 
~/target/share/gnuradio/examples/digital/narrowband/benchmark_rx.py -m gmsk -p 4 -r 333325 --rx-freq 1601.125e6 --rx-gain 38.0
```

When scripts on Rx nodes are running we are ready to start the respective scripts on Tx nodes. On node 1-1:
```
~/target/share/gnuradio/examples/digital/narrowband/benchmark_tx.py -m gmsk -p 4 -r 333325 --tx-freq 1601.125e6 --tx-gain 87.0 --tx-amplitude 1.00 --packets 5000
```

* On node 20-1:
``` 
~/target/share/gnuradio/examples/digital/narrowband/benchmark_tx.py -m gmsk -p 4 -r 333325 --tx-freq 1600.0e6 --tx-gain 87.0 --tx-amplitude 1.00 --packets 5000
```

Following these steps we run the experiments for the rest of the transmitting amplitude values and get similar results like in *Figure2*.

##Results visualization
For our figures we used [plot.ly](https://plot.ly). Plotly is an online analytics and data visualization tool. You can create an acount on plotly for free. As you logged into plotly you create a new grid project. Now you can fill your experiments data in the grid’s collumns.
When you are ready to plot your data, at the menu bar you select `CHOOSE PLOT TYPE` and then `Line plots`.

![](/blog/content/images/2016/03/Screen-Shot-2016-03-07-at-4-50-30-AM.png)

Next set your node’s 1-2 transmitting amplitude as **x axis** and your several throughput data as **y axis**. Then choose to plot all those data as a `Line plot` and plot your data by pressing `Line plot` on top left.

![](/blog/content/images/2016/03/Screen-Shot-2016-03-07-at-5-03-56-AM-1.png)

You can see the grids of the previous experiments series here:
[Figure1 data](https://plot.ly/~Charalampos/4/?share_key=S9DDtChL4HLes7jMYNKuvN), [Figure2 data.](https://plot.ly/~Charalampos/6/?share_key=BqpiRiYzj8cdQolD7CZuVu)