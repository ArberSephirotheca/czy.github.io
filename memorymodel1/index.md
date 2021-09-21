# Hardware Memory Models

**Memory Model Part1**
<!--more-->
## Introduction
* 假设我们有程序:
  ```c
  // Thread 1           // Thread2
  x = 1;                    while(done == 0)
  done = 1;                 print(x);
  ```
* 如果 thread1 和 thread2 各自在自己的processor上运行。运行完成后，程序会输出0吗？
* 输出的结果取决于hardware和compiler:
  - 如果是**x86**构架，就会输出1.
  - 如果是**ARM**或者**POWER** 构架，就会输出0.
  - 不管hardware是什么，compiler的优化都会使得程序输出0或者进行无限循环.
* 在多线程情况下，程序的执行结果在不同的平台有可能会造成不同的结果.
* 所以我们需要 *memory model* 来定义*hardware, compiler, programming language*操作内存时所遵循的模型.
* 本篇主要讨论的是 *hardware* 的 *memory model*.

## Sequential Consistency
* *Lesile Lamport* 曾提出了一个概念 *sequential consistency*:
  - 每个线程内部的指令都是按照程序规定的顺序
  - 线程执行的交错顺序可以是任意的，但是所有线程所看见的整个程序的总体执行顺序都是一样的.
* SC的实现模型之一:
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/memorymodel1/mem-sc.png "Sequential Consistency")
* 这里的模型实现了**强顺序一致性**，每次当一个处理器进行 write/read 请求，这个请求就会跳转到共享内存中。 所以内存的访问顺序决定了执行的顺序，达到了顺序一致性.
* 但强顺序一致性会导致硬件的性能变差，所以现代硬件都偏离了SC.

## x86 Total Store Order
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/memorymodel1/mem-tso.png "x86-TSO")
**什么是TSO?**:
* TSO guarantees that the sequence in which store, FLUSH, and atomic load-store instructions appear in memory for a given processor is identical to the sequence in which they  were issued by the processor.

**x86如何实现TSO?**:
* 所有 processors 都连接到同一个 shared memory中.
* 每次 write 都会加入到本地的 write queue 中，每当 一个 write 从本地的 write queue 移动到 shared memory 中， 处理器就执行一个新的指令.
* 每次 read 访问 shared memory 之前都会询问本地的 write queue，作用是 processor 可以先于其他 processor 看到自己的 write.
* 每当一个write到达 shared memory 之后，所有之后的read都会看到并且使用write的数值.
* Write queue 遵循了 FIFO rule.
* Example:
  1. *message passing*: 以下程序能否看见 *r1 = 1*, *r2= 0*?
  ```c
    // Thread 1         // Thread 2
    x = 1                   r1 = y
    y = 1                  r2 = x
  ```
    - SC: no
    - TSO: no
    - SC 和 TSO 都保持**相同**的结果.

    - *x = 1* 必须发生在 *y = 1* 之前，所以 *r1 = y* 不可能在 *r2 = x* 之前见到 *y* 的新值.

  2.  *write queue*: 以下程序能否看见: *r1 = 0*, *r2 = 0*?
  ```c
    // Thread 1         // Thread 2
    x = 1                   y = 1
    r1 = y                  r2 = x
    ```
    - SC: no
    - TSO: yes
    - SC 和 TSO 保持**不同**的结果.

    - 在 SC 中: *x = 1* 或者 *y = 1*  必须先于 read 发生，所以 *r1* 和 *r2* 不可能同时为 0.
    - 在 TSO 中:  thread1 和 thread2 会把各自的 write 放到 write queue 中. read时，有可能 write 还没有从 write queue 放到 shared memory 中， 所以 *r1* 或者 *r2* 有可能同时为 0.

    - 那如何保证x86遵行 Sequential Consistency呢?: **Write Barrier**
      - Write barrier 保证每个线程在 read 操作之前都会把 write 置入 shared memory 当中. 
      ```c
        // Thread 1         // Thread 2
      x = 1                   y = 1
      barrier                 barrier
      r1 = y                  r2 = x
      ```
      - barrier 保证了 *r1* 和 *r2* 必须在前面2个 write 操作置入 shared memory 之后，才执行 read. 
  
  3. *Independent reads of independent writes*以下程序能否看见 *r1 =1, r2 = 0, r3 = 1, r4 = 0*? (Thread 3 和 Thread 4 是否会见到 不同的 *x* 和 *y*)
  ```c
  // Thread 1   // Thread 2    // Thread 3    // Thread 4
  x = 1             y = 1              r1 = x           r3 = y
                                             r2 = y           r4 = x
  ```
  - SC: no
  - TSO: no
  
  - 因为是 *Total Store Order*，所以所有 processors 都同意这个顺序.

## ARM/POWER Relaxed Memory Model
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/memorymodel1/mem-weak.png "ARM/POWER Relaxed Memory Model")
* 每个 processor 的 read/write 都是在 local memory 中发生.
* 每个 write 操作都会传递给其他 processor 用于同步.
* ARM/POWER没有实现TSO:
  - 每个 read 的操作可被推迟，直到需要 read 的结果.
* Example:
  1. *message passing*: 以下程序能否见到 *r1 = 1, r2 = 0*?
  ```c
  // Thread 1         // Thread 2
  x = 1                   r1 = y
  y = 1                   r2 = x
  ```
  - SC: no
  - TSO: no
  - ARM/POWER: yes

  - 如果 thread 1 在向 thread 2 传递 *x=1* 的写操作之前先传递了 *y=1* 的写操作，并且 thread 2 在 2个写操作传递之间完成了，就会出现上述的结果.

  2. *load buffer*: 以下程序能否见到 *r1 = 0, r2 = 0*
```c
// Thread 1         // Thread 2
x = x                   y = 1
r1 = y                 r2 = x
```
  - SC: no
  - TSO: yes
  - ARM: yes
  
  - 如果对 *x* 和 *y* 的写操作储存到了 local memory, 但并没有传递到其他 processor 上，就会出现上述结果.


## Weak Ordering and Data-Race-Free Sequential Consistency
**什么是 Weak ordering?**:
  - Let a synchronization model be a set of constraints on memory accesses that specify how and when synchronization needs to be done.
  - Hardware is weakly ordered with respect to a synchronization model if and only if it appears sequentially consistent to all software that obey the synchronization model.

Adve 和 Hill 提出了一种 synchronization model: *data-race-free*, 假设了 hardware 中除了写操作和读操作, 还有 synchronization operations 用于对 write 和 read 重新排序，从而达到顺序一致性.
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/memorymodel1/mem-adve-2.png "Data-Race Before Synchronization")
  上面的图片中，我们无法保证两个写作操的顺序.
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/memorymodel1/mem-adve-3.png "Data-Race After Synchronization")
  通过 synchronization variable *a* 我们可以强制使得写操作遵循某种顺序.

  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/memorymodel1/mem-adve-4.png "Data-Race Before Assigns a Intermediate Thread")
  上面的图片中，即使使用了 synchronization variablel, 也无法消除读操作产生的 race condition.
  ![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/memorymodel1/mem-adve-5.png "Data-Race After Assigns a Intermediate Thread")
  我们可以通过在中间加入一个 intermedaite thread 来解决这个问题.

总而言之, Adve 和 Hill 通过 *data-race-free* 提出 *weaking ordering* 就像是软件和硬件之间的协议. 为了防止 data-race 的产生，硬件通过一系列的限制来使得软件的执行结果跟遵循了Sequential Order后的执行是一模一样的.
