#### free 数据的来源

- 查询手册技能

用 man 命令查询 free 的文档，就可以找到对应指标的详细说明。比如，我们执行 man free ，就可以看到下面这个界面。

```
      buffers
              Memory used by kernel buffers (Buffers in /proc/meminfo)

       cache  Memory used by the page cache and slabs (Cached and Slab in /proc/meminfo)

       buff/cache
              Sum of buffers and cache
```

从 free 的手册中，你可以看到 buffer 和 cache 的说明。

 - Buffers 是内核缓冲区用到的内存，对应的是 /proc/meminfo 中的 Buffers 值。
 - Cache 是内核页缓存和 Slab 用到的内存，对应的是 /proc/meminfo 中的 Cached 与 SReclaimable 之和。

#### proc文件系统
/proc 是 Linux 内核提供的一种特殊文件系统，是用户跟内核交互的接口

- 查询手册技能
执行 man proc ，你就可以得到 proc 文件系统的详细文档。注意这个文档比较长，你最好搜索一下（比如搜索 meminfo）


```
Buffers %lu
    Relatively temporary storage for raw disk blocks that shouldn't get tremendously large (20MB or so).

Cached %lu
   In-memory cache for files read from the disk (the page cache).  Doesn't include SwapCached.
...
SReclaimable %lu (since Linux 2.6.19)
    Part of Slab, that might be reclaimed, such as caches.
    
SUnreclaim %lu (since Linux 2.6.19)
    Part of Slab, that cannot be reclaimed on memory pressure.
```

- Buffers 是对原始磁盘块的临时存储，也就是用来缓存磁盘的数据，通常不会特别大（20MB 左右）。这样，内核就可以把分散的写集中起来，统一优化磁盘的写入，比如可以把多次小的写合并成单次大的写等等。

- Cached 是从磁盘读取文件的页缓存，也就是用来缓存从文件读取的数据。这样，下次访问这些文件数据时，就可以直接从内存中快速获取，而不需要再次访问缓慢的磁盘。

- SReclaimable 是 Slab 的一部分。Slab 包括两部分，其中的可回收部分，用 SReclaimable 记录；而不可回收部分，用 SUnreclaim 记录。

#### 测试磁盘io

```shell
# 清理文件页、目录项、Inodes等各种缓存
$ echo 3 > /proc/sys/vm/drop_caches

```
```shell

# 每隔1秒输出1组数据
$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
0  0      0 7743608   1112  92168    0    0     0     0   52  152  0  1 100  0  0
 0  0      0 7743608   1112  92168    0    0     0     0   36   92  0  0 100  0  0
```


- buff 和 cache 就是我们前面看到的 Buffers 和 Cache，单位是 KB。

- bi 和 bo 则分别表示块设备读取和写入的大小，单位为块 / 秒。因为 Linux 中块的大小是 1KB，所以这个单位也就等价于 KB/s。


测试写入
```shell
#执行 dd 命令，通过读取随机设备，生成一个 500MB 大小的文件：
$ dd if=/dev/urandom of=/tmp/file bs=1M count=500

# 首先清理缓存
$ echo 3 > /proc/sys/vm/drop_caches
# 然后运行dd命令向磁盘分区/dev/sdb1写入2G数据
$ dd if=/dev/urandom of=/dev/sdb1 bs=1M count=2048
```
测试读取
```shell

# 首先清理缓存
$ echo 3 > /proc/sys/vm/drop_caches
# 运行dd命令读取文件数据,清理缓存后，从文件 /tmp/file 中，读取数据写入空设备：
$ dd if=/tmp/file of=/dev/null


# 首先清理缓存
$ echo 3 > /proc/sys/vm/drop_caches
# 运行dd命令读取文件
$ dd if=/dev/sda1 of=/dev/null bs=1M count=1024
```
总结
**Buffer 是对磁盘数据的缓存，而 Cache 是文件数据的缓存，它们既会用在读请求中，也会用在写请求中。**

##### 金句:
在排查性能问题时，由于各种资源的性能指标太多，我们不可能记住所有指标的详细含义。那么，准确高效的手段——查文档，就非常重要了。