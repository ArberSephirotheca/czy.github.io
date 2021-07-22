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
      * 存放key-value的数据结构
      * 使用哈希表作为底层
      * 用拉链法(每个bucket有一个链表)来解决哈希冲突
      * 包含**2**个哈希表，是为了在扩容时，把rehash过后的key-value放到另外一个字典上，完成交换
      * rehash的过程是**渐进式**的，每次CRUD的时候，都会顺带做一点rehash的工作
  * 跳跃表
    ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/tree/master/redis/skiplist.png "Skip List")
      * Zest 的底层实现
      * 基于多指针有序列表
      * 在查找时，从上层指针开始查找，找到对应的区间之后再到下一层去查找。
      * 插入速度快，相比起红黑树等不需要旋转，支持无锁操作
      * 插入，删除，搜索都是 **O(logn)**

## 特点
* 速度快，因为数据存在内存中，并且底层kv使用hashmap来实现，查找和操作都是O(1)
* 单线程， 不用担心 **Race Condition**

## 主从配置


