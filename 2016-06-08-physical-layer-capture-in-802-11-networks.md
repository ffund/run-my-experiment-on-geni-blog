
### Introduction ###
This project is about the *capture effect* of Wi-Fi (IEEE802.11a) physical layer. Based on capture effect, sometimes the shared wireless channel can carry more than on packet at the same time without collision. There is a theoretical model for IEEE802.11a in the following paper which considers this effect:

**[1] Chang, Hoon, Vishal Misra, and Dan Rubenstein. "A General Model and Analysis of Physical Layer Capture in 802.11 Networks." INFOCOM. 2006.**    

We are going to evaluate figure 1 of this paper by a set of extensive experiments on ORBIT test-bed. The total time it may take to do this experiments is about 14 hours. 


### Background ###

In most of the models suggested to analyze the performance of Wi-Fi, it is assumed that if there are two or more packets in the shared wireless channel, collision happens and those packets should be sent again. However, in practice sometimes it is possible that two or more packets transmitted successfully thorough the shared channel, because correct detection at the receiver only depends on the signal to the noise plus interference power (SINR) of the bits. This is called capture effect and we will study it in this project. In fact, SINR is the key quantity and if it is above a threshold, then the detection is successful with a high probability. This means that even if we have interference due to transmission of other packets, if the signal power is enough high and also the SINR threshold is enough low, then the packet can be detected successfully. The SINR threshold is proportional to the target data rate of the link. If the transmitter (TX) and receiver (RX) agree on a higher data rate, then they use a more complicated modulation with denser constellation which requires higher minimum SINR for successful detection at the RX. To evaluate the simulation results of [1], we are going to do a set of experiments. In each experiment we have a number of Wi-Fi ad-hoc links which are active with a certain target rate for all of them. The number of the links and the target rate are varied across the experiments and the cumulative throughput (sum throughput of the links) is obtained for each case. 
 
### Results  ###

Below figure, the simulation results of [1], shows the behavior of the network throughput as the number of active links grow for different target rates.  

