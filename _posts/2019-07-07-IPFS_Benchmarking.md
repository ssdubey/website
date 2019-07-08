IPFS (InterPlanetary file system) is a distributed file system built over peer to peer systems. It aims at creating a permanent web by involving peers to share data and make them available for other peers, instead of putting it at central location making it vulnerable to problems like Denial of service or Broken link issues. 

How does it work
IPFS uses trackerless bittorrent based protocol to communicate among the participating nodes. It uses Kademlia DHT protocol with some modifications influenced by S/Kademlia and Coral DSHT.
Every participating node would maintain an IPFS directory structure describing the backend datastore and other configurations. 
IPFS offers 3 different datastores in the backend:
File system
Level DB
Badger DB

How to switch between datastores:
File system is the default datastore provided by IPFS. (how fs works)
Level DB and badger DB are other key-value data stores. (some thing about kv stores compared to fs)
Describe the steps to create level db and badger db

IPFS also offers an option to become a part of public IPFS network or create your own private network. A private IPFS network would identify the peer nodes based on a swarm key, which would be unique to that private network. If any public node shares that swarm key, then all the hosts would get exposed to the public network.

Major operations supported by IPFS would be:
Add: `ipfs add` command would accept a file, a directory or an input from stdin. Further it generates a hash corresponding to the data and adds it.

Cat: `ipfs cat` command takes a hash as input and prints the content of file or stdin on the screen.

Every object added to IPFS are considered as IPLD objects. `ipfs get` and `ipfs put` commands are used to manipulate IPLD objects.

We are working on integrating Irmin library datastore with IPFS to use its Routing system and content addressability capabilities to make Distributed applications based on Irmin more efficient. Since there are multiple storage options available with IPFS, we wanted to use the most optimal one in terms of load balancing, throughput, etc. This article is about our observation on benchmarking the IPFS on different parameters.

We have used all the three datastore backends provided by the IPFS for all the experiments. Also we attempted to compare the performance of IPFS working on a limited number of Private node collection and Public DHT. 
In this article we would mention the observations of the benchmark and also some of the interesting features of IPFS which we discovered on the way. 
Please feel free to comment and let us know if you want to know further details of the experiment or find any discrepancy in our understanding. 

First parameter on our benchmark list is Size.
As mentioned before IPFS uses Bittorent like protocol for distributing data over the network. Typically bittorrent protocol divides the data into small blocks. By default the block size for IPFS is 256KB.
Our experiment makes the observation separately on data less than block size and more than that. Below are the graphical representation of our findings. 

<!---   
<img src="/assets/size/TC14to16/private_badger.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC14to16/public_badger.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
<B>Private Badger DB</B>   |  <B>Public Badger DB</B>        
:-------------------------:|:-------------------------:
<img src="/assets/size/TC14to16/private_fs.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC14to16/public_fs.png" alt="drawing" width="800"/> 
<B>Private FS</B> 		   |		<B>Public FS</B> 		   
:-------------------------:|:-------------------------:
<img src="/assets/size/TC14to16/private_level.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC14to16/public_level.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
<B>Private Level DB</B>    |  <B>Public Level DB</B>     
:-------------------------:|:-------------------------:
-->
<img src="/assets/size/TC1to9/BadgerDBPrivate.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC1to9/BadgerDBPublic.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
<B>Private Badger DB</B>   |  <B>Public Badger DB</B>        
:-------------------------:|:-------------------------:
<img src="/assets/size/TC1to9/FSPrivate.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC1to9/FSPublic.png" alt="drawing" width="800"/> 
<B>Private FS</B> 		   |		<B>Public FS</B> 		   
:-------------------------:|:-------------------------:
<img src="/assets/size/TC1to9/LevelDBPrivate.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC1to9/LevelDBPublic.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
<B>Private Level DB</B>    |  <B>Public Level DB</B>     
:-------------------------:|:-------------------------:      

Fig. above shows the average time taken by `ipfs add` operation by various datastores in private and public scopes. The vertical line on top of each bar represents the standard deviation from the average.|

The above graphs show the performance of IPFS when the input size is less than the block size. It can be observed that if there is a large difference between the file size, we can observe some differences in processing time of `ipfs add` command. The “not so significant” difference in time taken for execution of command indicates the presence of an overhead component as part of the command execution. We will talk about this overhead in the later experiments.

It is worth to note that even if some of the graphs, like public and private FS shows relatively more variations in the time duration of the `add` operation, the variation is still less than 100ms, which can be attributed to the non-ideal conditions of the experimental setup. The vertical lines showing standard deviation points out the range of variations in extreme cases. 

Overall, we can state that variation in size of data does not play any significant role in overall performance of IPFS.

Another noticable point is the magnitude of time taken by the datastores for the same size of the file. For both Badger DB and FS, the time taken is around 400 to 500 milliseconds, while the Level DB does the same within 40 to 50 milliseconds. 

One interesting quetion would be, if there is any difference in public and private datastores while adding the data. <B>DO IT</B>

