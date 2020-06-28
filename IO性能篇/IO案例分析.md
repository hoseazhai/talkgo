### 磁盘IO性能指标
IO性能指标如下：
![](https://static001.geekbang.org/resource/image/b6/20/b6d67150e471e1340a6f3c3dc3ba0120.png)
### 性能工具

根据IO性能指标对应的工具
![](https://static001.geekbang.org/resource/image/6f/98/6f26fa18a73458764fcda00212006698.png)

根据工具查找指标
![](https://static001.geekbang.org/resource/image/c4/e9/c48b6664c6d334695ed881d5047446e9.png)
分析方向图
![](https://static001.geekbang.org/resource/image/18/8a/1802a35475ee2755fb45aec55ed2d98a.png)

### 分析思路
IO性能问题分析

1. - 首先可以通过top 查看机器的整体负载情况，一般会出现CPU 的iowait 较高的现象

2. - 运用iostat确认磁盘IO的性能瓶颈

3. - 使用 pidstat -d 1 找到读写磁盘较高的进程

4. - 找到进程后通过 strace -f -TT 进行跟踪，查看系统读写调用的频率和时间

5. - 通过lsof 找到 strace 中的文件描述符对应的文件 opensnoop可以找到对应的问题位置

6. - 最后，结合应用程序的原理，分析这些 I/O 的来源
