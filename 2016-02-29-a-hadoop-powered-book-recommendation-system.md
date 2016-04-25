This project shows how to use Hadoop to process a large text corpus (in this case, the Project Gutenberg public domain book dataset) and use it to power a simple book recommendation engine. 

This experiment should take about 60-90 minutes to run.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes).

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)


## Background

Apache's [Hadoop](http://hadoop.apache.org/) is an open-source storage and processing framework that can scale out to thousands of servers, and handle petabytes of data. Hadoop was produced shortly after [Google's 2004 File System research paper](http://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf).

Hadoop's processing works by splitting both data and processing jobs onto clusters of commodity hardware. It can use many machines to process jobs in parallel, and also keep working even if one machines fails (an idea known as fault-tolerance).

Hadoop's distributed storage system is also scalable. This means that users can continue to add capacity to a full file system by adding more workers to the cluster, without disproportionately influencing the performance of the cluster. 

### Our Recommendation System

Our project uses Hadoop to find instances of relevant keywords in the [Project Gutenberg](https://www.gutenberg.org/) text corpus and recommend books that contain many instances of those keywords.

We approached the problem of creating a recommendation system as follows:

We generated a word frequency list from the entire corpus (a one-time step that makes subsequent queries much faster). We downloaded all of the available Project Gutenberg books over a period of a couple of days, storing them on a GENI VM (an [XOXLarge](https://wiki.exogeni.net/doku.php?id=public:experimenters:resource_types:start) type). Once we had the books (~36,000 books at 17GB), we created a word frequency list (~ 3.6GB), where every line contained a [stemmed](https://en.wikipedia.org/wiki/Stemming) word, a particular book id, the total number of words in that book, and the number of times the stemmed word appeared.

So, when given a set of words to look for, we only need to process that list, rather than the entire corpus. 

A single [MapReduce](https://en.wikipedia.org/wiki/MapReduce) job is then used to process the file. 

In the map stage, the lines containing the words of interest are printed. These lines are the input to the reducer job, which applies a simple term frequency algorithm to produce a relevancy score S<sub>b</sub> for every book passed to it


$$ S\_b = \frac{1}{W} \sum\_{w=1}^{W} TF\_{w}  $$

where _W_ is the number of search terms, and TF<sub>w</sub> is the term frequency of word w in a book  and is computed as

$$ TF\_{w} = \frac{c}{t \alpha} $$

where _c_ is the number of times word _w_ appeared in a particular book, _t_ is the total number of words in that book, and _&alpha;_ is just an amplifying constant.

The books with the highest scores are then passed to the user.  

## Results

The following books were recommended for the terms "frog" and "prince":

![](/blog/content/images/2016/02/hadoop-recs.png)


## Run my experiment

On the GENI portal we'll use Jacks to reserve several Exogeni VMs. In a new slice, click "Add Resources", scroll down to the "Choose RSpec" section and select the "URL" option. Copy the [URL](https://raw.githubusercontent.com/rollingcoconut/hadoopExprOnGeni/master/rspec.xml) of the RSpec (http://tinyurl.com/h4eftjj) and enter it into the text box for the URL field, then click "Select".


It will load the following topology:

![](/blog/content/images/2016/02/kw-hadoop-topology.png)

Click on "Site 1" on the canvas and choose an available ExoGENI aggregate. Then reserve your resources. Return to your slice page and wait for your nodes to be ready to log in.

SSH into the master VM you created. You are now logged into the VM that coordinates the processing that goes on in the cluster.

First make sure all the machines on the cluster can communicate. Test for connectivity with the following ping commands. (If you are unable to ping to any of your worker machines from master, delete your resources on Jacks and get another.)

```
sudo ping -c 5 worker-0
sudo ping -c 5 worker-1
sudo ping -c 5 worker-2
```
Get the files you need to run the experiment by typing the following:


```
git clone https://github.com/rollingcoconut/hadoopExprOnGeni.git
```


Navigate to the "hadoopExprOnGeni" folder and execute the script "install.sh" to install the  linux tools bc, unzip, and httpd, and also configure your experiment environment. Type the following to change directory and execute "install.sh":

```
cd hadoopExprOnGeni
sudo ./install.sh
```

Now start the Hadoop application and place the 1GB Gutenberg dataset you just downloaded into the Hadoop's distributed filesystem with the following commands:
 
```
sudo su hadoop -
cd /home/hadoop
./startHadoop.sh   
```


If you haven't changed directories you should still be in /home/hadoop and logged in as the hadoop user. 

To get a book recommendation, run the "exprStart.sh" script with a list of terms of interest, e.g.:
```
./exprStart.sh 1 frog prince
```

(The first argument, here "1", represents the factor  by which to increase the logical block size a map job works on.)

To see your book recommendation, find out the public IP address of your master node by running:

```
wget http://ipinfo.io/ip -qO -
```

Then visit that IP address in a browser to see your recommendations.

When you're finished, don't forget to delete your resources to free them up for other experimenters!

## Notes

### Acknowledgements

This project was supported by the National Science Foundation and by GENI as a Research Experience for Undergraduate (REU) project.

The Rspecs in this experiment are based on the ones in this [GENI tutorial](http://groups.geni.net/geni/wiki/GENIExperimenter/Tutorials/jacks/HadoopInASlice/ObtainResources). A more recent Hadoop tutorial is available [on the ExoBlog](http://www.exogeni.net/2015/09/hadoop-tutorial/).


### Software versions

* CentOS Linux release 7.2.1511 (Core) 
* Python 2.7.5
* Github repository: https://github.com/rollingcoconut/hadoopExprOnGeni

### Possible extensions

If you're interested in modifying our algorithm for calculating relevance scores, you may want to edit the [reducer.py](https://github.com/rollingcoconut/hadoopExprOnGeni/blob/master/AUX_FILES/reducer.py) file. That's where the relevance scores are computed.

### Known issues

If you interrupt your MapReduce run, the next time you run "exprStart.sh" your job may be stuck at 0%. This can be resolved by following the steps outlined in "restartYARN.sh"