Our next benchmark test over the size parameter is to check if the multiple `add` operations over the same data is optimized or not. 
In simple words, `ipfs add` command does two things internally. 
* It takes the data and calculates its hash
* Adds the hash into its keystore (**check the terminology**) and points it to the corresponding data. 
Calculating hash would not be optional but if the hash turns out to be same as any existing key, IPFS can avoid the second step. 

Below are the graphs from our second set of experiment. These are only a few graphs from our experiment. The whole set of output can be found on the github repo (<B>Refer it</B>)

<!---  Removing individual graphs of 11(a) because I think graphs of 11(b) are sufficient to deduce the conclusions of both observations. Graphs of 11(a) are still in the assets folder.
:-------------------------:|:-------------------------:
<img src="/assets/size/TC11(a)/PrivateBadger16B.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC11(a)/PrivateBadger512KB.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
<img src="/assets/size/TC11(a)/PrivateFS16B.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC11(a)/PrivateFS512KB.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
<img src="/assets/size/TC11(a)/PrivateLevel16B.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC11(a)/PrivateLevel512KB.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
-->
(**Scale down the level DB graph**)



<img src="/assets/size/TC11(b)/batch_16B.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC11(b)/batch_512KB.png" alt="drawing" width="800"/> 


The graphs presented above are representing the smallest and largest dataset we have worked on. Other samples of the graph which are not shown here also follow the similar pattern and can be verified at (**github repo address**)

Experiment no. in the graph refers to the number of times same data is pushed to the IPFS one after another. In badger DB and FS, a clear drop in the operation time can be observed from first attempt to second attempt and further. It indicates that the first attempt does more processing which is saved if the data is repeated again, clearly following our hypothesis mentioned above.

We have observed that the second step of adding the hash into the keystore (**check the terminology**) takes approximately 70% of time in Badger DB in private scope and 85% in public scope. (**remove this if operation is same in private and public**) 
Similarly the time taken in second step is around 90% and 87% in private and public scopes respectively for FS datastore.

Leveldb, however, does not follow the suit of other datastores and addition time for it is mearly 5% to 8% of total time. The overall performance is almost 10 times faster than others. 

With the above graph we can also compare the relative performance of each datastores. For the first run FS and Badger are comparable, but FS is relatively costlier. Subsequently, Badger is found to be relatively costlier than FS for repeated data. We cannot rule out the non-ideal conditions of computing environment for slight variations. 

(**mention somewhere that computing environment is containers**)

Level DB outruns both Badger and FS for the first run, but is almost comparable to FS for subsequent runs.

We can also observe from the above graphs showing a difference of size as big as 500KB or two IPFS blocks, that the effect of size on operation is almost negligible throughout the datastores.

Until now we saw that if the same data is added repeatedly to the IPFS, it is optimised and there is significant improvement in the performance. From the theory we also understood that like bittorrent, IPFS also divides the data into blocks. Our next experiment was to check if the optimization we observed above was also extended upto block level, and we were not disappointed. 

Below is the graph drawn for average time taken by each datastore to add some data. The dataset contains files of 400KB each (**change in graph**), in which one block or 256KB of data is same in each file and rest 144KB is 
* different for one set of experiment, and 
* same for another set of experiment.

(**perform experiment with 400KB of all different data and edit the graph**)


<img src="/assets/size/TC12/graph.png" alt="drawing" width="800"/> 

(**write the observation after modifying the graph as above**)

The next experiment is the extension of previous one in which we test how the diversity of blocks in a single data file affects the performance of IPFS over different datastores. The dataset contains 8 files of a kind, each containing data equivalent to one block size to 8 block sizes.  The files are of three kinds:
* Each block contains the same data. The data is also same in each file. It is represented in the graph as: A, AA, AAA ...
* Each block contains the same data, but it is different in each file. It is represented in the graphs as: A, BB, CCC ...
* Each block contains different data. It is represented in the graph as: A, BC, DEF ...

The correspoding graph is shown below:


<img src="/assets/size/TC14to16/private_badger.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC14to16/public_badger.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
<B>Private Badger DB</B>   |  <B>Public Badger DB</B>        
:-------------------------:|:-------------------------:
<img src="/assets/size/TC14to16/private_fs.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC14to16/public_fs.png" alt="drawing" width="800"/> 
<B>Private FS</B> 		   |		<B>Public FS</B> 		   
:-------------------------:|:-------------------------:
<img src="/assets/size/TC14to16/private_level.png" alt="drawing" width="800"/>  |  <img src="/assets/size/TC14to16/public_level.png" alt="drawing" width="800"/> 
:-------------------------:|:-------------------------:
<B>Private Level DB</B>    |  <B>Public Level DB</B>     
:-------------------------:|:-------------------------:

We can see the similar pattern in each of the datastores with varying magnitude which could again be attributed to the non-ideal experimental environment. The observation is inline with the previous experiment that, IPFS efficiently utilizes the data block which is already present with it. It greatly affects the efficiency and performance in case the datastore takes significant time searching, if the key is already present. Relatively fast datastores such as Level DB in this case, shows consistent behaviour for any level of diversity in the dataset.
