In this experiment, you will explore the 5G RAN by bringing up an end-to-end 5G network on a single server. You will see:

* how a 5G RAN can be split with the CU (centralized unit) handling SDAP, PDCP, and RRC layers, and the DU (distributed unit) handling RLC, MAC, PHY layers
* how a CU and DU communicate over the F1 interface

Note that to run this experiment, you will need to have reserved access to compute resources in advance. Refer to the "reserve a server" instructions in order to complete this step several days before you plan to run the experiment.

We will run this experiment on the [POWDER testbed](https://powderwireless.net/). You should already have set up access to POWDER before you begin this experiment. Then, log in to [https://powderwireless.net/](https://powderwireless.net/) with your CloudLab or POWDER username and password.

## Background

In this experiment, we will explore the 5G Radio Access Network (RAN). The RAN is responsible for managing access to radio resources, and transferring datagrams between the UEs and the core network using those radio resources.

![](/blog/content/images/2024/04/ran-overall.svg)

<small>Image source: [Jim Kurose](https://gaia.cs.umass.edu/cs590ae/schedule.html)</small>

Some of the functionality provided by the RAN includes:

* RRC (Radio Resource Control): configures the user and control planes, including connection establishment and release, and radio bearer establishment.
* SDAP (Service Data Adaptation Protocol): (new in 5G, not illustrated above) responsible for QoS flow handling.
* PDCP (Packet Data Convergence Protocol): provides data integrity and security services for user and control plane packets.
* RLC (Radio Link Control): segmentation and reassembly of link layer frames, and reliable transfer of frames using ARQ.
* MAC (Media Access Control): scheduling frames on the radio link.
* PHY (Physical Layer): Modulation and coding.

Historically, all of these functionalities have been implemented in a single "monolithic" base station. However, in the 5G RAN, these functions can be split into up to three "units" - a radio unit (RU), a distributed unit (DU), and a central unit (CU).


![](/blog/content/images/2024/04/split-ran.svg)

<small>Image source: [Jim Kurose](https://gaia.cs.umass.edu/cs590ae/schedule.html)</small>

In this experiment, we will bring up an end-to-end 5G network, including a 5G core and a 5G RAN, with a CU-DU split: the PDCP layer and above is implemented in a CU, and the RLC layer and below is implemented in a DU.

![](/blog/content/images/2024/04/5g-ran-diagram.svg)

The CU and DU are connected by the F1 interface. The control plane signaling on this interface uses a protocol called F1AP, which uses SCTP transport, and the user plane packets are transferred using UDP transport. We will observe the packets exchanged over the F1 interface, as well as packets exchanged between the CU and the core network, in this experiment.

## Run my experiment

### Reserve a server

For this experiment, you will need one `d430`-type server on the [Emulab cluster](https://docs.emulab.net/hardware.html). Since you are reserving an entire bare-metal server, you should:

* set aside time to work on this lab. You should reserve 6-12 hours (depending on how conservative you want to be) - the lab should not take that long, but this will give you some extra time in case you run into any issues.
* **reserve** a `d430` server for that time at least a few days in advance
* and during your reserved time, complete the entire lab assignment in one sitting.

To reserve a server, click on Experiments > Reserve Resources in the POWDER menu. Then,

* identify the **Project** that you will reserve resources in
* select the Emulab **cluster**, `d430` **hardware**, and put `1` in the **nodes** box
* leave the **Frequency Range** section blank
* set a **start and end time** for your reservation
* and in the **Reason** box, write: "Learn about the 5G core network for my wireless networks course."

Click "Check" to verify that the hardware you have requested is available during the time that you have specified. If it is not, you can use the resource visualization on the right side of the page to identify a better time.

When your request is verified, you will see a pop up asking "Would you like to submit this reservation request?". Check the **Yes** box and then click **Submit**.

### Instantiate a profile

At the beginning of your reservation time, open the POWDER profile at the following link:

[https://www.powderwireless.net/p/244cecf4e88b119a16e5fb7d3fa5bf0a91a571a2](https://www.powderwireless.net/p/244cecf4e88b119a16e5fb7d3fa5bf0a91a571a2)

First, you'll see a brief description of the profile. Click "Next". On the following page, select the "Project" for this course and click "Next". 

On the "Schedule" page, look for the reservation you had created earlier. The "start" time should be in green, indicating that your reservation has started. Check the box next to your username, to automatically fill in the end time of your experiment with the end time of your reservation.

![Schedule experiment using your pre-existing reservation.](/blog/content/images/2024/03/powder-schedule.svg)

Then, click "Finish".

Wait until the server is ready to log in. This can take 10-20 minutes.

* on the "topology view", the server should be green with a check mark in the top right corner
* on the "list view" the status should say "ready" and the startup state should say "Finished"

Then, use SSH to log in to the server.

### Deploy a 5G core network

Before we begin our exploration of the 5G RAN, we will deploy a 5G core network for the RAN to connect to.

Use

```
cd /opt/oai-cn5g-fed/docker-compose  
sudo python3 ./core-network.py --type start-mini --scenario 1  
```

to deploy the 5G core network functions using Docker. At the end, it should say

```
OAI 5G Core network is configured and healthy....
```

<blockquote>
<b>Note:</b> If you need to stop the core network (e.g. to restart it), use
```
cd /opt/oai-cn5g-fed/docker-compose  
sudo python3 ./core-network.py --type start-mini --scenario 1  
```
</blockquote>

### Build the 5G RAN

The 5G RAN in our experiment is implemented in [open source software](https://gitlab.eurecom.fr/oai/openairinterface5g) by the OpenAirInterface project.  We are going to deploy a RAN in simulated RF mode, i.e. there will be a simulated link between the gNB and UE.

However, we will first need to build this RAN from the source code, which takes a little bit of time. 

```
sudo chown -R $USER:$GROUP /opt/openairinterface5g 
cd /opt/openairinterface5g
source oaienv
cd cmake_targets
./build_oai -c -C -I --ninja
```

Once this is done, you should see the message

```
BUILD SHOULD BE SUCCESSFUL
```

Then, run

```
./build_oai -c -C --gNB --nrUE -w SIMU --build-lib all --ninja
```

and again, wait until


```
BUILD SHOULD BE SUCCESSFUL
```

### Start the CU, DU, and UE

For the next part of this experiment, you will need six SSH sessions:

* one each for launching the CU, DU, and UE
* one each for capturing packets with `tcpdump` on the F1 interface (between CU-DU) and the N2 and N3 interface (between CU-AMF and CU-UPF)
* and one for watching the AMF log

In one SSH session, start capturing packets on the N2 interface:

<pre>
sudo tcpdump -i demo-oai -f "not arp and (src 192.168.70.129 or dst 192.168.70.129)" -w ~/start-cu-n2-n3.pcap
</pre>

and in a second session, monitor the AMF logs:

```
sudo docker logs -f oai-amf
```

Note that initially, the AMF is not aware of any gNB or any UE:

<pre>
|----------------------------------------------------gNBs' information-------------------------------------------|
|    Index    |      Status      |       Global ID       |       gNB Name       |               PLMN             |
|      -      |          -       |           -           |           -          |               -                |
|----------------------------------------------------------------------------------------------------------------|

|----------------------------------------------------------------------------------------------------------------|
|----------------------------------------------------UEs' information--------------------------------------------|
| Index |      5GMM state      |      IMSI        |     GUTI      | RAN UE NGAP ID | AMF UE ID |  PLMN   |Cell ID|
|----------------------------------------------------------------------------------------------------------------|
</pre>

Next, we will bring up the CU:

<pre>
cd /opt/openairinterface5g/cmake_targets/
sudo RFSIMULATOR=server ./ran_build/build/nr-softmodem --rfsim --sa -O /local/repository/etc/cu.conf
</pre>


After a few moments, you should now see in the AMF log that it is aware of the gNB:

<pre>
|----------------------------------------------------gNBs' information-------------------------------------------|
|    Index    |      Status      |       Global ID       |       gNB Name       |              PLMN              |
|      1      |    Connected     |         0xe000        |       gNB-Eurecom-CU |            208, 95             | 
|----------------------------------------------------------------------------------------------------------------|
</pre>

Use Ctrl+C to stop the `tcpdump`. Then, use 

<pre>
tshark -r ~/start-cu-n2-n3.pcap -Nn -H /local/repository/etc/hosts
</pre>

to play back the packet capture. Observe the communication between CU and AMF. This is the similar to the exchange you would see between a monolithic gNB and AMF when the gNB attaches to the core network.

Before we bring up the DU, we will start capturing packets on the F1 interface:

<pre>
sudo tcpdump -i lo -f "sctp or port 2153" -w ~/start-du-f1.pcap
</pre>

Now, we bring up the DU:

<pre>
cd /opt/openairinterface5g/cmake_targets/
sudo RFSIMULATOR=server ./ran_build/build/nr-softmodem --rfsim --sa -O /local/repository/etc/du.conf
</pre>

Once you start to see

```
[NR_MAC]   Frame.Slot X
```

output in the DU session, it is ready to use!

Use Ctrl+C to stop the `tcpdump`. Then, use 

<pre>
tshark -r ~/start-du-f1.pcap -Nn -H /local/repository/etc/hosts
</pre>

to play back the packet capture. Observe the communication between CU and DU over the F1 interface. (The DU endpoint is labeled "DU-F1C" to denote the **c**ontrol plane F1 interface.) 


<pre>
    1   0.000000       DU-F1C → CU-F1        SCTP 114 INIT 
    2   0.000070        CU-F1 → DU-F1C       SCTP 338 INIT_ACK 
    3   0.000095       DU-F1C → CU-F1        SCTP 310 COOKIE_ECHO 
    4   0.000155        CU-F1 → DU-F1C       SCTP 50 COOKIE_ACK 
    5   0.000516       DU-F1C → CU-F1        F1AP 278 F1SetupRequest, MIB, SIB1
    6   0.000538        CU-F1 → DU-F1C       SCTP 62 SACK 
    7   0.001035        CU-F1 → DU-F1C       F1AP 98 F1SetupResponse
    8   0.001059       DU-F1C → CU-F1        SCTP 62 SACK 
    9   0.001089        CU-F1 → DU-F1C       F1AP 118 GNBCUConfigurationUpdate SIB2[UNKNOWN PER: too many extensions][Malformed Packet]
   10   0.001918       DU-F1C → CU-F1        F1AP 94 SACK , GNBCUConfigurationUpdateAcknowledge
   11   0.202424        CU-F1 → DU-F1C       SCTP 62 SACK 
</pre>


---

**Lab report**: Refer to section "8.5 F1 Startup and cells activation" in [3GPP TS  138.401](https://www.etsi.org/deliver/etsi_ts/138400_138499/138401/17.07.00_60/ts_138401v170700p.pdf). Show your `tshark` output and label the messages illustrated in "Figure 8.5-1: F1 startup and cell activation".

**Lab report**: Examine these packets in more detail with

```
tshark -r ~/start-du-f1.pcap -Nn -H /local/repository/etc/hosts -Y 'f1ap' -V
```

In the F1 setup request, which cell(s) does the DU say it is ready to activate? What are the PLMN IDs (MCC and MNC)? of these cell(s)? In the gNB-CU configuration update sent from the CU, which cell(s) does it say to activate? Show evidence from the detailed `tshark` output or from Wireshark to support your answer.

---

Before we attach the UE, we will monitor both the F1 interface and the N2 and N3 interfaces.

In one SSH session, start capturing packets on the N2 and N3 interface:

<pre>
sudo tcpdump -i demo-oai -f "not arp and (src 192.168.70.129 or dst 192.168.70.129)" -w ~/start-ue-n2-n3.pcap
</pre>

In another SSH session, start capturing packets on the F1 interface:

<pre>
sudo tcpdump -i lo -f "sctp or port 2153" -w ~/start-ue-f1.pcap
</pre>

Finally, start the UE:

<pre>
cd /opt/openairinterface5g/cmake_targets
sudo RFSIMULATOR=127.0.0.1 ./ran_build/build/nr-uesoftmodem -O /local/repository/etc/ue.conf -r 106 -C 3619200000 --sa --nokrnmod --numerology 1 --band 78 --rfsim --rfsimulator.options chanmod
</pre>

You should see in the AMF log output that it is now aware of the UE:

<pre>
|----------------------------------------------------UEs' information---------------------------------------------|
| Index |      5GMM state      |      IMSI        |     GUTI      | RAN UE NGAP ID | AMF UE ID |  PLMN   |Cell ID |
|      1|       5GMM-REGISTERED|   208950000000031|               |      2224453656|          1| 208, 95 |14680064|
|-----------------------------------------------------------------------------------------------------------------|
</pre>

Use Ctrl+C to stop the `tcpdump` processes, and to stop monitoring the AMF log.

To see the communication over both F1 and N2 interfaces in order, we will merge the two packet captures:

<pre>
mergecap ~/start-ue-n2-n3.pcap ~/start-ue-f1.pcap -w ~/start-ue-all.pcap
</pre>

Then, use 

<pre>
tshark -r ~/start-ue-all.pcap -Nn -H /local/repository/etc/hosts
</pre>

to play back the packet captures.


<pre>
    3   1.514369       DU-F1C → CU-F1        F1AP/NR RRC 254 RRC Setup Request
    4   1.515493        CU-F1 → DU-F1C       F1AP/NR RRC 250 SACK , RRC Setup
    5   1.592801       DU-F1C → CU-F1        F1AP/NR RRC/NAS-5GS 158 SACK , RRC Setup Complete, Registration request MAC=0x00000000
    6   1.594171          gNB → AMF          NGAP/NAS-5GS 146 InitialUEMessage, Registration request
    7   1.648255          AMF → gNB          NGAP/NAS-5GS 630 DownlinkNASTransport, Authentication request
    8   1.648707        CU-F1 → DU-F1C       F1AP/NR RRC/NAS-5GS 166 SACK , DL Information Transfer, Authentication request MAC=0x00000000
    9   1.687973       DU-F1C → CU-F1        F1AP/NR RRC/NAS-5GS 142 SACK , UL Information Transfer, Authentication response MAC=0x00000000
   10   1.688413          gNB → AMF          NGAP/NAS-5GS 146 UplinkNASTransport, Authentication response
   11   1.691739          AMF → gNB          NGAP/NAS-5GS 462 DownlinkNASTransport, Security mode command
   12   1.692123        CU-F1 → DU-F1C       F1AP/NR RRC/NAS-5GS 146 SACK , DL Information Transfer, Security mode command MAC=0x00000000
   13   1.735382       DU-F1C → CU-F1        F1AP/NR RRC/NAS-5GS 170 SACK , UL Information Transfer MAC=0x00000000
   14   1.735804          gNB → AMF          NGAP/NAS-5GS 174 UplinkNASTransport
   15   1.741138          AMF → gNB          NGAP/NAS-5GS 1398 InitialContextSetupRequest
   16   1.741717        CU-F1 → DU-F1C       F1AP/NR RRC 126 SACK , Security Mode Command MAC=0x25525c73
   17   1.782378       DU-F1C → CU-F1        F1AP/NR RRC 118 SACK , Security Mode Complete MAC=0x5efc1e77
   18   1.782792        CU-F1 → DU-F1C       F1AP/NR RRC 130 SACK , UE Capability Enquiry MAC=0xfcaf52a3
   19   1.827818       DU-F1C → CU-F1        F1AP/NR RRC 130 SACK , UE Capability Information MAC=0x66aa74df
   20   1.828237        CU-F1 → DU-F1C       F1AP 166 SACK , UEContextSetupRequest
   21   1.828277          gNB → AMF          NGAP 122 UERadioCapabilityInfoIndication
   22   1.829565       DU-F1C → CU-F1        F1AP 242 SACK , UEContextSetupResponse
   23   1.830162        CU-F1 → DU-F1C       F1AP/NR RRC/NAS-5GS 366 SACK , RRC Reconfiguration MAC=0xdba06cc2
   24   1.874542       DU-F1C → CU-F1        F1AP/NR RRC 118 SACK , RRC Reconfiguration Complete MAC=0x73dca43e
   25   2.031981          AMF → gNB          SCTP 62 SACK 
   26   2.032035          gNB → AMF          NGAP 86 InitialContextSetupResponse
   27   2.075977        CU-F1 → DU-F1C       SCTP 62 SACK 
   28   2.235983          AMF → gNB          SCTP 62 SACK 
   29   2.879210       DU-F1C → CU-F1        F1AP/NR RRC/NAS-5GS 114 UL Information Transfer MAC=0x46ad728f
   30   2.879744          gNB → AMF          NGAP/NAS-5GS 118 UplinkNASTransport
   31   3.079980        CU-F1 → DU-F1C       SCTP 62 SACK 
   32   3.079981          AMF → gNB          SCTP 62 SACK 
   33   3.080018       DU-F1C → CU-F1        F1AP/NR RRC/NAS-5GS 138 UL Information Transfer MAC=0xc6c3553d
   34   3.080542          gNB → AMF          NGAP/NAS-5GS 146 UplinkNASTransport
   35   3.095685          AMF → gNB          NGAP/NAS-5GS 266 PDUSessionResourceSetupRequest
   36   3.096274        CU-F1 → DU-F1C       F1AP 154 SACK , UEContextModificationRequest
   37   3.097301       DU-F1C → CU-F1        F1AP 294 SACK , UEContextModificationResponse
   38   3.097923        CU-F1 → DU-F1C       F1AP/NR RRC/NAS-5GS 382 SACK , RRC Reconfiguration MAC=0xace84678
   39   3.142785       DU-F1C → CU-F1        F1AP/NR RRC 118 SACK , RRC Reconfiguration Complete MAC=0xfc4240f0
   40   3.143453          gNB → AMF          NGAP 122 PDUSessionResourceSetupResponse
   41   3.343986          AMF → gNB          SCTP 62 SACK 
   42   3.343991        CU-F1 → DU-F1C       SCTP 62 SACK 
</pre>

---

**Lab report**: Refer to section "Figure 8.1-1: UE Initial Access procedure" in [3GPP TS  138.401](https://www.etsi.org/deliver/etsi_ts/138400_138499/138401/17.07.00_60/ts_138401v170700p.pdf).  Show your `tshark` output, and label the packets shown in "Figure 8.1-1: UE Initial Access procedure". 

(Note that you have captured packets between the DU and CU, and between the gNB-CU and AMF. On the interface between the CU and AMF, the CU is labeled "gNB" in your `tshark` output.)

**Lab report**: When the AMF needs to send a message to the UE via the RAN, it sends a DOWNLINK NAS TRANSPORT message to the RAN. This process is described in section "8.6.2 Downlink NAS Transport" in [3GPP TS  138.413](https://www.etsi.org/deliver/etsi_ts/138400_138499/138413/17.07.00_60/ts_138413v170700p.pdf). Then, the RAN will forward this message to the UE.  Similarly, if the RAN receives a message from a UE that should be forwarded to the AMF, it uses the Uplink NAS Transport procedure described in section "8.6.3 Uplink NAS Transport". In your packet capture, identify messages sent between the UE and the AMF, as they appear (1) on the F1 interface, and (2) on the N2 interface. 


---

### Generate traffic between external data network and UE


Finally, we are going to generate some traffic between the UE and an external data network. We will monitor both the F1 interface and the N2 and N3 interfaces.

In one SSH session, start capturing packets on the N2 and N3 interface:

<pre>
sudo tcpdump -i demo-oai -f "icmp or udp" -w ~/ue-ext-dn-n2-n3.pcap
</pre>

In another SSH session, start capturing packets on the F1 interface:

<pre>
sudo tcpdump -i lo -f "sctp or port 2153" -w ~/ue-ext-dn-f1.pcap
</pre>


In another session (you can use the session in which you were previously monitoring the AMF log), we are going to send an ICMP echo request from the external data network to the UE. First, we get the UE IP address; then we send an ICMP echo request, and get a response, from inside the `oai-ext-dn` Docker container:

<pre>
UEIP=$(ip -o -4 addr list oaitun_ue1 | awk '{print $4}' | cut -d/ -f1)
sudo docker exec -it oai-ext-dn ping -c 1 $UEIP 
</pre>

After the echo response is received, stop the `tcpdump` sessions with Ctrl+C.

To see the communication over both F1 and N3 interfaces in order, we will merge the two packet captures:

<pre>
mergecap ~/ue-ext-dn-n2-n3.pcap ~/ue-ext-dn-f1.pcap -w ~/ue-ext-dn-all.pcap
</pre>

Then, use 

<pre>
tshark -r ~/ue-ext-dn-all.pcap -Nn -H /local/repository/etc/hosts
</pre>

to play back the packet captures. Note that the DU endpoint is labeled "DU-F1U to denote the **u**ser plane F1 interface.) 

<!--

<pre>
    3   2.594836       EXT-DN → 12.1.1.156   ICMP 98 Echo (ping) request  id=0x00b9, seq=1/256, ttl=64
    4   2.594951       EXT-DN → 12.1.1.156   GTP <ICMP> 142 Echo (ping) request  id=0x00b9, seq=1/256, ttl=63
    5   2.595074        CU-F1 → DU-F1U       UDP 137 2153 → 2153 Len=95
    6   2.637009       DU-F1U → CU-F1        UDP 137 2153 → 2153 Len=95
    7   2.637113   12.1.1.156 → EXT-DN       GTP <ICMP> 142 Echo (ping) reply    id=0x00b9, seq=1/256, ttl=64 (request in 4)
    8   2.637192   12.1.1.156 → EXT-DN       ICMP 98 Echo (ping) reply    id=0x00b9, seq=1/256, ttl=63 (request in 3)
</pre>
-->

---

**Lab report**: Show the `tshark` output, and label each of the following packets in it:

1. the ICMP echo request that is originated by the external data network, and is sent to the UE with the UPF as the next hop
2. the ICMP echo request that has been encapsulated in a GTP tunnel and forwarded from the UPF to the RAN over the N3 interface
3. the ICMP echo request that is forwarded over the F1-U interface from CU to DU inside a UDP packet
4. the ICMP echo response that is forwarded over the F1-U interface from DU to CU inside a UDP packet
5. the ICMP echo response that has been encapsulated in a GTP tunnel and forwarded from the RAN to the UPF over the N3 interface
6. the ICMP echo response that is delivered to the external data network.


---

### Transfer packet captures to your laptop with `scp`

Before your experiment ends, make sure to transfer all of your packet captures to your laptop with `scp`. You can use these packet captures to help you answer the lab report questions.

