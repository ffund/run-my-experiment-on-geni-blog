In this experiment, you'll set up a slice on GENI that uses has the same network topology as a part of an active community network in Austria. 

It should take about 60 minutes to run this experiment.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes). If you're not sure if you have those skills, you may want to try [Lab Zero](http://tinyurl.com/geni-labzero) first.

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

Availability of computers and Internet has essentially become a universal right, yet still Internet remains inaccessible in some rural and low-income parts of the world. Commercial Internet Service Providers are essentially the only form of high-speed internet access people have, limiting the consumers' choices and setting inescapable high prices [[1](#references)]. These providers can even prevent people from getting Internet by refusing to extend network lines to certain areas, such as rural territories, as there is not much demand and they will not make significant profit. 

![Internet Use by Income](/blog/content/images/2016/09/internet-use-by-income.svg)

_Data Source: July 2015 Computer and Internet Use Supplement to US Census_

The image above shows the computer and Internet usage of households in the United States, split into 4 different income brackets. There is a huge disparity between computer/Internet usage for households with higher incomes and households with lower incomes. Households with higher incomes (>75k) have nearly 100% Internet usage or access, while households with lower incomes (<25k) have around 50%, showing that those with lower incomes often cannot obtain Internet access.


Community networks have been presented as a potential solution to the problem of Internet availability [[2](#references)]. Community networks are an alternate way for underserved areas to obtain Internet connection. They are run by the users, for the users, without the need for every user to have Internet access from an ISP. Each individual person's wireless router can become a node, in which they can obtain Internet access as well as share it with others. 

![Map of Community Networks around the World](/blog/content/images/2016/09/cnmap.svg)

This map of the world depicts some of the more well-known community networks, to show their world-wide spread. There are many other smaller networks not shown.

A popular example of one is Guifi in Spain [[10](#references)]. They are a successful community network (and the biggest) with over 20,000 nodes, stemming from a rural area of Catalonia which had no ISPs to provide service to them. They have been so successful that they have even signed agreements with local government to bring Internet access to new areas. Another inspiring community network is one in Sarantaporo, Greece [[7](#references)]. This agricultural region had never had Internet access, and were basically isolated from the world, and each other. They were able to build a wireless community network that connected the many different villages in the region, to enliven their economy and allow better access to basic services like medical aid. The villages were able to communicate with each other, as well as the rest of the world through the network. The people of Sarantaporo are excited to get up in the morning and check the common website, where they can interact and organize events, and children find the villages more attractive now, whereas before they would often grow up to leave the region. There have been many similar initiatives all around the world to connect people and provide them with what many may think of as a human right, Internet access.

In my project, I create a tool to help researchers experiment on community networks. This is important because community networks are user-run and are built from the bottom up, and so they have different characteristics from the conventional networks provided by ISPs:

![A Conventional Network and Community Network](/blog/content/images/2016/09/network-comparison.svg)

_This drawing shows the heirarchy within the conventional model of Internet access, with ISPs. Connection goes through many levels before it is passed to the public. In community networks, there is little hierarchy and all of the nodes are interconnected with each other._

There are multiple benefits to using community networks over conventional ones. For one, they are generally relatively inexpensive to operate and free to use. Community networks increase consumer choice: in urban areas, there is a bit more freedom to make an individual choice in how you would like to obtain Internet, so you are not forced to use ISPs, and in rural areas, it may even be the only choice.  There is also an opportunity for increased security, privacy, and neutrality with community networks. Since everything is decentralized, it is more difficult for oppressive governments to exploit points in the network infrastructure to collect mass data or to censor some sources of information. With ISPs, there is a more centralized network, and they can control the mass majority of user's interaction [[1](#references)]. Similarly, conventional networks may accept payment from content providers to favor their traffic (a net neutrality violation) but community networks treat all traffic equally.

The decentralized model of community networks proves beneficial in crises and failures. When a node fails and goes down in a mesh/community network, it does not affect the rest of the network, as they can still obtain connection from another node [[3](#references)]. In a conventional/centralized model, if the central node goes down then all nodes lose access. For example, when Hurricane Sandy hit the East Coast and most of New York could not communicate with one another, Red Hook WiFi, a community mesh network in Brooklyn, was still up and running and able to provide a means of communication between people [[4](#references)].

Some characteristics of community networks have a negative impact on user experience. Community networks are composed mainly of wireless links, which have higher packet loss. Community networks are also mesh networks, leading to more latency because data has to go through multiple hops to reach a destination [[5](#references)].

However, creators of applications and other Internet related products don't really cater their products to consumers running them on community networks, which ultimately means decreased quality for those using the network (even though they already have slower speeds). Furthermore, it would be useful to create products specifically for community network users. For instance, they would benefit from peer-to-peer video streaming where each node can stream a video content to another node, rather than each node trying to get a video individually from the Internet [[9](#references)].

To make applications for community network users, it may seem like the most accurate thing to do would be to test them directly on the CNs [[6](#references)], but this is not a controlled environment and is not ideal for testing applications or alternative networking protocols to be deployed on them. Instead, the goal of our research is to make it easy for researchers to study the performance of community networks in a controlled environment, and use this environment to make Internet use better for them.

GENI [[11](#references)] is a research testbed where researchers can create an arbitrary configuration of computers connected by links, and then actually log into them to test and download applications. There are hundreds of nodes or virtual machines across the country at many universities that GENI researchers can gain access to from their computer.

To help researchers represent community networks on GENI, we wrote a Python script that generates a "resource specification" (RSpec) file that researchers can use to set up a network on GENI for testing. It gets data from an active community network, specifically Funk Feuer Graz in Austria, a fairly large community network with hundreds of nodes [[8](#references)]. As the data source, we downloaded a file that represents the nodes and the links of the FFGraz community network topology as a graph. 

Since the total topology includes hundreds of nodes, and it is more useful (and practical) to study small parts of it at a time, we then partition the graph into various subgraphs, separated by color in the image below. The partitioning process exploits the modularity that is characteristic of these networks: networks with high modularity have modules with dense connections between the nodes and sparse connections between nodes in different modules. So each subgraph includes nodes that are closely connected to one another, but not closely connected to nodes in other subgraphs, making them ideal to study in isolation.

![Colored CN Graph](/blog/content/images/2016/09/subgraph-highlight.svg)

_This graph shows the topology split into communities with different colors for each community. Subgraph 3, which we use for the rest of this discussion, is in pale green at the bottom with a box around it._

Next, we had to address cliques. Every node in a clique is connected to all other nodes in a clique (fully connected). In a wireless network, nodes that are all close to one another are fully connected through a single shared link. However, in the graph of the topology, it depicted these as point-to-point links, where each node had one individual link with every other node. When we represent the network on GENI, we want to represent the clique as a single shared link so that (1) traffic between one pair of nodes will affect other nodes on the link, which is more realistic, and (2) it is easier to successfully reserve the topology on GENI, since it requires fewer links overall. To solve this, our script finds all of the cliques in our subgraph and connects all of the nodes within the clique to one link.

![Subgraph selected for this example](/blog/content/images/2016/09/clique-detection-2.svg)

_This image shows one subgraph (subgraph 3) from the topology, with boxes surrounding some cliques._

From this topology we generate an RSpec with which you can reserve resources on GENI to accurately represent this part of the FF Graz community network.

## Results

The resulting topology on GENI looks this this:

![GENI Community Network Topology](/blog/content/images/2016/09/geni-portal.png)


## Run my experiment

There are many different python libraries you will need to reproduce this experiment. For example, community, matplotlib, seaborn, pydotplus, and networkx. To make this easier, you can reserve a single VM on GENI and log onto it and use these commands.

    sudo apt-get install python-pip
    sudo pip install networkx
    sudo apt-get install pkg-config
    sudo apt-get install libpng-dev
    sudo apt-get install libfreetype6-dev
    sudo apt-get install libjpeg8-dev 
    sudo apt-get install python-dev
    sudo pip install matplotlib
    sudo apt-get install gfortran libopenblas-dev liblapack-dev
    sudo apt-get install python-numpy python-scipy
    sudo pip install seaborn
    sudo apt-get install mercurial
    hg clone https://bitbucket.org/taynaud/python-louvain
    cd python-louvain
    sudo python setup.py install
    sudo pip install pydotplus

Next, you will want to clone this repository, then go into the repository. To do this, you can use 

    git clone https://github.com/amayamunoz/Community-Networks.git
    cd Community-Networks
    
This repository already include the current FF Graz network topology representation. If you want to use some other topology, such as a historical topology of FF Graz from [its archive](http://stats.ffgraz.net/topo/DOTARCHIV/), you can use this code with the URL of the dot file you want to use and replace the current `topology.dot.plain` file:

    wget http://stats.ffgraz.net/topo/topology.dot.plain -O topology.dot.plain

Once you're inside the `Community-Networks` directory, you should have the file `topology.py`. If you run this Python code with 

```
python topology.py
```

it will produce an Rspec in an xml file called `subgraph-3.xml`. If you want to change which subgraph you are using, change the integer value for the variable `subgraph`. This will also change the Rspec file name and the 3 will be replaced with whichever number you chose. 

You can then upload the Rspec to GENI and reserve the resources, and you have your own community network topology! In the GENI Portal, create a new slice and click on "Add Resources". Upload the Rspec file. Then click on "Site 1", choose an InstaGENI aggregate, and then click on "Reserve Resources".

As future work, we would like to use other sources of data on community networks to make our GENI links emulate realistic network conditions, such as packet loss, link capacity, and latency, using <code>netem</code>. 

## Notes

This research was supported by the NYU Tandon School of Engineering [Center for K12 STEM Education](http://engineering.nyu.edu/k12stem) via the [ARISE](http://engineering.nyu.edu/k12stem/arise/) program, and by the Pinkerton Foundation.


## References

[1] R. Lo Cigno and L. Maccari. "Urban wireless community networks: challenges and solutions for smart city communications." Proceedings of the 2014 ACM international workshop on Wireless and mobile technologies for smart cities. ACM, 2014. [http://dl.acm.org/citation.cfm?id=2633669](http://dl.acm.org/citation.cfm?id=2633669)

[2] M. Oliver, J. Zuidweg and M. Batikas, "Wireless Commons against the digital divide," 2010 IEEE International Symposium on Technology and Society, Wollongong, NSW, 2010, pp. 457-465.
[http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=5514608](http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=5514608)

[3] P. De Filippi, "It's Time to Take Mesh Networks Seriously", WIRED Magazine, 02 Jan 2014. [http://www.wired.com/2014/01/its-time-to-take-mesh-networks-seriously-and-not-just-for-the-reasons-you-think/](http://www.wired.com/2014/01/its-time-to-take-mesh-networks-seriously-and-not-just-for-the-reasons-you-think/)

[4] Noam Cohen, "Red Hook’s Cutting-Edge Wireless Network", New York Times, 24 Aug. 2014, [http://www.nytimes.com/2014/08/24/nyregion/red-hooks-cutting-edge-wireless-network.html?_r=0](http://www.nytimes.com/2014/08/24/nyregion/red-hooks-cutting-edge-wireless-network.html?_r=0)

[5] B. Braem et al. "A case for research with and on community networks." ACM SIGCOMM Computer Communication Review 43.3 (2013): 68-73. [http://dl.acm.org/citation.cfm?id=2500108](http://dl.acm.org/citation.cfm?id=2500108)

[6] A. Neumann, I. Vilata, X. León, P. E. Garcia, L. Navarro and E. López, "Community-lab: Architecture of a community networking testbed for the future internet," 2012 IEEE 8th International Conference on Wireless and Mobile Computing, Networking and Communications (WiMob), Barcelona, 2012, pp. 620-627. [http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6379141](http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6379141)

[7] "Building Communities of Commons in Greece", Documentary video produced by  Personal Cinema [https://www.youtube.com/watch?v=eA4u03iKmRA](https://www.youtube.com/watch?v=eA4u03iKmRA)

[8] Funkfeuer Graz Statistics [http://stats.ffgraz.net/](http://stats.ffgraz.net/)

[9] L. Baldesi, L. Maccari and R. Lo Cigno, "Improving P2P streaming in community-lab through local strategies," 2014 IEEE 10th International Conference on Wireless and Mobile Computing, Networking and Communications (WiMob), Larnaca, 2014, pp. 33-39. [http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6962146](http://ieeexplore.ieee.org/xpl/login.jsp?tp=&arnumber=6962146)

[10] Guifi [https://guifi.net/](https://guifi.net/)

[11] M. Berman et al. "GENI: A federated testbed for innovative network experiments." Computer Networks 61 (2014): 5-23. [http://dx.doi.org/10.1016/j.bjp.2013.12.037](http://dx.doi.org/10.1016/j.bjp.2013.12.037)
