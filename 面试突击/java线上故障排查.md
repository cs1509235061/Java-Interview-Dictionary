# java线上故障排查

## APM分析排查

APM，全称Application Performance Management，应⽤性能管理，⽬的是通过各种探针采集数据，收集关键指标，同时搭配数据呈现以实现对应⽤程序性能管理和故障管理的系统化解决⽅案。通过分布式链路调⽤跟踪系统，通过在系统请求中透传 trace-id，将所有相关⽇志进⾏聚合，然后⽇志统⼀采集和分析后，以图形化的形式展示给⼯程师们，⽽他们在排查问题的时候，可以简单粗暴且直观的调度出问题最根本的原因。 业务⽇志较优于解决单体服务异常，但现在的应⽤通常使⽤的是分布式架构，⽽在分布式系统中，仅通过单服务节点的⽇志，很难将错误信息与请求链路关联在⼀起，也就是通过系统中某个服务的异常⽇志信息很难定位到问题的根本原因。并且我们还需要关注各服务执⾏过程中的耗时情况，及时的发现异常服务，并根据异常信息进⾏修复。要满⾜这种需求，仅通过分析单个服务的⽇志信息是不够的，此时则需要APM进⾏全链路分析，通过请求链路监控，实时的发现链路中相关服务的异常情况。

### 物理环境排查

物理环境是指⼯程所依附的物理环境，⽐如服务器、宿主机、容器等，细分为服务器负载、CPU、内存、磁盘、⽹络⼏个⽅⾯。

CPU分析

排查CPU的⽬的主要是查看服务器CPU的使⽤率， 使⽤top命令分析CPU使⽤情况

内存分析

使⽤free -m命令查看内存使⽤情况

磁盘分析

使⽤df -h、iostat、lsof等命令查看磁盘IO情况，找到读写异常的进程

⽹络分析

使⽤dstat、vmstat等命令查看⽹络流量、TCP连接等情况，分析异常流量

应⽤服务排查

应⽤排查，排查应⽤本身最有可能引发的问题，针对各种场景进⾏对应分析

CPU分析

使⽤jstack等命令进⾏JVM分析

内存分析

使⽤jmap等命令分析内存使⽤情况

云⼚商或运营商问题排查

排查到了这⼀步的话，只需关注云⼚商或运营商官⽅公告即可。

假设我们从业务⽇志、APM分析中都没分析出问题所在，那么此时基本只能连接到⽣产环境中进⾏排查了。

### 常⽤Linux分析命令

CPU

CPU使⽤率是衡量系统繁忙程度的重要指标。但是CPU使⽤率的安全阈值是相对的，取决于你的系统的IO密集型还是计算密集型。⼀般计算密集型应⽤CPU使⽤率偏⾼load偏低，IO密集型相反。 top命令是Linux下常⽤的 CPU 性能分析⼯具，能够实时显示系统中各个进程的资源占⽤状况，常⽤于服务端性能分析。

```html
top
```

top 命令显示了各个进程 CPU 使⽤情况，⼀般 CPU 使⽤率从⾼到低排序展示输出。其中 LoadAverage 显示最近1分钟、5分钟和15分钟的系统平均负载。 我们⼀般会关注 CPU 使⽤率最⾼的进程，正常情况下就是我们的应⽤主进程。参数说明：

- PID : 进程id
- USER : 进程所有者的⽤户名
- PR : 进程优先级
- NI : nice值。负值表示⾼优先级，正值表示低优先级
- VIRT : 进程使⽤的虚拟内存总量，单位kb
- SHR : 共享内存⼤⼩
- %CPU : 上次更新到现在的CPU时间占⽤百分⽐
- %MEM : 进程使⽤的物理内存百分⽐
- TIME+ : 进程使⽤的CPU时间总计，单位1/100秒
- COMMAND : 命令名称、命令⾏

内存

内存是排查线上问题的重要参考依据，free 是显示的当前内存的使⽤，-h 表示⼈类可读性。

```html
free -h
```

参数说明：

- total ：内存总数
- used：已经使⽤的内存数
- free：空闲的内存数
- shared：被共享使⽤的物理内存⼤⼩
- buffers/buffer：被 buffer 和 cache 使⽤的物理内存⼤⼩
- available： 还可以被应⽤程序使⽤的物理内存⼤⼩

磁盘

磁盘满了很多时候会引发系统服务不可⽤等问题

```html
df -h
```

⽹络

dstat 是⼀个可以取代vmstat，iostat，netstat和ifstat这些命令的多功能产品。

```html
dstat
```

