The aim of this experiment is to explain the "frequency offset" described in many questions on the GNU Radio and USRP mailing lists. 

It should take about 30 minutes to run this experiment, but you will need to have reserved that time in advance. This experiment uses wireless resources (specifically, the sb7 sandbox at [ORBIT](http://geni.orbit-lab.org)), and you can only use wireless resources on GENI during a reservation.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). The project lead of the project you belong to must have [enabled wireless for the project](https://portal.geni.net/secure/wimax-enable.php). Finally, you must have reserved time on the sb7 sandbox at [ORBIT](http://geni.orbit-lab.org) and you must run this experiment during your reserved time.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

Members of the usrp-users and discuss-gnuradio mailing lists are often puzzled by an observed frequency offset in their signal. For example, [this question by Bruhtesfa Ebrahim](https://lists.gnu.org/archive/html/discuss-gnuradio/2008-11/msg00358.html) asks:

> I am using USRP with two XCVR2450 Trancievers (working at 2.4-2.5GHz and 4.9-5.0GHz).
I generated a carrier at 2.45GHz from the USRP using
>  
>  **$ python usrp_siggen.py -TA -f2.45e9**
> 
> But when i see the generated signal in a spectrum analyzer, i see that  it is shifted by 12.5KHz from the center frequency( i see the carrier at 2.45GHz minus 12.5KHz). I try for other carrier frequencies but the shift remain almost the same.
> 
> Also, I generated a carrier of 2.45GHz from a signal generator , received  it in the usrp and I observed the spectrum.  Again I see that the signal that is demodulated by the USRP and received in the computer through the USB is not a baseband signal as expected. Rather it is a carrier of about 12.5KHz.
> 
> This means their is a fequency shift of about 12.5KHz both in transmission and in reception.When I use a carrier of frequency 4.9-5.0GHz, the frequency shift almost doubles to 25KHz.
Does anyone experienced such a problem before?
So, What can i do to correct this frequency shift?  

The frequency offset described here is often due to the frequency tolerance of the crystal oscillator used as the timing reference in the radio. In the USRP2 and USRP N210, for example, a 100 MHz crystal oscillator acts as the reference for a [phase locked loop frequency synthesizer](https://en.wikipedia.org/wiki/Frequency_synthesizer).

However, the crystal oscillator used as the frequency reference is not perfect;  while it is supposed to oscillate at 100 MHz, its frequency may deviate from the nominal value. The maximum amount by which it is expected to deviate is expressed in parts per million (ppm). A tolerance of &plusmn;20 ppm, for example ([as in the USRP2](http://gnuradio.org/redmine/projects/gnuradio/wiki/USRP2GenFAQ#What-is-the-USRP2-reference-clock-stability)), means that the oscillator may actually deviate from the nominal frequency of 100 MHz by up to 100e6*20/1e6 = 2 kHz in either direction.

While the tolerance may be small when expressed in ppm, this offset is also multiplied whenever the reference frequency is. For example, if the frequency synthesizer multiplies the 100 MHz reference by 40 to create a signal that is nominally at 4 GHz, the actual generated signal may actually deviate from the nominal 4 GHz frequency by as much as 80 kHz! Thus, the accuracy of the reference oscillator becomes especially important for high carrier frequencies. Furthermore, because the transmitter and receiver can potentially have opposite worst-case errors, system designers must expect the total difference between the receiver and transmitter to be up to 160 kHz.


## Results

The following series of images show the received signal on a USRP2, when a tone is transmitted from another USRP2 at 750, 1250, 1750, 2250, 2750, 3250, 3750, and 4250 MHz respectively.

For a transmitted signal at 750 MHz, the received signal is close to the expected center frequency. However, as the center frequency increases, the effect of the frequency offset increases as well, with the total offset at 4.25 GHz clocking in at over 50 kHz.

![Transmitted signal at 750 MHz](/blog/content/images/2016/08/offset-750.png)

![Transmitted signal at 1250 MHz](/blog/content/images/2016/08/offset-1250.png)

![Transmitted signal at 1750 MHz](/blog/content/images/2016/08/offset-1750.png)

![Transmitted signal at 2250 MHz](/blog/content/images/2016/08/offset-2250.png)

![Transmitted signal at 2750 MHz](/blog/content/images/2016/08/offset-2750.png)

![Transmitted signal at 3250 MHz](/blog/content/images/2016/08/offset-3250.png)

![Transmitted signal at 3750 MHz](/blog/content/images/2016/08/offset-3750.png)

![Transmitted signal at 4250 MHz](/blog/content/images/2016/08/offset-4250.png)

The USRP2 has an oscillator rated at [20 ppm](http://gnuradio.org/redmine/projects/gnuradio/wiki/USRP2GenFAQ#What-is-the-USRP2-reference-clock-stability). The more recent USRP N210 has a TXCO rated at [2.5 ppm](https://www.ettus.com/content/files/07495_Ettus_N200-210_DS_Flyer_HR_1.pdf). The following image shows the received signal at one USRP N210, when another USRP N210 transmits at 4250 MHz:

![](/blog/content/images/2016/08/usrpn210-offset-1.png)

We can see that even at this high frequency, there is very little offset with the more accurate oscillator. (Of course, some USRP2 devices will also have very little offset; the tolerance rating describes the _maximum_ error a device may have, but an individual device may have much less.)

## Run my experiment

Make a reservation on sandbox 7 on the [ORBIT testbed](http://geni.orbit-lab.org/). This sandox currently has two nodes, each with a USRP2 device and SBX daughterboard.

During the reserved time slot, SSH into your testbed - `sb7.orbit-lab.org` - using your GENI wireless username and keys.

Once you are on the testbed console, load a baseline SDR disk image onto the nodes with the command:

```
omf load -i baseline-sdr.ndz -t system:topo:all
```


When this process finishes, turn on the node using the command:

```
omf tell -a on -t system:topo:all
```

and wait a few minutes for the nodes to boot up.

Then, SSH in to one node with:

```
ssh root@node1-1
```

and into the other with

```
ssh root@node1-2
```

(in another terminal).

The baseline SDR disk image already has the UHD drivers and GNU Radio software installed.

Make sure each node recognizes the USRP device attached to it by running

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

To observe the received signal, we'll use an ASCII art DFT receiver that is distributed along with the UHD drivers. To generate the signal, we'll use a waveform generator that is also distributed along with the UHD drivers. Choose one node to act as transmitter and the other to act as receiver.


To generate the 750 MHz plot, I used the following command on the transmitter side,

```
/usr/local/lib/uhd/examples/tx_waveforms --freq 0.75e9 --rate 200e3 --ampl 0.7 --gain 20
```

and on the receiver side,

```
/usr/local/lib/uhd/examples/rx_ascii_art_dft --rate 200e3 --freq 0.75e9 --gain 10 --ref-lvl -30 --dyn-rng 70 
```

You may have to adjust the reference level and dynamic range to see the complete signal.

To generate the other plots, I varied the `freq` argument in both of the commands above. I used the values: 0.75e9, 1.25e9, 1.75e9, 2.25e9, 2.75e9, 3.25e9, 3.75e9, 4.25e9.

To generate the plot for the USRP N210 (with 2.5 ppm frequency tolerance), I repeated the steps above for sandbox 3, which has two nodes with USRP N210 devices attached.

## Notes

The results shown here were with a USRP2 r3 motherboard with SBXv3 daughterboard. (Use `uhd_usrp_probe` to find out what motherboard and daughterboard versions you are using.)