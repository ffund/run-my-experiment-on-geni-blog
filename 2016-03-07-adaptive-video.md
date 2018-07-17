This experiment explores the tradeoff between different metrics of video quality (average rate, interruptions, and variability of rate) in an adaptive video delivery system.

It should take about 60-120 minutes to run completely, from start (reserve resources) to finish (plot experiment results.) However, the first set of data (with only one experiment run for each value of utilization) will be available in 45 minutes.


To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes).


## Background

### About adaptive video

Streaming video content is typically delivered to Internet users as follows:

* User's device keeps a video buffer (at the application layer)
* The video client downloads video from a server on the Internet. As video is received, it is placed in the buffer.
* As the video is played back, it is removed from the buffer.
* The amount of video in the buffer at a given time is called the buffer occupancy.

The video buffer is often limited in size, due to space constraints on the device or due to access restriction policies on the part of the content owners. The buffer may therefore become temporarily full, at which point no more video will be downloaded until more already-played video is removed from the buffer.

The video buffer may also become empty. This occurs when the video is being played back (and therefore, removed from the buffer) faster than it is being retrieved - i.e., the playback rate is higher than the download rate. This causes the user to experience rebuffering - when playback is interrupted and the user has to wait for more video to be downloaded (as shown above). This state is obviously undesirable, and something we wish very much to avoid.

To create a positive user experience for streaming video, therefore, requires a delicate balancing act.

* On the one hand, increasing the video playback rate too much (so that it is higher than the download rate) causes the undesired rebuffers.
* On the other hand, decreasing the video playback rate also decreases the user-perceived video quality.

Consider the following two video frames. The first shows a video encoded at 200kbps:

![](/blog/content/images/2016/02/dash-200.png)

Here's the same frame at 500kbps, with noticeably better quality:

![](/blog/content/images/2016/02/dash-500.png)

Performing rate selection to balance rebuffer avoidance and quality optimization is an ongoing tradeoff. Adaptive video (roughly) refers to the broad range of techniques that do this, e.g.,identify the best possible rate at which stream a video, so as to maximize quality but avoid rebuffering adjust this rate on an ongoing basis as necessary (e.g., if the network becomes more congested and the download rate decreases).

There are many different commercial adaptive video products: Microsoft Smooth Streaming, Apple HTTP Live Streaming (HLS), Adobe HTTP Dynamic Streaming (HDS). These all adopt approximately the same approach, but because they are not standardized, video prepared for one adaptive video product cannot be directly used for the other. Recent efforts to support interoperability have led to the Dynamic Adaptive Streaming over HTTP (DASH) standard, which works roughly as follows. 

The video file is first encoded into different versions, each having a different rate and/or resolution. These are called representations or media presentations. The representations of a video all have the same content, but they differ in quality.

Each of these is further subdivided in time into segments of equal lengths (e.g., four seconds).

![](/blog/content/images/2016/02/dash-stored.png)

The content server then stores all of the segments of all of the representations (as separate files).

The DASH manifest file, called the Media Presentation Description (MPD), is an XML file that identifies the various representations, identifies the video resolution and playback rate for each, and gives the location of every segment in each representation.

The manifest file is stored alongside the video files on the content server. 

Once the MPD and video files are in place, clients can start requesting DASH video.

First, the client requests the MPD file. It parses the MPD file, learns what representations are available, and decides what representation to request for the first segment. It then retrieves that specific file using the URL given in the MPD, and starts playing it back.

Each time a client finishes retrieving a file, it makes a new decision as to what representation to get for the next segment.

For example, the client might request the following representations for the first four segments of video:

![](/blog/content/images/2016/02/dash-requested.png)

The cumulative set of decisions made by the client is called a decision policy. The decision policy is a set of rules that determine which representation to request, based on some kind of client state - for example, what the current download rate is, or how much video is currently stored in the buffer.

The decision policy is not specified in the DASH standard. Many decision policies have been proposed by researchers, each promising to deliver better quality than the next!

### In this experiment 

You will run a series of experiments trying to evaluate 3 DASH rate adaptation policies. 

