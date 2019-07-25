In this work we will study the performance anomaly of the 802.11 WLAN network when users with low link rates are present along with users with high link rates. The original work is presented in 

> Heusse, Martin, et al. "Performance anomaly of 802.11b." INFOCOM 2003. Twenty-Second Annual Joint Conference of the IEEE Computer and Communications Societies. Vol. 2. IEEE, 2003.

It should take 3-4 hours to run this experiment, but you will need to have reserved that time in advance. This experiment uses wireless on the grid at [ORBIT](http://geni.orbit-lab.org)), and you can only use wireless resources on GENI during a reservation. Users without prior experience on ORBIT may want to reserve a 6-hour slot to have sufficient time to become familiar with the testbed and run the experiment.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have reserved time on the grid sandbox at [ORBIT](http://geni.orbit-lab.org) and you must run this experiment during your reserved time.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background ##

In this experiment we will see how channel contention and the presence of hosts with channel impairments (low link rate) adversely affects the performance of the entire 802.11b network. In fact there are two take away points from this experiment:

* The throughput of each user decreases when the number of hosts in the network increases due to contention for the shared channel.

* When one host has a poor link rate it will occupy the channel longer, in turn degrading the throughput of other users.

This experiment will give us an insight into the performance of wireless networks with shared channels.

## Results ##

The following image is from the original paper. It shows the useful data throughput obtained by any node versus the number of nodes in the network:

![1208921-fig-3-large.gif](https://bitbucket.org/repo/LBKe9r/images/4212215149-1208921-fig-3-large.gif) 

The next figure is our reproduction of the work presented by Heusse, Martin, et al. The throughput experienced by one user is plotted against the number of users in the system. We vary the number of nodes from 1 to 15.


![fig1_1.jpg](https://bitbucket.org/repo/LBKe9r/images/2982952279-fig1_1.jpg) 

In general, we see good agreement between our results and the original paper. In our experiment, due to practical concerns we limited the total number of nodes to 15 and also used a relatively short experiment duration. This may be the reason for the slight variations from the original result.

## Reproducing the experiment ##

In this section I will delineate the steps to reproduce the above results. The materials needed to reproduce this experiment can be found in the [EL7353-802.11b](https://bitbucket.org/Sourjya9016/el7353-802.11b) repository on Bitbucket.

For the sake of simplicity we will assume that there are 5 hosts and one access point (AP) in the network. The experiment can be very easily extrapolated to 15 or more hosts. We will use the grid on the ORBIT test bed.

We require to create a set up with one AP which supports the 802.11b WLAN and connect N hosts to it. In this experiment we vary N from 1 to 15. For the sake of brevity I describe a scaled down version of the experiment. At the AP we run the Iperf UDP server and at from  N hosts (Iperf clients) we send UDP packets to the server. 

At each of the client we will measure the **throughput** in Mb/s.

The measurements are taken over a period of 120 sec. You are encouraged to use longer durations if time permits.

### Initial set up ###

Find the nodes on the grid that have the **ath5k** wireless card. 

We declare a environment variable as

    GROUP=node1-1.grid.orbit-lab.org,node1-2.grid.orbit-lab.org,node2-1.grid.orbit-lab.org, node2-2.grid.orbit-lab.org,node4-1.grid.orbit-lab.org,node5-1.grid.orbit-lab.org

assuming that these nodes support the **ath5k** wireless card. The configurations may change and so please check which nodes support **ath5k** before starting the experiment. Load the image of the group

    omf load -i baseline.ndz -t "$GROUP"

and turn on all the nodes once imaging is done

    omf tell -a on -t "$GROUP"


### Create an AP ###
We chose node1-1 to be the AP. Use ssh to access the node 

    ssh root@node1-1

Once inside the node use the following commands

    apt-get update
    apt-get -y install git dnsmasq hostapd iperf
    modprobe ath5k
    ifconfig wlan0 up
    hw_mode=b
    /etc/init.d/hostapd restart
    iwconfig wlan0 rate 11M
    iwconfig wlan0 channel 6

The last command gives an error but check the current channel is use using

    iwlist wlan0 channel

and you will see that the required change has been made. Check the last line of the list to see the current channel in use.

To run the AP

    git clone https://github.com/oblique/create_ap.git
    cd create_ap
    make install
    essid=$(hostname -s)
    ./create_ap -n wlan0 "$essid" SECRETPASSWORD

Your access point is ready.

### Configure 802.11b hosts ###

To configure the 802.11b hosts ssh to the individual nodes and execute the following commands

    apt-get update
    apt-get -y install isc-dhcp-client iperf
    modprobe ath5k
    ifconfig wlan0 up
    hw_config=b
    /etc/init.d/hostapd restart
    iwconfig wlan0 channel 6

Run

    iwlist wlan0 scan 
 
to see the available access points. Connect to the access point as

    wpa_passphrase node1-1 SECRETPASSWORD > wpa.conf 

Remember to change the ESSID if it is not node1-1 in your experiment. Finally obtain an IP address as

    dhclient wlan0

Theses steps might be daunting to execute on a large number of machine. Thus in this repository you should find the **initial.sh** bash script. Run this script from the grid terminal (without ssh to any particular node) as

    ./initial.sh node1-2 node2-1 node2-2

to set up node1-2 node2-1 node2-2 as 802.11b hosts. Add more nodes to the argument list if needed. You should ssh to all of the nodes and ping other nodes to verify that the connection has beed properly established.

### Run the test ###
Running the test is simple.

* Step 1: Run Iperf server on the AP in the UDP mode.

        ssh root@node1-1
        iperf -s -u

* Step 2: On each of the hosts with fast link rate set the link rate as 11 Mbps and run Iperf in the client mode as

        ssh root@node2-2
        iwconfig wlan0 rate 11M
        iperf -c <server_ip> -u -b 11M -t 120 > iperfnode2-2.out

* Step 3: On the slow host, set the link rate to a lower permissible value (1,2 or 5.5 Mbps) and run perf in the client mode

        ssh root@node1-2
        iwconfig wlan0 rate 2M
        iperf -c <server_ip> -u -b 2M -t 120 > iperfnode1-2.out

For both Steps 2 and 3 do not forget to redirect the Iperf output to a file.

It is a daunting task to do this on a large number of nodes. Moreover that will also affect the time synchronization. Thus you can use the script **runTest1.sh** from the grid terminal (not from any specific node) as

    ./runTest1.sh 2M node1-1 exp2 node1-2 node2-1 node2-2

where the first argument is the rate of the slow node, argument 2 is the server (AP) node, argument 2 is the folder inside which the iperf data and error files are stored and arguments 4 to N are names of the host nodes. Put different folder name for different runs, other wise the files will be concatenated.

Fixing the rate of the slow host increase the number of fast hosts in the system from 0 to N-1. Thus for the case when the slow host has 2 Mb/s link the experiment will proceed as follows:

a) Run with a single host in the system

    ./runTest1.sh 2M node1-1 exp2 node1-2

b) Run with one slow and one fast host

    ./runTest1.sh 2M node1-1 exp2 node1-2 node2-1 

c) Run with one slow and two fast hosts
    
    ./runTest1.sh 2M node1-1 exp2 node1-2 node2-1 node2-2

and so on. Do this for the four possible slow link rates.

The script **runTest2M.sh** performs the above experiment but with all the nodes having a degraded link rate of 2Mbps. Run the script as

    ./runTest2M.sh node1-1 exp2M node1-2 node2-1 node2-2


### Collect Data ###

Timing is crucial in shared resources. As soon as you are sone with the experiments collect your data from the nodes using **scp**. All your data will be lost once the nodes are reset, i.e you lose access to the grid. The following commands are needed

    mkdir exp2
    scp root@node1-2:~/exp2/* ~/exp2/

This will bring all the files from the **exp2** folder in node1-2 to the **exp2/** folder on your Orbit terminal. The 8th line of the Iperf data file contains the summary of the experiment. Read that on a file as

    sed '8q;d' iperfnode1-2.out >> exp2-1.txt

Do this for all the files gathered from all the nodes for a particular experiment. Obviously this file is not suitable to be used by data processing tools like MATLAB, Excel etc. Extract the throughput from this file as

    cat exp2-1.txt | awk '{IF($8=="Kbits/sec") print $7/1000; else print $7}'

which writes all the throughput data in Mbps. A MATLAB script with the data files obtained can be found within the **data.zip** file.

The number of data files generated using Iperf is large in this experiment and naming may vary from user to user. Using the above two commands and a loop in the shell script one can very easily extract the data.

### Additional Notes ###

* It is best to run a small scale implementation in one of the sand boxes first. Sandbox 4 is ideal for this case.
* Be sure you can ssh to all the hosts before executing any of the scripts.
* In this work the experiment run time is 120 sec. This is a limiting factor for large number of hosts. But running longer experiments is not practical on shared resources.