默认情况下，dstat每秒都会刷新数据。需要注意的是报告的第⼀⾏，由于dstat会通过上⼀次的报告来给出⼀个总结，所以第⼀次运⾏时是没有平均值和总值的相关数据。 默认输出解释： CPU状态：显示了⽤户，系统和空闲部分，便于分析CPU当前的使⽤状况 磁盘统计：磁盘的读写操作，显示磁盘的读、写总数。 ⽹络统计：⽹络设备发送和接受的数据，显示了⽹络收、发数据总数。 分⻚统计：系统的分⻚活动。 系统统计：这⼀项显示的是中断（int）和上下⽂切换（csw）。

到了这⼀步，如果CPU、内存、磁盘、⽹络这四个地⽅都没有问题的话，那就进⼊了应⽤本身的问题排查阶段。下⼀步就该使⽤jvm命令进⾏问题定位了，但很多研发可能由于⾃身⼯作经验不⾜、对Java内存模型理解不深、尚未掌握Jvm排查命令等原因对Jvm相关排查畏⼿畏脚，不够⾃信，进⽽影响排查进度。

## JVM问题定位命令

在 JDK 安装⽬录的 bin ⽬录下默认提供了很多有价值的命令⾏⼯具。每个⼩⼯具体积基本都⽐较⼩，因为这些⼯具只是 jdk\lib\tools.jar 的简单封装。

其中，定位排查问题时最为常⽤命令包括：jps（进程）、jmap（内存）、jstack（线程）、jinfo（参数）等。

- jps：查询当前机器所有Java进程信息
- jmap：输出某个 Java 进程内存情况
- jstack：打印某个 Java 线程的线程栈信息
- jinfo：⽤于查看 jvm 的配置参数

### jps

jps ⽤于输出当前⽤户启动的所有进程 ID，当线上发现故障或者问题时，利⽤ jps 快速定位对应的 Java进程 ID。

```html
jps -m
```

参数解释：

-m：输出传⼊ main ⽅法的参数 -l：输出完全的包名，应⽤主类名，jar的完全路径名

当然，我们也可以使⽤ Linux 提供的查询进程状态命令也能快速获取 Tomcat 服务的进程 id。⽐如：

```html
ps -ef | grep tomcat
```

### jmap

jmap（Java Memory Map）可以输出所有内存中对象的⼯具，甚⾄可以将 VM 中的 heap，以⼆进制输出成⽂本，使⽤⽅式如下： jmap -heap：

```html
jmap -heap pid 输出当前进程JVM堆内存新⽣代、⽼年代、持久代、GC算法等信息
```

jmap -histo：

```html
jmap -histo:live pid | head -n 20 输出当前进程内存中所有对象包含的⼤⼩
```

输出当前进程内存中所有对象实例数（instances）和⼤⼩（bytes），如果某个业务对象实例数和⼤⼩存在异常情况，可能存在内存泄露或者业务设计⽅⾯存在不合理之处。

jmap -dump：

```html
jmap -dump:format=b,file=/apps/logs/gc/dump.hprof {pid}
以⼆进制形式输出当前内存的堆情况，便于导⼊MAT等⼯具进⾏分析情况
```

⼀般我们要求给 JVM 添加参数 -XX:+Heap Dump On Out Of Memory Error OOM 确保应⽤发⽣OOM 时 JVM 能够保存并 dump 出当前的内存镜像。当然如果你决定⼿动 dump 内存时，dump 操作占据⼀定 CPU 时间⽚、内存资源、磁盘资源等，因此会带来⼀定的负⾯影响。 此外，dump 的⽂件可能⽐较⼤,⼀般我们可以考虑使⽤zip命令对⽂件进⾏压缩处理，这样在下载⽂件时能减少带宽的开销。在下载 dump ⽂件完成之后，由于 dump ⽂件较⼤可将 dump ⽂件备份⾄制定位置或者直接删除，以释放磁盘在这块的空间占⽤。

### jstack

jstack⽤于打印某个 Java 线程的线程栈信息，通常⽤以下三步分析：

```html
printf '%x\n' tid 10进制⾄16进制线程转换
jstack pid | grep tid -C 30 --color
ps -mp 8278 -o THREAD,tid,time | head -n 40
```

举个栗⼦，某 Java 进程 CPU 占⽤率⾼，我们想要定位到其中 CPU 占⽤率最⾼的线程，如何定位？ 1、利⽤ top 命令可以查出占 CPU 最⾼的线程 pid

```html
top -Hp pid
```

2、 占⽤率最⾼的线程 ID 为 22021，将其转换为16进制形式（因为 java native 线程以16进制形式输出）

```html
printf '%x\n' 22021
```

3、 利⽤ jstack 打印出 Java 线程调⽤栈信息

