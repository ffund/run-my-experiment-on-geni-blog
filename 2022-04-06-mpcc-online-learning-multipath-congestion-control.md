## Background

Multipath transport deals with the transmission and reception of internet traffic using multiple network paths at the same time. Multipath transport capability can be used to increase the effective network throughput for bandwidth hungry applications. Alternatively, multiple network paths can also be used to increase the reliability of communication. Several multipath protocols have been designed to enable multipath communication on the internet. One such protocol is Multipath TCP (MPTCP) which is the multipath extension of the standard Transmission Control Protocol (TCP).

An integral component of TCP is congestion control which ensures reliable end-to-end transmission of packets, at a rate which allows utilisation of available network capacity while keeping a check on network congestion. While the theory and design of congestion control protocols is quite extensive, congestion control protocols specifically designed for the multipath scenario are relatively new. MPTCP congestion control relies on the principles of resource pooling and load balancing. In MPTCP , each path has its own congestion control to react to the congestion seen on that path. At the same time, the congestion control parameters are linked across paths in order to move traffic away from the most congested paths.

Several MPTCP congestion control techniques have been designed in the last decade or so - e.g., [OLIA](https://www.ietf.org/proceedings/87/slides/slides-87-iccrg-7.pdf) and [BALIA](https://datatracker.ietf.org/doc/html/draft-walid-mptcp-congestion-control-00). Most of these protocols are loss-based AIMD (additive increase, multiplicative decrease) congestion control schemes, similar in nature to TCP Cubic and TCP Reno. The loss-based nature of these schemes can lead to several performance issues which are also common to loss-based single path TCP protocols, such as :

 - High self induced queuing delay due to the buffer-filling nature of loss-based schemes ( the ''bufferbloat'' problem)
 - Bad adaptation to network changes
 - Reaction to non-congestion related losses

To overcome some of these issues, the paper [``MPCC : Online Learning Multipath Transport''](https://dl.acm.org/doi/abs/10.1145/3386367.3433030) proposes a new multipath congestion control protocol. This scheme takes an online learning based approach with the goal of achieving high throughput, low queueing delay and low packet loss for MPTCP. In the next section, we will introduce this scheme. The **goal of this experiment** is to reproduce one result from this paper using experiments on the [Cloudlab](https://www.cloudlab.us/) networking testbed. 


#### MPCC congestion control

MPCC is the multipath extension of the PCC congestion control protocol. PCC, or performance-oriented congestion control is an online learning based single path TCP scheme, originally proposed in 2015. An updated version of PCC released in 2018, known as [PCC-Vivace](https://www.usenix.org/conference/nsdi18/presentation/dong) targets low queueing delay in addition to high throughput and low packet loss. 

PCC is a rate based algorithm, i.e.,  it directly adjusts the sending rate of the TCP sender in response to congestion. At any point of time during an active TCP connection, let's assume that the current sending rate of the PCC sender is *r*. 

Then, PCC sends traffic at rate *r + &epsilon;* and rate *r - &epsilon;* during the next two monitor intervals (typically = *1  RTT*), one by one. Basically, it tries out a slightly higher and a slightly lower sending rate in order to explore for better sending rates. For both these rates, it calculates a utility function. Based on the observed utility, PCC adjusts its sending rate in the direction of higher utility. It then takes a gradient-descent step in the direction of higher utility. The utility function for PCC Vivace is shown in the figure below. It increases with the throughput, penalizes any increase in latency due to the RTT gradient term, and also penalizes high packet losses.

![](/blog/content/images/2021/12/PCC_utility.png)

For the multipath version of PCC, utility functions should be designed such that they give high performance for individual subflows but also induce global properties like stability and fairness. Additionally, MPCC needs to enable reactions on different timescales, since the RTT of each path might be different from the others. The solution proposed in the paper is to have a separate utility function for each subflow, with some loose coupling between the subflows. The utility function of flow *i* in MPCC is shown in the figure below. 
![](/blog/content/images/2021/12/MPCC_utility.png)

The utility depends on the sending rates of all subflows, but only takes into account the latency gradient and loss rate of subflow *i*. This enables reaction at different timescales for different flows as subflow *i* does not have to keep track of the performance metrics of other subflows. The MPCC paper provides some theoretical analysis and shows that MPCC is guaranteed to converge to an equilibrium that is max-min fair in the case of multiple competing MPTCP flows.

####  Target Result 

For this experiment, our goal is to reproduce figure 9 of the [MPCC paper](https://dl.acm.org/doi/abs/10.1145/3386367.3433030). This result is shown in the figure below, taken from the original paper. 

![](/blog/content/images/2021/12/result.png)

This result compares the latency performance of different multipath CC protocols as the bottleneck buffer size is increased. The network under consideration is a two-link network with the bottleneck capacity on each link of *100 Mbps* and the base RTT of each link is *30 ms*. The buffer size is simultaneously increased for both links. This result is important for two main reasons : 

 - It shows that MPCC can achieve lower delays compared to other multipath congestion control schemes, satisfying one of the main goals of its design and utility function.
 - It verifies that loss-based multipath congestion control, which is the current default for all MPTCP deployments, induces high queueing delays and may not be suitable for low delay applications.

## Run my experiment

To run these experiments, you will need an account on [CloudLab](https://cloudlab.us/). If you already have a GENI account, you can use it to log in to CloudLab. Once you are logged in to CloudLab, open our  profile named `mptcp-ashutosh`: [https://www.cloudlab.us/show-profile.php?uuid=8c3a26d9-a6db-11eb-8fd9-e4434b2381fc](https://www.cloudlab.us/show-profile.php?uuid=8c3a26d9-a6db-11eb-8fd9-e4434b2381fc)

Click on ``Instantiate``, then choose CloudLab Wisconsin and start your experiment. Wait until all nodes have booted and are ready to log in, then click on the ``List view`` tab to get SSH login instructions for your nodes. Then, use SSH to open a shell session on each node in the experiment.

After instantiating the experiment on Cloudlab, follow these instructions : 

First, we configure some basic routing rules on all the nodes of the experiment.

On the `client` node: 
```

sudo ifconfig enp6s0f0 down; sudo ifconfig enp6s0f0 up
sudo ifconfig enp6s0f1 down; sudo ifconfig enp6s0f1 up
sudo route add -net 192.168.3.0/24 gw 192.168.10.1 
sudo route add -net 192.168.4.0/24 gw 192.168.20.1
```

On the `emulator1` node: 

```
sudo ifconfig enp6s0f0 down; sudo ifconfig enp6s0f0 up
sudo ifconfig enp6s0f1 down; sudo ifconfig enp6s0f1 up
sudo route add -net 192.168.3.0/24 gw 192.168.1.2 
```

On the `emulator2` node:

```
sudo ifconfig enp6s0f0 down; sudo ifconfig enp6s0f0 up
sudo ifconfig enp6s0f1 down; sudo ifconfig enp6s0f1 up
sudo route add -net 192.168.4.0/24 gw 192.168.2.2 
```
On the `router1` node:

```
sudo ifconfig enp6s0f0 down; sudo ifconfig enp6s0f0 up
sudo ifconfig enp6s0f1 down; sudo ifconfig enp6s0f1 up
sudo route add -net 192.168.10.0/24 gw 192.168.1.1 
```

On the `router2` node:

```
sudo ifconfig enp6s0f0 down; sudo ifconfig enp6s0f0 up
sudo ifconfig enp6s0f1 down; sudo ifconfig enp6s0f1 up
sudo route add -net 192.168.20.0/24 gw 192.168.2.1 
```

On the `server` node:

```
sudo ifconfig enp6s0f0 down; sudo ifconfig enp6s0f0 up
sudo ifconfig enp6s0f1 down; sudo ifconfig enp6s0f1 up
sudo route add -net 192.168.10.0/24 gw 192.168.3.2 
sudo route add -net 192.168.20.0/24 gw 192.168.4.2
```

Now, we will download some software tools needed for this experiment and also configure basic MPTCP related settings.

On both `client` and `server`
  
```
sudo apt update; sudo apt -y install iperf3; sudo apt -y install moreutils
sudo sysctl -w net.mptcp.mptcp_enabled=1
sudo modprobe mptcp_balia
sudo sysctl -w net.ipv4.tcp_congestion_control=balia
sudo sysctl -w net.mptcp.mptcp_checksum=0
```

Install `mptcp-iproute` package on the `client` and `server`

```
sudo git clone https://github.com/multipath-tcp/iproute-mptcp.git
cd iproute-mptcp
sudo make
sudo make install
cd
```

On both `client` and `server`, we will disable multipath on the control interface and ensure that it is enabled on the experiment interfaces

First, on the `client` node:

```
sudo ip link set dev $(ip route get 8.8.8.8 | grep -oP "(?<=dev )[^ ]+") multipath off
sudo ip link set dev $(ip route get 192.168.10.1 | grep -oP "(?<=dev )[^ ]+") multipath on
sudo ip link set dev $(ip route get 192.168.20.1 | grep -oP "(?<=dev )[^ ]+") multipath on
```

On the `server` node:
```
sudo ip link set dev $(ip route get 8.8.8.8 | grep -oP "(?<=dev )[^ ]+") multipath off
sudo ip link set dev $(ip route get 192.168.3.2 | grep -oP "(?<=dev )[^ ]+") multipath on
sudo ip link set dev $(ip route get 192.168.4.2 | grep -oP "(?<=dev )[^ ]+") multipath on
```

 Add delay at the `emulator` nodes

At `emulator 1`, 

```
sudo tc qdisc replace dev $(ip route get 192.168.3.1 | grep -oP "(?<=dev )[^ ]+") root netem delay 30ms limit 60000
```

At `emulator 2`,
```
sudo tc qdisc replace dev $(ip route get 192.168.4.1 | grep -oP "(?<=dev )[^ ]+") root netem delay 30ms limit 60000
```

Now we will set bottleneck capacity and buffer size at both routers. 

At `router 1`,
```
sudo tc qdisc del dev $(ip route get 192.168.3.1 | grep -oP "(?<=dev )[^ ]+") root
sudo tc qdisc replace dev $(ip route get 192.168.3.1 | grep -oP "(?<=dev )[^ ]+") root handle 1: htb default 3
sudo tc class add dev $(ip route get 192.168.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1: classid 1:3 htb rate 100mbit
sudo tc qdisc add dev $(ip route get 192.168.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 375000 
```

At `router 2`,

```
sudo tc qdisc del dev sudo tc qdisc add dev $(ip route get 192.168.4.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 375000  root
sudo tc qdisc replace dev sudo tc qdisc add dev $(ip route get 192.168.4.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 375000  root handle 1: htb default 3 
sudo tc class add dev sudo tc qdisc add dev $(ip route get 192.168.4.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 375000  parent 1: classid 1:3 htb rate 100mbit
sudo tc qdisc add dev sudo tc qdisc add dev $(ip route get 192.168.4.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 375000  parent 1:3 handle 3: bfifo limit 375000
```

Configure routing rules at the `client`

```
sudo ip rule add from 192.168.10.2 table 1
sudo ip rule add from 192.168.20.2 table 2

sudo ip route add 192.168.10.0/24 dev $(ip route get 192.168.10.1 | grep -oP "(?<=dev )[^ ]+") scope link table 1
sudo ip route add 192.168.20.0/24 dev $(ip route get 192.168.20.1 | grep -oP "(?<=dev )[^ ]+") scope link table 2

sudo ip route add 192.168.3.0 via 192.168.10.1 dev $(ip route get 192.168.10.1 | grep -oP "(?<=dev )[^ ]+") table 1
sudo ip route add 192.168.4.0 via 192.168.20.1 dev $(ip route get 192.168.20.1 | grep -oP "(?<=dev )[^ ]+") table 2
```

In the end, your routing table should look like the one shown in the figure below ( you may need to add or delete certain routing rules in addition to the above set of commands)

![](/blog/content/images/2021/12/routing_config.png)

Clone the MPCC repo at the `client` node

```
sudo git clone https://github.com/mpccopensource/mpcc_release.git
cd mpcc_release
sudo make
sudo insmod mptcp_pacing_sched.ko
sudo insmod tcp_mpcc_loss.ko
sudo insmod tcp_mpcc.ko

cd


sudo cp mpcc_release/mptcp_pacing_sched.ko /lib/modules/4.19.126.mptcp/kernel/net/mptcp
sudo cp mpcc_release/tcp_mpcc.ko /lib/modules/4.19.126.mptcp/kernel/net/mptcp
sudo cp mpcc_release/tcp_mpcc_loss.ko /lib/modules/4.19.126.mptcp/kernel/net/mptcp
```

At both `client` and `server`, load all mptcp congestion control and scheduling modules

```
sudo modprobe mptcp_coupled
sudo modprobe mptcp_olia
sudo modprobe mptcp_balia
sudo modprobe mptcp_wvegas
```

Generate a public-private key pair at the `client`

```
sudo ssh-keygen -t rsa
```

By default, the key should be saved at the location *`/users/”your_username”/.ssh/id_rsa`*

On both the `router` nodes, copy-paste the public key of the client node generated in the previous step into the `authorized_keys` file of the router:

```
cd /users/as12730/.ssh

vim authorized_keys
```

Copy the following bash script into a file named `client-iperf.sh` at the client node 

```
t=200
  
runtime="200 seconds"

for j in {1..5}
do
        for b in {375000,400000,550000,700000,850000,1000000}
        do

                for i in {reno,pcc,balia,bbr,lia,olia,wvegas,pcc_loss}
                do
                        echo "$j" | tr '\n' ',' >>  results-trial.txt
                        echo "$i" | tr '\n' ',' >>  results-trial.txt
                        echo "$b" | tr '\n' ',' >>  results-trial.txt

                        endtime=$(date -ud "$runtime" +%s)

                        ssh router1 -f sudo tc qdisc replace dev enp6s0f0 parent 1:3 handle 3: bfifo limit $b

                        ssh router2 -f sudo tc qdisc replace dev enp6s0f1 parent 1:3 handle 3: bfifo limit $b

                        sudo sysctl -w net.ipv4.tcp_congestion_control=$i

                        if [[ "$i" == "bbr" ]]
                        then
                                sudo sysctl -w net.mptcp.mptcp_scheduler=default_pacing
                        elif [[ "$i" == "pcc" ]]
                        then
                                sudo sysctl -w net.mptcp.mptcp_scheduler=default_pacing
                        elif [[ "$i" == "pcc_loss" ]]
                        then
                                sudo sysctl -w net.mptcp.mptcp_scheduler=default_pacing
                        else
                                sudo sysctl -w net.mptcp.mptcp_scheduler=default
                        fi



                        iperf3 -f m -c 192.168.4.1 -C "$i" -P 2 -i 0.1 -t $t | ts '%.s' | tee $i-$j-$b-trial-iperf.txt > /dev/null & rm -f $i-$j-$b-trial-ss.txt;while [[ $(date -u +%s) -le $endtime ]]; do ss --no-header -eipn dst 192.168.3.1 or dst 192.168.4.1 | ts '%.s' | tee -a $i-$j-$b-trial-ss.txt > /dev/null;sleep 0.1; done
                        sleep 5;
                        cat $i-$j-$b-trial-iperf.txt | grep "sender" | awk '{print $8}' | tr '\n' ',' >> results-trial.txt
                        sed '/fd=3/d' $i-$j-$b-trial-ss.txt > $i-$j-$b-trial-ss1.txt
                        cat $i-$j-$b-trial-ss1.txt | sed -e ':a; /<->$/ { N; s/<->\n//; ba; }'  | grep "iperf3" > $i-$j-$b-trial-ss-processed.txt
                        cat $i-$j-$b-trial-ss-processed.txt | grep -oP '\brtt:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' '  | cut -d '/' -f 1   > srtt-$i-$j-$b-trial.txt
                        awk '{ total += $1; count++ } END { print total/count }' srtt-$i-$j-$b-trial.txt >> results-trial.txt;
                        rm $i-$j-$b-trial-ss.txt;rm $i-$j-$b-trial-iperf.txt;rm $i-$j-$b-trial-ss-processed.txt;rm srtt-$i-$j-$b-trial.txt;rm $i-$j-$b-trial-ss1.txt;

                done
        done
done
```

Download and install `screen` on both the client and the server 

Open one screen each on client and server

On the server screen run
```
iperf3 -s -i 0.1
```
On the client screen run
```
bash client-iperf.sh 
```
Leave them  running and exit both screens using “Ctrl-A + D” 

The experiment should finish in 13.5 hours. You can then log back in and get the results in a file named `results-trial.txt`. The columns of this log contain the following information : `experiment_number, congestion control, buffer size, average throughput in Mbps and average latency in ms`. The experiment runs 5 trials using each combination of buffer size and multipath congestion control protocol. For TCP Reno and TCP BBR which are not MPTCP congestion controls, the experiment runs two independent single-path TCP flows instead of a single multipath flow.


Below is an additional script that can be used at any of the `routers` if you want to monitor their queue while running the experiment. Make sure you change the interface name according to the configuration of your experiment.

```
t=200
sleep 0.1

for j in {1..5}
do
        for b in {375000,400000,550000,700000,850000,1000000}
        do

                for i in {reno,pcc,balia,bbr,lia,olia,wvegas,pcc_loss}
                do
                        echo "$j-$i-$b" >>  results-router1.txt
                        bash queuemonitor.sh enp6s0f0 $t 0.1 | tee $j-$i-$b-router1.txt > /dev/null

                        sleep 5;
                done
        done
done
```

##Result of the experiment

The result obtained from our Cloudlab experiment is shown below. 

![](/blog/content/images/2021/12/result_latency.png)

Comparing it with the result shown in figure 9 of the MPCC paper, we can observe that the two plots are very similar. This verifies that our testbed experiment is in agreement with the results shown in the paper. For the MPTCP protocols (LIA, OLIA, BALIA) as well as other loss-based protocols like TCP Reno and MPCC-loss, the latency increases as the bottleneck buffer size is increased. This happens due to the ``bufferbloat'' problem of loss-based TCP.

For the latency version of MPCC, the observed latency stays low even for large buffer sizes. The delay for TCP BBR is slightly higher than MPCC's delay but is lower when compared to the loss-based schemes. When compared with the paper's result, BBR's delay for high buffer sizes is slightly lower. We still have to look into the reason for this discrepancy.

The latency of the wVegas protocol stays very close to the base RTT of the network. This is because wVegas which is a delay-based scheme severely under-utilizes the bottleneck links and never fills up the queues. Its almost zero queueing delay comes at the cost of very low throughput.






