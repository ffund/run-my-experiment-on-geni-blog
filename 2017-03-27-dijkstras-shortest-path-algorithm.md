In this experiment, we will use Dijkstra's algorithm to find the shortest path from one node in a six-node topology, to all other nodes. We will then install routing rules at each node to implement the shortest-path tree produced by Dijkstra's algorithm. 

It should take about 120 minutes to run this experiment.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.


* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Results

In executing Dijkstra's algorithm, we found the following individual link costs (with round-trip latency as the cost metric):

![](/blog/content/images/2017/03/dijkstra-topology-costs.svg)

and produced the following table:

![](/blog/content/images/2017/03/dijkstra-table.svg)

The procedure is shown in the following video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FLr3MwhyO6A" frameborder="0" allowfullscreen></iframe>

From this table, we found the shortest-path tree. Here,the links that are part of the shortest-path tree are shown in red, and the links that are not are in grey:

![](/blog/content/images/2017/03/dijkstra-topology-routes.svg)

We then installed routes so that traffic from the source node would follow the paths recommended by Dijkstra's algorithm: 

<iframe width="560" height="315" src="https://www.youtube.com/embed/uqgBVHHFtLM" frameborder="0" allowfullscreen></iframe>

Finally, we verified that the routes and the total path cost from the source node to every other node are consistent with our expectations:

<iframe width="560" height="315" src="https://www.youtube.com/embed/G5G1oWy2Z9c" frameborder="0" allowfullscreen></iframe>


## Run my experiment

In this experiment, we will find the shortest path (in terms of latency) from the node named "dijkstra" to each of the other nodes (all named for famous engineers or computer scientists) in this topology:

![](/blog/content/images/2017/03/dijkstra-topology.svg)


You may reserve this topology in either of two ways:

