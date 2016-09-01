In this experiment, you'll experience what it is like to use the Internet as a user from an emerging market with poor Internet connectivity. 

It should take about 60 minutes to run this experiment.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

Developers and engineers who create Internet services for others to use typically connect to the Internet themselves with a high-quality connection. As a result, they may not fully understand the experience of their users, many of whom are in developing countries or underserved areas and do not have access to fast broadband. To better serve these users, developers and engineers need to test their services on a range of network conditions. 

As described in a [Facebook Engineering blog post](https://code.facebook.com/posts/1561127100804165/augmented-traffic-control-a-tool-to-simulate-network-conditions/):

> When we launch a new app or feature on Facebook, we usually do it from a powerful wireless network at our offices in North America. These networks are typically what we use to test and build services. However, for a large percentage of people around the world, accessing Facebook requires connecting to a slower, less reliable wireless network. We want as many people as possible to be able to access our services at their full potential. So we need to be able to test our features on wireless connections that more accurately reflect the types of connections people on Facebook will ultimately use.


To fill this need, Facebook developed a tool called Augmented Traffic Control (ATC). It can be used to test an application across a variety of networks, from high-speed home networks to severely impaired wireless networks in developing countries.


By testing on network conditions that reflect the full variety of networks seen "in the wild", [Facebook engineers found that they were able to design better applications](https://code.facebook.com/posts/1561127100804165/augmented-traffic-control-a-tool-to-simulate-network-conditions/):

> The other teams that used ATC found that it helped save time and pointed out weaknesses in their work. The team working on an update to Messenger that would make the app run faster and more reliably used ATC to check how things would run when the network connection wasn’t as strong. They ran automated tests in which the bandwidth connections were changed in order to look at how long Messenger should wait before timing out and how many retries were optimal to send messages. This information allowed the Messenger team to choose a selection of small A/B tests to run. By zeroing in on where the weaknesses were in different networks, they were able to more quickly identify problems and save development time. The Messenger team also uses ATC APIs in automation to measure call quality on different network configurations.

Facebook also took steps to more generally bridge the "empathy gap" between developers at Facebook, and users in emerging markets with poor Internet connectivity, with an initiative called "2G Tuesdays". As they announced in a [blog post](https://code.facebook.com/posts/1556407321275493/building-for-emerging-markets-the-story-behind-2g-tuesdays/):


> Today we're taking another step toward better understanding by implementing “2G Tuesdays” for Facebook employees. On Tuesdays, employees will get a pop-up that gives them the option to simulate a 2G connection. We hope this will help us understand how people with 2G connectivity use our product, so we can address issues and pain points in future builds.


Facebook released ATC as an open source tool so that other developers and researchers can test their applications over realistic network conditions. For example, Jason Ernst, Chief Networking Scientist at Internet company [Left](http://left.io/) wrote about [setting up a dedicated ATC router](https://medium.com/@csgrad/turning-a-netgear-r7000-into-an-augmented-traffic-control-atcd-router-f0c9db861fd7#.6clc456nt) and commented: 



> At work we’re developing apps that are being used in developing countries and half of the office works out of Vancouver where our networks are very good. Unfortunately, this means that we often don’t think about user experience problems and bugs that occur only when the app is operating with a poor quality connection.
> 
> To combat this, we are taking motivation from Facebook, which offers “2G Tuesdays” to employees so they can experience what it’s like for people in other parts of the world. Facebook also released a tool call Augmented Traffic Control which allows you to simulate these types of conditions with your own equipment.


In this experiment, you'll set up ATC and connect it to your local browser so you, too, can experience the Internet from the perspective of a user of a 2G or other non-ideal network.


## Results

In this experiment, we experience the Internet as a user in a developing market might.

For example, here is what we would see in real time as an [EDGE](https://en.wikipedia.org/wiki/Enhanced_Data_Rates_for_GSM_Evolution) network user trying to load the Google homepage:

<iframe width="420" height="315" src="https://www.youtube.com/embed/VG2tfQC5Rg8" frameborder="0" allowfullscreen></iframe>

For comparison, here is what we could see as a user of a cable network:

<iframe width="560" height="315" src="https://www.youtube.com/embed/pHLfSK7fsNU" frameborder="0" allowfullscreen></iframe>

## Run my experiment

For this experiment, we will set up a topology in GENI that involves an OpenVPN server and a Squid HTTP proxy server, connected by a link:

![](/blog/content/images/2016/08/2g-tuesday-1.svg)

Then, will will configure the local host from which we run the experiment (e.g. a laptop) as follows:

1. Connect to the OpenVPN server.
2. Configure our browser to use the Squid proxy for web traffic.
3. Route traffic to the Squid proxy through the VPN tunnel.

Web traffic will go from the laptop browser, through the VPN tunnel, over the link between OpenVPN and Squid - where we will shape the traffic - and then on to the Internet. The reverse traffic will follow the same path in reverse.

First, create a new slice in the GENI Portal, and then click "Add Resources". Load the Rspec in [this gist](https://gist.github.com/ffund/2f43862ceebef6596b538112db75029c) from the URL: [https://git.io/viUtO](https://git.io/viUtO) 

(You can load the Rspec directly from the URL in the "Choose Rspec" area.) This will load the following topology in your canvas:

![](/blog/content/images/2016/08/2g-tuesdays-canvas-1.png)

Click on "Site 1", choose an InstaGENI site to bind to. Try to choose a site that is close to your location: you want the Internet link you experience to be shaped by ATC, not bottlenecked by the link between you and GENI. Click "Reserve Resources". Wait until both nodes are ready to log in.

First, we will set up a VPN server on the ATC gateway. Log in to the "openvpn" node and run

```
openvpn --genkey --secret static.key
```

This will create a file called `static.key`. Use `scp` to copy this file from the node to your local host (e.g. the laptop from which you are running this experiment). The syntax is:

```
scp -P PORT GENI-USERNAME@HOSTNAME:~/static.key .
```

to download `static.key` to the directory from which you run the `scp` command, and you replace `PORT`, `GENI-USERNAME`, and `HOSTNAME` with the values provided by the Portal for your reserved nodes.

We need to make sure the OpenVPN instance will tunnel traffic to the proxy, so on "openvpn"  run:

```
sudo sysctl -w net.ipv4.ip_forward=1
```

Then, on the "openvpn" node, create a file called `server.ovpn` with the following contents:

```
dev tun
ifconfig 10.8.0.1 10.8.0.2
secret static.key
```

and run

```
sudo openvpn server.ovpn
```

to start the OpenVPN server. Leave this running.

Next, we're going to use OpenVPN on your local host (e.g. laptop) and connect to this VPN server. OpenVPN is available for a variety of platforms; find the appropriate installation instructions for your platform and install it. 

The instructions that follow are for Ubuntu Linux, but you can modify them to work on any other platform:

On your local host (e.g. laptop), create a file `client.ovpn` in the same directory to which you downloaded `static.key`, with the following contents:

```
remote HOSTNAME
dev tun
ifconfig 10.8.0.2 10.8.0.1
secret static.key
route 10.91.1.0 255.255.255.0
```

but replace the word `HOSTNAME` with the actual hostname of the "openvpn" node on GENI (provided by the portal). For example, in the following image, the hostname is `openvpn.realistic-atc.ch-geni-net.genirack.nyu.edu`:

![](/blog/content/images/2016/08/2g-hostname.png)

Then run

```
sudo openvpn client.ovpn
```

and verify that you can connect to the VPN.

Finally, log in to the "proxy" node and run

```
sudo route add -net 10.8.0.0/24 gw 10.91.1.1
```

so that the proxy will route traffic destined for the VPN tunnel through the VPN server. 

Now, on your client, verify that you can reach both the VPN server and the proxy server:

```
ping 10.8.0.1
ping 10.91.1.2
```

We will use [squid](http://www.squid-cache.org/) as our HTTP proxy. On the "proxy" node and set up squid:

```
sudo apt-get update
sudo apt-get -y install squid3
```

Download the squid configuration file from [this gist](https://gist.github.com/ffund/c5f5ba86b2c49898549b8c4e01ede418) with the following command:

```
sudo wget --output-document=/etc/squid3/squid.conf https://git.io/v6mej
```

Then run:

```
sudo service squid3 restart
```

to restart squid with the new configuration. 

Finally, in your browser, tell it to use 10.91.1.2 (port 3128) as an HTTP and HTTPS proxy. In Firefox, you would open the menu, click on the Advanced tab, and then click on the Network tab, and set up proxy settings like this:

![](/blog/content/images/2016/08/firefox-proxy.png)

You can find similar instructions for other browsers online.


To verify that your browser is using the proxy, visit http://amibehindaproxy.com/ and make sure it shows "Your IP address" as 10.8.0.2, and that you are using a proxy (the proxy address and name will vary depending on what InstaGENI site you are using).

![](/blog/content/images/2016/08/2g-proxy.png)


Now, we will set up ATC on the gateway node according to the instructions [here](https://github.com/facebook/augmented-traffic-control) and [here](http://facebook.github.io/augmented-traffic-control/).

Open a new connection to the "openvpn" node, and install software and get the utility scripts with:

```
sudo apt-get update
sudo apt-get -y install curl python-pip python-dev=2.7.5-5ubuntu3 vim

sudo pip install atc_thrift atcd django-atc-api django-atc-demo-ui django-atc-profile-storage

# Fix packages that were installed with unsupported versions
# as per https://github.com/facebook/augmented-traffic-control/issues/253
sudo pip install -U django-bootstrap-themes==3.1.3
# as per http://quabr.com/38382447/django-rest-framework-import-error
sudo pip install -U djangorestframework==3.3.3
# this one should go last
sudo pip install -U django==1.7

git clone https://github.com/facebook/augmented-traffic-control.git
```

Now we will set up the ATC web interface:

```
django-admin startproject atcui
cd atcui
```

Edit the file `atcui/settings.py` and add to `INSTALLED_APPS` so that it looks like this:

```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Django ATC API
    'rest_framework',
    'atc_api',
    # Django ATC Demo UI
    'bootstrap_themes',
    'django_static_jquery',
    'atc_demo_ui',
    # Django ATC Profile Storage
    'atc_profile_storage',
)
```

and edit `atcui/urls.py` so it looks like this:

```
from django.conf.urls import patterns, include, url
from django.contrib import admin
from django.views.generic.base import RedirectView
from django.conf.urls import include


urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    # Django ATC API
    url(r'^api/v1/', include('atc_api.urls')),
    # Django ATC Demo UI
    url(r'^atc_demo_ui/', include('atc_demo_ui.urls')),
    # Django ATC profile storage
    url(r'^api/v1/profiles/', include('atc_profile_storage.urls')),
    url(r'^$', RedirectView.as_view(url='/atc_demo_ui/', permanent=False)),
)
```

Then update the Django DB:

```
python manage.py migrate
```

and start it with

```
python manage.py runserver 0.0.0.0:8000
```

Finally, we'll set up some prepared network profiles. Open a third connection to "openvpn", and run:

```
cd ~/augmented-traffic-control/utils/
bash restore-profiles.sh localhost:8000
```

You should see some output similar to this:

```
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/2G-DevelopingRural.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/2G-DevelopingUrban.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/3G-Average.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/3G-Good.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/Cable.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/DSL.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/Edge-Average.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/Edge-Good.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/Edge-Lossy.json
Added profile /users/ffund01/augmented-traffic-control/utils/profiles/NoConnectivity.json
```

Then start the `atcd` daemon with

```
sudo atcd --atcd-wan eth1 --atcd-lan tun0
```

For the next steps, you'll need another browser, one _not_ using a proxy (e.g.: if you set up a proxy in Firefox, use Chrome for this next part). In this browser, visit http://10.8.0.1:8000

Click to expand the "Authentication" section. You should see your own IP address listed as 10.8.0.2, i.e. your IP address on the VPN:

![](/blog/content/images/2016/08/atcui-auth-1.png)

At the top of the page, click on the "Turn On" button:

![](/blog/content/images/2016/08/atcui-turnon.png)

It should turn into a red "Turn Off" button, indicating that traffic shaping is now on:

![](/blog/content/images/2016/08/atcui-turnoff.png)

Now, we'll select the network profile to use. If you expand the "Profile" section, you should see a list of network profiles:

![](/blog/content/images/2016/08/atcui-profiles.png)

Click "Select" next to the profile you want to try - for example, try "EDGE - Good". Then, scroll back to the top and click on "Update Shaping".

Now, go to the browser that you set up to use a proxy, and try browsing the Internet as you normally would, but as an EDGE user. For example, trying to load the Google homepage might look like this:


<iframe width="420" height="315" src="https://www.youtube.com/embed/VG2tfQC5Rg8" frameborder="0" allowfullscreen></iframe>

When you're finished, release your resource in the GENI Portal to free them for use by other experimenters!

## Notes

Traffic shaping at high data rates may not work very well with a virtual machine. For this reason, we use a Raw PC for the "openvpn" node. However, because a Raw PC is a scarce resource, it may be somewhat difficult to successfully reserve your topology.