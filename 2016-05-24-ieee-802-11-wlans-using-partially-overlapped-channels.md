
Wireless communication inherently involves the transmission of information over a wireless medium. The IEEE 802.11 standard defines the spectrum, protocols and policies used in WLANs. Although a large number of overlapping channels are present in the spectrum (the 2.4 GHz spectrum to be precise), traditionally only non-overlapping channels are used in order to avoid the adverse effects of adjacent channel interference. In this post we investigate whether it might actually be beneficial to use partially overlapped channels in IEEE 802.11 WLANs. We first present a brief theoretical framework as given in literature. We then proceed to implement actual experiments run on the ORBIT testbed which provides us with a 20x20 grid of programmable radio nodes which can be interconnected into specific topologies. Detailed instructions on how to reproduce the experiment are also provided.

This experiment should take 40 minutes per topology to reproduce. So if you choose to test on 10 different topologies, the total experiment time should be around 6 hours.

#### Background ####
Wireless Communication systems use a certain portion of the spectrum allocated to them to transmit information wirelessly. This spectrum is typically divided into several channels with a given transmitter-receiver pair operating on one particular channel. For IEEE 802.11(a/b/g) WLANs, the spectrum used is the 2.4 GHz spectrum. This is divided into 11 channels, each with a bandwidth of a 44 MHz. However, the channel separation is only 5 MHz. This means that only three of the eleven channels (channels 1,6 and 11) are non-overlapping. Consequently, nodes operating on overlapped channels in close proximity with each other would potentially interfere to a certain degree, a phenomenon known as adjacent channel interference. Nodes operating on the the same channel in close proximity interfere fully with each other, which is known as co-channel interference. Co-channel interference is resolved through the CSMA/CA protocol defined in the IEEE 802.11 standard. However, there is no such conflict resolution mechanism for adjacent channel interference and typically this is avoided by operating nodes only on non-overlapped channels.

However, we are interested in the proposition laid out in [1] which claims that the use of these partially overlapped channels may in fact bestow significant advantages in terms of collision reduction and better throughput performance. We consider a wireless network consisting of several BSS (an AP along with its clients forms a BSS). Each BSS operates on a single channel for communication. To increase the average throughput in the network, multiple BSS across the network may be assigned partially overlapped channels according to our proposition. We now present a brief summary of the theoretical framework for this claim.
	
The communication system can be modeled as two-ray ground propagation model. Under this model, the received power is directly proportional to, besides other factors, the interference factor *I(i,j)*  which represents the interference experienced by the receiver when the transmitter is on channel *i*  and the receiver is on channel *j*.  

The interference factor is an experimentally measured quantity which varies according to the channel separation between the transmitter and the receiver and the wireless standard and protocol used for communication. In [2], the authors measure this interference factor for IEEE 802.11 networks. The interference factor is normalized to a quantity between 0 and 1, with 1 denoting maximum possible interference. 

The interference region for a particular node can thus be thought of as made up of concentric circles, with the largest circle corresponding to the interference region when there is zero channel separation and the smallest circle corresponding to the interference region when there is a channel separation of 4. For channel separation of 5 and above, there is no interference at all even if the transmitting nodes are in close proximity of each other.

Given this model, we now consider the proposition laid out by the authors in [1]. The claim is that using partially overlapping channels within a WLAN could prove to be beneficial in terms of average throughput and network performance. The key problem, however, is to figure out how to intelligently allocate partially overlapped channels in a WLAN. We are therefore concerned with implementing an appropriate channel assignment algorithm which allocates partially overlapped channels in the WLAN and then measuring the average throughput to gauge the network performance.


#### Results ####
We randomly chose eleven different topologies and ran our experiments on those. The results obtained are shown in Figure 1.

