# Research Paper - MapReduce: Simplified Data Processing on Large Clusters

**Discuss MapReduce research paper**
<!--more-->
## Programming Model
* There are **two** functions for MapReduce library for computations:
  * Map function:
    1. takes a set of **input** key/value pairs.

    2. produces a set of **intermediate** key/value pairs.

    3. group all intermediate values associated with same intermediate key **I** and pass them to *Reduce* function,
  * Reduce function:
    1. Accept an intermediate key **I** and a set of values (usually supplied via an iterator so that it does not cost much memory) for that key.

    2. Merges together values to form a possibly smaller set of values (typically zero or one output).
* Example
  * Distributed Grep
  * Count of URL Access Frequency
  * Reverse Web-Link Graph
  * Distributed Sort
    ```pseudo
    map(String key, String value):
    // key: document name
    // value: document contents
    for each word w in value:
      EmitIntermediate(w, "1");
    reduce(String key, Iterator values):
      // key: a word
      // values: a list of counts
    int result = 0;
    for each v in values:
      result += ParseInt(v);
    Emit(AsString(result));
    ```
## Implementation
* Execution overview
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/mapreduce/execution.png "Execution overview")
  1. Mapreduce library in user program splits input files into **M** pieces (typically 16MB to 64MB),
    and start many copies of program on a cluster of machines.

  2. One of the copies of program is **master** and rest are **workers**. Master program assign works to workers.
    There are **M** map tasks and **R** reduce tasks.

  3. Worker who is assigned a map task reads and parses contents of input split into map function.
    Map function then produces **intermediate** key/values, which are buffered in **memory**.
  
  4. Periodically, the buffered pairs are written to the disk, partitioned into **R** regions.
    The location of regions are passed back to **master**.

  5. When a **reduce** worker is forwarded by **master** about regions location,
    it uses RPC to read buffered data from disk of different **map** workers.
    Then, it sorts data by **intermediate** key to eliminate multiple same keys.
  
  6. **Reduce** worker iterates over the sorted **intermediate** data.
    For each unique key, it passes set of values of the key to **reduce** function.
    The **reduce** function then outputs result to a file.

  7. When all **map** tasks and **reduce** tasks are done, the master worker wake up user program.
    **MapReduce** function call is finally returned.

* Master Data Structures
  * It stores the state of [map]/[reduce] tasks(**idle, in-progress or completed**).
  * It stores the identity of worker machine(for non-idle tasks).
  * It stores the location and size of **R** intermediate file regions for completed map tasks.
  * It sends information about intermediate file regions to workers that has **in-progress** reduce tasks.

* Fault Tolerance
  * Work Failure:
    * **Master** ping every **worker** periodically, and if master does not get response by a worker,
      master will rest all tasks that are **completed** or **in-progress** on worker machine to **idle** states.
    * The completed **map** tasks on failed machine will be re-executed as it stored on local machine. 
    * Reduce tasks are notified with the re-execution and redirection of read machine.

  * Master failure:
    * Abort MapReduce function if **master** failed.

* Locality
  * In order to save scare **Network bandwidth**, input data will be stored on local disks of machines.
  * The **[GFS](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/gfs-sosp2003.pdf)** will save serveral copies (typically **3**) of each split on different machines.

* Task Granularity
  * **master** must make **O(M+R)** scheduling decisions and keeps **O(M * R)** states in memory.

* Backup Tasks
  * When a **MapReduce** operation is close to completion, the **master** program backup execution of remaining **in-progress** tasks.   
  * Task is marked as **completed** whenever either the primary or the backup execution completes. 

***Reference: https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/mapreduce-osdi04.pdf***
