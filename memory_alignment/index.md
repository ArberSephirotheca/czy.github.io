# OS - Memory Alignment

**这篇文章讨论了内存对齐的原理**
<!--more-->
##  内存条的结构
- 每个内存条一面是一个 Rank
    - 每面Rank一般包含8个 Chips.
        - 每个Chips包含8个 Banks.
        - 计算机通过选择Banks的 **row** 和 **col** 来定位地址.

## 结构的优点
* 通过对8个 Chips 进行[并行]^(parallelism)操作，提高了内存访问效率.

##  结构的缺陷
* 虽然 [8 byte]^(8 字节) 物理上不是连续存在，但共用8个Chips的同一个地址.
* 这也导致了address只能是[8的倍数]^(%8=0).

## 解决方案
- CPU分两次读
    - 如果当用户想要从不属于Chips(这里是8)**倍数**的地址开始读取数据，CPU会读取[前后8个bytes的地址]^(16字节)来获得数据.
    - 但这也导致了性能的降低.


## 更优秀的解决方案
* 内存对齐
    * 为了保持高效的运行，编译器会把[不同类型]/[不同大小]的数据安排到合适的地点并占用合适的长度.
    - 内存对齐要求数据储存的地址是对其边界的**倍数**
        - `int32` 的对齐边界是 [4 bytes]^(4 字节)，所以它所在的地址一定是 [**4** 的倍数]^(%4=0).
        - `int16` 的对齐边界是 [2 bytes]^(2 字节)，所以它所在的地址一定是 [**2** 的倍数]^(%2=0).
* 举例:
    ```go
    type a struct{
        A int8
        B int16
        C int32
        D int64
    }

    type b struct{
        B int16
        A int8
        C int64
        D int32
    }
    ```
    虽然上面2个`struct`内容相同，但`data type`的排序却会导致[**后者**]^(b struct)占用较多的字节.
    * 第一种排序占用 [16 bytes]^(16 字节).
          |0x00|0x01|0x02|0x03|0x04|0x05|0x06|0x07|0x08|0x09|0xa0|0xa1|0xa2|0xa3|0xa4|0xa5|
          |:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
          | 8  |    | 16 | 16 | 32 | 32 | 32 | 32 | 64 | 64 | 64 | 64 | 64 | 64 | 64 | 64 |
          
    * 第二种排序占用 [24 bytes]^(24 字节).
          |0x00|0x01|0x02|0x03|0x04|0x05|0x06|0x07|0x08|0x09|0xa0|0xa1|0xa2|0xa3|0xa4|0xa5|0xa6|0xa7|0xa8|0xa9|0xb0|0xb1|0xb2|0xb3| 
          |:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
          | 16 | 16 | 8  |    |    |    |    |    | 64 | 64 | 64 | 64 | 64 | 64 | 64 |    |    |    |    |    | 32 | 32 | 32 | 32 |
* `Golang` 语言下可以用 `unsafe.sizeof` 来查看 `struct` 的大小.
<br/><br/>
<br/><br/>
***Reference:*** *https://www.bilibili.com/video/BV1hv411x7we?p=3*
