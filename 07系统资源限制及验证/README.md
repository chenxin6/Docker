# 系统资源限制及验证

在 docker run 或者 docker create 的时候设置资源限制

## 内存相关的参数说明

| --memory-swap | --memory | 功能 |
| ----- | ----- | ----- |
| 正数 S | 正数 M | 容器可用总空间为 S，其中 ram 为 M，swap为 S - M，若 S = M，则无可用 swap 资源 |
| 0 | 正数 M | 相当于未设置 swap 即为 unset |
| unset | 正数 M | 若主机 Docker Host 启用了 swap，则容器的可用 swap 为两倍的 M |
| -1 | 正数 M | 若主机 Docker Host 启用了 swap，则容器可使用最大至主机上的所有 swap 空间的 swap 资源 |

**注意：在容器内使用 free 命令可以查看到的 swap 空间并不具有其所展现出的空间指示意义**

--oom-kill-disable 不允许该容器因为 OOM 而被杀掉，使用该参数前必须设置好前面所说的两个参数，否则的话宿主机为了保持这个容器的正常运行会去杀死宿主机的一些系统进程，从而去释放内存养这个无法因为 OOM 而被杀死的容器。

--memory-swappiness 数值从0到100，0表示能不用交换分区就不用交换分区，100表示能使用交换分区就使用交换分区。可以理解为交换分区 swap 和物理分区 ram 的使用比例。

--memory-reservation 如果 Docker 发现宿主机内存资源紧张时，在系统的下次内存回收时，系统会回收容器的部分内存页，强迫容器的内存占用回到 --memory-reservation 设置的值大小。

--kernel-memory 容器能够使用的 kernel memory 大小，最小值为 4m。

## CPU 相关的参数说明

--cpu-shares CPU 共享权值（相对权重），也就是说按照比例分配 CPU，例如现在有两个容器 --cpu-shares 的具体数值分别是1024和512，假设两个容器都很需要 CPU 的资源，则会按照2比1的比例分配 CPU 即前者 66.6% 后者 33.3%，但如果第一个不是很需要 CPU 的时候，后者也是有可能直接占用 100% 的 CPU 资源。**能够这么做的主要原因是 CPU 是可压缩资源**

--cpus=\<value\> 限制该容器最多能够使用的核数，比如说一个四核的计算机，那么它就有 400% 的 CPU 资源。所以如果 --cpus 的具体数值是2（这个数可以是浮点数）那么该容器最多能够使用的 CPU 资源为 200%，且有可能是每个核 50%

--cpuset-cpus 规定这个容器能够使用的内核集，还是假设四核的计算机，则这四个核心的编号分别为0，1，2，3。限制容器运行在哪些核上并不是一个很好的做法，因为它需要事先知道主机上有多少 CPU 核，而且非常不灵活。除非有特别的需求，一般并不推荐在生产中这样使用。

--cpu-period 和 --cpu-quota 这两个参数是相互配合的，–-cpu-period 是用来指定容器对 CPU 的使用要在多长时间内做一次重新分配，而 -–cpu-quota 是用来指定在这个周期内，最多可以有多少时间用来跑这个容器。跟 –-cpu-shares 不同的是这种配置是指定一个绝对值，而且没有弹性在里面，容器对 CPU 资源的使用绝对不会超过配置的值。比如说某容器配置的 –-cpu-period=100000 --cpu-quota=50000，那么该容器就可以最多使用 50% 个 CPU 资源，如果配置的 –-cpu-quota=200000，那就可以使用 200% 个 CPU 资源。

# 使用 stress 做压测
## 测试内存

```
# 最多分配 256M 内存，容器内有两个进程的内存消耗为默认的 256M
docker run --name stress -it --rm -m 256m lorel/docker-stress-ng:latest stress --vm 2
```

另开命令行终端运行命令`docker top stress`

```
chenxindeMacBook-Pro:~ chenxin$ docker top stress
PID                 USER                TIME                COMMAND
12938               root                0:00                /usr/bin/stress-ng stress --vm 2
12979               root                0:00                {stress-ng-vm} /usr/bin/stress-ng stress --vm 2
12980               root                0:00                {stress-ng-vm} /usr/bin/stress-ng stress --vm 2
13006               root                0:01                {stress-ng-vm} /usr/bin/stress-ng stress --vm 2
13007               root                0:00                {stress-ng-vm} /usr/bin/stress-ng stress --vm 2
```

运行命令`docker stats`

```
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
9cab7a84ca8d        stress              173.46%             254MiB / 256MiB     99.21%              1.04kB / 0B         19.4GB / 51.2GB     5
```

## 测试 CPU

```
# 最多使用两个核，容器内使用八个进程
docker run --name stress -it --rm --cpus 2 lorel/docker-stress-ng:latest stress --cpu 8
```

另开命令行终端运行命令`docker top stress`

```
chenxindeMacBook-Pro:~ chenxin$ docker top stress
PID                 USER                TIME                COMMAND
13703               root                0:00                /usr/bin/stress-ng stress --cpu 8
13739               root                0:38                {stress-ng-cpu} /usr/bin/stress-ng stress --cpu 8
13740               root                0:32                {stress-ng-cpu} /usr/bin/stress-ng stress --cpu 8
13741               root                0:32                {stress-ng-cpu} /usr/bin/stress-ng stress --cpu 8
13742               root                1:01                {stress-ng-cpu} /usr/bin/stress-ng stress --cpu 8
13743               root                0:46                {stress-ng-cpu} /usr/bin/stress-ng stress --cpu 8
13744               root                1:03                {stress-ng-cpu} /usr/bin/stress-ng stress --cpu 8
13745               root                0:46                {stress-ng-cpu} /usr/bin/stress-ng stress --cpu 8
13746               root                0:55                {stress-ng-cpu} /usr/bin/stress-ng stress --cpu 8
```

运行命令`docker stats`

```
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
e743e6a74141        stress              198.36%             36.18MiB / 1.952GiB   1.81%               1.04kB / 0B         0B / 0B             9
```