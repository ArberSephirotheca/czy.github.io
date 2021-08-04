# Research Paper - GFS

**Discuss GFS research paper**
<!--more-->
## What is GFS
* A distributed file system for large distributed data-intensive application.

## Why GFS
* Big, fast 
* Global data - visit data across many data center.
* Sharding - split data into serveral servers for parallelism.
* Automatic recovery - recover data from failure automatically.

## Big Storage is hard
* Performance -> sharding(split data into serveral severs in order to operate in parallel)
* Faults -> Tolerance 
* Tolerance -> replication
* Replication -> Inconsistency
* Consistency -> Low performance

## Structure of GFS
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/GFS/read.jpg "GFS Architecture")

```
C1, C2, C3 ... Cn
  |
  |
  |
Master 
  |
  |
  |
Chunk server 1
Chunk server 2
Chunk server 3
...
``` 
* Chunk server:
  * Responsible for **reading** data.
  * Usually has **three** copies of same chunk on other chunkservers.

* Master data:
  * Responsible for **writing** data and controll chunkservers.
  * Store file, chunk namespaces, mapping of files to chunks, locations of each chunks' replcias in **RAM**.
  * file name -> array of chunk handles
  * hanlde -> list of chunkservers
              version number
              primary(what chunkserver is primary, what are copies)
              lease expiration 
  * Store log(operation such as write, read), checkpoint(version number) in **disk**.

## Read
1. name, offset -> Master
2. Master send chunk handle, list of servers 
3. Client read -> Chunkserver
4. Chunkserver return data -> client


## Write
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/GFS/write.jpg "Write Control and Data Flow")
1. Client asks Master for **primary** chunk which holds lease.
2. If no primary on master:
  1. Find **up to date** replicas.
  2. Pick one of them to be **primary**, other to be secondary.
  3. **Increment** version number.
  4. Tells primary, secondary the version number. 
  5. Master writes version number to disk.
3. Client sends data 
3. Primary picks offset.
4. All replicas include primary told to write appended record to offset.
5. If primary receives all 'yes' reply from secondary, primary return to client.
6. If primary failed to receives, client could restart from **step 1**.

## Consistency
* **consistence** - a file region is *consistent* if all clients will always see the same data.
* **defined** - a region is *defined* after a file data mutation if it is **consistent** and
  client will see what the mutation writes in its entirely.
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/GFS/region.png "File Region State After Mutation")

* Consistentcy on file/directory:
  * Master use **prefix compression** to make a lookup table mapping full pathnames to metadata (etc. `/d1/d2/.../dn`)
  * each master operation acquires a set of locks before runs.
    * example: when it invokes `/d1/d2/.../dn/leaf`, it acquires read-lock on directory `/d1/d2.../dn`.
      Then, it acquires either a read-lock or write-lock on `/d1/d2/.../dn/leaf`
    * locks are acquired by following order to prevent **deadlock**: 
      1. level by namespace tree
      2. lexicographically within same level

## Availability
* *Fast Recovery*
  * Both master and chunkservers are designed to restore their states and start in seconds.
* *Chunk Replication*
* *Master Replication*
  * Its operation log and checkpoints are replicated on multiple machines.
  * A mutation to the state is committed only after its log record has been
    flushed to disk locally on all replicas.
  * Once it fails, infrastructure outside GFS starts a new master process.
  * **shadow master** is used which provides read-only access when master process is down.

***Reference: https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/gfs-sosp2003.pdf***