```html
jstack 21993 | grep '0x5605' -A 50 --color
```

### jinfo

jinfo可以⽤来查看正在运⾏的 java 应⽤程序的扩展参数，包括Java System属性和JVM命令⾏参数；也可以动态的修改正在运⾏的 JVM ⼀些参数。

```html
jinfo pid
```

### jstat

jstat命令可以查看堆内存各部分的使⽤量，以及加载类的数量。

```html
jstat -gc pid
```

## 内存分析⼯具 MAT

MAT（Memory Analyzer Tool），⼀个基于 Eclipse 的内存分析⼯具，是⼀个快速、功能丰富的 JAVAheap 分析⼯具，它可以帮助我们查找内存泄漏和减少内存消耗。使⽤内存分析⼯具从众多的对象中进⾏分析，快速的计算出在内存中对象的占⽤⼤⼩，看看是谁阻⽌了垃圾收集器的回收⼯作，并可以通过报表直观的查看到可能造成这种结果的对象。

## GC分析

### GC ⽇志详细分析

Java 虚拟机GC⽇志是⽤于定位问题重要的⽇志信息，频繁的GC将导致应⽤吞吐量下降、响应时间增加，甚⾄导致服务不可⽤。

```html
-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/apps/logs/gc/gc.log -
XX:+UseConcMarkSweepGC
```

我们可以在 Java 应⽤的启动参数中增加-XX:+PrintGCDetails 可以输出 GC 的详细⽇志，例外还可以增加其他的辅助参数，如 -Xloggc 制定 GC ⽇志⽂件地址。如果你的应⽤还没有开启该参数,下次重启时请加⼊该参数。

### CMS GC ⽇志分析

Concurrent Mark Sweep（CMS）是⽼年代垃圾收集器，从名字（Mark Sweep）可以看出，CMS 收集器就是“标记-清除”算法实现的，分为六个步骤：

初始标记（STW initial mark） 并发标记（Concurrent marking） 并发预清理（Concurrent precleaning） 重新标记（STW remark） 并发清理（Concurrent sweeping） 并发重置（Concurrent reset）

其中初始标记（STW initial mark） 和 重新标记（STW remark）需要“Stop the World”。 初始标记 ：在这个阶段，需要虚拟机停顿正在执⾏的任务，官⽅的叫法 STW（Stop The Word）。这个过程从垃圾回收的"根对象"开始，只扫描到能够和"根对象"直接关联的对象，并作标记。所以这个过程虽然暂停了整个 JVM，但是很快就完成了。 并发标记 ：这个阶段紧随初始标记阶段，在初始标记的基础上继续向下追溯标记。并发标记阶段，应⽤程序的线程和并发标记的线程并发执⾏，所以⽤户不会感受到停顿。 并发预清理 ：并发预清理阶段仍然是并发的。在这个阶段，虚拟机查找在执⾏并发标记阶段新进⼊⽼年代的对象（可能会有⼀些对象从新⽣代晋升到⽼年代， 或者有⼀些对象被分配到⽼年代）。通过重新扫描，减少下⼀个阶段"重新标记"的⼯作，因为下⼀个阶段会 Stop The World。 重新标记 ：这个阶段会暂停虚拟机，收集器线程扫描在 CMS 堆中剩余的对象。扫描从"跟对象"开始向下追溯，并处理对象关联。 并发清理 ：清理垃圾对象，这个阶段收集器线程和应⽤程序线程并发执⾏。 并发重置 ：这个阶段，重置 CMS 收集器的数据结构，等待下⼀次垃圾回收。 CMS 使得在整个收集的过程中只是很短的暂停应⽤的执⾏，可通过在 JVM 参数中设置 -XX:UseConcMarkSweepGC 来使⽤此收集器，不过此收集器仅⽤于 old 和 Perm（永⽣）的对象收集。CMS 减少了 stop the world 的次数，不可避免地让整体 GC 的时间拉⻓了。 Full GC 的次数说的是 stop the world 的次数，所以⼀次 CMS ⾄少会让 Full GC 的次数+2，因为 CMSInitial mark 和 remark 都会 stop the world，记做2次。⽽ CMS 可能失败再引发⼀次 Full GC。

异常情况有： 伴随 prommotion failed，然后 Full GC： [prommotion failed：存活区内存不⾜，对象进⼊⽼年代，⽽此时⽼年代也仍然没有内存容纳对象，将导致⼀次Full GC] 伴随 concurrent mode failed，然后 Full GC： [concurrent mode failed：CMS回收速度慢，CMS完成前，⽼年代已被占满，将导致⼀次Full GC]

频繁 CMS GC：

[内存吃紧，⽼年代⻓时间处于较满的状态]















