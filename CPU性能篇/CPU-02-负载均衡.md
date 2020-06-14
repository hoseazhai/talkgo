#### 基础概念

##### 平均负载

- 平均负载
> 平均负载是指单位时间内，系统处于**可运行状态**和**不可中断状态**的平均进程数，也就是平均活跃进程数（直观上的理解就是单位时间内的活跃进程数，但它实际上是活跃进程数的指数衰减平均值）。

- 可运行状态的进程
> 可运行状态的进程是指正在使用 CPU 或者正在等待 CPU 的进程，也就是我们常用 ps 命令看到的，处于 R 状态（Running 或 Runnable）的进程。

- 不可中断状态的进程
> 不可中断状态的进程则是正处于内核态关键流程中的进程，并且这些流程是不可打断的，比如最常见的是等待硬件设备的 I/O 响应，也就是我们在 ps 命令中看到的 D 状态（Uninterruptible Sleep，也称为 Disk Sleep）的进程。

**不可中断状态实际上是系统对进程和硬件设备的一种保护机制**

##### CPU使用率
- CPU利用率
> CPU 使用率是单位时间内 CPU 繁忙情况的统计，跟平均负载并不一定完全对应。

##### 区别
> 平均负载是指单位时间内，处于可运行状态和不可中断状态的进程数。所以，它不仅包括了正在使用 CPU 的进程，还包括等待 CPU 和等待 I/O 的进程。

- CPU 密集型进程，使用大量 CPU 会导致平均负载升高，此时这两者是一致的；

- I/O 密集型进程，等待 I/O 也会导致平均负载升高，但 CPU 使用率不一定很高；

- 大量等待 CPU 的进程调度也会导致平均负载升高，此时的 CPU 使用率也会比较高。

#### 平均负载案例
案例环境 centos7.4
```shell
yum install stress
```
由于centos中的sysstat版本比较老推荐源码安装sysstat(来自文章评论区)
```shell
git clone --depth=50 --branch=master https://github.com/sysstat/sysstat.git sysstat/sysstat
cd sysstat/sysstat
git checkout -qf 6886152fb3af82376318c35eda416c3ce611121d
export TRAVIS_COMPILER=gcc
export CC=gcc
export CC_FOR_BUILD=gcc
./configure --disable-nls --prefix=/usr/local/
make &&make install
```

场景一：CPU 密集型
```shell
# 在三个窗口中执行以下命令
#查看时间，服务启动时间和系统一分钟五分钟十五分钟的平均负载
$ uptime
# 模拟一个 CPU 使用率 100% 的场景
$ stress --cpu 1 --timeout 600
# 实时监控 uptime
$ watch -d uptime
# -P [ -P { <cpu_list> | ALL } ]指定CPU列表或者监控所有的 CPU 的使用率情况，每五秒显示一次
$ mpstat -P ALL 5
# 查询进程 CPU、wait 占用情况，每五秒显示一次
$ pidstat -u 5 1
```
场景二：I/O 密集型进程
```shell

# 在三个窗口中执行以下命令

$ uptime
# 模拟 I/O 压力
$ stress -i 1 --timeout 600   # 此命令可能模拟不出来
$ stress-ng -i 1 --hdd 1 --timeout 600
# # 其他的参考前面场景一

```

场景三：大量进程的场景
```shell
# 在三个窗口中执行以下命令

$ uptime
# 当系统中运行进程超出 CPU 运行能力时，就会出现等待 CPU 的进程。
# 模拟 8 个进程
$ stress -c 8 --timeout 600
# 其他的参考前面场景一
```

**注意：当平均负载高于 CPU 数量 70% 的时候，你就应该分析排查负载高的问题了**