As part of these policies, we'll track four variables for each segment:

* The `empirical_rate` is the measured download rate of the previous segment.
* The `buffer_percent` is the buffer occupancy at the time of decision.
* The `decision_rate` is a function of the `empirical_rate` and `buffer_percent` (explained below).
* The `chosen_rate` is the rate of the representation that is actually selected for the segment (i.e., the outcome of the decision). This will be the video playback rate for the segment.



As the client downloads a segment, it keeps track of its download rate (the rate history part of the policy). At the end of the download, it sets `empirical_rate` based on this download rate. For example, if Segment 1 is downloaded at 278 kbps, then the `empirical_rate` considered for Segment 2 will be 278 kbps. A second variable called `decision_rate` is initially set to the `empirical_rate`.

In addition to considering rate history, it also checks buffer occupancy as a percent of the maximum buffer length, and stores this value in `buffer_percent`. If the buffer is almost empty (less than 30%), we want to allow the buffer to recover and prevent a rebuffer interruption. So, we will set the `decision_rate` to zero. Otherwise, we leave `decision_rate` as is.

Next, the `chosen_rate` (the actual video rate we'll use for the next segment) is selected as follows:

* Try to set `chosen_rate` to the rate of the highest representation listed in the MPD that is less than `decision_rate`. For example, if `decision_rate` is still 278 kbps, and representations are available at 100, 200, 400, and 800 kbps, the `chosen_rate` would be set to 200 kbps.
* If no representations in the MPD qualify for the first condition, set `chosen_rate` to the rate of the lowest representation in the MPD.

Finally, we retrieve the next segment with the representation matching `chosen_rate`. The following policies for setting `chosen_rate` will be evaluated:

* Policy 1: When the buffer is less than 30% full, the `decision_rate` is zero, and the `chosen_rate` is the lowest representation available.
* Policy 2: The `decision_rate` is not adjusted, i.e. it is always equal to the actual bitrate observed over the download of the last segment (`empirical_rate`).
* Policy 3: When the buffer is less than 30% full, the `decision_rate` is equal to half of the actual bitrate observed over the download of the last segment (`empirical_rate`).

These policies involve a tradeoff between video rate and buffer status. Obviously, we want to download segments of a high video rate, because these offer better quality. However, we also want to avoid freezes (the state where there is zero video in the buffer) and we want the video playback to be smooth, avoiding big jumps in video rate.




The scenario for today's experiment is very simple. We'll run an experiment in which part of a DASH video is downloaded onto a client over the network. The download rate at any given time will depend on the quality of the link (we emulate that by limiting the outgoing bandwidth of the server).

Afterwards, we'll consider how good the video experience was in terms of:

**Rate**. Is the video rate selected usually a high representation, or a low one? If there are _N_ segments, and _r<sub>n</sub>_ is the rate chosen for the _n<sub>th</sub>_ segment, we can measure average rate as 

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mi>R</mi>
  <mo>=</mo>
  <mfrac>
    <mn>1</mn>
    <mi>N</mi>
  </mfrac>
  <munderover>
    <mo>&#x2211;<!-- ∑ --></mo>
    <mrow class="MJX-TeXAtom-ORD">
      <mi>n</mi>
      <mo>=</mo>
      <mn>1</mn>
    </mrow>
    <mrow class="MJX-TeXAtom-ORD">
      <mi>N</mi>
    </mrow>
  </munderover>
  <msub>
    <mi>r</mi>
    <mrow class="MJX-TeXAtom-ORD">
      <mi>n</mi>
    </mrow>
  </msub>
</math>

<br>

where a higher value of _R_ is better.

**Interruptions**. Does the video ever freeze due to rebuffering? We measure the number of interruptions with

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mi>I</mi>
  <mo>=</mo>
  <munderover>
    <mo>&#x2211;<!-- ∑ --></mo>
    <mrow class="MJX-TeXAtom-ORD">
      <mi>n</mi>
      <mo>=</mo>
      <mn>1</mn>
    </mrow>
    <mrow class="MJX-TeXAtom-ORD">
      <mi>N</mi>
    </mrow>
  </munderover>
  <mo stretchy="false">(</mo>
  <msub>
    <mi>b</mi>
    <mrow class="MJX-TeXAtom-ORD">
      <mi>n</mi>
    </mrow>
  </msub>
  <mo>&lt;=</mo>
  <mn>0</mn>
  <mo stretchy="false">)</mo>
</math>
<br>

where _b<sub>n</sub>_ is the buffer occupancy (as a percent, from 0 to 100) observed at the beginning of the download of the _n<sub>th</sub>_ segment. A lower value is better for this metric.

**Variability**. Are the changes in video quality smooth or abrupt? One way to measure variability is with

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mi>S</mi>
  <mo>=</mo>
  <mfrac>
    <mn>1</mn>
    <mrow>
      <mi>N</mi>
      <mo>&#x2212;<!-- − --></mo>
      <mn>1</mn>
    </mrow>
  </mfrac>
  <munderover>
    <mo>&#x2211;<!-- ∑ --></mo>
    <mrow class="MJX-TeXAtom-ORD">
      <mi>n</mi>
      <mo>=</mo>
      <mn>2</mn>
    </mrow>
    <mrow class="MJX-TeXAtom-ORD">
      <mi>N</mi>
    </mrow>
  </munderover>
  <mrow>
    <mo>|</mo>
    <mi>log</mi>
    <mo>&#x2061;<!-- ⁡ --></mo>
    <mo stretchy="false">(</mo>
    <msub>
      <mi>r</mi>
      <mrow class="MJX-TeXAtom-ORD">
        <mi>n</mi>
      </mrow>
    </msub>
    <mo stretchy="false">)</mo>
    <mo>&#x2212;<!-- − --></mo>
    <mi>log</mi>
    <mo>&#x2061;<!-- ⁡ --></mo>
    <mo stretchy="false">(</mo>
    <msub>
      <mi>r</mi>
      <mrow class="MJX-TeXAtom-ORD">
        <mi>n</mi>
        <mo>&#x2212;<!-- − --></mo>
        <mn>1</mn>
      </mrow>
    </msub>
    <mo stretchy="false">)</mo>
    <mo>|</mo>
  </mrow>
</math>

<br>
where _r<sub>n</sub>_ is the rate chosen for the _n<sub>th</sub>_ segment. A low value is better for this metric.


## Results

The results of our measurements for all three policies are as follows:
![](/blog/content/images/2016/03/dash-results-1.svg)

In the plots above we can see that in  the first policy, in which the perceived available bitrate is zero when buffer nears depletion, there are large jumps in video quality, although it is successful in preventing freezes in video playback (which would appear as a zero value for buffer status). In the second policy, in which buffer status is not considered, there is a higher average video rate, and the video rate is smooth, but suffers from many freezes in video playback. The third policy is a good compromise, with reasonable video quality overall and subtle transitions in video rate, while only one freeze with very small duration.

In the following table we can see the overall metrics for all three policies:

```
            Average Rate (bps)       Freezes     Rate Variability
Policy 1       1948505                 4              0.059
Policy 2       2442917                13              0.122
Policy 3       2934482                 6              0.094
```

## Run my experiment

To start, create a new slice on the [GENI portal](https://portal.geni.net/). Create a client-server topology with two VMs connected with a link, by downloading and using [this RSpec](https://git.io/vxpBt) (In the Portal, create a new slice, click "Add Resources", scroll down to "Choose RSpec", and select "From File"):

![](http://witestlab.poly.edu/repos/genimooc/run_my_experiment/dash_experiment/dash_topology.png)

Then choose an aggregate and log in to your resources.

Now let's install all the required software. 

**On Client Node**. Install software dependencies for this experiment:

```
sudo su # if you are not logged in as root
cd ~
sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/tools:/mytestbed:/stable/xUbuntu_12.04/ /' >> /etc/apt/sources.list.d/oml2.list"
apt-get update
apt-get -y build-dep vlc
apt-get -y --force-yes install subversion liboml2-dev

wget https://www.freedesktop.org/software/vaapi/releases/libva/libva-1.1.1.tar.bz2
tar -xjvf libva-1.1.1.tar.bz2 
cd libva-1.1.1
./configure
make
make install 
ldconfig
```

Download the VLC client software, compile it and install it

```
cd ~
svn co http://witestlab.poly.edu/repos/genimooc/dash_video/vlc-2.1.0-git
cd vlc-2.1.0-git
./configure LIBS="-loml2" --enable-run-as-root --disable-lua --disable-live555 --disable-alsa --disable-dvbpsi --disable-freetype
make
make install
mv /usr/local/bin/vlc /usr/local/bin/vlc_app
echo '#!/bin/sh' > /usr/local/bin/vlc
echo 'export LD_LIBRARY_PATH="/usr/local/lib${LD_LIBRARY_PATH:+:}${LD_LIBRARY_PATH}"' >> /usr/local/bin/vlc
echo 'export TERM=xterm' >> /usr/local/bin/vlc
echo 'vlc_app "$@"' >> /usr/local/bin/vlc
chmod +x /usr/local/bin/vlc
```

While this is running (it will take a while), open a second terminal and set up the server node.

**On Server Node**. start by installing/setting up Apache, as we need a server for the video file.

```
sudo su # if you are not logged in as root
cd ~
apt-get update # refresh local information about software repositories
apt-get -y install apache2 ruby1.9.1
cd /var/www/html
wget http://witestlab.poly.edu/repos/genimooc/dash_video/BigBuckBunny_2s_480p_only.tar.gz
tar -zxvf BigBuckBunny_2s_480p_only.tar.gz
```

Download the MPD file:

```
cd video
rm bunny_Desktop.mpd
wget -nH --no-parent "http://witestlab.poly.edu/repos/genimooc/run_my_experiment/dash_experiment/bunny_Desktop.mpd"
```

Let's test that everything works up until now. On client node execute

```
vlc -I dummy -V dummy http://192.168.1.200/video/bunny_Desktop.mpd --oml-id client --oml-domain "VLC" --oml-collect "file:/dev/null" --quiet --oml-log-level 0 --sout "#duplicate{dst=display,dst=std{access=file,mux=ps,dst=/tmp/test.mp4}}" --dash-policy 1
```

The output should be something like this

```
VLC media player 2.1.0-git Rincewind (revision 1.3.0-git-6058-g2f0e36e)
Feb 25 13:53:49 INFO	OML Client 2.11.0 [OMSPv5] Copyright 2007-2014, NICTA
INFO	File_stream: opening local storage file '/dev/null'
chosenRate_bps=101492 empiricalRate_bps=0 decisionRate_bps=0 buffer_percent=0
chosenRate_bps=101492 empiricalRate_bps=0 decisionRate_bps=0 buffer_percent=0
INFO	/dev/null: Connected
chosenRate_bps=101492 empiricalRate_bps=5988064 decisionRate_bps=0 buffer_percent=0
chosenRate_bps=101492 empiricalRate_bps=222384442 decisionRate_bps=0 buffer_percent=9
chosenRate_bps=101492 empiricalRate_bps=208178498 decisionRate_bps=0 buffer_percent=13
chosenRate_bps=101492 empiricalRate_bps=194267895 decisionRate_bps=0 buffer_percent=18
chosenRate_bps=101492 empiricalRate_bps=171997078 decisionRate_bps=0 buffer_percent=23
chosenRate_bps=5991271 empiricalRate_bps=196050159 decisionRate_bps=196050159 buffer_percent=30
chosenRate_bps=5991271 empiricalRate_bps=208852276 decisionRate_bps=208852276 buffer_percent=39
chosenRate_bps=5991271 empiricalRate_bps=650956631 decisionRate_bps=650956631 buffer_percent=59
chosenRate_bps=5991271 empiricalRate_bps=886964692 decisionRate_bps=886964692 buffer_percent=74
chosenRate_bps=5991271 empiricalRate_bps=922833287 decisionRate_bps=922833287 buffer_percent=77
chosenRate_bps=5991271 empiricalRate_bps=962394258 decisionRate_bps=962394258 buffer_percent=80
chosenRate_bps=5991271 empiricalRate_bps=1001082564 decisionRate_bps=1001082564 buffer_percent=83
chosenRate_bps=5991271 empiricalRate_bps=1123459517 decisionRate_bps=1123459517 buffer_percent=94
```

Now we are ready to run this experiment.

**On Server Node**. To make our experiment more realistic, we will adjust the rate of the link between client and server so that there is an outbound limit to the server. We have created a series of scripts that perform this action.

First we need a script that initializes this rate limit:

```sh
#!/bin/bash
RATE=$1
tc qdisc del dev eth1 root
tc qdisc add dev eth1 root handle 1: htb default 30
tc class add dev eth1 parent 1: classid 1:1 htb rate $RATE burst 15k
tc qdisc add dev eth1 parent 1:1 sfq perturb 10
tc filter add dev eth1 parent 1: protocol ip prio 1 u32 match ip dst 192.168.1.201 flowid 1:1
```

download this script with:

```
wget -nH --no-parent "http://witestlab.poly.edu/repos/genimooc/run_my_experiment/dash_experiment/src/server/init_rate.sh"
```

Then we have created a script that can change the rate:

```sh
#!/bin/bash
RATE=$1
echo "new rate: $RATE"
tc class change dev eth1 parent 1: classid 1:1 htb rate $RATE burst 15k
```

download this script with:

```
wget -nH --no-parent "http://witestlab.poly.edu/repos/genimooc/run_my_experiment/dash_experiment/src/server/change_rate.sh"
```

Finally we have created a ruby script that orchestrates the scripts above and randomly changes the rate every 10 seconds between 1 mbit and a maximum value.

```ruby
#!/usr/bin/env ruby
INTERVAL = 10
BPS = ARGV[0].to_i
init_cmd = " sh init_rate.sh #{BPS}mbit"
`#{init_cmd}`

while(true)
  sleep(INTERVAL)
  new_rate = rand(1..BPS)
  cmd = "sh change_rate.sh #{new_rate}mbit"
  puts "changing rate to #{new_rate}mbit"
  `#{cmd}` if BPS > 1
end
```


download this script with:

```
wget -nH --no-parent "http://witestlab.poly.edu/repos/genimooc/run_my_experiment/dash_experiment/src/server/change_rate.rb"
```

And execute it like this

```
ruby change_rate.rb MAX_RATE
```

where the rate will randomly change between 1 mbit and MAX_RATE mbit. Suggested value is 6, e.g. `ruby change_rate.rb 6`, because the maximum rate at which the video is encoded is 6Mbps.

The output will be something like this, a new line will appear every 10 seconds:

```
RTNETLINK answers: No such file or directory
changing rate to 6mbit
changing rate to 1mbit
...
```

Now we need to run the vlc command from the **Client Node** and parse/save the results. To that purpose we have created another ruby script.

```ruby
#!/usr/bin/env ruby

require 'open3'
require 'csv'

POLICY      = ARGV[0]
VIDEO_OUT   = "/tmp/test_#{POLICY}.mp4"
parsed_data = []
tokens      = ["chosenRate_bps","empiricalRate_bps","decisionRate_bps","buffer_percent"]

vlc_cmd = "vlc -I dummy -V dummy http://192.168.1.200/video/bunny_Desktop.mpd --oml-id client --oml-domain 'VLC' --oml-collect 'file:/dev/null' --quiet --oml-log-level 0 --sout '#duplicate{dst=display,dst=std{access=file,mux=ps,dst=#{VIDEO_OUT}}}' --dash-policy #{POLICY}"

def write_data(data)
  fname   = "./dash_out_#{POLICY}.csv"
  i       = 1
  sum     = 0
  avg     = 0
  freezes = 0
  r1      = nil
  var_sum = 0
  var     = 0

  CSV.open(fname, "wb") do |csv|
    data.each do |d|
      next if d.empty?
      sum += d["chosenRate_bps"]
      avg = sum / i
      unless r1.nil?
        var_sum += Math.log(d["chosenRate_bps"]) - Math.log(r1)
        var = var_sum / (i - 1) 
      end
      r1 = d["chosenRate_bps"]
      csv << [i, d["chosenRate_bps"], d["empiricalRate_bps"], d["decisionRate_bps"], d["buffer_percent"], avg, var]
      freezes += 1 if d["buffer_percent"] == 0
      
      i += 1
    end
  end

  [fname, avg, var, freezes]
end

begin
  stdin, stdout, stderr = Open3.popen3(vlc_cmd)
  while data = stderr.gets
    puts data
    pdata = {}
    tokens.each do |token|
      split_tok = data.split("#{token}=")
      if split_tok && split_tok[1]
        left_part = split_tok[1]
        pdata[token] = left_part.split(' ')[0].to_i if left_part.split(' ') && left_part.split(' ')[0]
      end
    end
    parsed_data << pdata
  end
rescue Interrupt => e
  `pkill -9 vlc`
  puts "\nThe video output is: #{VIDEO_OUT}"
  print "Saving data to file: "
  fname, avg, var, freezes = write_data(parsed_data)
  puts fname
  puts "The overall average video rate is:    #{avg}"
  puts "The variability of the video quality: #{var}"
  puts "The overall number of freezes are:    #{freezes}"
end
```


download this script with:

```
wget -nH --no-parent "http://witestlab.poly.edu/repos/genimooc/run_my_experiment/dash_experiment/src/client/dash_video_experiment.rb"
```

And execute it like this:

```
ruby dash_video_experiment.rb POLICY_ID
```

where the `POLICY_ID` defines the DASH policy that will be used. Valid values are 1, 2 and 3.

You can let it run as long as you like, typically 60 seconds will give you valid results for inspection. You can press CTRL+C anytime to stop it.


The output of this experiment includes:

* An mp4 file in /tmp/test\_POLICY\_ID.mp4
* A csv file in the same folder of the script

First let's download and watch the mp4 file:

```
scp root@CLIENT_PUBLIC_IP:/tmp/test_1.mp4 .
vlc test.mp4 #you need to have vlc installed on your PC
```

where CLIENT\_PUBLIC\_IP is the IP address or hostname of the control interface of the client (e.g. the one you use to log in.) You can get this from the GENI porta.

If the experiment was correctly executed you should be able to see video quality fluctuate.

We can then plot the experiment results with gnuplot. First install gnuplot  on the client node:

```
apt-get -y install gnuplot
```

Then, assuming the output has been redirected to a file named "dash\_out\_3.csv" (policy 3 was selected for this file), we have created a script that will draw two plots, one for the rates and one for the buffer size:

```
set datafile separator ','  
set xtics font "Times-Roman, 20"  
set ytics font "Times-Roman, 20"  
set key font "Times-Roman, 18"  
set xlabel "Time" font "Times-Roman, 24"  
set ylabel "Bandwidth (bps)"  font "Times-Roman, 24"  
set key outside right center  
set terminal postscript eps enhanced color font 'Times-Roman, 20'  
set output "dash_3.eps"
set yrange [0:7000000]

set title "Rates for Policy 3"  
plot 'dash_out_3.csv' using 1:6 title 'Average Rate' with lines, 'dash_out_3.csv' using 1:2 title 'Chosen Rate' with lines

set ylabel "Percent (%)" font "Times-Roman, 24"  
set yrange [0:110]
set output "dash_buffer_3.eps"  
set title "Buffer size for Policy 3"  
plot 'dash_out_3.csv' using 1:5 title 'Buffer size' with lines
```

Download this script with:

```
wget -nH --no-parent "http://witestlab.poly.edu/repos/genimooc/run_my_experiment/dash_experiment/src/client/dash_plot.gnu"
```

And execute it like this:

```
gnuplot dash_plot.gnu
```

This will generate two plot files in eps format. You will need to download those using scp. If you wish to generate plots for other policies you will have to edit this gnuplot script to change the input file as well as the titles of the plots.

## Implementing your own DASH policy

Navigate to the directory containing the VLC source files:

```
cd vlc-2.1.0-git
```

Now open the file where the DASH policies are implemented with your preferred text editor:

```
modules/stream_filter/dash/adaptationlogic/RateBasedAdaptationLogic.cpp
```
 
You should find the three policies discussed in the lab in the following lines:

```c++
if(this->policy != 2)
    {
        if(this->getBufferPercent() < MINBUFFER)
        {
            if(this->policy == 1)
                bitrate = 0;
            else
                bitrate = this->getBpsAvg() / 2;
        }
    }
```

In order to understand the above if-statement you should know the matching variables with the terms `decision_rate`, `empirical_rate` and `chosen_rate`. These are the following:

* `bitrate` is the `decision_rate`
* `this->getBpsAvg()` gives the `empirical_rate`
* `rep->getBandwidth()` is the `chosen_rate` which is calculated based on the `decision_rate`

In all the three policies the `decision_rate` is set to the `empirical_rate` and it changes only in policy 1 and 3 in the case the buffer is lower than 30%. More specifically if you have chosen policy 1 then `bitrate`-`decision_rate` is set to zero, whereas in policy 2 `bitrate`-`decision_rate` is set to half of the `this->getBpsAvg()`-`empirical_rate`.

Now that you have seen how these policies are implemented you are free to use these metrics in order to implement your own decision policy. These are the offered metrics, but you are free to use also network metrics that will help you to better estimate the current channel conditions. An easy way to insert extra metrics is to write everything you want in a file and then read this file inside the code you will be writing for your own policy. This is just an example of how you can insert more metrics, you are free to choose your own path of implementation.

When you have edited the file with the policies, you should run the following commands which will compile your new code and install the modified VLC application.

```
./configure LIBS="-loml2" --enable-run-as-root --disable-lua --disable-live555 --disable-alsa --disable-dvbpsi --disable-freetype
make
make install
mv /usr/local/bin/vlc /usr/local/bin/vlc_app
echo '#!/bin/sh' > /usr/local/bin/vlc
echo 'export LD_LIBRARY_PATH="/usr/local/lib${LD_LIBRARY_PATH:+:}${LD_LIBRARY_PATH}"' >> /usr/local/bin/vlc
echo 'export TERM=xterm' >> /usr/local/bin/vlc
echo 'vlc_app "$@"' >> /usr/local/bin/vlc
chmod +x /usr/local/bin/vlc
```


## Notes and References

The VLC DASH plugin in this experiment is described in:

Christopher Müller and Christian Timmerer. 2011. A VLC media player plugin enabling dynamic adaptive streaming over HTTP. In *Proceedings of the 19th ACM international conference on Multimedia (MM '11)*. ACM, New York, NY, USA, 723-726. ([URL](http://dx.doi.org/10.1145/2072298.2072429))

The version of this plugin that measurements to an OML database is described in:

Fraida Fund, Cong Wang, Yong Liu, Thanasis Korakis, Michael Zink and Shivendra S. Panwar. 2013. Performance of DASH and WebRTC Video Services for Mobile Users. In *Proceedings of the 2013 20th International Packet Video Workshop*. San Jose, CA, 1-8. ([URL](http://dx.doi.org/ 10.1109/PV.2013.6691455))

and its source code is available via SVN:

```
svn co http://witestlab.poly.edu/repos/omlapps/vlc/vlc-2.1.0-git
```

A README describing its installation is available [here](http://witestlab.poly.edu/svn/filedetails.php?repname=omlapps&path=%2Fvlc%2FREADME.txt). 

The DASH dataset used in this experiment is described in:

Stefan Lederer, Christopher Müller, and Christian Timmerer. 2012. Dynamic adaptive streaming over HTTP dataset. In *Proceedings of the 3rd Multimedia Systems Conference (MMSys '12)*. ACM, New York, NY, USA, 89-94. ([URL](http://dx.doi.org/10.1145/2155555.2155570))

and is available at [this link](http://www-itec.uni-klu.ac.at/dash/?page_id=207) (it's the one referred to as the "older version of our MPEG-DASH dataset".)



