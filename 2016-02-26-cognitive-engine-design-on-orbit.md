This experiment describes how to run our submission for phase 1 of the Spectrum-SharC student cognitive radio design contest. 

It should take about 60 minutes to run this experiment, but you will need to have [reserved that time](http://geni.orbit-lab.org) in advance. This experiment uses wireless resources (specifically, on the [ORBIT](http://geni.orbit-lab.org) testbed - either sb3, sb2, or sb7), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have [reserved time on the ORBIT testbed](http://geni.orbit-lab.org) on either sb2, sb3, or sb7, and you must run this experiment during your reserved time.

## Background

The Spectrum - [ShaRC (spectrum sharing radio contest)](http://radiocontest.wireless.vt.edu/) uses the cognitive radio test system [(CRTS)](https://www.cornet.wireless.vt.edu/new_CRTS_Flyer_20151222.pdf), a framework developed for cognitive radio experimentation and performance measurement, and Virginia Tech's internet-acessible [CORNET cognitive radio testbed](http://cornet.wireless.vt.edu/) to measure performance of student-designed cognitive or adaptive dynamic spectrum access radios in challenging operational environments.

The focus of first phase of the competition was on the physical layer (PHY). Each team had to design one or two Cognitive Engines (CEs) based on the Extensible Cognitive Radio (ECR). The ECR is a CR framework developed by Virginia Tech to enable flexible development of CEs. Score was based on the sum data transmitted between the two CRs in a set period of time in a set of interference scenarios that vary in terms of perceived difficulty/complexity. 

In general, a scenario comprised of two CR nodes and one or more nearby interferers which was transmitting dynamically in time and frequency.

We carried out several initial tests on ORBIT test bed and later validated the results on CORNET testbed.

### Our approach for Phase 1 (PHY)

Our first step was to try and understand each Cognitive Radio Engine (CE) and scenario provided to us by the Virgina Tech organizers.

We mainly experimented with 3 engines for Phase 1, namely `Two_Node_FDD_Configuration`,`FEC_Adaptation` and `Mod_Adaptation`:

 * **Two\_Node\_FDD\_Configuration**: This is the most basic two node CR network. No actual cognitive/adaptive behavior is defined by the cognitive engine.
 * **FEC\_Adaptation**: This engine is designed to adapt the transmit Forward Error correction (FEC) scheme based on feedback from the receiver.
 * **Mod\_Adaptation**: This engine is designed to adapt the transmit modulation scheme based on feedback from the receiver.

We tested the performance of each of the above CEs, in a no-interference scenario. We gathered results and considered this as a baseline design. 

We then tried to develop a new cognitive engine. However, none of our designs had better performance than the provided CEs, so we abandoned these designs and focused on modifying the parameters of the CEs (rather than their logic) to improve the performance of our radio.

For each of the three cognitive engines, we  identified configuration parameters which appeared to give us the best results in no interference scenario. We fine tuned parameters like gain (digital gain, analog gain, TX gain and RX gain) and  channel separation.

## Results

![Test results.jpg](http://s6.postimg.org/gayeq3ar5/orbit_results.png)


Exp-1, Exp-2, and Exp-3 in the image above show the performance of the three cognitive engines under consideration, with original parameters. 

In the image above, Exp-4, Exp-5, and Exp-6 use the same cognitive engines, but with improved parameters - namely, modified gain and increased separation between the two channels used for bidirectional communication. 

Our final configuration file was derived from the above experimental results. Since we got the best results with the FEC adaptation engine with modified parameters, we made that our final design. 

In the [phase 1 rankings](http://radiocontest.wireless.vt.edu/pfs/SpectrumSharc%20Round%201%20Results%20Press%20Release%2020160212.pdf) for this competition, our team, the Spectral Rangers, were in a 3-way tie for first place.

## Run my experiment

Make a reservation in ORBIT using [ORBIT webpage](http://geni.orbit-lab.org/). We wrote these instructions for testbed sb3, but sb7 and sb2 also have USRP N210 devices.

During the reserved slot, SSH into your testbed, e.g. `sb3.orbit-lab.org`, using your GENI wireless username and keys.

Once you are on the testbed console, load a  baseline SDR disk image onto two nodes with the command:

```

omf load -i baseline-sdr.ndz -t node1-1.sb3.orbit-lab.org,node1-2.sb3.orbit-lab.org

```

The above step might take some time. After that, turn on both the nodes using the command:

```

omf tell -a on -t node1-1.sb3.orbit-lab.org,node1-2.sb3.orbit-lab.org

```


Wait for some time for nodes to turn on. Then, SSH in to both the nodes from separate consoles using commands:
 

```
ssh root@node1-1


```


and


```
ssh root@node1-2

```
The baseline SDR disk image already has the drivers and other software for interfacing with the SDR. 


However, you need to do some additional setup tasks 
to install some more libraries for CRTS. On each node, run:

```
apt-get update
apt-get -y install libconfig-dev
apt-get -y install git
apt-get -y install automake

# Install libfec
git clone https://github.com/Opendigitalradio/ka9q-fec.git
cd ka9q-fec
./bootstrap
./configure
make
make install
ldconfig
cd

# Install liquid-dsp
git clone https://github.com/jgaeddert/liquid-dsp.git
cd liquid-dsp
./bootstrap.sh
./configure
make
make install
ldconfig
cd

```

Then get our clone of the CRTS code:

```
cd ~/ # go back to home directory
git clone https://bitbucket.org/spectralrangers/phase-1-code.git
cd ~/phase-1-code
make
```


Find the IP address of each node relative to the other. This is required for one of the later steps, In order to find the IP address:

On node1-1, run
```
ping node1-2
```

and on node1-2 run
```
ping node1-1
```

and make a note of the IP address that it pings. On sb3, node1-1 has IP 10.13.1.1 and node1-2 has IP 10.13.1.2. On sb2, the second octet will be 12, and on sb7, the second octet will be 17.

If you are running this experiment on sb2 or sb7, you need to update the IP addresses in the scenario files, which refer to sb3. 

```
cd ~/phase-1-code/scenarios

# on sb7:
grep -rl '10.13.1' ./ | xargs sed -i 's/10.13.1/10.17.1/g'

# on sb2:
grep -rl '10.13.1' ./ | xargs sed -i 's/10.13.1/10.12.1/g'

cd ..
```




Now, you can run the code. We need to use a controller and two nodes for TX and RX (each node both transmits and receives, on adjacent channels). We will use node1-1 as the controller and the first TX/RX node, and node1-2 as the second TX/RX node. 

On node1-1 which is now chosen as controller

    cd ~/phase-1-code  # You have to start the controller from this directory!
    ./CRTS_controller -m -f Exp5_FEC_Adap_modified.cfg

Immediately, you should see a whole lot of output that tells you more about the configuration that's being run. Then the controller process will wait until it's contacted by the CR nodes.

On the other node(node1-2), run

    cd ~/phase-1-code
    ./CRTS_CR -a 10.13.1.1 | tee node2_output.txt # on sb3
    # on sb2, use ./CRTS_CR -a 10.12.1.1 | tee node2_output.txt
    # on sb7, use ./CRTS_CR -a 10.17.1.1 | tee node2_output.txt

When you start this, you should see "node parameters" output on the CR node as below:

````
root@node1-2:~/phase-1-code# ./CRTS_CR -a 10.13.1.1
linux; GNU C++ version 4.8.2; Boost_105400; UHD_003.008.002-86-g566dbc2b

Received settings for scenario
Received 6024 bytes

--------------------------------------------------------------
-                    node parameters                         -
--------------------------------------------------------------
General Settings:
    Node type:                         CR
    CR type:                           ecr
    CORNET IP:                         10.13.1.2

````
 
etc.

Now,on the controller node, you'll see in the output

    Node 1 has connected. Sending its parameters...

in the output.

Then, open another SSH session to the node running the controller (node1-1) and run the second CR process on that node (since here we are using 
a scenario with two nodes.) Run the same thing:

    cd ~/phase-1-code 
    ./CRTS_CR -a 10.13.1.1 | tee node1_output.txt # on sb3
    # on sb2, use ./CRTS_CR -a 10.12.1.1 | tee node1_output.txt 
    # on sb7, use ./CRTS_CR -a 10.17.1.1 | tee node1_output.txt 


On this terminal also, you'll see the node parameters as below:

````
root@node1-1:~/phase-1-code# ./CRTS_CR -a 10.13.1.1
linux; GNU C++ version 4.8.2; Boost_105400; UHD_003.008.002-86-g566dbc2b

Received settings for scenario
Received 6024 bytes

--------------------------------------------------------------
-                    node parameters                         -
--------------------------------------------------------------
General Settings:
    Node type:                         CR
    CR type:                           ecr
    CORNET IP:                         10.13.1.1

Virtual Network Interface Settings:
    CRTS IP:                           10.0.0.2
    Target IP:                         10.0.0.3
    Traffic pattern:                   stream
    Average throughput:                2000000.00
````

On the terminal where the controller process is running, you'll also see 

    Node 2 has connected. Sending its parameters...
    Listening for scenario termination message from nodes



After everything runs successfully, on the controller you'll see

    Node 2 has sent a termination message...
    Node 1 has sent a termination message...
    Ending scenario 1 because all nodes have sent a termination message

and on the CR nodes, if the scenario involves log files, you may see something like


```
Updating USRP parameters
linux; GNU C++ version 4.8.2; Boost_105400; UHD_003.008.002-86-g566dbc2b

Log file name: logs/bin/Sharc4_Node2_CR_NET_RX.log
Output file name: logs/octave/Sharc4_Node2_CR_NET_RX.m
linux; GNU C++ version 4.8.2; Boost_105400; UHD_003.008.002-86-g566dbc2b

Log file name: logs/bin/Sharc4_Node2_CR_NET_TX.log
Output file name: logs/octave/Sharc4_Node2_CR_NET_TX.m
linux; GNU C++ version 4.8.2; Boost_105400; UHD_003.008.002-86-g566dbc2b

Log file name: logs/bin/Sharc4_Node2_CR_PHY_RX.log
Output file name: logs/octave/Sharc4_Node2_CR_PHY_RX.m
linux; GNU C++ version 4.8.2; Boost_105400; UHD_003.008.002-86-g566dbc2b

Log file name: logs/bin/Sharc4_Node2_CR_PHY_TX.log
Output file name: logs/octave/Sharc4_Node2_CR_PHY_TX.m
CRTS: Reached termination. Sending termination message to controller


```

and on all nodes, you should get your terminal prompt back. 

Finally, we need to calculate the total number packets received correctly inorder to find the throughput. Inorder to do that, you can do

On node1-1, run

```
cat node1_output.txt | grep "Payload Valid:    1" | wc -l
```

On node1-2,run

```
cat node2_output.txt | grep "Payload Valid:    1" | wc -l

```

The above commands will give you the total number of packets received on each node.

To run the other engines, repeat the steps above, with a different "-f" parameter passed to the controler: 

```
# For Two_Node_FDD_Network with original parameters: Exp-1
./CRTS_controller -m -f Exp1_Two_Node.cfg 

# For FEC_Adaptation with original parameters: Exp-2
./CRTS_controller -m -f Exp2_FEC_Adap.cfg 

# For Mod_Adaptation with original parameters: Exp-3
./CRTS_controller -m -f Exp3_Mod_Adap.cfg 

# For Two_Node_FDD_Network with our parameters: Exp-4
 ./CRTS_controller -m -f Exp4_Two_Node_modified.cfg 

# For FEC_Adaptation with our parameters: Exp-5
./CRTS_controller -m -f Exp5_FEC_Adap_modified.cfg 

# For Mod_Adaptation with our parameters: Exp-6
./CRTS_controller -m -f Exp6_Mod_Adap_modified.cfg 

```








