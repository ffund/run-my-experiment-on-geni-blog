
The aim of this experiment is to explain the "spectral droop" described in many questions on the GNU Radio and USRP mailing lists. 

It should take about 30 minutes to run this experiment, but you will need to have reserved that time in advance. This experiment uses wireless resources (specifically, either the sb2, sb3, or sb7 sandbox at [ORBIT](http://geni.orbit-lab.org)), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have reserved time on the sb2, sb3, or sb7 sandbox at [ORBIT](http://geni.orbit-lab.org) and you must run this experiment during your reserved time.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)



## Background

"Why does my spectrum "droop" at the edges?" is a question that is frequently asked on the GNU Radio and USRP mailing lists.

For example, consider [this question from the GNU Radio list](https://www.ruby-forum.com/topic/4418862):

> I want to use USRP to receive real 802.11a/g wifi signal, so I set sample rate 20MHz and appropriate centre frequency. But frequency spectrum of the receiving OFDM signal is strange. (see picture:)
> 
> ![](/blog/content/images/2016/06/fft_router2422_20m.jpg)
> Besides that, I did another experiment. using a USRP generate a 300kHz bandwith OFDM signal, anther receiving end also using 300kHz sample rate, and we can see a perfect OFDM frequency spectrum wave.(see picture:)
> 
> ![](/blog/content/images/2016/06/fft300-cropped.jpg)
> Does anyone know why sample rate increase, OFDM frequency in receiving end turn into this shape?

Here is a lightly edited version of [another example from the USRP users list](http://lists.ettus.com/pipermail/usrp-users_lists.ettus.com/2014-September/010537.html):

> I am using USRP N210, GNU Radio 3.7.2.1 (version of last friday) over
Ubuntu 14.04. I uploaded some screenshots of the FFT of the received signal. Tried with other frequencies and sample rate. I looked to have a center frequency not a factor of sample rate.
> 
> Sampling at 9 MSps:
> ![](/blog/content/images/2016/06/fft9MSps.png)
> 
> Sampling at 10 MSps:
> ![](/blog/content/images/2016/06/fft10MSps.png)
> 
> Sampling at 11 MSps:
> ![](/blog/content/images/2016/06/fft11MSps.png)
> I can see that when I use 10 MSps the signals is much more flat than the signals
captured with 9 or 11 MSps. Why is this?

And here is [another](http://lists.ettus.com/pipermail/usrp-users_lists.ettus.com/2016-February/018497.html):

> I am using a 2 x USRP N210 + SBX40MHz daughter board for measuring channels. I use a OFDM (802.11g) transmission signal @2.45 GHz carrier frequency for estimating the channel in frequency domain. Everything works fine, I even receive data packets without error. But when I use a coax cable between Tx and Rx, I do not obtain a flat transfer function as I expect (cp. Attached image). From the center frequency, the magnitude drops by some 8 dB at the band edge. 
> ![](/blog/content/images/2016/06/gnuradio-attachment-resized.png)

All of these users are experiencing the effects of [filter rolloff](https://en.wikipedia.org/wiki/Roll-off), often called "spectral droop" or "CIC rolloff" in this particular context on the GNU Radio and USRP mailing lists. 

The "spectral droop" is created by the signal processing elements that change the sampling rate of the signal, which is why it appears with some sampling rates and not others.

Mathematically, sampling involves taking a continuous signal _s(t)_ and measuring its value every _T_ seconds to produce the discrete signal, _s(nT)_ for integer values of _n_. The sampling rate _f<sub>s</sub>_ is the number of samples measured per second, thus _f<sub>s</sub> = 1/T_.



![](https://upload.wikimedia.org/wikipedia/commons/thumb/5/50/Signal_Sampling.png/640px-Signal_Sampling.png)
<small>Sampling: the continuous signal is represented with a green colored line while the discrete samples are indicated by the blue vertical lines. Public domain image via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Signal_Sampling.png).</small>

In a radio receiver, an [analog to digital converter (ADC)](https://en.wikipedia.org/wiki/Analog-to-digital_converter) is used to convert the analog, continuous signal received from the antenna to a stream of digital, discrete signal samples by [sampling](https://en.wikipedia.org/wiki/Sampling_(signal_processing)) and [quantizing](https://en.wikipedia.org/wiki/Quantization_(signal_processing)) the signal. 

In the USRP N210 (the hardware used by the experimenters who posted the mailing list questions reproduced above), the ADC samples the received signal at a rate of _f<sub>s</sub>_ = 100 MSps. To reduce the sampling rate beyond that (e.g. so that it can be processed in real time by the host computer), filters in the FPGA [decimate](https://en.wikipedia.org/wiki/Decimation_(signal_processing)) the signal to our desired sampling rate. The decimation factor determines the final sampling rate (if for example we decimate by 4 in the FPGA, the signal entering the host computer will have a rate of 25 MSps).

The figure below shows the FPGA architecture for the USRP N210. The receive chain (the path followed by the received signal) is at the bottom. The upper and lower parts on the right side of the receive chain are for processing the [in-phase and quadrature](https://en.wikipedia.org/wiki/In-phase_and_quadrature_components) signals. The ADC is highlighted in blue, and the filters for decimation in pink and green:

![](/blog/content/images/2016/06/usrp-architecture-2.svg)
<small> Image adapted from: John Malsbury and Matt Ettus. 2013. Simplifying FPGA design with a novel network-on-chip architecture. In *Proceedings of the second workshop on Software radio implementation forum (SRIF '13)*. ACM, New York, NY, USA, 45-52. ([URL](http://dx.doi.org/10.1145/2491246.2491251)) </small>

Note that there are two kinds of filters that may be used for decimation of a signal: a **CIC** filter colored in pink and labeled **&darr;N**, and two **halfband** filters in green and labeled **&darr;2**. 


* The CIC filter provides decimation by an arbitrary programmable integer decimation factor of 1 to 128, but has very significant pass band roll off. (See the red line in the image below.)
* The two halfband filters have fixed decimation factor of 2 each, but have a flatter passband response. (See the blue and green lines in the image below.) These filters are not always used; they may be bypassed.

![](/blog/content/images/2016/06/lab2-filters.png)
<small>Magnitude response of the CIC and halfband filters on the USRP N210.  Image via [GNU Radio wiki](http://gnuradio.org/redmine/projects/gnuradio/wiki/UsrpFAQDDC).</small>

When a UHD or GNU Radio user requests a particular sample rate _f<sub>s</sub>_, the software will find the decimation factor _N_ required to achieve that sample rate. Since the sample rate coming out of the ADC is 100 MSps,

$$ N = \frac{100 \times 10^6}{f_s} $$


The configuration of filters limits the decimation factors that may be realized in the FPGA: we can achieve integer decimation factors from 1 through 128, even values from 128 through 256, and values from 256 to 512 that are factors of four.  So in practice, we may not achieve exactly _f<sub>s</sub>_, but only some value close to it.

Then, the number of halfband filters used, and in turn the "flatness" of the [passband](https://en.wikipedia.org/wiki/Passband), depends on the value of _N_:

* If _N_ is odd, no halfband filters are used. 
* If _N_ is even, one halfband filter is used, and the response is slightly flatter.
* If _N_ is a factor of four, two halfband filters are used, and this gives the flattest response.

We may choose the sampling rate deliberately to achieve a flat response (by choosing a sampling rate _f<sub>s</sub>_ so that 100e6/_f<sub>s</sub>_ is a factor of four, or at least a factor of two). 

In the mailing list questions referenced above, the first user experienced a "droopy" signal at a 20 MSps sampling rate because 100e6/20e6=5 is odd. When this user requested a sampling rate of 300 kSps, the nearest realizable sampling rate was 301.2 kSps with a decimation factor of 332, which is factor of 4 and gives a much flatter response. The next user saw a flat response for 10 MSps (100e6/10e6=10, which is even) but a "droopy" response for 9 MSps (odd decimation factor, 11) and 11 MSps (odd decimation factor, 9).

## Results

We will see "spectral droop" in a display of received ambient noise when we use a sampling rate of 14.285714 MSps. With this configuration, the USRP N210 will use only a CIC filter to achieve the decimation rate of 100e6/14.285714e6=7, and so there will be significant passband rolloff. The received signal (which should be flat, for an ideal receiver) seems to droop down at the edges:

![](/blog/content/images/2016/06/7-rolloff-1.gif)

We will see a much flatter spectrum in a display of received ambient noise when we use a sampling rate of 12.5 MSps. With this configuration, the USRP N210 will use two halfband filters after the CIC to achieve the decimation rate of 100e6/12.5=8, and so there will be much less rolloff:

![](/blog/content/images/2016/06/8-norolloff-1.gif)


## Run my experiment

Make a reservation on one of the SDR sandboxes on the [ORBIT testbed](http://geni.orbit-lab.org/). We wrote these instructions for sb3, but sb7 and sb2 also have USRP N210 devices.

During the reserved time slot, SSH into your testbed, e.g. `sb3.orbit-lab.org`, using your GENI wireless username and keys.

Once you are on the testbed console, load a baseline SDR disk image onto a node with the command:

```
omf load -i baseline-sdr.ndz -t node1-1.sb3.orbit-lab.org
```

If you're using something other than sb3, modify the command above accordingly. 

When this process finishes, turn on the node using the command (again, modified for sb2 or sb7 if necessary):

```
omf tell -a on -t node1-1.sb3.orbit-lab.org
```

and wait a couple of minutes for it to turn on. Then, SSH in to the node with:

```
ssh root@node1-1
```

The baseline SDR disk image already has the UHD drivers and GNU Radio software installed.

Make sure your node recognizes the USRP device attached to it by running

```
uhd_find_devices
```

The output should look like e.g.

```
--------------------------------------------------
-- UHD Device 0
--------------------------------------------------
Device Address:
    type: usrp2
    addr: 192.168.10.2
    name: 
    serial: F297B6
```

(potentially with a different address and serial number).

To observe spectral droop (in this case, just in the received ambient noise), we'll use an ASCII art DFT receiver that is distributed along with the UHD drivers.

First, we'll run it with a sampling rate of 14.285714 MSps, i.e. a decimation factor of 7. With this configuration, only the CIC filter is used and we expect to see spectral "droop" due to the passband rolloff. To try it, run

```
/usr/local/lib/uhd/examples/rx_ascii_art_dft --freq 700e6 --gain 30 --rate 14.285714e6 --frame-rate 15 --ref-lvl -50 --dyn-rng 50
```

(note that the command above is all one line). You should start to see the observed spectrum. You may have to tune the reference level and/or dynamic range in order to see the noise floor.

Observe the spectrum with the sampling rate of 14.285714 MSps, then press Ctrl+C to stop.

Next, we'll run it with a sampling rate of 12.5 MSps, i.e. a decimation factor of 8. With this configuration, two halfband filters follow the CIC filter, and so we expect a much flatter response. Try it with

```
/usr/local/lib/uhd/examples/rx_ascii_art_dft --freq 700e6 --gain 30 --rate 12.5e6 --frame-rate 15 --ref-lvl -50 --dyn-rng 50
```

Observe the spectrum with the sampling rate of 12.5e6 MSps, then press Ctrl+C to stop.


## Notes

The results shown here were with a USRP N210r4 software radio device with SBXv3 daughterboard