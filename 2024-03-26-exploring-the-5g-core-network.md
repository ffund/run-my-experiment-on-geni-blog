In this experiment, you will explore the 5G core network by bringing up an end-to-end 5G network on a single server. You will see:

* how 5G core network functions can be deployed in containers
* the HTTP2/TCP interfaces between network functions
* the NGAP/SCTP interface between the 5G gNodeB and the 5G core network AMF
* how traffic is exchanged between a UE and an external data network using GTP tunnels

Note that to run this experiment, you will need to have reserved access to compute resources in advance. Refer to the "reserve a server" instructions in order to complete this step several days before you plan to run the experiment.

We will run this experiment on the [POWDER testbed](https://powderwireless.net/). You should already have set up access to POWDER before you begin this experiment. Then, log in to [https://powderwireless.net/](https://powderwireless.net/) with your CloudLab or POWDER username and password.

## Background

In this experiment, we will set up the following end-to-end 5G network:

![](/blog/content/images/2024/03/5g-core-network-diagram.svg)

and we will use it to observe operation of the 5G core network.

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

### Set up the 5G core network

The 5G core network in our experiment is implemented in [open source software](https://gitlab.eurecom.fr/oai/cn5g) by the OpenAirInterface project. 

The 5G core has been called a "cloud-native" core - it is designed to run on cloud infrastructure, using cloud computing principles like containerization. Each of the network functions in our 5G core will run in a Docker container - a portable, standalone, executable package that includes the network function software and its dependencies, runtime environment, and anything else it needs to run. 

Run

```
sudo docker images oai*
```

to list the network function container images, which are already downloaded to your server. 

```
REPOSITORY       TAG       IMAGE ID       CREATED         SIZE
oai-upf-vpp      v1.4.0    e4b272a861f9   20 months ago   303MB
oai-spgwu-tiny   v1.4.0    b1d0c95000bc   20 months ago   127MB
oai-smf          v1.4.0    639a618a3707   20 months ago   149MB
oai-amf          v1.4.0    890b5b8e62ec   20 months ago   153MB
oai-udr          v1.4.0    fcfc4be7b807   21 months ago   148MB
oai-nssf         v1.4.0    84b0e0494976   21 months ago   96MB
oai-udm          v1.4.0    4eec9f6b1651   22 months ago   143MB
oai-nrf          v1.4.0    c22af18f9da0   23 months ago   102MB
oai-ausf         v1.4.0    bd704e547e4e   23 months ago   140MB
```

Our 5G core will include the following control plane network functions:

<table>
<tr><th>Abbr</th><th>Network Function</th><th></th></tr>
<tr><td>AMF</td><td>Access and Mobility Management Function</td><td></td></tr>
<tr><td>AUSF</td><td>Authentication Server Management Function</td><td></td></tr>
<tr><td>NRF</td><td>Network Repository Function</td><td></td></tr>
<tr><td>NSSF</td><td>Network Slicing Selection Function</td><td></td></tr>
<tr><td>SMF</td><td>Session Management Function</td><td></td></tr>
<tr><td>UDM</td><td>Unified Data Management</td><td></td></tr>
<tr><td>UDR</td><td>Unified Data Repository</td><td></td></tr>
<tr></tr>
</table>

and a combined service gateway, packet gateway user plane function (SPGW-U UPF).

To bring up the core network using these Docker containers, run

```
cd /opt/oai-cn5g-fed/docker-compose
sudo python3 ./core-network.py --type start-basic --scenario 1
```

which will start and configure each of the containers. The final line of output should say 

```
OAI 5G Core network is configured and healthy....
```

and if you run

```
sudo docker ps 
```

you can see the list of running containers (your container IDs in the first column will be different than mine, though!).

<pre>
CONTAINER ID   IMAGE                   COMMAND                  CREATED              STATUS                        PORTS                          NAMES
035fed5b3ae5   trf-gen-cn5g:latest     "/bin/bash -c ' ip r…"   About a minute ago   Up 58 seconds                                                oai-ext-dn
98414ca1b031   oai-spgwu-tiny:v1.4.0   "/bin/bash /openair-…"   About a minute ago   Up About a minute (healthy)   2152/udp, 8805/udp             oai-spgwu
8434cac196eb   oai-smf:v1.4.0          "/bin/bash /openair-…"   About a minute ago   Up About a minute (healthy)   80/tcp, 8080/tcp, 8805/udp     oai-smf
d7369af8e8e1   oai-amf:v1.4.0          "/bin/bash /openair-…"   About a minute ago   Up About a minute (healthy)   80/tcp, 9090/tcp, 38412/sctp   oai-amf
7c1e4c36889c   oai-ausf:v1.4.0         "/bin/bash /openair-…"   About a minute ago   Up About a minute (healthy)   80/tcp                         oai-ausf
d8f22369a085   oai-udm:v1.4.0          "/bin/bash /openair-…"   About a minute ago   Up About a minute (healthy)   80/tcp                         oai-udm
17b97b9c7ddf   oai-udr:v1.4.0          "/bin/bash /openair-…"   About a minute ago   Up About a minute (healthy)   80/tcp                         oai-udr
f92fcd9e41c7   mysql:5.7               "docker-entrypoint.s…"   About a minute ago   Up About a minute (healthy)   3306/tcp, 33060/tcp            mysql
f2eb2973d7f8   oai-nrf:v1.4.0          "/bin/bash /openair-…"   About a minute ago   Up About a minute (healthy)   80/tcp, 9090/tcp               oai-nrf
</pre>

Note that addition to the containers running the functional components of the 5G core, there is also a container named `oai-ext-dn` that will serve as external data network for the UE to communciate with, once we attach a UE to the network.

> **Tip:** If your 5G core network does not come up successfully, you can stop the deployment with `sudo python3 ./core-network.py --type stop-basic --scenario 1` and then, try again!


---

**Lab report**: Show the output of `sudo docker ps` indicating that all containers are running and healthy. (You can take a screenshot or copy/paste from the terminal into your lab report.) Include the terminal prompt, the command, and the output. 

Also, briefly describe the role of each of the NFs in *this* 5G core network deployment. (If you use external resources other than the lecture slides/notes and the reference texts, please cite your sources.)

---


Although there is no gNB or UE attached to the network yet, we can still see some communication among the containers implementing different network functions. Let's capture the next 150 packets exchanged on the network that connects these containers - 

```
sudo tcpdump -i demo-oai -c 150 -f "not arp and not port 53 and not host archive.ubuntu.com and not host security.ubuntu.com" -w ~/core-before-ran.pcap
```

and wait until the `tcpdump` stops.


Let us start by looking at the HTTP traffic exchanged between network functions. We will use `tshark` to summarize the relevant packets:

```
tshark -r ~/core-before-ran.pcap -Nn -H /local/repository/etc/hosts -Y 'http'
```

We can see that the AUSF, UDM, UDR, SMF, and the SPGW-U communicate with the NRF on a regular schedule (every 10 seconds in this deployment - the value is deployment-specific):

<pre>
    6   4.257917         AUSF → NRF          HTTP/JSON 292 PATCH /nnrf-nfm/v1/nf-instances/7fb4a233-9a84-44ee-a97a-74aded716952 HTTP/1.1 , JavaScript Object Notation (application/json)
    8   4.258739          NRF → AUSF         HTTP 163 HTTP/1.1 204 No Content 
   16   4.702826          SMF → NRF          HTTP/JSON 322 PATCH /nnrf-nfm/v1/nf-instances/5349e5b9-39ef-4aac-8c72-033e9d05218e HTTP/1.1 , JavaScript Object Notation (application/json)
   18   4.703203          NRF → SMF          HTTP 163 HTTP/1.1 204 No Content 
   26   8.420875          UDR → NRF          HTTP/JSON 292 PATCH /nnrf-nfm/v1/nf-instances/065c2f77-2b04-45e2-82fc-7576a4a5456c HTTP/1.1 , JavaScript Object Notation (application/json)
   28   8.421670          NRF → UDR          HTTP 163 HTTP/1.1 204 No Content 
   36   9.357589       SPGW-U → NRF          HTTP/JSON 292 PATCH /nnrf-nfm/v1/nf-instances/a02f8095-fd05-4e28-9a86-4a57eb5515f6 HTTP/1.1 , JavaScript Object Notation (application/json)
   38   9.358004          NRF → SPGW-U       HTTP 163 HTTP/1.1 204 No Content 
   46   9.442078          UDM → NRF          HTTP/JSON 292 PATCH /nnrf-nfm/v1/nf-instances/99564313-7893-4b4b-9274-89c67128f6b0 HTTP/1.1 , JavaScript Object Notation (application/json)
   48   9.442431          NRF → UDM          HTTP 163 HTTP/1.1 204 No Content 
</pre>

The NRF is a network function discovery service, and allows NFs to register themselves and discover other NFs:

* Each NF will initially register itself with the NRF by sending its profile to `/nf-instances/{nfInstanceID}` with an HTTP PUT operation.
* After that, it will contact the NRF periodically by sending a "patch" that replaces its status with the value "REGISTERED" or "UNDISCOVERABLE" to `/nf-instances/{nfInstanceID}` with an HTTP PATCH operation. The NRF will return "204 No Content". This is how the NRF knows that the NF is still active - if it does not send this heartbeat on a regular schedule, it will change the NF status to "SUSPENDED".

These heartbeat exchanges are the HTTP packets that you captured. You can use `tshark` again but with the `-V` argument to see these exchanges in more detail:

<pre>
tshark -r ~/core-before-ran.pcap -Nn -H /local/repository/etc/hosts -Y 'http' -V
</pre>

You can also see the logs from the NRF container with

```
sudo docker logs -f oai-nrf
```

and note how each heartbeat is logged. (Use Ctrl+C to stop viewing the log.) And, you can see the logs from other NFs as they send these heartbeats, e.g.

```
sudo docker logs -f oai-ausf
```

and so on.

---

**Lab report**: Show *one* NF Heartbeat exchange between the AUSF and the NRF:

* show *one* HTTP exchange (AUSF to NRF and the response from NRF to AUSF) from the `tshark` output
* from the verbose `tshark` output with the `-V` flag, also show the complete HTTP header and body from the HTTP request 
* also show the corresponding log file lines for *one* heartbeat exchange from the NRF container (use the `tshark` output to identify which NF instance ID belongs to the AUSF)
* and show the corresponding log file lines for *one* heartbeat exchange from the AUSF container

---


You will also have captured a different type of heartbeat, between the SMF and the SPGW-U. Run

<pre>
tshark -r ~/core-before-ran.pcap -Nn -H /local/repository/etc/hosts -Y 'pfcp'
</pre>

to see these packets:

```
    1   0.000000          SMF → SPGW-U       PFCP 58 PFCP Heartbeat Request
    2   0.000282       SPGW-U → SMF          PFCP 58 PFCP Heartbeat Response
```

The Packet Forwarding Control Protocol (PFCP) protocol is used on the N4 interface that connects the SMF and SPGW-U, and this PFCP heartbeat is used to ensure that the path is still valid.

---

**Lab report**: Show the PFCP heartbeats you have captured with `tshark`. What is the frequency of these heartbeats in this deployment?

---


### Set up the 5G RAN

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


For the next part of this experiment, you will need four SSH sessions:

* one each for launching the gNB and the UE
* one for capturing packets with `tcpdump`
* and one for watching NF log files


In one SSH session, start capturing packets:

<pre>
sudo tcpdump -i demo-oai -f "not arp and not port 53 and not host archive.ubuntu.com and not host security.ubuntu.com" -w ~/connect-gnb.pcap
</pre>

and in another session, monitor the AMF logs:

```
sudo docker logs -f oai-amf
```

When the gNB comes up, it connects to the AMF through the N2 interface using the NGAP/SCTP protocol. The AMF is the only control plane NF that the gNB communicates with.

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

since we have not attached any to the core network yet.

In a third terminal, we are going to bring up a monolithic gNodeB - one with no CU/DU/RU split. Run:

<pre>
cd /opt/openairinterface5g/cmake_targets
sudo RFSIMULATOR=server ./ran_build/build/nr-softmodem -O /local/repository/etc/gnb.conf --sa --rfsim
</pre>

and leave this running. Here, we use `-O` to specify a configuration file path, `--sa` to use 5G STANDALONE mode, and `--rfsim` to pass baseband IQ samples in software rather than using real radio hardware.

Once you start to see

```
[NR_MAC]   Frame.Slot X
```

output in the gNB session, it is ready to use! You should now see in the AMF log that it is aware of the gNB:

<pre>
|----------------------------------------------------gNBs' information-------------------------------------------|
|    Index    |      Status      |       Global ID       |       gNB Name       |              PLMN              |
|      1      |    Connected     |         0xe000        | gNB-Eurecom-5GNRBox  |            208, 95             | 
|----------------------------------------------------------------------------------------------------------------|
</pre>

Use Ctrl+C to stop the `tcpdump` and to stop monitoring the AMF log, but leave the gNB process running.

Scroll up in the AMF log and find where it establishes a new connection with the gNB using SCTP:

```
[debug] SCTP Association Change event received
[debug] Add new association with id (2)
[info ] ----------------------
[info ] Local addresses:
[info ]     - IPv4 Addr: 192.168.70.132
[info ] ----------------------
[info ] Peer addresses:
[info ]     - IPv4 Addr: 192.168.70.129
[info ] ----------------------
[debug] Ready to handle new NGAP SCTP association request (id 2)
[debug] Create a new gNB context with assoc_id (2)
```

The gNB then sends an NGSetupRequest (using the NGAP protocol over this SCTP connection), letting the AMF know its configuration. Look for the contents of this request in the AMF log, and save it for your lab report.

Finally, the AMG returns an NGSetupResponse. Look for the contents of this response in the AMF log, and save it for your lab report.

You can play back the SCTP exchange from your packet capture:

<pre>
tshark -r ~/connect-gnb.pcap -Nn -H /local/repository/etc/hosts -Y 'sctp'
</pre>

and you can see the detail of the NGAP protocol traffic with

<pre>
tshark -r ~/connect-gnb.pcap -Nn -H /local/repository/etc/hosts -Y 'ngap' -V
</pre>


---

**Lab report**: Show the SCTP exchange you captured over the N2 interface when the gNB attaches to the core network.


**Lab report**: In the AMF log, it shows a table of gNB information:

<pre>
|----------------------------------------------------gNBs' information-------------------------------------------|
|    Index    |      Status      |       Global ID       |       gNB Name       |              PLMN              |
|      1      |    Connected     |         0xe000        | gNB-Eurecom-5GNRBox  |            208, 95             | 
|----------------------------------------------------------------------------------------------------------------|
</pre>

including a 4-digit hex code that identifies the gNB, a human-readable name for the gNB called the RAN Node Name, and a PLMN ID that identifies the network (this includes an MCC, mobile country code, and an MNC, mobile network code). Find each of these values in the `NGSetupRequest` sent from the gNB to the AMF (either from the AMF log, or from the verbose `tshark` output). Annotate the `NGSetupRequest` to highlight these values.

---


While the gNB is still running, we will prepare to attach a UE to the network!

In one SSH session, start capturing packets:

<pre>
sudo tcpdump -i demo-oai -f "not arp and not port 53 and not host archive.ubuntu.com and not host security.ubuntu.com" -w ~/connect-ue.pcap
</pre>

and in another session, monitor the AMF logs:

```
sudo docker logs -f oai-amf
```

In a third session, attach the UE:

<pre>
cd /opt/openairinterface5g/cmake_targets
sudo RFSIMULATOR=127.0.0.1 ./ran_build/build/nr-uesoftmodem -O /local/repository/etc/ue.conf -r 106 -C 3619200000 --sa --nokrnmod --numerology 1 --band 78 --rfsim --rfsimulator.options chanmod
</pre>

Note that the UE configuration file sets various credentials (IMSI, Ki, OPC, etc) to match records that exist in the basic OAI 5G core network deployment and includes a channel model configuration file for the RF simulator.


You should see in the AMF log output that it is now aware of the UE:

<pre>
|----------------------------------------------------UEs' information---------------------------------------------|
| Index |      5GMM state      |      IMSI        |     GUTI      | RAN UE NGAP ID | AMF UE ID |  PLMN   |Cell ID |
|      1|       5GMM-REGISTERED|   208950000000031|               |      2224453656|          1| 208, 95 |14680064|
|-----------------------------------------------------------------------------------------------------------------|
</pre>

Use Ctrl+C to stop the `tcpdump` and to stop monitoring the AMF log, but leave the gNB and UE processes running.

When the UE attaches to the network, some other NFs are involved in authenticating the UE. To see how this works, use

<pre>
tshark -r ~/connect-ue.pcap -Nn -H /local/repository/etc/hosts -Y 'ngap or http'
</pre>

to play back your packet capture.

---

**Lab report**: Describe the sequence of messages exchanged between the RAN and the core NFs, and between NFs within the core, when the UE connects to the network. Draw a diagram showing the exchange of messages. Then, show each message in sequence from your `tshark` output, and briefly explain its purpose. (Exclude the NRF heartbeat messages.)

You may use any outside sources you like to help you answer this question, but you must cite your sources.

---

### Generate traffic between external data network and UE


Finally, we are going to generate some traffic between the UE and an external data network.

You should still have the gNB and UE processes running. 


In one SSH session, start capturing packets:

<pre>
sudo tcpdump -i demo-oai -f "not arp and not port 53 and not host archive.ubuntu.com and not host security.ubuntu.com" -w ~/ue-ext-dn.pcap
</pre>

In another session, we are going to send an ICMP echo request from the external data network to the UE. First, we get the UE IP address; then we send an ICMP echo request, and get a response, from inside the `oai-ext-dn` Docker container:

<pre>
UEIP=$(ip -o -4 addr list oaitun_ue1 | awk '{print $4}' | cut -d/ -f1)
sudo docker exec -it oai-ext-dn ping -c 1 $UEIP 
</pre>

After the echo response is received, stop the `tcpdump` with Ctrl+C.

To understand this exchange, we need to know the MAC address of the SPGW-U UPF. Find out by running `ip addr` inside the `oai-spgwu` Docker container: 

```
sudo docker exec -it oai-spgwu ip addr
```

Also run

```
ip addr list demo-oai
```

to get the MAC address of the gNB - where the RAN is attached to the core network.

And run

```
sudo docker exec -it oai-ext-dn ip addr
```

to get the MAC address of the first hop in the external data network.

With these addresses in hand, look at the exchange using `tshark`, with Ethernet source and destination addresses included:

<pre>
tshark -T fields -e frame.number -e _ws.col.Source -e _ws.col.Destination -e eth.src -e eth.dst -e _ws.col.Protocol -e ip.len  -e _ws.col.Info -r ~/ue-ext-dn.pcap 'icmp'
</pre>

and save the output for your lab report.

---

**Lab report**: Show the `tshark` output and the `ip addr` output, and use this to explain how IP data is carried between the UE and the external data network:

* When IP traffic is forwarded from the external data network to the 5G core network, to which NF is it sent? (Use the destination MAC address!)
* How is the IP packet encapsulated, and by which NF, before it is transferred to the RAN?
* When a packet is transferred from the RAN to the core network, to which NF is it transferred? (Use the destination MAC address!)
* Which NF un-encapsulates the IP packet and transfers it to the external data network?

---

### Transfer packet captures to your laptop with `scp`

Before your experiment ends, make sure to transfer all of your packet captures to your laptop with `scp`. You can use these packet captures to help you answer the lab report questions.

