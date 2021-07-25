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

* TODO
