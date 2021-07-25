# MIT 6.824 - Lecture 2 RPC and Threads

**MIT 6.824 Lecture 2 Note**
<!--more-->
## Thread
* Feature:
  * local stack.
  * local registers.
  * local sotrage.
  * program counter.
  * unshared code except for read-only code.
* Why thread:
  1. I/O Concurrency:
      * Able to send multiple request to the network and waiting for many replies at same time.
  2. CPU Parallelism:
      * threads can run truly in parallel.
      * Could use multiple CPU cycles per second. 
  3. Convenience:
      * unblocking main process while doing other jobs.
* Thread Challenge:
  * Race Condition
    * multiple threads doing: `n += 1`, where `n` is shared data.
  * Coordination
    * Channels - way of interaction between threads.
    * sync.Cond - good when multiple readers wait for the shared resources to be available.
    * sync.waitGroup - good for launching a known number of goroutines and waiting for them to finish.
  * Dead Lock
    * example - T1 holds Lock A, T2 holds Lock B, then T1 needs Lock B and T2 needs Lock A. 
