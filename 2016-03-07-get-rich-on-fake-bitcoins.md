This experiment explores the problem of consensus in distributed systems in the context of Bitcoin, a distributed currency.

This experiment should take about 60-120 minutes to run.

To reproduce this experiment on GENI, you will need an account on the [GENI Portal](http://groups.geni.net/geni/wiki/SignMeUp), and you will need to have [joined a project](http://groups.geni.net/geni/wiki/JoinAProject). You should have already [uploaded your SSH keys to the portal and know how to log in to a node with those keys](http://groups.geni.net/geni/wiki/HowTo/LoginToNodes).

* Skip to [Results](#results)
* Skip to [Run my experiment](#runmyexperiment)

## Background

This experiment explores the problem of reaching **consensus** in a distributed system. "Consensus" is the problem of getting members of a network to agree on something, e.g. a value. In some systems, there is a centralized control unit who can decide on the value and then broadcast it to the rest of a network. In a distributed consensus system, members of the group have to collectively reach consensus without the benefit of a centralized unit. Further complicating the problem, some members of the group may be lying or otherwise manipulating the group to try and reach a consensus that favors them over the "true" value.

There are various solutions to the problem of distributed consensus. In this experiment, we examine the approach used by **Bitcoin**, a distributed currency with no central bank or other authority. In order for a distributed currency to work, members of the network must agree on how many units of the currency each member holds at all times, in order to prevent members from **double spending**, i.e. re-using the same units of currency in multiple **transactions**. This is, of course, a distributed consensus problem.

Bitcoin uses a process called **mining** to reach a consensus. Members of the network who choose to take part in the process of reaching a distributed consensus are called **miners**. Mining involves forming a **block** containing a series of transaction records, then finding a valid **proof of work** for that block that satisfies certain rules. Specifically, miners increment a **nonce** until they find a value that gives the block's hash a certain number of leading zeros, thereby "finding" the next block in the **blockchain**. For example:


Once a block has been "found", it is broadcast to all other nodes in the network, who validate the transactions it holds and accept the block if it is valid. 

Overall, the process is:

1. New transactions are broadcast to all nodes in the network.
2. Miner nodes collect the new transactions into a block. 
3. Miner nodes try to find a proof of work for the block. 
4. When a miner node finds a proof of work, it broadcasts it to all nodes in the network. 
5. Receiving nodes validate the new block and then start work on the next block, containing transactions that have not yet been placed in a block.
6. The node that found the block is rewarded with some new Bitcoins, and also the value of the transactions fees for the transactions in the block.

Miners choose which transactions to include in the block they are mining based on priority algorithms that include metrics such as the age of the transaction and the value of its transaction fees. So it is possible for two nodes to arrive at different "next blocks," potentially at the same time, and broadcast different blocks to the rest of the network. If this occurs, the blockchain is said to be **forked**, and the network needs some way to reach a consensus on which block to accept as the next block. 

Bitcoin clients operate on a simple rule: always trust the longest chain of blocks. If a node received two "next blocks", it will save both and start to work on the next block in one of the chains. If in the meantime it receives a new block for one of the chains, it will discard the shorter chain and start to work on a next block for the longer chain. (Miners have an incentive to work on the longest chain, rather than continuing to work on the chain following from the **orphaned** blocks, because they want to earn the Bitcoins associated with finding the next block in the longest chain.)

![](https://upload.wikimedia.org/wikipedia/commons/9/98/Blockchain.svg)

<i>The main chain (black) consists of the longest series of blocks from the genesis block (green) to the current block. Orphan blocks (purple) exist outside of the main chain. Image by Theymos [CC BY 3.0](http://creativecommons.org/licenses/by/3.0), via Wikimedia Commons</i>

A key property of these blocks is that in order to replace a block that is already in the blockchain with one of its own, a node would have to compute the proof of work for that block and all subsequent blocks that have been added to the blockchain after it, since in order to get Bitcoin clients to accept your version of the blockchain, you need to have the longest chain. 

Finding proof of work is computationally difficult, so the more blocks have been added to the blockchain after the block containing a particular transaction, the more secure it is and the less likely it is that the block will be replaced (rolling back the transaction). Most Bitcoin clients consider the a Bitcoin transaction to be **confirmed** after six more blocks to be added to the chain, and the Bitcoins awarded for finding a new block are considered to be confirmed after 100 blocks. 

This kind of consensus is not completely invulnerable to double spending. Suppose an attacker submits a transaction to the network that pays some of its Bitcoins to another party, while simultaneously mining a private blockchain fork in which it re-uses the same currency for another transaction. After waiting for _n_ confirmations (i.e. _n_ more blocks are added to the blockchain that the rest of the network sees), the other party considers the transaction to be secure and sends the attacker the good or service that it paid for. At this point, if the attacker has more than _n_ blocks in its privately mined fork, he can broadcast his forked chain, including the double-spend transaction, to the network and effectively use the same Bitcoins twice. If not, he can continue to try extending his private fork in hopes of surpassing the rest of the network, at which point he will release the fork. The probability of this succeeding depends on _n_ and on how much **hashing power** the attacker has relative to the rest of the network.

If an attacker controls more than half of the hashing power in the network (i.e. there is a 51% chance or greater that it computes the next proof of work), then the attack described above will succeed with 100% probability. Since the attacker can generate blocks faster than the rest of the network (on average), eventually his private fork _will_ surpass the blockchain constructed by the rest of network. This is known as a **majority attack**. 

There have been times when **pools** of Bitcoin miners have controlled more than 51% of the hashing power in the network, but they have not used it to carry out a majority attack. It is typically more profitable for a majority pool to use its hashing power honestly to generate new coins, than to use it to steal back payments made to other people.

In this experiment, we will see how multiple confirmations are required before a Bitcoin transaction or mining rewards become spendable. We will also see how a Bitcoin network reaches consensus when the blockchain is forked.

## Results

In this experiment, we allow a fork to develop in a Bitcoin network. One side of the network has one has for the last block in the chain:

```
ffund01@node-4:~$ bitcoin-cli -regtest getblockhash 121
5eaf42996bc3062e96f096eee12131f34f1e48c137eb4cfe2e9cf75f5a4826fc
```

while the other side of the network has another

```
ffund01@node-0:~$ bitcoin-cli -regtest getblockhash 121
4e5545b5f3bf4f57914c11cf835aeccc5fd51e38139d9c25c4ecd67904ea9869
```

After reconnecting the network, both forks still persist because they are of equal length. However, as soon as one more block is generated, all nodes in the network converge on the longer chain:

```
ffund01@node-4:~$ bitcoin-cli -regtest getblockhash 121
4e5545b5f3bf4f57914c11cf835aeccc5fd51e38139d9c25c4ecd67904ea9869
```

## Run my experiment

Download the following [RSpec](https://gist.githubusercontent.com/ffund/5b5165df52c342b7bd163d3c7ca76855/raw/70f6fb663a86dbe35a1d9bdc262428444e0f3d19/bitcoin.xml). In the GENI Portal, load this RSpec in a new slice (click "Add resources", scroll down to "Use RSpec", and choose "From File.")

This will load the following topology onto your canvas:

![](/blog/content/images/2016/03/bitcoin.png)

and will also pre-install the bitcoin tools onto the nodes. Click on "Site 1", then bind to any InstaGENI aggregate and reserve your resources.

Once your resources are ready to log in, open five  terminals, and in each, log in to one node. (You may have to wait a few minutes after the nodes come up for it to finish installing the Bitcoin software.)

### Set up Bitcoin network

Run the following on each node:

```
mkdir ~/.bitcoin
echo "rpcuser=test\nrpcpassword=test\n" > ~/.bitcoin/bitcoin.conf
```

Then run

```
bitcoind -regtest -daemon 
```

to run the bitcoin daemon in ["regtest" mode](https://bitcoin.org/en/developer-examples#regtest-mode). This mode lets us create a private blockchain to experiment with, and create new blocks on demand (otherwise it would be impossibly difficult for us to mine blocks on our ordinary CPUs.)

On each node, add all the other nodes as peers:

```
bitcoin-cli -regtest addnode node-0 onetry
bitcoin-cli -regtest addnode node-1 onetry
bitcoin-cli -regtest addnode node-2 onetry
bitcoin-cli -regtest addnode node-3 onetry
bitcoin-cli -regtest addnode node-4 onetry
```

Verify that each node sees all the others with

```
bitcoin-cli -regtest getpeerinfo
```

### Generate blocks

Now that all our peers are connected, we are going to generate blocks, and watch them get propagated through the network.

Generate a block on node-0 with

```
bitcoin-cli -regtest generate 1
```

and check that all the other nodes become aware of it with

```
bitcoin-cli -regtest getblockcount
```

(all nodes should see a blockcount of 1.)

Check whether the node that generated the block has earned a reward by running

```
bitcoin-cli -regtest getbalance
```

on that node. It should still have a balance of 0, because the block it generated has not yet been confirmed by 100 additional blocks.

Let's add more blocks to the blockchain; run

```
bitcoin-cli -regtest generate 20
```

on each of the five node. Verify that the blockchain length seen on all nodes is 101 with 

```
bitcoin-cli -regtest getblockcount
```

Also, if you watch the end of the bitcoin log file on any of the nodes, 

```
tail --lines=100 ~/.bitcoin/regtest/debug.log
```
you should see "update tip" messages indicating that new blocks have been added to the end of the blockchain.

Now that there are 101 blocks in the blockchain, if you run

```
bitcoin-cli -regtest getbalance
```

on node-0 - the node that generated the first block - you should see that it has earned 50 bitcoins for its troubles. The other nodes still have a balance of zero, until the blocks they contributed are, in turn, confirmed.

If you run

```
bitcoin-cli -regtest listunspent
```

on node-0, you should see further details about its unspent bitcoins:

```
[
  {
    "txid": "bfb9adfbba1625a546dfe5c599e9cf7e5be086078441c636f035af39db6cad21",
    "vout": 0,
    "address": "n2ZKnMQYnefuqX4diKppT1bjUnHeen4rrn",
    "scriptPubKey": "2103c24d0c6f3a606f281dbbf0a8f1438c86b5f7762b5466efa241692c9247c574edac",
    "amount": 50.00000000,
    "confirmations": 101,
    "spendable": true
  }
]

```

### Fork the network

Now we are going to split our Bitcoin network, allow unconnected parts of the network to generate blocks, and see what happens when they reconnect.

On node-3, use

```
ifconfig
```

and look for the interface that has IP address "10.10.3.2 ". This is the interface that connects node-3 and node-4 to nodes 0-2. Find the name of the interface (e.g. eth1, eth2) and bring it down with

```
sudo ifconfig eth2 down
```

Generate some blocks on node-4, e.g.

```
bitcoin-cli -regtest generate 20  
```

Make a note of the output, e.g.

```
[
  "00da8527c32bd760e9cb013a5f510ab79f97f8aadd8ed726822e0bc201ad8bdd", 
  "3fd4570d7dcac3daa136789f47afc80e4c4030bfcb47f06d53717bd07fca6760", 
  "4a38c6a7e290e493e73b6fafe938b317bc78a26a4f19787dbe51130eb020c910", 
  "50fe171e3d7c7f237348a9cf1a00299900e9cb1b0965a607b0a6823b97dfbf86", 
  "16bbeb3c328d042069021025596fc32ec2d0bb21f8914087fcca8ccda8f68c95", 
  "756671548336ba9570f69497bf69b667afbc5d2ffde0f8244d459cce1930a2a3", 
  "2357563ce2beb4dba9338cac2549ffcf642ab77afc80a922b312c19d51029d27", 
  "34030f813a32bc5be168632efc68bccb4fba359fed4070edbe3fbb323885b1d8", 
  "7cb43a9327ffee018ac84a1b03b6376c1bb55db153d34a9417c311128a10ba85", 
  "34bf87b4e80b95812950a667240535b10494e14a0e9cef08c1769c392fabbfad", 
  "5220e1d2bd2988ab5ac97cc53286701fb1c2b0d13128fa63800a7d746d0b8064", 
  "23423dc33f8c20b109c4886e9b5ccdce91033ad263ce1d309468de689858ac3f", 
  "1b4c5418082561659276e03d7b622a004103cc2710df64e63e147d97f9b30686", 
  "1a285e611daf6befbbb17157470b107460df2cd7aa2629c89f9a8bee85296720", 
  "3b6173c8689006f506306b5e7fd831e6d3e7054d563d57881d00dd31e3a04fc7", 
  "5c9b08cb067a02b0df988c8bfdfd087536a0a58500331e343591dda6dde36b35", 
  "029b78163126873e49972547eb11f9a9282298264d3b72f7e617171ad9397c1a", 
  "545385392453c866ade3b50738f1bca4ba116c60c62ed31b49e03e99aa19b3bf", 
  "7c7bd51d0877b96dc640707cd923f923e3c595342db350b2538d26c76df21981", 
  "5eaf42996bc3062e96f096eee12131f34f1e48c137eb4cfe2e9cf75f5a4826fc"
]
```
Use 

```
bitcoin-cli -regtest getblockcount  
```

on both sides of the network. On node-3, you should see a blockchain length of 121, and if you 

```
tail -f .bitcoin/regtest/debug.log
``` 
you can see that it got exactly the blocks generated by node-4. On the other hand, node-0, node-1, and node-2 will still have a blockchain length of 101.

Now generate 20 blocks on the other side of the network. On node-0, run

```
bitcoin-cli -regtest generate 20  
```

and make a note of the output, e.g.

```
[
  "74de92645268b89305d4f2784de25c6d0fd53b872efff9a610c45f24f67676c4", 
  "5e70564189d75fe9b2c7d6d07e930dab853969379dac427e8d97922bdc16ac99", 
  "282fd323e1d67bee1852b1eae0040503aca237278b8c7c4ab1254a72f0518a4e", 
  "7d7026a24da640b9afbfc5cea886c3845f088874bd8ec0a3a2ce12c7e46622f8", 
  "4ef569b87223d44cf8d2abd8d09c886aeb4bdb7cecb5d4142d16a51fd1e1c8f1", 
  "4ad8e81c186fa6f1f79eca0174ad1f1e224e69b3fa9c6840c88bd1f6f22afe7a", 
  "1014d53ce3a9356b38f63a1598c7d169b1b02e014fb65930257c7c6363a8aba7", 
  "6fdb4893b26195554c0c27135c120a751c94ccee09f7cf785c51854e2b67804d", 
  "6b0d089063effc8079aa71aac77327df3ef82c67ab30d40137e4150a93afb381", 
  "2ad2950003aaac613760272f9299c457f3f394707a0f9366242d8a25c6716d67", 
  "1e32a3e677799c19b404d0d97a33df84e12517b3f33d7c6e358a332f387d610e", 
  "75e12f97a5b99b5220bfce9a5867567e0dced9e64d60fa70a57fd3c5342a2521", 
  "1e79ab1089dd5a365449fc84a2205f0114eaba7fc233970330f0cad1c1ce41b5", 
  "31fc274c17783bd8b9008df6259350bcf8dffbd38d308cbb1d632b9cb98bdf62", 
  "423a88799b1930bd41846e96596e56e5d271c6c0280724c3d768b17a52e9dc1c", 
  "3388476ef3d53503f09b3b5e4f9142f41544e280acf0814a8086859a13c0f082", 
  "42124b151e775bf3dee1b2c8e1de014b3ac0faa722bfac48e522f89672f8e750", 
  "416e5527c5129af23ff2f62c45bec55ef08cc5ac7e8242eba6ceb3666a38a95b", 
  "22d342715fd0cdd5572c8cda9d6dc9a46d228fe4cfb82e3f186b3c216ca56e90", 
  "4e5545b5f3bf4f57914c11cf835aeccc5fd51e38139d9c25c4ecd67904ea9869"
]
```

We now have a blockchain with 101 blocks in common between both sides of the network, followed by 20 different blocks. Let's reconnect the network. On node-3, run

```
sudo ifconfig eth2 up
```

substituting the name of the interface you brought down before.

All nodes in the network should still respond with "121" to 

```
bitcoin-cli -regtest getblockcount
```

however, the specific blocks in their blockchains will vary. Compare the 121st block in the blockchain on both sides with

```
bitcoin-cli -regtest getblockhash 121
```

Let's reconcile the conflict. On node-0, run

```
bitcoin-cli -regtest getblockhash 121
bitcoin-cli -regtest generate 1  
```


Now repeat

```
bitcoin-cli -regtest getblockcount
bitcoin-cli -regtest getblockhash 121
```

on all the nodes to see how the network has "agreed" on a specific blockchain, with identical blocks, and abandoned what has become the shorter blockchain.

### Bitcoin transactions

At this point, some of your Bitcoin nodes should have spendable bitcoins. Now we'll see how the network requires multiple confirmations, in order to guard against double spending.

On each node, check your balance with 

```
bitcoin-cli -regtest getbalance  
```

Now, we will send some bitcoins from node-0 to node-4.

On node-4, run

```
bitcoin-cli -regtest getnewaddress
```

to get a new Bitcoin address.

Then, on node-0, run

```
bitcoin-cli -regtest sendtoaddress ADDRESS 10.00
```


where ADDRESS is the address you generated in the previous step, to send 10 bitcoins to node-4. This command will return a transaction ID.

Now, if you run 

```
bitcoin-cli -regtest getbalance
```

on node-0, you should see that slightly more than 10 bitcoins have been deducted from this node's balance. This includes the 10 bitcoins that will be sent to node-4, and a small transaction fee.

However, if you run 

```
bitcoin-cli -regtest getbalance
```

on node-4, you will see that its balance has *not* increased. The transaction hasn't been embedded in the blockchain yet, so as far as the rest of the network is concerned, it doesn't exist. 

Let's generate six blocks on node-1 and see what this does with respect to our transaction. On node-1, run

```
bitcoin-cli -regtest generate 6
```

Now, on node-4, run

```
bitcoin-cli -regtest getbalance
```
and you should see that it has been incremented by 10.

The node that embedded the transaction in the blockchain, node-1, should have earned a transaction fee. However, this fee is not confirmed yet, so if you run 

```
bitcoin-cli -regtest getbalance
```

on node-1, it won't have earned that fee yet. However, if you run 

```
bitcoin-cli -regtest generate 100
```

on node-2, the block containing the transaction will be confirmed, and when you run


```
bitcoin-cli -regtest getbalance
```

on node-1, the transaction fee should now be reflected in its balance. You can also run

```
bitcoin-cli -regtest listunspent
```

on node-1, and look for the transaction for which it has earned a little more than 50 bitcoins, e.g.:

```
  {
    "txid": "efa54bb99286436ac5ecc3c1f83cf4937eabf621d75de2ada78360bb98b9a1ed",
    "vout": 0,
    "address": "mjbHJkmvcZVmvQ13qctGuXw7oo67v5nFnQ",
    "scriptPubKey": "2102ee35c878387d17d57094cddb3ca0120c6717f3fbc3b4742251b3073d4d530ad1ac",
    "amount": 50.00003840,
    "confirmations": 106,
    "spendable": true
  }, 
```


### Clean up

Please delete your resources in the GENI Portal when you're done, to free them up for other experimenters!
