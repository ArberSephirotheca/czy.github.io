# Research Paper - HDFS

**Discuss HDFS research paper**
<!--more-->
## What is HDFS
* HDFS is a file system component of Hadoop, which store metadata and application data separately.

## Why HDFS
* HDFS is designed to store very large data sets reliably, and to stream those data sets at high bandwidth to user applications.
* By distributing storage and computation across many servers, the resource can grow with demand while remaining economical at every size.

## Architecture
* **NameNode**:
    * maintain the **namespace tree** and mapping of file blocks(phyiscal address) to DataNodes.
    * namesapce is a hierarchy of files and directories, and is stored in **RAM**.
    * Files and directories are represented by *Inode*, which record permission, modification and access time.

* **DataNode**:
  * Two files: one store **application data** and another one store **metadata**.
  * File content is split into large blocks and each block is replicated at multiple DataNodes.
  * Intially with no *namespace ID*, and able to join cluster and receive the cluster's *namespace ID*.
  * After joining a NameNode for a first time, DataNode registers a unique *storage ID*.  
  * Each time DataNode connects to a NameNode, *namespace ID* and *software version* will be verified, DataNode will be shutting down if either of them match.
  * Send *Hearbeat message* periodically to NameNode to confirm that DataNode is operating.
  
* **HDFS Client**:
 * User application that accesses file system.
 * Process of reading data:
   1. Asks NameNode for list of DataNodes.
   2. Connect to a DataNode and request the transfer of the desired block.
   3. **Pipelines** data to the chosen DataNodes.
   4. Confirm the creation of the block replicas to the NameNode.
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/hdfs/structure.png "Client writing data")

## Block Placement
* HDFS replica placement policy:
  1. No Datanode contains more than one replica of any block.
  2. No rack contains more than two replicas of the same block.
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/hdfs/cluster.png "Cluster Topology")


## Consistency
* Write:
  * HDFS maintains consistency  by allowing only **single** client at a time to write the file using **Write Leases**, but it does prevent other clients from reading the file.
* Read:
  * Data written to the current block is not guaranteed to be visual to other clients, which read the content of the file. Only when the writing client concluded its write operation and closed the file, the new content is visible for sure.

## Availability
* A DataNode stores checksums in a metadata file separate from the block’s data file. When HDFS reads a file, each
  block’s data and checksums are shipped to the client. The client then can fetch a difference replica once the checksum is not right.
* However, HDFS does not support the case of multiple correlated failures (ie. three failed DataNodes with same HDFS block)

## Parititon tolerance
* When write, Client will wait for ACK from replicas, and the transaction will be successful if the number of DataNodes that acknowledge 
  the reception of all the packages is equal or greater than minimum number of replicas.
* The NameNode endeavors to ensure that each block always
  has the intended number of replicas.
* The NameNode make sures that not all replicas of a block are located on one **rack** using block placement policy.