![figure1.png](https://bitbucket.org/repo/8k8RGo/images/4083723413-figure1.png)

Figure 1 illustrates that transmitting at lower rates yields higher throughput as the number of nodes increases. This is because packets are captured with lower probability for high transmission rates. When a sender-receiver pair has only a small number of neighbors, packets transmitted at low rates are not lost and throughput linearly increases with the number of nodes. As the number of neighbors is increased, packet loss rates escalate
and throughput reduces as a result [1].


Figure 2, illustrates the empirical counterpart of Figure 1.

![empirical result.png](https://bitbucket.org/repo/8k8RGo/images/697869767-empirical%20result.png)

Let R and N denote target rate and number of the links, respectively. Although the shape of the curves is different from Figure 1, we can see some similar trends. If we look at the curves which correspond to R = 18,24,36 Mbps, we observe that the rate increases almost linearly as the number of the links increases up to N=12 and does not change much after it. However, for R = 48, 54 Mbps, the network throughput first increases and N = 8, it starts to decrease. This is because of the higher SINR threshold required for correct detection in higher target rates. Besides, it seems that the network throughput does not change much after N = 12 for all of the target rates. The reason is after some point, the marginal throughput that is added to the network throughput due to adding a link is almost mitigated by the interference that link adds to the network. On the other hand, we can observe some points in Figure 1 which are not supported by empirical results in Figure 2. Based on Figure 1, as the network gets crowded due to the increase in the number of the links, higher target rates lead to lower network throughput. Clearly, it is because of the higher SINR thresholds the RX needs for higher target rates to detect the packet correctly. However, we do not observe this behavior in empirical results. The reason can be the smaller network area in the experiments. To compensate for this difference in the network area, different transmission powers could be used, but unfortunately when I tried 16, 6 and 2 dBm as the transmit power, the behaviors did not change. Other potential reason can be the node placement method. In the simulations of reference paper [1], the number of active links goes from 1 to 64 (Figure 1). However, due to the limitations of ORBIT testbed, it is almost impossible to have more than 20 active links at the same time. Some of these limitations are:

* As opposed to what we say, ORBIT grid has about 150 available nodes and not 400 nodes (20*20).
* Among the available nodes, some of them have problems and you cannot use them for the experiments.
* Among the available nodes which can work correctly, some of them do not support IEEE 802.11a.

Like always, the problems in experiment design also can be one of the origins of the differences which I hope is not the case.

### Run my experiment ###

Before starting, please note that the following specifications have been used to produce Figure 2:

* Linux version: Ubuntu 14.04.4 LTS
* kernel version 3.13.0-79-generic
* iperf version (traffic generator): 2.0.5-3
* bc version (calculator): 1.06.95-8 ubuntu1

In this section, a step by step guidance is presented to run the experiments and reproduce the results. To do the 
experiments first you need to reserve grid network of [ORBIT testbed](https://geni.orbit-lab.org/wiki/Tutorials/a0Basic/Tutorial1#ORBITBasics:SixStepExperimentLifeCycle). To reserve the time slots, you should log on to your GENI portal and select ORBIT from "Partners" in top menu. Then, click on "Control Panel", and then "Online Scheduler". To get more familiar with ORBIT testbed see these: [link1](https://geni.orbit-lab.org/wiki/Hardware/bDomains/aGrid#Grid), [link2](https://geni.orbit-lab.org/wiki) 


After reserving ORBIT grid, you should do the following steps:

1- Log on to the grid using your GENI/ORBIT ID. To do this, open a terminal (e.g. git bash terminal) and use *ssh* to log on to the grid. For example, if you use ORBIT with your GENI account you should use the following command:

```
#!bash
ssh geni-GENI_USER_NAME@grid.orbit-lab.org 

```
Remember that you should be logged on through the whole next steps.

2- You need to prepare the nodes to set up Wi-Fi ad-hoc links. For this, first you need to create a group of nodes to prepare them to be the potential transmitters/receivers on the links. Note that if you want to have *N* ad-hoc links, you need to have *2N* nodes. Please also note that for this particular experiment, every node should have *ath5k* wireless driver. To check, go to this [ORBIT web site](https://geni.orbit-lab.org/), click on "Control Panel", then click on "Status Page" and finally click on "grid". If the group has more nodes, preparation takes more time. Following example tells you how to create the group. Remember that to have a reliable experiment, all of the nodes which you want to use should be included.

```
#!bash

GROUP=node17-11.grid.orbit-lab.org,node17-10.grid.orbit-lab.org,node14-11.grid.orbit-lab.org,node14-10.grid.orbit-lab.org,node16-16.grid.orbit-lab.org,node15-15.grid.orbit-lab.org,node14-14.grid.orbit-lab.org,node13-13.grid.orbit-lab.org,node15-1.grid.orbit-lab.org,node16-1.grid.orbit-lab.org,node7-10.grid.orbit-lab.org,node7-11.grid.orbit-lab.org,node4-10.grid.orbit-lab.org,node4-11.grid.orbit-lab.org,node20-10.grid.orbit-lab.org,node20-12.grid.orbit-lab.org,node5-5.grid.orbit-lab.org,node6-6.grid.orbit-lab.org,node7-7.grid.orbit-lab.org,node8-8.grid.orbit-lab.org,node11-4.grid.orbit-lab.org,node10-4.grid.orbit-lab.org,node10-1.grid.orbit-lab.org,node11-1.grid.orbit-lab.org,node4-1.grid.orbit-lab.org,node5-1.grid.orbit-lab.org,node1-11.grid.orbit-lab.org,node1-12.grid.orbit-lab.org,node1-13.grid.orbit-lab.org,node1-15.grid.orbit-lab.org,node1-15.grid.orbit-lab.org,node1-16.grid.orbit-lab.org,node1-17.grid.orbit-lab.org,node1-18.grid.orbit-lab.org,node5-16.grid.orbit-lab.org,node6-15.grid.orbit-lab.org,node7-14.grid.orbit-lab.org,node8-13.grid.orbit-lab.org,node10-20.grid.orbit-lab.org,node9-20.grid.orbit-lab.org,node8-20.grid.orbit-lab.org,node7-20.grid.orbit-lab.org

```
You can use above group in this experiment or you can use your own.

3- Run the following command in the terminal and wait until it is done and a new command line opens in your terminal.

```
#!bash
omf load -i baseline.ndz -t "$GROUP"

```

4- Run the following command in the terminal and wait until it is done and a new command line opens in your terminal.

```
#!bash
omf tell -a on -t "$GROUP"

```
A report will be printed on the screen which shows the status of the nodes of the group. Note that you should see the word "OK" in front of every node name. Otherwise, you cannot use those nodes reliably. In this case, if the nodes with "OK" are not enough for you, either restart the experiment from step 2 with the same group of nodes or restart from step 2 with another group of nodes. 
Below figure, shows a successful try: 

![Capture2.PNG](https://bitbucket.org/repo/8k8RGo/images/1270539292-Capture2.PNG)


5- Now, the nodes are prepared and you should set up the links. Each link needs two end nodes which should be introduced to each other. To make the life easier, there is a bash script (*linkprep.sh*) in this repository which you can run to set the links automatically. To use it, first you should move it to ORBIT testbed VM node using *scp* command. Besides, since two other scripts (*wlink.sh* and *apt.sh*) are called in *linkprep.sh*, you need to copy them on the ORBIT VM too. After moving these three scripts to the VM you use, run the following command:


```
#!bash
bash linkprep.sh N

```
where N is the number of the links and should pass as an argument to *linkprep.sh* script. You can set N to 1, 2, 4, 8, 12 and 16.
In any of these cases, the script sets up the links automatically for you. However, if you want to use another N, you should use 0 and modify linkprep.sh to include your own configuration. To do this, you can use *nano* editor to open the script and modify the following part of the code:


```
#!bash

if [ "$link" == "0" ]
then

	# define your links here. 
	declare -a TXNODES=("")
	declare -a RXNODES=("")
	declare -a TXIP=("")
	declare -a RXIP=("")
	declare -a LINKNUM=("")

fi

```

After running *linkprep.sh*, the script tries to set up the links. Sometimes, the script cannot log on to one or more nodes and you get an error in which it tells you which nodes have a problem. For example you may see the following error on the screen:

```
#!bash

ssh: connect to host node1-11 port 22: No route to host
```



This means that node7-7 is a part of the experiment and *linkprep.sh* wanted to log on to that node to set the link parameters, but it could not log on. The first thing to do is to run *linkprep.sh* a few more times with some delay between the trials. Below figure describes one of these situations that may happen in this step. As this figure shows, after running *linkprep.sh* with N=8, the script could not log on to node1-11, but in the second try it could and the problem has been resolved:

![Capture2.PNG](https://bitbucket.org/repo/8k8RGo/images/2840511962-Capture2.PNG)

If the errors are not resolved, you should try another solution. Let us say node X (e.g. X=node1-11) has a problem based on the printed report on the screen. In this case, you can try to log on to the node manually using the following command:

```
#!bash

ssh root@nodeX
```
Sometimes you should try this command a few times to enter the node. If you could log on the node use *exit* command to exit the node and try this process for all of other nodes which have a problem. Then, run *linkprep.sh* again and this time you should not see any error. In the case you see error again, there is a problem in one of the previous steps or one of the nodes does not support IEEE802.11a. My suggestion is to start from step 2 again in those cases. Of course, if you use the node group in example of step 2 and one of the predefined set of links (i.e. choosing N from {1,2,4,8,12,16}), then you do not encounter with such repetitive errors frequently.


6- In this step, you should activate the links. To do this, TX node of each link should send UDP packets to RX node. To make this happen, you can use *supervisor.sh* and *activate.sh* bash scripts from this repository (the second script is called in the first one). You should copy them to the ORBIT VM using *scp* command. Then you should run *supervisor.sh* with an argument:


```
#!bash

bash supervisor.sh N
```
where N is the number of links. You should use the value that you used in step 5 for N. After running *supervisor.sh*, the links are activated and you should wait for the experiment to be done. In each run of *supervisor.sh*, the experiment is done for each values of link target rate (R ={18,24,36,48,54} Mbps). After each experiment ends, the value of network throughput is printed on the screen with some hints to help you. You should wait until 5 values for network throughput are printed on the screen. Below figure depicts the screen after the experiment is done.

![Capture5.PNG](https://bitbucket.org/repo/8k8RGo/images/3161897837-Capture5.PNG)

7- To reproduce Figure 2, first you should do steps 1-4. Then you do step 5 to set the number of the links and fix the network configuration. After it you should do step 6 to find the network throughput for each target rate and the number of the links you set in step 5. I suggest to repeat step 6 a few times and take the average of the network throughput values for each target rate. This is necessary because this experiment includes randomness and at least it needs some iterations. Now, you can do step 5 again to change the number of the links. We conclude that for each value of N, you should do step 5 first and then do step 6 a few times to get the data. Then you can do data analysis and reproduce Figure 2 by plotting the average network throughput with respect to the number of the links for each target rate (5 curves).