# Redis

**这篇文章讨论了redis的基础数据结构和特点**
<!--more-->
## 数据类型
* String
* List
    * Linked Lists.
* Set
    * collection of **unique**, **unsorted** elemnts.
* Hash
    * 
* Zest

## 数据结构
* 字典
    * 存放key-value的数据结构.
    * 使用哈希表作为底层.
    * 用拉链法(每个bucket有一个链表)来解决哈希冲突.
    * 包含**2**个哈希表，是为了在扩容时，把rehash过后的key-value放到另外一个字典上，完成交换.
    * rehash的过程是**渐进式**的，每次CRUD的时候，都会顺带做一点rehash的工作.
* 跳跃表
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/redis/skiplist.png "Skip List")
  * Zest 的底层实现.
  * 基于多指针有序列表.
  * 在查找时，从上层指针开始查找，找到对应的区间之后再到下一层去查找.
  * 插入速度快，相比起红黑树等不需要旋转，支持无锁操作.
  * 插入，删除，搜索都是 **O(logn)**.

## 特点
* 速度快，因为数据存在内存中，并且底层kv使用hashmap来实现，查找和操作都是O(1).
* 单线程， 不用担心 **Race Condition**.

## 集群
* 主从复制
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/redis/master_slave.png "master-slave replication")
  * redis 提供了 [复制]^(replication)功能，可以实现当**master**数据库更新后，自动将更新的数据同步到其他**slaves**数据库中.
  * **master**数据库可以进行**读写**操作.
  * **slave**数据库只能进行**读**操作.
  * 优点:
    * 容灾恢复.
    * 读写分离，分担**master**读写的压力.
    * 在同步期间，**master**和**slave**服务器是非阻塞(non-blocking)，客户端可以操作.
  * 缺点:
    * Redis不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败.
    * Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂.
* [哨兵]^(Sentinel)模式
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/redis/sentinel.png "sentinel replication")
  * 哨兵是独立进程，通过向redis服务器发送request并等待response来监控运行多个redis实例.
  * 当哨兵检测到**master**服务器宕机，就会将其中一个**slave**服务器切换成**master**并通知其他服务器.
  * 优点:
    * 主从可以自动切换，系统更健壮，可用性更高.
  * 缺点:
    * Redis较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂.
* [集群]^(Cluster)模式
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/redis/cluster.png "cluster")
  * 在每台redis节点上储存**不同**的数据.
  * 服务器之间互相连接，一个服务器可以访问任意一个服务器.
  * 采用**hash slot**来分配节点地址.
    * redis cluster 有16384个hash slot, 通过对每个key进行 mod 16384来决定节点的位置.
    * 每个节点负责一部分的hash slot.

## 同步
* **slave**服务器向**master**服务器发送sync命令.
* 收到sync命令后，**master**服务器执行bgsave命令，用来生成rdb文件，并在一个缓冲区中记录现在开始执行的写命.
* bgsave执行完后，发送给**slave**服务器使其更新数据.
* **master**服务器再将缓冲区记录的写命令发送给从服务器，**slave**服务器执行完这些写命令后，此时的数据库状态便和**master**服务器一致了.
## 持久化
* redis因为是内存型rdb，为了保证数据不丢失，会在一定频率下把内存数据保存到硬盘当中.
* 可用在AOF文件写指令来设置保存数据的频率(etc. always, everysec, no).
