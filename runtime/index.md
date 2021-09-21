# Runtime

**Talks about Golang Runtime**
<!--more-->
## Runtime
* Golang Runtime 是 Go语言运行所需要的基础设施:
    1. 协程调度，内存分配，GC.
    2. 操作系统及CPU相关的操作的封装.
    3. pprof, trace, race 检测.
    4. map, channel, string等内置类型及反射的发现.

## 调度
* Goroutine 实现:
    * 线程的运行是通过调度队列切换不同的执行流，而goroutine就是执行流.
    * Goroutine的结构体:
        ```go
        type g struct {
            goid int64 // 协程的id
            status uint32 // 协程的状态
            stack struct{
                lo uintptr // 协程拥有的栈低位
                hi uintptr // 协程拥有的栈高位
            }
            sched gobuf // 切换时保存的上下文
            startfunc uintptr // 程序地址
        }

        type gobuf struct {
            sp uintptr // 栈指针位置
            pc uintptr // 运行到的程序位置
        }
        ```
        每个 go func 都会编译成 runtime.newproc 函数, 最终有一个 runtime.g 对象放入调度队列.  函数的指针设置在 runtime.g 的 startfunc 字段, 参数会在 newproc 函数里拷贝到    stack 中, sched 用于保存协程切换时的 pc 位置和栈位置.
    * Processor的结构体:
        ```go
        type p struct {
            id int32
            status uint32
            m muintpr // 与之关联的m
            mcache *mcache // per-p分配的cache
            runqhead uint32 
            runqtail uint32
            runnext guintpr
            gFree struct {
                gList
                n int32
            }
            palloc persistentAlloc // per-P, 用于分配一些runtime里的特殊结构.
            gcw gcWork
        }
        ```
    * machine的结构体:
        ```go
        type m struct {
            g0 *g // 每个m绑定的g
            procid uint64 // 底层的线程id
            tls [6]uint64 // 传给FS寄存器的线程局部变量
            mstartfn func() // m启动时的函数，传给clone
            curg *g // 现在运行的代码的g
            p puintptr // 在运行代码时绑定的p
            id int64
            spinning bool // m找不到可运行的g，spin
            mcache *mcache // 运行代码时绑定的p中的mcache
            lockedg guintptr // 是否与某个g一直绑定
        }
        ```
## GC
 * 三色标记法:
    1. 一开始所有对象都是白色
    2. 从root开始标记，把所有可达到的对象标记为灰色
    3. 从灰色对象集合取出对象并标记灰色，把自己标记为黑色
    4. 重复第三步直到灰色集合是空
    5. 剩下的白色就是不可到达的对象

* 写屏障:
    * 万一出现漏标记对象的现象：A指向nil并被标记，B指向C，A被标记后指向C，B指向nil，这样B被标记的时候就忽略了C，所以我们有了写屏障:
        * 强三色不变性 — 黑色对象不会指向白色对象，只会指向灰色对象或者黑色对象
        * 弱三色不变性 — 黑色对象指向的白色对象必须包含一条从灰色对象经由多个白色对象的可达路径
    * 插入写屏障:
        * 如果两个对象之间新建立引用，那么引用(黑色)指向的对象就会被标记为灰色以满足强三色不变性，这是一种相对保守的屏障技术。
    * 删除屏障:
       * 如果一个灰色对象指向一个白色对象的引用被删除，那么在删除之前写屏障检测到内存变化，就会把这个白色对象标灰
