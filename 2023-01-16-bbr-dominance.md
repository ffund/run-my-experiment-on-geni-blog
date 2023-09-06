## Background 

Low latency TCP congestion control (CC) is a key enabler of delay-sensitive applications such as cloud gaming, remote driving, and virtual/augmented reality(VR/AR). In recent years, TCP BBR has emerged as a popular choice for low latency CC, with it already having widespread adoption. However, a critical question that needs to be studied is how BBR will coexist with the current most dominant CC protocol on the internet, i.e., TCP Cubic. A recently published study, titled, [``Are we heading towards a BBR-dominant internet?''](https://dl.acm.org/doi/abs/10.1145/3517745.3561429) proposed a mathematical model for the coexistence of TCP BBR and TCP Cubic. This study on BBR dominance predicted that the internet will reach a stable mixed distribution of BBR and Cubic flows, resulting in a Nash Equilibrium where no traffic will have any incentive to switch between BBR and Cubic. 

This blog post presents a fully reproducible experiment on the [FABRIC testbed](https://fabric-testbed.net/) that replicates the major experiments from the BBR dominance study. In addition to reproducing the original results, we further show that the predictions of the proposed model do not hold up in specific scenarios that are highly relevant to today's low-latency networks. Our work motivates future work to improve the analysis of BBR and Cubic's coexistence. 


## Results

The first experiment **(Experiment 1)** looks at the **per-flow throughput and queuing delay at a shared bottleneck**. This reproduces the result from Figure 8 of the BBR-dominance paper where 10 TCP flows run for 120 seconds over a line network with a bottleneck capacity of 100 Mbps, based delay of 40 ms, and a buffer size of 2 BDP. The result obtained from our FABRIC testbed experiment is shown below:

![](/blog/content/images/2023/01/Screen-Shot-2023-01-16-at-4.47.22-PM.png)

**Experiment 2** is about **observing Nash Equilibria with a mixed BBR-Cubic bottleneck**. This tries to replicate the observations shown in Figure 9 of the BBR-dominance paper. This experiment is designed to verify the existence of Nash Equilibrium in a mixed BBR-Cubic network, and to validate the accuracy of the mathematical model proposed in the original work. Our results from this experiment obtained from our FABRIC testbed setup are shown below:

![](/blog/content/images/2023/01/Screen-Shot-2023-01-16-at-6.09.47-PM.png)

The above results use the same network conditions as were used in the original BBR-dominance paper. Our results somewhat agree, with most of the Nash Equlibrium points falling within the blue-shaded regions denoting the “Nash Region” predicted by the model (for the experiment parameters and factor levels considered in the original paper). 

However, our experimental results disagree with the original paper with respect to one high-level conclusion. The authors claim that when the buffer size is expressed as a multiple of BDP, "the region predicted by the model is exactly the same regardless of the base RTT or the bottleneck link capacity," and that the experimentally observed Nash Equilibria agree with this trend. In our experiment, we observe that in the networks with smaller BDP (e.g. 50 Mbps capacity, 20 ms RTT), the theoretical model underestimates the proportion of BBR flows at the Nash Equilbrium for small or moderately-sized buffers. The two figures below show our results for **Experiment 2** when tested on small base delays and high bottleneck capacities respectively.

![](/blog/content/images/2023/01/Screen-Shot-2023-01-16-at-6.16.14-PM.png)

To confirm that when the base delay is small, there is no Nash Equilibrium with both BBR and Cubic flows co-existing, we re-ran **experiment 1** for a network with base RTT of 5 ms. The rest of the conditions remain the same as before. The result is shown below:

![](/blog/content/images/2023/01/Screen-Shot-2023-01-16-at-6.20.01-PM.png)


## Run my experiment

To reproduce our experiments on FABRIC, log in to the FABRIC testbed's JupyterHub environment. Open a new terminal from the launcher, and run:

```
git clone https://github.com/ashutoshs25/bbr-dominance-experiments
```

Then, run the following notebooks with the following parameters:

* To reproduce the results showing per-flow throughput and queuing delay at a shared bottleneck, i.e., **Experiment 1**, run `Figure_2.ipynb`. (This takes about 30 minutes). Make sure that the experiment parameters are configured as follows:

```python 
btl_capacity = 100 # mbit
base_owd     = 20  # ms, to be applied in each direction
n_bdp        = 2
btl_limit    = int(1000*n_bdp*btl_capacity*2*base_owd/8)
```

* To reproduce the results for Experiment 1 with 100Mbps capacity, 5ms base RTT, run `Figure_5.ipynb`. (This takes about 30 minutes). Make sure that the experiment parameters are configured as follows:

```python 
btl_capacity = 100 # mbit
base_owd     = 2.5  # ms, to be applied in each direction
n_bdp        = 2
btl_limit    = int(1000*n_bdp*btl_capacity*2*base_owd/8)
```
* For reproducing the results of **experiment 2**, i.e. the Nash Equilibrium experiments, you will need to run the notebooks `Nash_equilibirium_experiments/slice_x.ipynb` inside the `bbr-dominance-experiments` Github repo. Here `x`(notebook number) goes from 1 to 12 since we need to generate 12 figures, each for a different network condition. Each notebook, when run will reserve a separate slice on the FABRIC testbed and run the experiment for the specified given network condition. For example, in `slice_1.ipynb` we specify:
```
btl_capacity = 50 # mbit
base_owd     = 2.5  # ms, to be applied in each direction
```
   This configures the network to have a capacity of 50Mbs, and a total base delay of 5ms (one-way delay is 2.5ms and is applied in both directions).

Each experiment will take around 13 hours to run. Hence, it is recommended that you run the twelve notebooks (`slice_1.ipynb` to `slice_12.ipynb`) in parallel. 

Once the notebook corresponding to slice `x` is fully executed, the processed results will be saved in `slice_x/bbr_results.csv` and `slice_x/cubic_results.csv` 

* For calculating the experimentally observed Nash equilibrium points in **Experiment 2** and plotting them for each network scenario, run the notebook `plot_fig_9.ipynb`. 

## Publication

This work was published at the IEEE INFOCOM International Workshop on Computer and Networking Experimental Research using Testbeds (CNERT) 2023.

**Citation and link**:

*Ashutosh Srivastava, Fraida Fund, and Shivendra Panwar. “Some of the internet may be heading towards BBR dominance: an experimental study”. In 2023 IEEE INFOCOM International Workshop on Computer and Networking Experimental Research using Testbeds (CNERT 2023), New York area, USA, May 2023.* [Link](https://witestlab.poly.edu/~ffund/pubs/bbr_cnert23.pdf)