![alt text](https://bitbucket.org/AffanJaved/el7353-project/downloads/figure.jpg)

Figure 1 shows that  for topologies 1-6 and 11, partially overlapped channels outperformed non-overlapped channels. However, for topologies 7-10, the performance of partially overlapped channels was poorer than that of non-overlapped channels. Moreover, note that even in the cases where UDP throughput for partially overlapped channels is higher, the improvement factor is not as high as that shown in the original results. Thus, our results diverge from the original results on two counts. First, the original results show that partially overlapped channels *consistently* outperform non-overlapped channels whereas our results show that the performance of non-overlapped channels is better for some topologies. Secondly, the original results show that UDP throughput when partially overlapped channels are used is nearly 3 times higher than when non-overlapped channels are used. However, our results show that even when partially overlapped channel outperform non-overlapped channel, they do so by a small margin.

The error bars in Figure 1 show the standard deviation in the average UDP throughput amongst all the clients. There's a large standard deviation in almost all the topologies which shows that the throughput distribution amongst the clients is quite unequal: some clients get a large proportion of the bandwidth whilst other only get a small proportion.

#### How to Run my Experiment ####
We will be using the following shorthand notation when naming our channel allocation scripts:
PO: Partially Overlapped 
NO: Non-Overlapped

###### Log in to ORBIT and Load Image ######
You need to reserve time on the ORBIT testbed to run this experiment. If you are unfamiliar with the ORBIT testbed, you can find basic instructions on how to get started [here.](http://www.orbit-lab.org/wiki/Documentation/CGettingStarted#Howtogetstarted)
During your reserved time, login to the testbed using:

                ssh geni-maj407@grid.orbit-lab.org
 where you will replace "geni-maj407" with your own ORBIT username.
 
 On the main grid console, clone the repository containing all our scripts by executing the following command:
 
       git clone git@bitbucket.org:AffanJaved/el7353-project.git

Then change your current directory to the folder you just cloned by executing 

         cd el7353-project
 
You will have all the scripts you need to run the experiment in this directory.

The next step is to load omf images on the nodes in the grid you wish to run your experiment on.Execute `bash loadImage.sh` to run the script.

There are approximately 100 nodes which have ath9k capability. This script will load images on all of them. You can choose to do it for a smaller subset of nodes if you want.

**Note**: For the topology, you have to choose 10 nodes which will serve as Access Points and 20 nodes which will serve as Clients. Two clients and one AP will form a BSS. You can choose any node randomly as long as it has ath9k capability.
By default, the topology included in the loadImage file consists of **ALL** the nodes in the grid which have the ath9k wireless card.

###### Updating and Configuring Clients and APs ######
Before you proceed further, you will need to choose 10 APs and 20 Clients randomly from the grid. You can do this by choosing 10 APs randomly, then selecting two nearby nodes as the clients for each AP. The node IDs need to be declared in arrays. An example is as     follows:

            declare -a APs=("3-19" "13-20" "13-7" "1-6" "1-13" "8-13" "20-10" "4-19" "10-1" "2-2")
            declare -a Clients=("3-20" "4-3" "14-7" "14-10" "13-8" "13-13" "1-19" "1-20" "1-15" "1-16" "8-18" "8-20" "20-12" "20-19" "5-5" "5-16" "10-4" "10-7" "2-19" "3-1")

The AP array needs to be copied into the following scripts:
1. configureMain
2. connectClienttoAP.sh
3. iperfServers.sh
4. copyFiles.sh

The Client array needs to be copied into the following scripts:
1. configureMain
2. connectClienttoAP.sh
3. iperfClients.sh
4. retrieve.sh

Make sure you modify these scripts with the correct Client and AP arrays because they will need this information in order to run properly.
After you are done loading the omf images, you need to configure the clients and APs that you have chosen.
Then, run:

            bash configureMain

This script will execute code at all the APs and Clients you have specified that will update their basic software and install the packages required for them to work as clients and access points correctly.

###### Running Distributed Channel Assignment Algorithm ######
Now that the basic software has been set up at all the Clients and APs, it is time to create APs and allocate channels to them. To create an AP, we use a GitHub repository called [create_ap](https://github.com/oblique/create_ap). Before creating an AP, we need to decide which channel to allocate to it.

###### Create the Access Points ######

Execute `bash copyFiles.sh` on the main grid console. This will copy the channel allocation scripts to all the APs that you have chosen. Then open separate terminal windows and log-in to each AP. There are 10 APs, so you will have to open 10 terminal windows to log in to each of them. If, for example, you have chosen node3-2 as an AP you can login to it by running:

            ssh root@node3-2
on the main grid console.
Then in each terminal window run the main channel allocation script, specifiying the name(ESSID) of the Access Point:

            bash PO_DIST_main AP1

The Access Point will scan the environment for all conflicting APs. Given this interference information it will decide on the best channel for itself. It will then run code to create an Access Point with ESSID "AP1" on this particular channel.
The output should also show you the channel assigned to this particular AP.
**Note**: Do this for 10 nodes randomly selected in the grid. The only requirement is that the nodes should be equipped with the **ath9k** wireless card. The APs should be named **AP1..AP10**; no other ESSID should be used for the APs, otherwise rest of the code won't work properly.
    
After you are done with this, you will have created 10 APs on different channels which were assigned to them using the distributed algorithm. The create_ap function is a **blocking call**, so you will need to keep all the Terminal windows open till the end of the experiment.

###### Connect Clients to their respective APs and measure bandwidth ######
The next step is to associate two clients with each AP, so that one AP and two clients form a BSS. To do this, execute `bash connectClienttoAP.sh` on the main grid console. Note that the vectors containing client and AP IDs are used in the connect.sh script so open that up and modify the vectors according to your topology.
This script will ssh into all the clients and connect them with their respective APs one by one.
Now our network has been completely created and we just need to measure the bandwidth at each client which is our metric for network performance.

###### Measure Bandwidth at all the Clients ######
We use iperf to measure the bandwidth at each client. Once again, this process has been automated so you only need to run two scripts on the main grid console. Run:

            bash iperfServers.sh
            bash iperfClients.sh
The iperfServers script will start iperf servers on all the APs. The iperfClients script will then make each client iperf to its respective AP. We use UDP traffic and each client tries to transmit at the maximum possible rate. The iperf result then shows the actual bandwidth allocation. Bandwidth is measured over a period of 2 minutes and this is repeated 5 times. So once you run this script, it will take 10 minutes for the complete experiment for this particular topology to finish. This output is stored in log files at each client node. If you want, you can ssh into a client and track the progress of the experiment by opening the log. For example, if node3-2 is a client you can ssh into it and open the log by running `nano iperf3-2.out`.

Next we run a simple script to retrieve all these log files from the client nodes and transfer them to the main grid console. The script will also extract the bandwidth from the log files and store them in a separate file called **final_log.txt**. To retrieve the files, run

            bash retrieve.sh

You can then see the results by opening the final_log.txt file. The file should have 5 columns and 20 rows. So each row corresponds to the bandwidth of one client, and each column corresponds to one 2-min run of the experiment. 

Copy this into MATLAB or Excel. Average across all rows and then across all columns to represent this 2D Matrix by a single number which is the average UDP throughput per client for this topology.

Make sure to delete the final_log.txt file before starting a new experiment on another topology. If you don't delete the file, the new results will be concatenated at the end of the same file when you run `bash retrieve.sh` again.

###### Repeat same experiment for non-overlapping channels ######
The experiment we just did was for the partially overlapped channels case, because we assigned partially overlapped channels to the APs. Now we wish to run the same experiment, but only assign non-overlapping channels to the APs.

To do this, run the scripts **NO\_DIST.rb** and **NO\_DIST\_main** while creating the APs. The remaining steps - which pertain to connecting clients to their APs and measuring the bandwidth - remain exactly the same as before. 
Once again, record your experiment results in a separate log file.

Now you will have the average UDP throughput per client for both partially overlapped and non overlapped cases. However, this is only for one topology. You need to repeat this entire experiment for 7-10 different,randomly chosen topologies. After getting all the results, you can then plot topology number vs UDP throughput to get a graph like the one shown in the results section.

When switching to a different topology, it is important that you erase all memory of the previous configuration from the nodes. To do this you can run the following piece of code to reboot the nodes of the previous topology:

    if [[ -n "$GROUP" ]]  
    then  
      omf tell -a reboot -t "$GROUP"
    else  
      echo "Please set the GROUP variable"
    fi  

Here, the variable GROUP would be defined to include the node IDs of the previous topology.

#### References ####
[1] 	Arunesh Mishra, Vivek Shrivastava, Suman Banerjee, and William 	Arbaugh. 		2006. 	Partially overlapped channels not considered harmful. In Proceedings of 		the joint international conference on Measurement and modeling of computer 		systems (SIGMETRICS '06/Performance '06). ACM, New York, NY, USA, 63-74

[2] 	Burton M. Channel overlap calculations for 802.11b networks.Technical Report, 		White Paper, Cirond TechnologiesInc., 2002.