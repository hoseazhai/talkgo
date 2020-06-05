#### 基础概念

- CPU使用率
>CPU 使用率是单位时间内 CPU 使用情况的统计，以百分比的方式展示

Linux 作为一个多任务操作系统，将每个 CPU 的时间划分为很短的时间片，再通过调度器轮流分配给各个任务使用，因此造成多任务同时运行的错觉

- 节拍率 HZ的由来
>为了维护 CPU 时间，Linux 通过事先定义的节拍率（内核中表示为 HZ），触发时间中断，并使用全局变量 Jiffies 记录了开机以来的节拍数。每发生一次时间中断，Jiffies 的值就加 1

- 查看节拍率
>为了维护 CPU 时间，Linux 通过事先定义的节拍率（内核中表示为 HZ），触发时间中断，并使用全局变量 Jiffies 记录了开机以来的节拍数。每发生一次时间中断，Jiffies 的值就加 1

##### top各个属性字段的介绍

- user（通常缩写为 us）
>代表用户态 CPU 时间。注意，它不包括下面的 nice 时间，但包括了 guest 时间。

- nice（通常缩写为 ni）
>代表低优先级用户态 CPU 时间，也就是进程的 nice 值被调整为 1-19 之间时的 CPU 时间。这里注意，nice 可取值范围是 -20 到 19，数值越大，优先级反而越低。

- system（通常缩写为 sys），
>代表内核态 CPU 时间。

- idle（通常缩写为 id）
>代表空闲时间。注意，它不包括等待 I/O 的时间（iowait）。

- iowait（通常缩写为 wa）
>代表等待 I/O 的 CPU 时间。

- irq（通常缩写为 hi）
>代表处理硬中断的 CPU 时间。softirq（通常缩写为 si），代表处理软中断的 CPU 时间。

- steal（通常缩写为 st）
>代表当系统运行在虚拟机中的时候，被其他虚拟机占用的 CPU 时间。
- guest（通常缩写为 guest），代表通过虚拟化运行其他操作系统的时间，也就是运行虚拟机的 CPU 时间。

- guest_nice（通常缩写为 gnice）
>代表以低优先级运行虚拟机的时间。

#### 性能工具
CPU百分比计算方法
为了计算 CPU 使用率，性能工具一般都会取间隔一段时间（比如 3 秒）的两次值，作差后，再计算出这段时间内的平均 CPU 使用率，即
![](/uploads/talkgo/images/m_d672380cdb1ea704ab466c18ed0ca44e_r.png)
性能分析工具给出的都是间隔一段时间的平均 CPU 使用率，所以要注意间隔时间的设置

- top
>按数字1可以查看每一个CPU的使用率

- pidstat

```shell

# 每隔1秒输出一组数据，共输出5组
$ pidstat 1 5
15:56:02      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
15:56:03        0     15006    0.00    0.99    0.00    0.00    0.99     1  dockerd

...

Average:      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
Average:        0     15006    0.00    0.99    0.00    0.00    0.99     -  dockerd

```

- perf
>使用 perf 分析 CPU 性能问题,可以查看指定函数的问题

 - 第一种常见用法
```shell
$ perf top
Samples: 833  of event 'cpu-clock', Event count (approx.): 97742399
Overhead  Shared Object       Symbol
   7.28%  perf                [.] 0x00000000001f78a4
   4.72%  [kernel]            [k] vsnprintf
   4.32%  [kernel]            [k] module_get_kallsym
   3.65%  [kernel]            [k] _raw_spin_unlock_irqrestore
...
```
执行信息分析介绍
>第一行包含三个数据，分别是采样数（Samples）、事件类型（event）和事件总数量（Event count

   - 第一列 Overhead
>是该符号的性能事件在所有采样中的比例，用百分比来表示。

   - 第二列 Shared
>是该函数或指令所在的动态共享对象（Dynamic Shared Object），如内核、进程名、动态链接库名、内核模块名等。

    - 第三列 Object
>是动态共享对象的类型。比如 [.] 表示用户空间的可执行程序、或者动态链接库，而 [k] 则表示内核空间。

     - 第四列 Symbol
>是符号名，也就是函数名。当函数名未知时，用十六进制的地址来表示。

 - 第二种常见用法

```shell
$ perf record # 按Ctrl+C终止采样
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.452 MB perf.data (6093 samples) ]

$ perf report # 展示类似于perf top的报告
```
用法介绍
>也就是 perf record 和 perf report。 perf top 虽然实时展示了系统的性能信息，但它的缺点是并不保存数据，也就无法用于离线或者后续的分析。而 perf record 则提供了保存数据的功能，保存后的数据，需要你用 perf report 解析展示,在实际使用中，我们还经常为 perf top 和 perf record 加上 -g 参数，开启调用关系的采样.

根据top获取进程号，再根据进程号获取函数
```shell
# -g开启调用关系分析，-p指定php-fpm的进程号21515
$ perf top -g -p 21515
```
查找代码
```shell
#针对php代码可以进行遍历查找
$ grep sqrt -r app/ #找到了sqrt调用
```
- 压测工具ab
```shell
# 并发10个请求测试Nginx性能，总共测试100个请求
$ ab -c 10 -n 100 http://192.168.0.10:10000/
This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, 
...
Requests per second:    11.63 [#/sec] (mean)
Time per request:       859.942 [ms] (mean)
...
```

#### 按照老师例子实验遇到的问题。解决方法摘自评论区(每天晒白牙)
> 我的系统是centos7，上次实战用 perf top -g -p pid没有看到函数名称，只能看到一堆十六进制的东西，然后老师给了解决方法，我转述下：
分析：当没有看到函数名称，只看到了十六进制符号，下面有Failed to open /usr/lib/x86_64-linux-gnu/libxml2.so.2.9.4, continuing without symbols 这说明perf无法找到待分析进程所依赖的库。这里只显示了一个，但其实依赖的库还有很多。这个问题其实是在分析Docker容器应用时经常会碰到的一个问题，因为容器应用所依赖的库都在镜像里面。

老师给了两个解决思路：
1. 在容器外面构建相同路径的依赖库。这种方法不推荐，一是因为找出这些依赖比较麻烦，更重要的是构建这些路径会污染虚拟机的环境。
2. 在容器外面把分析纪录保存下来，到容器里面再去查看结果，这样库和符号的路径就都是对的了。

操作：
（1）在Centos系统上运行 perf record -g -p <pid>，执行一会儿（比如15秒）按ctrl+c停止
（2）把生成的 perf.data（这个文件生成在执行命令的当前目录下，当然也可以通过查找它的路径 find | grep perf.data或 find / -name perf.data）文件拷贝到容器里面分析:
```shell
docker cp perf.data phpfpm:/tmp
docker exec -i -t phpfpm bash
$ cd /tmp/
$ apt-get update && apt-get install -y linux-perf linux-tools procps
$ perf_4.9 report
```

注意：
> 最后运行的工具名字是容器内部安装的版本 perf_4.9，而不是 perf 命令，这是因为 perf 会去跟内核的版本进行匹配，但镜像里面安装的perf版本有可能跟虚拟机的内核版本不一致。
注意：上面的问题只是在centos系统中有问题，ubuntu上没有这个问题


#### 思考
学习到的东西:

1. 体验排查思路
2. 运用top vmstat,mpstat,pidstat,perf等工具对系统排错，可以精确到某个函数。