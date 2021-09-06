# Pingcap Project - Tinykv

**记录下做tinykv的想法**
<!--more-->

## 高可用分布式储存系统*必要*的需求:
* 合理的副本数量
* 分配副本到不同的机器上
* 加入新节点后，要将其他节点上的副本迁移过来
* 节点下线后，要将下线节点的数据迁移

## 高可用分布式储存系统*可选*的需求:
* 维持每个集群的 Leader 分布均匀
* 维持每个节点的储存容量均匀
* 维持访问热点分布均匀
* 管理 Balance 的速度，避免影响服务
* 管理节点状态，包括上线/下线节点，以及自动下线失效节点


## Project 1
* Create DB first.
* CF is a key namespace where seperates same keys in different key namesapce.
* DB is based on **badger**.
        * It uses MVCC.
        * It runs transactions concurrently.
        * It has serializable snapshot isolation guarantees.
        * It uses LSM tree with value log to seperate keys from values.
* Each read/write opens a new transcation.
* Discard() after each transaction

## Project 2
* A:
  * Initialize the new raft.
  * Handle step to deal about state change and message received/sent.
  * Each time a follower/candidate encounters election timeout, it becomes a candidate to start a new election, the term + 1.
  * voter use first-come-first-served rule, which only votes once in a single term.
  * Use randomnized election timeout to prevent conflict that many followers become candiate at the same time.
  * Remeber to clean the votes map for each term raft becomes a candidate

* B:
  *   


## Reference
* https://pingcap.com/blog-cn/tidb-internal-1/
* https://pingcap.com/blog-cn/tidb-internal-3/.

