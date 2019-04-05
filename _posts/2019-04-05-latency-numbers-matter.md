---
layout: post
title:  "Latency Numbers Matter"
date: 2019-04-05 14:06:26 +0800
categories: blog
---

在优化存储系统的读写延迟的过程，要经常做 benchmark 跑数据。有了监控等数据就要进行分析，找出性能瓶颈。这时对计算机系统的各种操作访问的延迟数据的敏感性就非常重要。这里就根据 jeff dean 的 "what are the numbers that every computer engineer should know"，对这些数据进行简要分析。

<!--description-->

## latency numbers 
```
Latency Comparison Numbers (2012)
----------------------------------
 CPU cycle                                 0.42 ns (2.4 GHz)
 L1 cache reference                           0.5 ns
 Branch mispredict                            5   ns
 L2 cache reference                           7   ns                      14x L1 cache
 L3 cache reference                          20   ns (not accurate)
 Mutex lock/unlock                           25   ns
 Main memory reference                      100   ns    20x L2 cache, 200x L1 cache
 Context switch                           1,500   ns      1.5 us
 Compress 1K bytes with Zippy             3,000   ns        3 us
 Send 1K bytes over 1 Gbps network       10,000   ns       10 us
 Read 4K randomly from SSD              150,000   ns      150 us          ~1GB/sec SSD
 Read 1 MB sequentially from memory     250,000   ns      250 us
 Round trip within same datacenter      500,000   ns      500 us
 Read 1 MB sequentially from SSD      1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
 Disk seek                           10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
 Read 1 MB sequentially from disk    20,000,000   ns   20,000 us   20 ms  80x memory, 20X SSD
 Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms
 Physical System reboot                                                   2 min
```

- CPU 的速度看主频，如 "Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz"，主频 2.40 GHz. 为了让 CPU 发挥最大性能，请将 CPUfreq 调节器模式设置为 performance 模式，见 [CPUFREQ](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/power_management_guide/cpufreq_governors)。
- L1 缓存与 CPU 速度相当，这里能看出缓存的重要性，程序本身的 locality 特性加上指令层级上的优化，cache 访问的命中率很高，这最终能极大提高效率。L1 分指令和数据两个 Cache.
- 分支预测错误耗时是正常的 10 倍，需优化代码来降低分支预测错误的概率，如 [Stack overflow 问题](http://stackoverflow.com/questions/11227809/why-is-it-faster-to-process-a-sorted-array-than-an-unsorted-array)。这里与编译器优化和 CPU 指令设计联系密切。
- L2 也是 L1 的 10 倍左右。L1 和 L2 是用于单个 core 的。
- L3 ，目前已集成到 CPU 内部。在 Intel 处理器上，L1 是 4 个指令周期，L2 是 12 个，L3 是 26～31 个。L3 是一个 CPU 内的多个 core 共用的。可见[多级 CPU 缓存的作用](https://fgiesen.wordpress.com/2016/08/07/why-do-cpus-have-multiple-cache-levels/)
- 互斥锁的加解锁是需要 CPU 指令支持的，尤其是多核和多路处理器。需要结合操作系统来提供同步互斥机制，在并发编程中是很耗时的。
- 内存寻址（包括读取一个基本单元）。对于 NUMA 架构中（即多路服务器上），要考虑本地内存和远程内存的访问延迟不同。
- CPU 上下文切换（系统调用），估算参考 [文章](https://blog.tsunanet.net/2010/11/how-long-does-it-take-to-make-context.html)。上下文切换恐怖的事情在于，这段时间里 CPU 没有做任何有用的计算，只是切换了两个不同进程的寄存器、缓存和内存状态；而且这个过程还破坏了缓存，让后续的计算更加耗时。
- 压缩数据，考虑用计算换空间的时间代价。可看看其他常用压缩算法，如 Snappy 和 zstd。
- 千兆网络延迟，可考虑万兆网络及用户态网络驱动。以及网络的测试工具 iperf。
- SSD 的随机读取和顺序读取差别不大，可区分 sata ssd 和 pcie/nvme ssd。
- 从 Memory 中读取数据块。
- 数据中心内的延迟。考虑同城数据中心的延迟，idc 专线的带宽和延迟。
- 从 SSD 中读取大量数据。
- 机械硬盘寻址，把磁头移动到正确的磁道和扇区位置，然后才能读取指定扇区的内容。它虽然很浪费时间，但其实并没有办任何正事（读取磁盘内容）。
- 从 HDD 读大量数据。
- 异地数据中心或跨地域网络访问。世界上不同城市之间的网络来回，见[世界各地的 ping 报文时间](https://wondernetwork.com/pings/)。所有的程序和系统架构都会尽量避免进行不同城市甚至是跨国家的网络访问，CDN 就是一个解决方案，让用户和最接近自己的服务器进行交互，从而减少报文的网络传输延迟。
- 系统重启。


## 参考资料
- [What Every Programmer Should Know About Memory](https://www.akkadia.org/drepper/cpumemory.pdf)
- [Getting Physical With Memory](http://duartes.org/gustavo/blog/post/getting-physical-with-memory/)