* You can reserve resources that are distributed across GENI sites around the United States. It is more difficult to reserve resources this way, but it may be more interesting - the latency measurements will reflect actual distances between sites. To try this method, see [Using multiple GENI sites](#usingmultiplegenisites).
* You can reserve resources at a single site. In this case, an artificial latency will be applied to each link in the topology. This is an easier way to reserve resources. To try this method, see [Using one site](#usingonesite).


### Using multiple GENI sites

Reserving an experiment topology that uses resources from multiple GENI sites can be a little more complicated than experiments that only use resources at one GENI site. In order for everything to work,

* The resource reservation request must succeed, i.e. when you submit your resource request in the GENI Portal it returns "Status: Finished" indicating that all of the aggregates involved have indicated that they can satisfy your request.
* The VMs you have requested must come up (turn "green" on the slice page), indicating that they are ready to log in.
* The links _between_ aggregates, which are created using a process called [Stitching](#http://groups.geni.net/geni/wiki/GeniNetworkStitchingSites), must be operational. 

If there is a failure at _one_ GENI aggregate for any of these steps, the entire thing will fail; you need each of these steps to succeed at _all_ the GENI aggregates in your slice. Thus, you'll need some extra patience and willingness to try again. In this section, I will describe how to identify which aggregate is responsible for the failure of a slice, so that you can exclude that aggregate when you try again.

First, in the GENI Portal, create a new slice, and then click "Add Resources". Scroll down to the "Choose RSpec" section and select "URL"; enter the URL [https://git.io/vSqcn](https://git.io/vSqcn) and choose "Select" to load the multi-site topology:

![](/blog/content/images/2017/03/dijkstra-multisite.png)

You will notice that this topology includes multiple sites. To replace any site (e.g. if you have problems with a particular site), you can click on the name of the site, then use the menu on the left to select a different InstaGENI site. You should choose from InstaGENI sites that support multi-site links ("stitching") - there's a list [here](http://groups.geni.net/geni/wiki/GeniNetworkStitchingSites#GENINetworkStitchingSites).

You may also notice that some links are marked with a red warning symbol; you can safely ignore this warning.

When you click the button to reserve resources,  you may notice that the resource reservation takes longer than usual, since it must be coordinated between six different aggregates. To keep an eye on the process, click on "Detailed Progress". If all goes well, the output should include something this, that shows you when the allocation is finished at each of the six sites in your topology:

```
16:43:51 INFO    : Multi-AM reservation will include resources from these aggregates:
16:43:51 INFO    : 	<Aggregate ohmetrodc-ig>
16:43:51 INFO    : 	<Aggregate rutgers-ig>
16:43:51 INFO    : 	<Aggregate nyu-ig>
16:43:51 INFO    : 	<Aggregate missouri-ig>
16:43:51 INFO    : 	<Aggregate nps-ig>
16:43:51 INFO    : 	<Aggregate ucla-ig>
16:43:51 INFO    : Stitcher doing createsliver at <Aggregate ohmetrodc-ig>...
16:44:07 INFO    : ... Allocation at <Aggregate ohmetrodc-ig> complete.
16:44:07 INFO    : Stitcher doing createsliver at <Aggregate rutgers-ig>...
16:44:23 INFO    : ... Allocation at <Aggregate rutgers-ig> complete.
16:44:23 INFO    : Stitcher doing createsliver at <Aggregate nyu-ig>...
16:44:39 INFO    : ... Allocation at <Aggregate nyu-ig> complete.
16:44:39 INFO    : Stitcher doing createsliver at <Aggregate missouri-ig>...
16:44:57 INFO    : ... Allocation at <Aggregate missouri-ig> complete.
16:44:57 INFO    : Stitcher doing createsliver at <Aggregate nps-ig>...
16:45:19 INFO    : ... Allocation at <Aggregate nps-ig> complete.
16:45:19 INFO    : Stitcher doing createsliver at <Aggregate ucla-ig>...
16:45:54 INFO    : ... Allocation at <Aggregate ucla-ig> complete.
16:45:54 INFO    : All aggregates are complete.
```

If a request fails at a particular site, you can determine from the output which aggregate it was. You should then delete the resources in this slice (this, too, may take a few minutes longer than usual, since it has to delete resources at multiple aggregates). Create a new slice, and try your request in the new slice, _without_ the site that failed - replace that site in the topology before you reserve resources again.

Once the resource request succeeds, you should wait for your nodes to become available to log in. If a resource fails to come up (e.g. it shows as "Failed" in the Portal, you get an email alerting you that the node has failed to boot, or it remains in an "Unknown" state for a very, very, very long time), you should delete the resources in your slice. Then, try again, using a different aggregate in place of the one that just failed.

If _all_ nodes in your topology turn green, indicating that they are ready to log in, then you are ready for the last check - making sure the links between sites are operational. 

Wait until _all_ of your nodes are ready to log in. Then SSH into each of your nodes. When you log in, you will see a message indicating which of its direct neighbors are reachable. If one site is not connected, then you will see something like this example, which shows that all of the links to "lovelace" are down:

![](/blog/content/images/2017/03/dijkstra-linksdown.png)

If a link is down, you should delete the resources in this slice and try again. (In this case, since all of the links that are down are links to "lovelace", it would be advisable to avoid whatever GENI site "lovelace" is on in your next attempt.)

You can also find out the geographical location of each node by running

```
wget -qO- http://ipinfo.io
```

on each node. This hits a [website](http://ipinfo.io/) that [geolocates](https://en.wikipedia.org/wiki/Geolocation) the node based on its public IP address, and prints the output in the terminal.

When you're confident that your topology is ready to go, continue to [Run Dijkstra's algorithm](#rundijkstrasalgorithm).

### Using one site

Alternatively, you may prefer to reserve resources at only one GENI site.

In the GENI Portal, create a new slice, and then click "Add Resources". Scroll down to the "Choose RSpec" section and select "URL"; enter the URL [https://git.io/vSCdF](https://git.io/vSCdF) and choose "Select". This will load a six-node topology in your canvas.

Click on "Site 1" and select an InstaGENI site from the drop-down list on the left, then click "Reserve Resources". Wait until all of your nodes are up and ready to log in, then open six terminals and SSH into each node.



### Run Dijkstra's algorithm

We now have a topology with six nodes, and IP addresses defined on each as shown in the following image:

![](/blog/content/images/2017/03/dijkstra-topology-ips-2.svg)

Next, we will find the shortest path from the node named "dijkstra" to all other nodes, using Dijkstra's algorithm. 

In this section, I show how we would execute this algorithm on _my_ experiment topology - when _you_ run this experiment, you will have different link costs, and will have to modify the procedure accordingly (i.e. visit nodes in a different order than I did). 

For your convenience, you can use [this table](https://docs.google.com/spreadsheets/d/1d98qzdzui3sfPJlBrcXQ8vz6YT8r46Urc7wjpEJk3K0/) and fill it in as you work. Visit the link and click on "File > Make a Copy" to make an editable copy in your own Google Drive, or use "File > Download As" to download a copy to your computer and edit in a spreadsheet application. 

In the first iteration of Dijkstra's algorithm, we will visit the "dijkstra" node. From the figure above, we can identify each of the direct neighbors of the "dijkstra" node and their IP address on the link that they share with "dijkstra":

* "lovelace" on 10.10.4.1
* "baran" on 10.10.5.2


To get the cost of the link from "dijkstra" to "lovelace", we will ping "lovelace" on its 10.10.4.1 IP address 25 times, and take the average latency as the cost. From "dijkstra", run:

```
ping -c 25 10.10.4.1
```

Note the _average_ round trip time reported by the `ping` command, e.g. 32.742 ms in the example below:

<pre>
--- 10.10.4.1 ping statistics ---
25 packets transmitted, 25 received, 0% packet loss, time 24045ms
rtt min/avg/max/mdev = 32.272/<b>32.742</b>/33.986/0.448 ms
</pre>

In the table, fill in this value as the cost of the path from "dijkstra" to "lovelace", and note that the previous hop in this path is "dijkstra".

Then, repeat this step for "lovelace". From the "dijkstra" node, run:

```
ping -c 25 10.10.5.2
```

and record the result. 

The other entries in the table remain unchanged, so we can copy them from the previous row. Also, since we have already visited "dijkstra", we know that it will not change in future iterations, so we can copy its value all the way down the table. 

At the end of iteration 1, our table looks like this:

![](/blog/content/images/2017/03/dijkstra-iteration1.svg)

To choose the next node to visit, we compare the costs to each of the remaining unvisited nodes. Here, 

$$ 32.742 < 71.918 < \infty $$

so on the next iteration we visit "lovelace". Its direct _unvisited_ neighbors are: 

* "knuth" at 10.10.8.2
* "hopper" at 10.10.3.1

So from "lovelace", we will run `ping` commands to each of those nodes. In the table, we will record the cost of the _total_ path from "dijkstra" to each of "knuth" and "hopper":

* We measure 39.809 from "lovelace" to "knuth", for a total cost of 32.742+39.809=72.551. This is less than the previous (infinite) cost of the path to "knuth", so we update the table to note 72.551 as the cost to "knuth" and "lovelace" as the previous hop.
* We measure 24.254 ms from "lovelace" to "hopper", for a total cost of 32.742+24.254=56.996. This is less than the previous (infinite) cost of the path to "hopper", so we update the table to note 56.996 as the cost to "hopper" and "lovelace" as the previous hop

The other entries in the table remain unchanged, so we copy them from the previous row. Also, now that we have visited "lovelace", its value will not change again and I can copy it all the way down the table. After iteration 2, our table looks like this:

![](/blog/content/images/2017/03/dijkstra-iteration2.svg)

To find the next node to visit next, I compare the cost of the path to each of the remaining unvisited nodes. Since

$$ 56.996 < 71.918 < 72.551 < \infty $$

we will visit "hopper" next. 

After six iterations following a similar procedure, we have produced the following table:

![](/blog/content/images/2017/03/dijkstra-table.svg)

and measured these costs on each link:

![](/blog/content/images/2017/03/dijkstra-topology-costs.svg)

The following video shows how we executed Dijkstra's algorithm on this six-node topology:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FLr3MwhyO6A" frameborder="0" allowfullscreen></iframe>

### Route according to the shortest-path tree

Next, we will use the table to find the shortest-path tree rooted at the "dijkstra" node. This is a version of the topology that has no cycles (closed loops), and where the path from the "dijkstra" node to each other node is a minimum path.

We can read the tree off the table as follows. For each destination node (each column), we refer to the final row to find the node that is "previous hop" to that destination node. The link between them is part of the shortest-path tree.

For example, in my topology (your experiment may come out different):

* The last hop before "cerf" is "knuth", so the link between "cerf" and "knuth" is in the shortest-path tree.
* The last hop before "lovelace" is "dijkstra", so the link between "lovelace" and "dijkstra" is in the shortest-path tree.
* The last hop before "hopper" is "lovelace", so the link between "hopper" and "lovelace" is in the shortest-path tree.
* The last hop before "baran" is "dijkstra", so the link between "baran" and "dijkstra" is in the shortest-path tree.
* The last hop before "knuth" is "lovelace", so the link between "knuth" and "lovelace" is in the shortest-path tree.

We can now mark each link as either being part of the shortest path tree (red) or not (grey):

![](https://witestlab.poly.edu/blog/content/images/2017/03/dijkstra-topology-routes.svg)

Furthermore, we can mark each network interface as being part of the shortest path tree if they are on a "red" link, and not part of the shortest path tree if they are on a "grey" link.

To realize the shortest path tree on our topology, we will bring down each interface that is on a "grey" link. For every "grey" interface on your topology, SSH into the terminal of the node it is attached to. Run

```
ifconfig
```

to find out the name (e.g. `eth1`, `eth2`) of the interface with that IP address. Then bring down that interface with:

<pre>
sudo ifconfig <b>eth1</b> down
</pre>

where in place of the bold part above, substitute the name of interface that you have found in the previous step. For example, in my experiment I would bring down

* The interface on "hopper" with IP address 10.10.6.1
* The interface on "knuth" with IP address 10.10.6.2
* The interface on "hopper" with IP address 10.10.2.1
* The interface on "cerf" with IP address 10.10.2.2
* The interface on "baran" with IP address 10.10.1.2
* The interface on "cerf" with IP address 10.10.1.1


(Make sure to bring down each link that is not part of the shortest-path tree on _both_ sides of the link.)

Finally, for links and interfaces that _are_ part of the shortest-path tree, we will set up routing rules so that traffic will follow the shortest path.

We will see two kinds of routing rules:

1. Routes for traffic that is going to a local network (i.e. not traversing any router, which say what interface the traffic should be sent out of.
2. Routes for traffic that is _not_ going to a local network, which say what "next hop" IP address the traffic should be sent through. The "next hop" IP address _must_ be on the local network.

The first kind of rule is already set up for each interface on each node. (Without these rules, we would not have been able to send "ping" traffic between nodes in the first parts of this experiment.) If you run 

```
route -n
```

on any node, you'll see all of its routing rules. For example, on "dijkstra" I might see

<pre>
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.16.0.1      0.0.0.0         UG    0      0        0 eth0
<b>10.10.4.0       0.0.0.0         255.255.255.0   U     0      0        0 eth1</b>
<b>10.10.5.0       0.0.0.0         255.255.255.0   U     0      0        0 eth2</b>
172.16.0.0      0.0.0.0         255.240.0.0     U     0      0        0 eth0
</pre>

where the two routes shown in bold say:

* "Send traffic with destination IP address 10.10.4.0/24 (addresses whose first 24 bits - 3 octets - are the same as the first 24 bits of 10.10.4.0) through interface eth1."
* "Send traffic with destination IP address 10.10.5.0/24 through interface eth2."

(The other two rules shown above describe how to route traffic to and from the "control" interface, which you use in order to connect to the node over SSH.)

The second kind of routing rule, which involves traffic that is forwarded through a gateway, is not installed. We will need to define rules based on our Dijkstra's algorithm table, and add them ourselves. 

The syntax of that kind of rule is e.g.:

<pre>
sudo route add -net <b>10.10.3.0/24</b> gw <b>10.10.4.1</b>
</pre>

which says: "Forward traffic with destination IP address **10.10.3.0/24** through the next-hop router **10.10.4.1**". (The next hop should be on the local network.) 

These rules, too, can be read off the table that we completed with Dijkstra's algorithm, and the shortest path tree. For example, our table showed that the hop before "lovelace" is "hopper". We also note from the shortest-path tree that the address on which "dijkstra" will reach "hopper" is 10.10.3.1, and the address on which "hopper" will reach "disjktra" is 10.10.4.2. To set up this route, we would:

* **Set up the forward path**: Run `sudo route add -net 10.10.3.0/24 gw 10.10.4.1` on "dijkstra". This says to send traffic for the 10.10.3.0/24 network through "hopper".
* **Set up the reverse path**: Run `sudo route add -net 10.10.4.0/24 gw 10.10.3.2` on "hopper". This says to send traffic for the 10.10.4.0/24 network through "lovelace".

For routes with several hops, we need to make sure that routes are also set up at intermediate routers. For example, for the "dijkstra"-"lovelace"-"knuth"-"cerf" path:

* **Set up the forward path**: On "dijkstra", we will add a route to 10.10.7.0/24 through 10.10.4.1. Then, on "lovelace", we will add a route to 10.10.7.0/24 through 10.10.8.2.
* **Set up the reverse path**: On "cerf", we will add a route to 10.10.4.0/24 through 10.10.7.1. Then, on "knuth", we will add a route to 10.10.4.0/24 through 10.10.8.1.

Some paths through the tree may involve duplicate rules - for example, both the path to "cerf" and the path to "knuth" require a rule at "knuth" that says to send traffic to 10.10.4.0/24 through 10.10.8.1. When you try to set up a duplicate rule, you'll see an error

```
SIOCADDRT: File exists
```

which is not a cause for concern in this context. 

To test our routes, we can use `mtr`. This is a traceroute application that 

* shows us each hop along a path to a destination, and
* shows us the cost of the path "so far" at each hop.

For example, to see the path from "disjktra" to "cerf", I would run

```
mtr 10.10.7.2 --report --no-dns
```

where 

* `--report` says to run the traceroute, then print a report and return.
* `--no-dns` says to show IP addresses as numbers, rather than trying to resolve hostnames.

The output of this shows that the path is consistent with my Dijkstra's algorithm table:

<pre>
HOST: dijkstra.multisite-another. Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- <b>10.10.4.1</b>                  0.0%    10   52.0  <b>34.8</b>  32.2  52.0   6.0
  2.|-- <b>10.10.8.2</b>                  0.0%    10   78.3  <b>78.0</b>  76.6  79.2   0.6
  3.|-- <b>10.10.7.2</b>                  0.0%    10  109.7 <b>109.4</b> 108.5 110.0   0.4
</pre>

This output shows:

* The average cost to reach "cerf" is 109.4 ms, which is similar to the 103.842 cost listed in the table.
* the previous hop before "cerf" in the table is "knuth", so the second-to-last entry in the `mtr` report is 10.10.8.2. Furthermore, the average cost to "knuth" is 78.0, similar to the 72.551 listed in the table.
* the previous hop before "knuth" in the table is "lovelace", so the third-to-last entry in the `mtr` report is 10.10.4.1. Also, the average cost to "lovelace" is 34.8, similar to the 32.742 listed in the table. 

The following video shows how we set up the shortest-path tree and the routes on our GENI nodes:

<iframe width="560" height="315" src="https://www.youtube.com/embed/uqgBVHHFtLM" frameborder="0" allowfullscreen></iframe>

and also how we used `mtr` to verify that the routes and the total path cost from the source node to every other node are consistent with our expectations:

<iframe width="560" height="315" src="https://www.youtube.com/embed/G5G1oWy2Z9c" frameborder="0" allowfullscreen></iframe>

## Notes

### Exercise

For your topology, run Dijkstra's algorithm and submit:

* A table, like [this one](https://witestlab.poly.edu/blog/content/images/2017/03/dijkstra-table.svg).
* A figure showing *link* costs on each link (_not_ the *path* costs), like [this one](https://witestlab.poly.edu/blog/content/images/2017/03/dijkstra-topology-costs.svg).
* A figure showing the shortest-path tree, like [this one](https://witestlab.poly.edu/blog/content/images/2017/03/dijkstra-topology-routes.svg).
* Show the output of `mtr` when you run it on the "dijkstra" node, for each destination node. For each destination, explain whether the `mtr` output is consistent with your table. Make sure to highlight the relevant parts of your `mtr` output. For example:

<blockquote>
From "dijkstra" to "cerf": 
<pre>
ffund01@dijkstra:~$ mtr 10.10.7.2 --report --no-dns
Start: Mon Mar 27 20:47:36 2017
HOST: dijkstra.multisite-another. Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- <b>10.10.4.1</b>                  0.0%    10   52.0  <b>34.8</b>  32.2  52.0   6.0
  2.|-- <b>10.10.8.2</b>                  0.0%    10   78.3  <b>78.0</b>  76.6  79.2   0.6
  3.|-- <b>10.10.7.2</b>                  0.0%    10  109.7 <b>109.4</b> 108.5 110.0   0.4
</pre>
</blockquote>
> This output shows:
>
>* The average cost to reach "cerf" is 109.4 ms, which is similar to the 103.842 cost listed in the table.
>* the previous hop before "cerf" in the table is "knuth", so the second-to-last entry in the `mtr` report is 10.10.8.2. Furthermore, the average cost to "knuth" is 78.0, similar to the 72.551 listed in the table.
>* the previous hop before "knuth" in the table is "lovelace", so the third-to-last entry in the `mtr` report is 10.10.4.1. Also, the average cost to "lovelace" is 34.8, similar to the 32.742 listed in the table. 

