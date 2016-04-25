In this experiment, we'll capture surveillance messages from aircraft flying over our wireless testbed in Brooklyn, NY using an RTL-SDR software defined radio device. 

It should take about 60 minutes to run this experiment, but you will need to have [reserved that time](http://witestlab.poly.edu/respond/sites/witest/tutorial/make-a-reservation) in advance. This experiment uses wireless resources (specifically, on the [WITest](http://witestlab.poly.edu/) testbed), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have [reserved time on the WITest testbed](http://witestlab.poly.edu/respond/sites/witest/tutorial/make-a-reservation), and you must run this experiment during your reserved time.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

In this experiment, we'll use an [RTL-SDR](http://www.rtl-sdr.com/about-rtl-sdr/) software defined radio device to listen in on surveillance messages transmitted by aircraft flying over our wireless testbed in Brooklyn, NY.

 
Secondary surveillance radar (SSR) is a radar system used in air traffic control. Unlike primary radar systems that work by detecting reflected radio signals, SSR requests additional information from the aircraft itself such as its identity and altitude. It relies on targets equipped with a radar transponder, that replies to each interrogation signal by transmitting a response containing encoded data.

The most common SSR "interrogation" mode is Mode-S. These are the kind of messages we'll be watching today.


You can read more about SSR on its [Wikipedia page](https://en.wikipedia.org/wiki/Secondary_surveillance_radar).  The [Mode-S](https://en.wikipedia.org/wiki/Secondary_surveillance_radar#Mode_S) section in particular describes the specific uplink and downlink messages we are likely to see.


We may also see some ADS-B messages. Automatic Dependent Surveillance – Broadcast (ADS–B) can be used by ATC ground stations as a replacement for secondary radar. An ADS-B equipped aircraft determines its position via satellite navigation and periodically broadcasts it, enabling it to be tracked. This information can be received by other aircraft (in addition to ATC) to provide situational awareness and allow self separation.

You can read more about ADS-B in its [Wikipedia page](https://en.wikipedia.org/wiki/Automatic_dependent_surveillance_%E2%80%93_broadcast).

## Results

If you don't have X11 forwarding, you can see the results in a table, like this:

```
Hex     Mode  Sqwk  Flight   Alt    Spd  Hdg    Lat      Long   Sig  Msgs   Ti-
-------------------------------------------------------------------------------
A8AF5F  S                     7025                               11     3   11
AB967B  S     1002  DAL1098   3675  250  039   40.690  -73.969   33   294    0
A923A9  S                     9175                                7     3   27

```

If you have X11 forwarding, you can see a display like this:

![](/blog/content/images/2016/02/dump1090.png)


## Run my experiment

At the beginning of your reservation, SSH in to witestlab.poly.edu and load the baseline disk image onto an RTL-equipped node:

```
omf load -i baseline-witest.ndz -t omf.witest.node1
# or on node3:
# omf load -i baseline-witest.ndz -t omf.witest.node3
# or on node16:
# omf-5.4 load -i baseline-witest.ndz -t omf.witest.node16
# or on node17:
# omf-5.4 load -i baseline-witest.ndz -t omf.witest.node17
```

Wait for the disk load process to finish, then turn the node on with

```
omf tell -a on -t omf.witest.node1
# Or, if you are using a different node, use one of:
# omf tell -a on -t omf.witest.node3
# omf tell -a on -t omf.witest.node16
# omf tell -a on -t omf.witest.node17
```

Wait for the node to come online. Then log in, e.g.

```
ssh root@node1  
# or substitute name of the node you are using
```

Start by installing some software:

```
apt-get update  # update list of available software  
apt-get -y install git cmake libusb-1.0-0-dev
```

Get the RTL software libraries:

```
# On WITest ONLY, need to use a proxy to reach the 
# public Internet to retrieve libraries
export http_proxy="http://10.0.0.200:3128"  
export https_proxy="https://10.0.0.200:3128" 


# Remove other RTL-SDR driver, if it is loaded
modprobe -r dvb_usb_rtl28xxu

git clone https://github.com/steve-m/librtlsdr  
cd librtlsdr  
mkdir build  
cd build  
cmake ../  
make  
make install  
ldconfig  
cd
```

Then get and build the dump1090 application:

```
git clone https://github.com/MalcolmRobb/dump1090
cd dump1090
make
```

Finally, run

```
./dump1090 --aggressive --fix --no-crc-check
```

You should see some messages from aircraft, for example

```
*07e391edf17964;
CRC: fc4a2f (wrong)
DF 0: Short Air-Air Surveillance.
  VS             : Ground
  CC             : 1
  SL             : 7
  Altitude       : 0 meters
  ICAO Address   : fc4a2f
```

Here's an ADS-B message:

```
DF 17: ADS-B message.
  Capability     : 5 (Level 2+3+4 (DF0,4,5,11,20,21,24,code7 - is airborne))
  ICAO Address   : a874e1
  Extended Squitter  Type: 19
  Extended Squitter  Sub : 1
  Extended Squitter  Name: Airborne Velocity
    EW status         : Valid
    EW velocity       : 334
    NS status         : Valid
    NS velocity       : 299
    Vertical status   : Valid
    Vertical rate src : 1
    Vertical rate     : -3008
```

We've been decoding even messages that fail the CRC check, because the signal is noisy and most of our received messages *will* fail that check. 

Depending on various factors (weather, traffic in the air), you may be able to see some aircraft transmit messages that pass the CRC check, and also include flight code information and latitude and longitude information. Try watching the output of

```
./dump1090 --aggressive --fix --interactive
```

for a while.

If you do find some aircraft transmitting latitude and longitude coordinates, and you have X-11 forwarding on your laptop, you can even plot them on a map. Try running

```
./dump1090 --aggressive --fix --interactive --net
```

on the node. Then, in a local console (i.e. on your laptop), run

```
ssh -XC4c arcfour,blowfish-cbc GENI-WIRELESS-USERNAME@witestlab.poly.edu
```

substituting your own GENI wireless username for "GENI-WIRELESS-USERNAME". Run 

```
google-chrome
```

and, when the browser loads, visit port 8080 on the node that you're running dump1090 on. For example, if you are using node17, you would put

```
http://node17:8080/
```

in the URL bar. Center the map around NYC and you'll see the flight paths of aircraft that include latitude and longitude information in their messages.