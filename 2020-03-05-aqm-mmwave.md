

## Background

5G wireless networks work on mmWave links which suffer from highly variable capacity and intermittent outages, during which a temporary "bufferbloat" condition occurs. In traditional networks, AQM (Active Queue Management) schemes are used to overcome the bufferbloat problem. However, AQMs were not necessarily designed to address bufferbloat on mmWave links, and this begs the need to evaluate the current AQMs in mmWave environments. In this experiment, we evaluate two of the more common AQMs, PIE and FQ_CoDel, and compare how they perform with respect to a FIFO queue in a scenario with highly variable capacity and intermittent outages.

## Results

With a large FIFO queue, we observe extreme delay (up to 150 ms) during temporary degradation of the mmWave link because of a human blocker. Both the small FIFO queue and the AQMs are effective at mitigating this queuing delay. However, further research is needed to understand how these will perform with more realistic traffic patterns.

![Experiment results](/blog/content/images/2020/03/mmWave-buffer-management-.svg)

## Run my experiment




This experiment runs on  [CloudLab](https://cloudlab.us/). If you have a GENI account, you can sign in to CloudLab as follows:

-   Visit  [https://cloudlab.us/](https://cloudlab.us/)  and click on "Log In"
-   Click on the button that says "Geni User?"
-   In the pop-up window, click on the GENI logo
-   In the new pop up window, log on to your GENI Portal account
-   Authorize CloudLab to use your GENI account

Once you are signed in to CloudLab, reserve resources by visiting the link for our profile,  [https://www.cloudlab.us/p/7d41098b-a1a4-11e9-8677-e4434b2381fc](https://www.cloudlab.us/p/7d41098b-a1a4-11e9-8677-e4434b2381fc). Click on "Instantiate", then choose CloudLab Wisconsin and start your experiment. Wait until all nodes have booted and are ready to log in, then click on the "List view" tab to get SSH login instructions for your nodes.


###  Set up and install software


On **src** and **dstn** nodes, install **iperf3**
```
sudo apt-get update
sudo apt-get -y install iperf3
```
On **src** and **router** nodes, install **moreutils**
```
sudo apt-get update
sudo apt-get -y install moreutils 	
```

Next we will need to install some necessary data files and a script to be able to do bandwidth shaping. On the **router** node, run the following commands:

<pre>
wget https://gist.githubusercontent.com/Shreeshail-Hingane/ea39871943010085953f1bd7ef691d35/raw/8b4586054ab6d0cbb73f703c2508745596291c98/lb-tput.csv
wget https://gist.githubusercontent.com/Shreeshail-Hingane/ea39871943010085953f1bd7ef691d35/raw/4bf1add9a8c32d9c553f86a4ec45954283f9ad4b/mobb-tput.csv
wget https://gist.githubusercontent.com/Shreeshail-Hingane/ea39871943010085953f1bd7ef691d35/raw/4bf1add9a8c32d9c553f86a4ec45954283f9ad4b/sb-tput.csv
wget https://gist.githubusercontent.com/Shreeshail-Hingane/ea39871943010085953f1bd7ef691d35/raw/4bf1add9a8c32d9c553f86a4ec45954283f9ad4b/sl-tput.csv
wget https://gist.githubusercontent.com/ffund/7a02086edcbff5af7db025e08621f08d/raw/01bacb40db9ed5492f3f78b512971893d62c39bc/tputvary.sh
</pre>

Finally, run the following command on the router node to set up the bandwidth shaper using an `htb` (Hierarchical Token Bucket):

<pre>
sudo tc qdisc del dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") root handle 1: htb default 3
sudo tc class add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1: classid 1:3 htb rate 3gbit
</pre>

We will run the experiment for four different queuing disciplines **(pie, fq_codel, smallfifo, largefifo)** and 4 different scenarios **(static link(sl), short blockage(sb), long blockage(lb), mobility & blockage(mobb))** resulting in a total of 16 different cases.

Below we will demonstrate how to run one of these cases and later show how to change the queuing disciplines or scenario to run the other cases.

We will need six terminal windows open simultaneously to run an instance of the experiment. 

###  Run experiment with large FIFO queue

Open six terminal windows and start three SSH sessions on the source node, two on the router, and one on the destination node.

Set up a FIFO queue with capacity 60 Mbits on the router, on the link between the router and the destination node.


```
sudo tc qdisc add dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 7500000
```


Next, set the `EXPID` variable in every shell. In each of the six windows, run

```
EXPID=sl-largefifo-1
```

This indicates that we will be running the first iteration where we have a static link with a largefifo queue operating in the router.

Finally, prepare six commands by copying and pasting each of the following into the terminal windows, but don't start any of these yet - wait until all six commands are prepared. Then hit "Enter" in each terminal window, in the same order as which the commands are listed.


<ol>
  <li><b>dstn</b>: set up the iperf3 server: <pre>iperf3 -s -i 0.1 -1</pre></li>
  <li><b>src</b>: measure the delays seen by a separate ping flow, and save to a file: <pre>sudo ping -w 120 -i 0.1 10.0.3.1 | while read line; do echo `date +%s.%N` - $line; done | tee "$EXPID"-ping.txt</pre></li>
  <li><b>src</b>: measure the delays seen by TCP flows, and save to a file: <pre>rm "$EXPID"-ss.txt; while true; do ss --no-header -eipn dst 10.0.3.1 | ts '%.s' | tee -a "$EXPID"-ss.txt; sleep 0.1; done</pre></li>
  <li><b>router</b>: measure queue stats, and save to a file: <pre>rm "$EXPID"-queue.txt; while true; do tc -p -s -d qdisc show dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") | tr -d '\n' | ts '%.s' | tee -a "$EXPID"-queue.txt; echo "" | tee -a "$EXPID"-queue.txt; sleep 0.1; done</pre></li>
  <li><b>router</b>: bandwidth shaping: <pre>bash tputvary.sh sl</pre></li>
  <li><b>src</b>: send ten TCP flows: <pre>iperf3 -f m -c dstn -P 10 -i 0.1 -t 120 | ts '%.s' | tee "$EXPID"-iperf.txt</pre></li>
</ol>

Commands running on terminals 1,2,5,6 will automatically terminate after about  120 seconds. Once these commands finish running, manually shut down (using Ctrl+C) the commands running in terminals 3 and 4.
 


### Change the scenario and repeat experiment

To change the link scenario from static link(sl) to one of the others, change the `EXPID` shell variable and also change the argument to the `tputvary.sh` script in the commands listed above.

For the "long blockage" scenario, run   
```
EXPID=lb-largefifo-1
```
in all six shells, and then run commands 1-4,6 as written but replace 5 with
```
bash tputvary.sh lb
```

For the "short blockage" scenario, run 
  
```
EXPID=sb-largefifo-1
```
in all six shells, and then run commands 1-4,6 as written but replace 5 with
```
bash tputvary.sh sb
```

For the "mobility & blockage" scenario, run 
  
```
EXPID=mobb-largefifo-1
```
in all six shells, and then run commands 1-4,6 as written but replace 5 with

```
bash tputvary.sh mobb
```


### Change the queue and repeat experiment

To apply an AQM in place of a FIFO queue, you will change the queue at the router node, and you will also change the `EXPID` shell variable.

First, remove the old queue:

```
sudo tc qdisc del dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3:
```

Then add a new queue.

To run the experiment with FQ\_CoDel, run
```
EXPID=sl-fq_codel-1
```

in all six shells, and on the router, run

```
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: fq_codel limit 5000 target 10ms
```

Then repeat steps 1-6 as written.

To run the experiment with PIE, run
```
EXPID=sl-pie-1
```

in all six shells, and on the router, run

```
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: pie limit 5000 target 10ms
```

Then repeat steps 1-6 as written.

To run the experiment with a small FIFO queue, run
```
EXPID=sl-smallfifo-1
```

in all six shells, and on the router, run

```
sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 1875000
```

Then repeat steps 1-6 as written.




### Table of experiment configurations

For your convenience, the table below lists the commands for all possible combinations of 4 queue types and 4 link scenarios (16 experiments).

<table class="table-striped table-bordered">
    <thead>
        <tr>
            <th colspan="4">Table 1 : Experiment configurations</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Title</td>
            <td>Set EXPID</td>
          	<td>Argument to <code>tputvary.sh</code></td>
          	<td>Queue configuration</td>
        </tr>
        <tr>
              <td>Large FIFO, static link</td>
              <td><pre>EXPID=sl-largefifo-1</pre></td>
              <td><pre>bash tputvary.sh sl</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 7500000</code></pre></td>
         </tr>
        <tr>
              <td>Large FIFO, short blockage</td>
              <td><pre>EXPID=sb-largefifo-1</pre></td>
              <td><pre>bash tputvary.sh sb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 7500000</code></pre></td>
         </tr>

        <tr>
              <td>Large FIFO, long blockage</td>
              <td><pre>EXPID=lb-largefifo-1</pre></td>
              <td><pre>bash tputvary.sh lb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 7500000</code></pre></td>
         </tr>

        <tr>
              <td>Large FIFO, mobility & blockage</td>
              <td><pre>EXPID=mobb-largefifo-1</pre></td>
              <td><pre>bash tputvary.sh mobb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 7500000</code></pre></td>
         </tr>
        <tr>
              <td>FQ_Codel, static link</td>
              <td><pre>EXPID=sl-fq_codel-1</pre></td>
              <td><pre>bash tputvary.sh sl</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: fq_codel limit 5000 target 10ms</code></pre></td>
         </tr>
        <tr>
              <td>FQ_Codel, short blockage</td>
              <td><pre>EXPID=sb-fq_codel-1</pre></td>
              <td><pre>bash tputvary.sh sb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: fq_codel limit 5000 target 10ms</code></pre></td>
         </tr>

        <tr>
              <td>FQ_Codel, long blockage</td>
              <td><pre>EXPID=lb-fq_codel-1</pre></td>
              <td><pre>bash tputvary.sh lb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: fq_codel limit 5000 target 10ms</code></pre></td>
         </tr>

        <tr>
              <td>FQ_Codel, mobility & blockage</td>
              <td><pre>EXPID=mobb-fq_codel-1</pre></td>
              <td><pre>bash tputvary.sh mobb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: fq_codel limit 5000 target 10ms</code></pre></td>
         </tr>

        <tr>
              <td>PIE, static link</td>
              <td><pre>EXPID=sl-pie-1</pre></td>
              <td><pre>bash tputvary.sh sl</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: pie limit 5000 target 10ms</code></pre></td>
         </tr>
        <tr>
              <td>PIE, short blockage</td>
              <td><pre>EXPID=sb-pie-1</pre></td>
              <td><pre>bash tputvary.sh sb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: pie limit 5000 target 10ms</code></pre></td>
         </tr>

        <tr>
              <td>PIE, long blockage</td>
              <td><pre>EXPID=lb-pie-1</pre></td>
              <td><pre>bash tputvary.sh lb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: pie limit 5000 target 10ms</code></pre></td>
         </tr>

        <tr>
              <td>PIE, mobility & blockage</td>
              <td><pre>EXPID=mobb-pie-1</pre></td>
              <td><pre>bash tputvary.sh mobb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: pie limit 5000 target 10ms</code></pre></td>
         </tr>

        <tr>
              <td>Small FIFO, static link</td>
              <td><pre>EXPID=sl-smallfifo-1</pre></td>
              <td><pre>bash tputvary.sh sl</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 1875000  </code></pre></td>
         </tr>
        <tr>
              <td>Small FIFO, short blockage</td>
              <td><pre>EXPID=sb-smallfifo-1</pre></td>
              <td><pre>bash tputvary.sh sb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 1875000  </code></pre></td>
         </tr>

        <tr>
              <td>Small FIFO, long blockage</td>
              <td><pre>EXPID=lb-smallfifo-1</pre></td>
              <td><pre>bash tputvary.sh lb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 1875000  </code></pre></td>
         </tr>

        <tr>
              <td>Small FIFO, mobility & blockage</td>
              <td><pre>EXPID=mobb-smallfifo-1</pre></td>
              <td><pre>bash tputvary.sh mobb</pre></td>
              <td><pre><code>sudo tc qdisc replace dev $(ip route get 10.0.3.1 | grep -oP "(?<=dev )[^ ]+") parent 1:3 handle 3: bfifo limit 1875000  </code></pre></td>
         </tr>
    </tbody>
</table>


### Data analysis

To parse the router data, run (on the router node):

<pre>
for EXPID in sl-largefifo-1 sb-largefifo-1 lb-largefifo-1 mobb-largefifo-1 sl-fq_codel-1 sb-fq_codel-1 lb-fq_codel-1 mobb-fq_codel-1 sl-pie-1 sb-pie-1 lb-pie-1 mobb-pie-1 sl-smallfifo-1 sb-smallfifo-1 lb-smallfifo-1 mobb-smallfifo-1
do
    cat "$EXPID-queue.txt" | awk '{print $1","$24$30","$31}' | tr -d 'b' | tr -d 'p' > "$EXPID-queue.csv"
done
</pre>

This will create a file for each experiment ID whose columns are: timestamp, number of packets dropped, queue backlog in bytes, and queue backlog in packets.


And to get the data from SS and iperf, run (on the sender node):

<pre>
for EXPID in sl-largefifo-1 sb-largefifo-1 lb-largefifo-1 mobb-largefifo-1 sl-fq_codel-1 sb-fq_codel-1 lb-fq_codel-1 mobb-fq_codel-1 sl-pie-1 sb-pie-1 lb-pie-1 mobb-pie-1 sl-smallfifo-1 sb-smallfifo-1 lb-smallfifo-1 mobb-smallfifo-1
do
    cat $EXPID-ss.txt | sed -e ':a; /<->$/ { N; s/<->\n//; ba; }'  | grep "iperf3" > $EXPID-ss-processed.txt
    cat $EXPID-ss-processed.txt | awk '{print $1}' > ts-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\bcwnd:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' ' > cwnd-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\brtt:.*?(\s|$)' |  awk -F '[:,]' '{print $2}' | tr -d ' '  | cut -d '/' -f 1   > srtt-$EXPID.txt
    cat $EXPID-ss-processed.txt | grep -oP '\bfd=.*?(\s|$)' |  awk -F '[=,]' '{print $2}' | tr -d ')' | tr -d ' '   > fd-$EXPID.txt

    paste ts-$EXPID.txt fd-$EXPID.txt cwnd-$EXPID.txt srtt-$EXPID.txt -d ',' > $EXPID-ss.csv

    cat $EXPID-iperf.txt | grep "\[ " | grep "sec" | grep -v "0.00-1" | awk '{print $3","$4","$8","$10}' | tr -d ']' | sed 's/-/,/g' > $EXPID-iperf.csv
done
</pre>

This will create a file for each experiment ID whose columns are: timestamp, file descriptor of flow, CWND, and smoothed RTT in ms. It will also create another file for each experiment ID whose columns are: file descriptor of flow, *relative* timestamp from beginning of flow at beginning of interval, relative timestamp at end of interval, throughput in Mbps, and number of retransmissions. 