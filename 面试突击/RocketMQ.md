# RocketMQ

## 1.消息积压解决方案

### 1. 提高消费并行度

绝大部分消息消费行为都属于 IO 密集型，即可能是操作数据库，或者调用 RPC，这类消费行为的消费速度在于后端数据库或者外系统的吞吐量，通过增加消费并行度，可以提高总的消费吞吐量，但是并行度增加到一定程度，反而会下降。所以，应用必须要设置合理的并行度。 如下有几种修改消费并行度的方法：

同一个 ConsumerGroup 下，通过增加 Consumer 实例数量来提高并行度（需要注意的是超过订阅队列数的 Consumer 实例无效）。可以通过加机器，或者在已有机器启动多个进程的方式。 提高单个 Consumer 的消费并行线程，通过修改参数 consumeThreadMin、consumeThreadMax 实现。

### 2.批量方式消费

某些业务流程如果支持批量方式消费，则可以很大程度上提高消费吞吐量，例如订单扣款类应用，一次处理一个订单耗时 1 s，一次处理 10 个订单可能也只耗时 2 s，这样即可大幅度提高消费的吞吐量，通过设置 consumer 的 consumeMessageBatchMaxSize 返个参数，默认是 1，即一次只消费一条消息，例如设置为 N，那么每次消费的消息数小于等于 N。

### 3. 跳过非重要消息

发生消息堆积时，如果消费速度一直追不上发送速度，如果业务对数据要求不高的话，可以选择丢弃不重要的消息。例如，当某个队列的消息数堆积到 100000 条以上，则尝试丢弃部分或全部消息，这样就可以快速追上发送消息的速度。

### 4. 优化每条消息消费过程

举例如下，某条消息的消费过程如下：

- 根据消息从 DB 查询【数据 1】
- 根据消息从 DB 查询【数据 2】
- 复杂的业务计算
- 向 DB 插入【数据 3】
- 向 DB 插入【数据 4】

这条消息的消费过程中有 4 次与 DB 的 交互，如果按照每次 5ms 计算，那么总共耗时 20ms，假设业务计算耗时 5ms，那么总过耗时 25ms，所以如果能把 4 次 DB 交互优化为 2 次，那么总耗时就可以优化到 15ms，即总体性能提高了 40%。所以应用如果对时延敏感的话，可以把 DB 部署在 SSD 硬盘，相比于 SCSI 磁盘，前者的 RT 会小很多。

## 2.RocketMQ

### 集群化部署

RocketMQ是可以集群化部署的，可以部署在多台机器上。

### 存储海量消息

分布式存储，发送消息到MQ的系统会把消息分散发送给多台不同的机器，每台机器上部署的RocketMQ进程一般称之为Broker，每个Broker都会收到不同的消息，然后就会把这批消息存储在自己本地的磁盘文件里。

### 高可用保障

Broker主从架构以及多副本策略。

Broker是有Master和Slave两种角色的，Master Broker收到消息之后会同步给Slave Broker，这样Slave Broker上就能有一模一样的一份副本数据！

### 数据路由

有一个NameServer的概念，他也是独立部署在几台机器上的，然后所有的Broker都会把自己注册到NameServer上去，NameServer就知道集群里有哪些Broker了。

如果系统要从Broker获取消息，也会找NameServer获取路由信息，去找到对应的Broker获取消息。

## 3.核心的部分

- NameServer，负责去管理集群里所有Broker的信息，让使用MQ的系统可以通过他感知到集群里有哪些Broker。
- Broker集群，在多台机器上部署这么一个集群，而且还得用主从架构实现数据多副本存储和高可用。
- 生产者，向MQ发送消息的那些系统。
- 消费者，从MQ获取消息的那些系统。

## 4.NameServer

可以集群化部署，实现高可用性。

每个Broker启动都得向所有的NameServer进行注册，每个NameServer都会有一份集群中所有Broker的信息。

生产者和消费者自己主动去NameServer拉取Broker信息的，通过这些路由信息，每个系统就知道发送消息或者获取消息去哪台Broker上去进行了。

心跳机制，Broker会每隔30s给所有的NameServer发送心跳，告诉每个NameServer自己目前还活着。每次NameServer收到一个Broker的心跳，就可以更新一下他的最近一次心跳的时间。然后NameServer会每隔10s运行一个任务，去检查一下各个Broker的最近一次心跳时间，如果某个Broker超过120s都没发送心跳了，那么就认为这个Broker已经挂掉了。

容错机制，会定期重新从NameServer拉取最新的路由信息。

NameServer监听的接口默认就是9876。

## 5.Broker的主从架构原理

Master-Slave模式的，也就是一个Master Broker对应一个Slave Broker。

RocketMQ的Master-Slave模式采取的是Slave Broker不停的发送请求到Master Broker去拉取消息。采取的是**Pull模式**拉取消息。

在写入消息的时候，通常来说肯定是选择Master Broker去写入的。但是在拉取消息的时候，有可能从Master Broker获取，也可能从Slave Broker去获取。

作为消费者的系统在获取消息的时候会先发送请求到Master Broker上去，然后Master Broker在返回消息给消费者系统的时候，会根据当时Master Broker的负载情况和Slave Broker的同步情况，向消费者系统建议下一次拉取消息的时候是从Master Broker拉取还是从Slave Broker拉取。

如果Master挂掉了，在RocketMQ 4.5之后，有一种新的机制，叫做Dledger。

一个Master Broker对应多个Slave Broker，一旦Master Broker宕机了，就可以在多个副本，也就是多个Slave中，通过Dledger技术和Raft协议算法进行leader选举，直接将一个Slave Broker选举为新的Master Broker，然后这个新的Master Broker就可以对外提供服务了。整个过程也许只要10秒或者几十秒的时间就可以完成。

## 6.Broker是如何跟NameServer进行通信的？

Broker跟NameServer之间的通信是基于什么协议来进行的？在RocketMQ的实现中，采用的是**TCP长连接**进行通信。也就是说，Broker会跟每个NameServer都建立一个TCP长连接，然后定时通过TCP长连接发送心跳请求过去。

所以各个NameServer就是通过跟Broker建立好的长连接不断收到心跳包，然后定时检查Broker有没有120s都没发送心跳包，来判定集群里各个Broker到底挂掉了没有。

## 7.Topic

一个**数据集合**的意思。不同类型的数据你得放不同的Topic里去。

比如现在你的订单系统需要往MQ里发送订单消息，那么此时你就应该建一个Topic，他的名字可以叫做：topic_order_info，也就是一个包含了订单信息的数据集合。

然后你的订单系统投递的订单消息都是进入到这个“topic_order_info”里面去的，如果你的仓储系统要获取订单消息，那么他可以指定从“topic_order_info”这里面去获取消息，获取出来的都是他想要的订单消息了。

### Topic怎么在Broker集群里存储的

分布式存储，可以在创建Topic的时候指定让他里面的数据分散存储在多台Broker机器上。

每个Broke在进行定时的心跳汇报给NameServer的时候，都会告诉NameServer自己当前的数据情况，比如有哪些Topic的哪些数据在自己这里，这些信息都是属于路由信息的一部分。

### 生产者系统是如何将消息发送给Broker的

生产者系统跟NameServer建立一个TCP长连接，然后定时从他那里拉取到最新的路由信息，包括集群里有哪些Broker，集群里有哪些Topic，每个Topic都存储在哪些Broker上。然后生产者系统自然就可以通过路由信息找到自己要投递消息的Topic分布在哪几台Broker上，此时可以根据负载均衡算法，从里面选择一台Broke机器出来，比如round robine轮询算法，或者是hash算法，都可以。

Broker收到消息之后就会存储在自己本地磁盘里去。

## 8.压测准备

第一步，你需要对他部署的机器的OS内核参数进行一定的调整（也就是linux操作系统的一些内核参数）。

因为OS内核参数很多默认值未必适合生产环境的系统运行，有些参数的值需要调整大一些，才能让中间件发挥出来性能，我们看下面的的图。

接着下一步，一般中间件是Java开发的，在一台机器上部署和启动一个中间件系统，说白了就是启动一个JVM进程，由这个JVM进程来运行中间件系统内的所有代码，然后实现中间件系统的各种功能。JVM的各种参数也需要优化。比如内存区域的大小分配，垃圾回收器以及对应的行为参数，GC日志存放地址，OOM自动导出内存快照的配置，等等。

最后第三件事情，就是中间件系统自己本身的一些核心参数的设置，比如你的中间件系统会开启很多线程处理请求和工作负载，然后还会进行大量的网络通信，同时会进行大量的磁盘IO类的操作。

这个时候你就需要依据你的机器配置，合理的对中间件系统的核心参数进行调整。

## 9.OS内核参数的调整

（1）vm.overcommit_memory

“vm.overcommit_memory”这个参数有三个值可以选择，0、1、2。

如果值是0的话，在你的中间件系统申请内存的时候，os内核会检查可用内存是否足够，如果足够的话就分配内存给你，如果感觉剩余内存不是太够了，干脆就拒绝你的申请，导致你申请内存失败，进而导致中间件系统异常出错。

因此一般需要将这个参数的值调整为1，意思是把所有可用的物理内存都允许分配给你，只要有内存就给你来用，这样可以避免申请内存失败的问题。

可以用如下命令修改：echo 'vm.overcommit_memory=1' >> /etc/sysctl.conf。

（2）vm.max_map_count

这个参数的值会影响中间件系统可以开启的线程的数量，同样也是非常重要的。

如果这个参数过小，有的时候可能会导致有些中间件无法开启足够的线程，进而导致报错，甚至中间件系统挂掉。

他的默认值是65536，但是这个值有时候是不够的，因此建议可以把这个参数调大10倍，比如655360这样的值，保证中间件可以开启足够多的线程。

可以用如下命令修改：echo 'vm.max_map_count=655360' >> /etc/sysctl.conf。

（3）vm.swappiness

这个参数是用来控制进程的swap行为的，这个简单来说就是os会把一部分磁盘空间作为swap区域，然后如果有的进程现在可能不是太活跃，就会被操作系统把进程调整为睡眠状态，把进程中的数据放入磁盘上的swap区域，然后让这个进程把原来占用的内存空间腾出来，交给其他活跃运行的进程来使用。

如果这个参数的值设置为0，意思就是尽量别把任何一个进程放到磁盘swap区域去，尽量大家都用物理内存。

如果这个参数的值是100，那么意思就是尽量把一些进程给放到磁盘swap区域去，内存腾出来给活跃的进程使用。

默认这个参数的值是60，有点偏高了，可能会导致我们的中间件运行不活跃的时候被迫腾出内存空间然后放磁盘swap区域去。

因此通常在生产环境建议把这个参数调整小一些，比如设置为10，尽量用物理内存，别放磁盘swap区域去。

可以用如下命令修改：echo 'vm.swappiness=10' >> /etc/sysctl.conf。

（4）ulimit

这个是用来控制linux上的最大文件链接数的，默认值可能是1024，一般肯定是不够的，因为你在大量频繁的读写磁盘文件的时候，或者是进行网络通信的时候，都会跟这个参数有关系。

对于一个中间件系统而言肯定是不能使用默认值的，如果你采用默认值，很可能在线上会出现如下错误：error: too many open files。

因此通常建议用如下命令修改这个值：echo 'ulimit -n 1000000' >> /etc/profile。

### 总结

最后要调整的东西，无非都是跟磁盘文件IO、网络通信、内存管理、线程数量有关系的，因为我们的中间件系统在运行的时候无非就是跟这些打交道。

- 中间件系统肯定要开启大量的线程**（跟vm.max_map_count有关）**
- 而且要进行大量的网络通信和磁盘IO**（跟ulimit有关）**
- 然后大量的使用内存（跟vm.swappiness和vm.overcommit_memory有关）

## 10.JVM参数调整

在rocketmq/distribution/target/apache-rocketmq/bin目录下，就有对应的启动脚本，比如mqbroker是用来启动Broker的，mqnamesvr是用来启动NameServer的。

用mqbroker来举例，我们查看这个脚本里的内容，最后有如下一行：

sh ${ROCKETMQ_HOME}/bin/runbroker.sh org.apache.rocketmq.broker.BrokerStartup $@

这一行内容就是用runbroker.sh脚本来启动一个JVM进程，JVM进程刚开始执行的main类就是org.apache.rocketmq.broker.BrokerStartup。

runbroker.sh脚本，在里面可以看到JVM的一些参数配置内容。

**server**：这个参数就是说用服务器模式启动，这个没什么可说的，现在一般都是如此。

**-Xms8g -Xmx8g -Xmn4g**：这个就是很关键的一块参数了，也是重点需要调整的，就是默认的堆大小是8g内存，新生代是4g内存，高配物理机的话可以调高内存。

**-XX:+UseG1GC -XX:G1HeapRegionSize=16m**：这几个参数也是至关重要的，这是选用了G1垃圾回收器来做分代回收，对新生代和老年代都是用G1来回收.

这里把G1的region大小设置为了16m，机器内存比较多的话，所以region大小可以调大一些给到16m，不然用2m的region，会导致region数量过多的.

**-XX:G1ReservePercent=25**：这个参数是说，在G1管理的老年代里预留25%的空闲内存，保证新生代对象晋升到老年代的时候有足够空间，避免老年代内存都满了，新生代有对象要进入老年代没有充足内存了.

默认值是10%，略微偏少，这里RocketMQ给调大了一些.

**-XX:InitiatingHeapOccupancyPercent=30**：这个参数是说，当堆内存的使用率达到30%之后就会自动启动G1的并发垃圾回收，开始尝试回收一些垃圾对象.

默认值是45%，这里调低了一些，也就是提高了GC的频率，但是避免了垃圾对象过多，一次垃圾回收耗时过长的问题.

**-XX:SoftRefLRUPolicyMSPerMB=0**：这个参数默认设置为0了，在JVM优化专栏中，救火队队长讲过这个参数引发的案例，其实建议这个参数不要设置为0，避免频繁回收一些软引用的Class对象，这里可以调整为比如1000.

**-verbose:gc -Xloggc:/dev/shm/mq_gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m**：这一堆参数都是控制GC日志打印输出的，确定了gc日志文件的地址，要打印哪些详细信息，然后控制每个gc日志文件的大小是30m，最多保留5个gc日志文件。

**-XX:-OmitStackTraceInFastThrow**：这个参数是说，有时候JVM会抛弃一些异常堆栈信息，因此这个参数设置之后，就是禁用这个特性，要把完整的异常堆栈信息打印出来.

**-XX:+AlwaysPreTouch**：这个参数的意思是我们刚开始指定JVM用多少内存，不会真正分配给他，会在实际需要使用的时候再分配给他.

所以使用这个参数之后，就是强制让JVM启动的时候直接分配我们指定的内存，不要等到使用内存的时候再分配.

**-XX:MaxDirectMemorySize=15g**：这是说RocketMQ里大量用了NIO中的direct buffer，这里限定了direct buffer最多申请多少，如果你机器内存比较大，可以适当调大这个值。

**-XX:-UseLargePages -XX:-UseBiasedLocking**：这两个参数的意思是禁用大内存页和偏向锁，这两个参数对应的概念每个要说清楚都得一篇文章，所以这里大家直接知道人家禁用了两个特性即可。

### 总结

RocketMQ默认的JVM参数是采用了**G1垃圾回收器**，默认堆内存大小是8G.

### 11.RocketMQ核心参数调整

配置文件：rocketmq/distribution/target/apache-rocketmq/conf/dledger

在这里主要是有一个较为核心的参数：**sendMessageThreadPoolNums=16**

这个参数的意思就是RocketMQ内部用来发送消息的线程池的线程数量，默认是16.

其实这个参数可以根据你的机器的CPU核数进行适当增加，比如机器CPU是24核的，可以增加这个线程数量到24或者30，都是可以的。

## 12.压测注意点

找到最合适的最高负载。

意思就是**在RocketMQ的TPS和机器的资源使用率和负载之间取得一个平衡。**尽量找到一个最高的TPS同时机器的各项负载在可承受范围之内，这才是压测的目的。

（1）RocketMQ的TPS和消息延时

（2）cpu负载情况

（3）内存使用率

使用free命令就可以查看到内存的使用率。

（4）JVM GC频率

使用jstat命令就可以查看RocketMQ的JVM的GC频率，基本上新生代每隔几十秒会垃圾回收一次，每次回收过后存活的对象很少，几乎不进入老年代。

（5）磁盘IO负载

用top命令查看一下IO等待占用CPU时间的百分比。

执行top命令之后，会看到一行类似下面的东西：

Cpu(s): 0.3% us, 0.3% sy, 0.0% ni, 76.7% id, 13.2% wa, 0.0% hi, 0.0% si。

在这里的13.2% wa，说的就是磁盘IO等待在CPU执行时间中的百分比。

如果这个比例太高，说明CPU执行的时候大部分时间都在等待执行IO，也就说明IO负载很高，导致大量的IO等待。

（6）网卡流量

使用如下命令可以查看服务器的网卡流量：

sar -n DEV 1 2

通过这个命令就可以看到每秒钟网卡读写数据量了。

## 13.发送消息到RocketMQ

### 先引入依赖

```html
<dependency>
    <groupId>org.apache.rocketmq<groupId>
    <artifactId>rocketmq-client</artifacted>
    <version>4.3.0</version>
</dependency>
```

### 发送消息

```html
public class RocketMQProducer{
    //这是个RocketMQ的生产者类，发送消息到RocketMQ
    private static DefalutMQProducer producer;
    static{
        //这里就是构建一个Producer实例对象
        producer=new DefalutMQProducer("order_producer_group");
        //这个是为Producer设置NameServer地址，让他可以拉取路由信息
        //这样才知道每个Topic的数据分散在哪些Broker机器上
        //然后才可以把消息发送到Broker上去
        producer.setNamesrvAddr("localhost:9876");
        //这里是启动一个Producer
        producer.start();      
    }
    public static void send(String topic,String message) throws Exception{
        //这里是构建一条消息对象
        Message msg=new Message(
            topic,//这就是指定发送消息到哪个Topic上去
            "",//这是消息的Tag
            message.getBytes(RemotingHelper.DEFAULT_CHARSET)//这是消息
            );
        //利用Producer发送消息
        SendResult sendResult=producer.send(msg);
    }
}
```

通过上述代码就可以让订单系统把订单消息发送到RocketMQ的一个Topic里去了。

### 从RocketMQ中消费消息

```html
public class RocketMQConsumer{
    public static void start(){
        new Thread(){
            public void run(){
                try{
                    //这是RocketMQ消费者实例对象
                    //"credit_group"之类的就是消费者分组
                    //不同的系统给自己取不同的消费组名字
                    DefaultMQPushConsumer consumer=new DefaultMQPushConsumer("credit_group");
                    //这是给消费者设置NameServer地址
                    //这样就可以拉取到路由信息，知道Topic的数据在哪些broker上
                    //然后可以从对应的broker上拉取数据
                    consumer.setNamesrvAddr("localhost:9876");
                    //选择订阅"TopicOrderPaySuccess"的消息
                    //这样会从这Topic的broker机器上拉取订单消息过来
                    consumer.subscribe("TopicOrderPaySuccess","*");
                    //注册消息监听器来处理拉取到的订单消息
                    //如果consumer拉取到了订单消息，就会回调这个方法交给你处理
                    consumer.registerMessageListener(new MessageListenerConcurrently(){
                        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,ConsumeConcurrentlyContext context){
                            //在这里对获取到的msgs订单消息进行处理
                            //比如增加积分，发送优惠卷等
                            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                        }
                    });
                    //启动消费者实例
                    consumer.start();
                    while(true){//别让线程退出，就让创建好的consumer不停消费数据
                        Thread.sleep(1000);
                        
                    }                                    
                }catch(Exception e){
                    e.printStackTrace();
                }
            }
        }.start();
    }
}
```

通过上述代码，积分系统就可以从RocketMQ里消费“TopicOrderPaySuccess”中的订单消息，然后根据订单消息执行增加积分、发送优惠券之类的业务逻辑了。

## 14.发送消息到RocketMQ？

### 同步发送

所谓同步，意思就是你通过这行代码发送消息到MQ去，SendResult sendResult = producer.send(msg)，然后你会卡在这里，代码不能往下走了.

你要一直等待MQ返回一个结果给你，你拿到了SendResult之后，接着你的代码才会继续往下走。

这个就是所谓的同步发送模式。

### 异步发送

构造Producer

```html
//设置异步发送失败的时候重试次数为0
producer.setRetryTimesWhenSendAsyncFailed(0);
```

发送消息

```html
producer.send(message,new SendCallback(){
    @Override
    public void onSuccess(SendResult sendResult){
        
    }
    @Override
    public void onException(Throwable e){
        
    }
})
```

你把消息发送出去，然后上面的代码就直接往下走了，不会卡在这里等待MQ返回结果给你！

然后当MQ返回结果给你的时候，Producer会回调你的SendCallback里的函数，如果发送成功了就回调onSuccess函数，如果发送失败了就回调onExceptino函数。

这个就是所谓的异步发送，异步的意思就是你发送消息的时候不会卡在上面那行代码等待MQ返回结果给你，会继续执行下面的别的代码，当MQ返回结果给你的时候，会回调你的函数！

### 发送单向消息

```html
producer.sendOneway(msg);
```

这个sendOneway的意思，就是你发送一个消息给MQ，然后代码就往下走了，根本不会关注MQ有没有返回结果给你，你也不需要MQ返回的结果，无论发送的消息是成功还是失败，都不关你的事。

## 15.消费模式

### Push模式

就是Broker会主动把消息发送给你的消费者，你的消费者是被动的接收Broker推送给过来的消息，然后进行处理。这个就是所谓的Push模式，意思就是Broker主动推送消息给消费者。

### Pull模式

Broker不会主动推送消息给Consumer，而是消费者主动发送请求到Broker去拉取消息过来。

## 16.完整数据同步到别的系统

先同步一份初始数据。

### MySQL Binlog同步系统

这种系统会监听MySQL数据库的Binlog，所谓Binlog大致可以理解为MySQL的增删改操作日志。

然后MySQL Binlog同步系统会将监听到的MySQL Binlog（也就是增删改操作日志）发送给你的系统，让你来处理这些增删改操作日志。

采用Canal监听MySQL Binlog，然后直接发送到RocketMQ里.

然后大数据团队的数据同步系统从RocketMQ中获取到MySQL Binlog，也就获取到了订单数据库的增删改操作，接着把增删改操作还原到自己的数据库中去就可以。

## 17.MessageQueue是什么？

如果我们要使用RocketMQ，你先部署出来一套RocketMQ集群这个肯定是必须的，在有了集群之后，就必须根据你的业务需要去创建一些Topic。

创建Topic的时候需要指定一个很关键的参数，就是MessageQueue。你要指定你的这个Topic对应了多少个队列，也就是多少个MessageQueue。

MessageQueue就是RocketMQ中非常关键的一个数据分片机制，他通过MessageQueue将一个Topic的数据拆分为了很多个数据分片，然后在每个Broker机器上都存储一些MessageQueue。

通过这个方法，就可以实现Topic数据的分布式存储！

生产者从NameServer中就会知道，一个Topic有几个MessageQueue，哪些MessageQueue在哪台Broker机器上，哪些MesssageQueue在另外一台Broker机器上。

生产这将数据分散写入不同的MessageQueue中，这样就可以实现RocketMQ集群分布式存储海量的消息数据了！

## 18.某个Broker出现故障该怎么办？

通常来说建议大家在Producer中开启一个开关，就是sendLatencyFaultEnable。

一旦打开了这个开关，那么他会有一个自动容错机制，比如如果某次访问一个Broker发现网络延迟有500ms，然后还无法访问，那么就会自动回避访问这个Broker一段时间，比如接下来3000ms内，就不会访问这个Broker了。

这样的话，就可以避免一个Broker故障之后，短时间内生产者频繁的发送消息到这个故障的Broker上去，出现较多次数的异常。而是在一个Broker故障之后，自动回避一段时间不要访问这个Broker，过段时间再去访问他。

## 19.MessageQueue在数据存储中

实际上在ConsumeQueue中存储的每条数据不只是消息在CommitLog中的offset偏移量，还包含了消息的长度，以及tag hashcode，一条数据是20个字节，每个ConsumeQueue文件保存30万条数据，大概每个文件是5.72MB。

所以实际上Topic的每个MessageQueue都对应了Broker机器上的多个ConsumeQueue文件，保存了这个MessageQueue的所有消息在CommitLog文件中的物理位置，也就是offset偏移量。

## 20.如何让消息写入CommitLog文件近乎内存写性能的？

Broker是基于OS操作系统的**PageCache**和**顺序写**两个机制，来提升写入CommitLog文件的性能的。

首先Broker是以顺序的方式将消息写入CommitLog磁盘文件的，也就是每次写入就是在文件末尾追加一条数据就可以了，对文件进行顺序写的性能要比对文件随机写的性能提升很多。

另外，数据写入CommitLog文件的时候，其实不是直接写入底层的物理磁盘文件的，而是先进入OS的PageCache内存缓存中，然后后续由OS的后台线程选一个时间，异步化的将OS PageCache内存缓冲中的数据刷入底层的磁盘文件。

所以在这样的优化之下，采用**磁盘文件顺序写+OS PageCache写入+OS异步刷盘的策略**，基本上可以让消息写入CommitLog的性能跟你直接写入内存里是差不多的，所以正是如此，才可以让Broker高吞吐的处理每秒大量的消息写入。

## 21.同步刷盘与异步刷盘

### 异步刷盘

异步刷盘模式下，生产者把消息发送给Broker，Broker将消息写入OS PageCache中，就直接返回ACK给生产者了。

异步刷盘的的策略下，可以让消息写入吞吐量非常高，但是可能会有数据丢失的风险。

### 同步刷盘

同步刷盘模式的话，那么生产者发送一条消息出去，broker收到了消息，必须直接强制把这个消息刷入底层的物理磁盘文件中，然后才会返回ack给producer，此时你才知道消息写入成功了。

如果broker还没有来得及把数据同步刷入磁盘，然后他自己挂了，那么此时对producer来说会感知到消息发送失败了，然后你只要不停的重试发送就可以了，直到有slave broker切换成master broker重新让你可以写入消息，此时可以保证数据是不会丢的。

但是如果你强制每次消息写入都要直接进入磁盘中，必然导致每条消息写入性能急剧下降，导致消息写入吞吐量急剧下降，但是可以保证数据不会丢失。

## 22.基于DLedger技术的Broker主从同步原理

### Broker高可用架构原理

如果要让Broker实现高可用，那么必须有一个Broker组，里面有一个是Leader Broker可以写入数据，然后让Leader Broker接收到数据之后，直接把数据同步给其他的Follower Broker。

如果基于DLedger技术来实现Broker高可用架构，实际上就是用DLedger先替换掉原来Broker自己管理的CommitLog，由DLedger来管理CommitLog.每个Broker上都有一个DLedger组件.

### DLedger是如何基于Raft协议选举Leader Broker的？

如果我们配置了一组Broker，比如有3台机器，DLedger是如何从3台机器里选举出来一个Leader的？

实际上DLedger是**基于Raft协议来进行Leader Broker选举的**，那么Raft协议中是如何进行多台机器的Leader选举的呢？

确保有人可以成为Leader的核心机制就是一轮选举不出来Leader的话，就让大家随机休眠一下，先苏醒过来的人会投票给自己，其他人苏醒过后发现自己收到选票了，就会直接投票给那个人。

依靠这个随机休眠的机制，基本上几轮投票过后，一般都是可以快速选举出来一个Leader。

只有Leader可以接收数据写入，Follower只能接收Leader同步过来的数据。

### DLedger是如何基于Raft协议进行多副本同步的？

数据同步会分为两个阶段，一个是uncommitted阶段，一个是commited阶段。

首先Leader Broker上的DLedger收到一条数据之后，会标记为uncommitted状态，然后他会通过自己的DLedgerServer组件把这个uncommitted数据发送给Follower Broker的DLedgerServer。

接着Follower Broker的DLedgerServer收到uncommitted消息之后，必须返回一个ack给Leader Broker的DLedgerServer，然后如果Leader Broker收到超过半数的Follower Broker返回ack之后，就会将消息标记为committed状态。

然后Leader Broker上的DLedgerServer就会发送commited消息给Follower Broker机器的DLedgerServer，让他们也把消息标记为comitted状态。

### 如果Leader Broker崩溃了怎么办？

如果Leader Broker挂了，此时剩下的两个Follower Broker就会重新发起选举，他们会基于DLedger还是采用Raft协议的算法，去选举出来一个新的Leader Broker继续对外提供服务，而且会对没有完成的数据同步进行一些恢复性的操作，保证数据不会丢失。

新选举出来的Leader会把数据通过DLedger同步给剩下的一个Follower Broker。

## 23.消费者是如何获取消息处理以及进行ACK的

### 消费组

消费者组的意思，就是让你给一组消费者起一个名字。

不同的系统应该设置不同的消费组，如果不同的消费组订阅了同一个Topic，对Topic里的一条消息，每个消费组都会获取到这条消息。

### 集群模式消费 vs 广播模式消费

默认情况下我们都是集群模式，也就是说，一个消费组获取到一条消息，只会交给组内的一台机器去处理，不是每台机器都可以获取到这条消息的。

但是我们可以通过如下设置来改变为广播模式：

consumer.setMessageModel(MessageModel.BROADCASTING);

如果修改为广播模式，那么对于消费组获取到的一条消息，组内每台机器都可以获取到这条消息。但是相对而言广播模式其实用的很少，常见基本上都是使用集群模式来进行消费的。

### MessageQueue、CommitLog、ConsumeQueue之间的关系

Topic中的多个MessageQueue会分散在多个Broker上，在每个Broker机器上，一个MessageQueue就对应了一个ConsumeQueue，当然在物理磁盘上其实是对应了多个ConsumeQueue文件的，但是我们大致也理解为一 一对应关系。

对于一个Broker机器而言，存储在他上面的所有Topic以及MessageQueue的消息数据都是写入一个统一的CommitLog的.

然后对于Topic的各个MessageQueue而言，就是通过各个ConsumeQueue文件来存储属于MessageQueue的消息在CommitLog文件中的物理地址，就是一个offset偏移量。

### Push模式 vs Pull模式

这两个消费模式本质是一样的，都是消费者机器主动发送请求到Broker机器去拉取一批消息下来。

Push消费模式本质底层也是基于这种消费者主动拉取的模式来实现的，只不过他的名字叫做Push而已，意思是Broker会尽可能实时的把新消息交给消费者机器来进行处理，他的消息时效性会更好。

一般我们使用RocketMQ的时候，消费模式通常都是基于他的Push模式来做的，因为Pull模式的代码写起来更加的复杂和繁琐，而且Push模式底层本身就是基于消息拉取的方式来做的，只不过时效性更好而已。

当消费者发送请求到Broker去拉取消息的时候，如果有新的消息可以消费那么就会立马返回一批消息到消费机器去处理，处理完之后会接着立刻发送请求到Broker机器去拉取下一批消息。

当你的请求发送到Broker，结果他发现没有新的消息给你处理的时候，就会让请求线程挂起，默认是挂起15秒，然后这个期间他会有后台线程每隔一会儿就去检查一下是否有的新的消息给你，另外如果在这个挂起过程中，如果有新的消息到达了会主动唤醒挂起的线程，然后把消息返回给你。

### Broker是如何将消息读取出来返回给消费机器的？

消费消息的时候，本质就是根据你要消费的MessageQueue以及开始消费的位置，去找到对应的ConsumeQueue读取里面对应位置的消息在CommitLog中的物理offset偏移量，然后到CommitLog中根据offset读取消息数据，返回给消费者机器。

### 消费者机器如何处理消息

消费者机器拉取到一批消息之后，就会将这批消息回调我们注册的一个函数。

```html
consumer.registerMessageListener(new MessageListenerConcurrently(){
    @Overrride
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExit>msgs,ConsumeConcurrentlyContext context){
        //处理消息
        //标记消息已被成功消费
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
})
```

当我们处理完这批消息之后，消费者机器就会提交我们目前的一个消费进度到Broker上去，然后Broker就会存储我们的消费进度.

比如我们现在对ConsumeQueue0的消费进度假设就是在offset=1的位置，那么他会记录下来一个ConsumeOffset的东西去标记我们的消费进度.

那么下次这个消费组只要再次拉取这个ConsumeQueue的消息，就可以从Broker记录的消费位置开始继续拉取，不用重头开始拉取了。

### 如果消费组中出现机器宕机或者扩容加机器，会怎么处理？

这个时候其实会进入一个rabalance的环节，也就是说重新给各个消费机器分配他们要处理的MessageQueue。

## 24.消费者到底是根据什么策略从Master或Slave上拉取消息的？

### CommitLog基于os cache提升写性能

拉取消息的时候必然会先读取ConsumeQueue文件.

broker收到一条消息，会写入CommitLog文件，但是会先把CommitLog文件中的数据写入os cache(操作系统管理的缓存)中去,然后os自己有后台线程，过一段时间后会异步把os cache缓存中的CommitLog文件的数据刷入磁盘中去.

### ConsumeQueue文件是基于os cache的

broker对ConsumeQueue文件同样也是基于os cache来进行优化的.

对于Broker机器的磁盘上的大量的ConsumeQueue文件，在写入的时候也都是优先进入os cache中的.

而且os自己有一个优化机制，就是读取一个磁盘文件的时候，他会自动把磁盘文件的一些数据缓存到os cache中。

### CommitLog是基于os cache+磁盘一起读取的

在进行消息拉取的时候，先读os cache里的少量ConsumeQueue的数据，这个性能是极高的，然后第二步就是要根据你读取到的offset去CommitLog里读取消息的完整数据了。

**第一种可能**，如果你读取的是那种刚刚写入CommitLog的数据，那么大概率他们还停留在os cache中，此时你可以顺利的直接从os cache里读取CommitLog中的数据，这个就是内存读取，性能是很高的。

**第二种可能**，你也许读取的是比较早之前写入CommitLog的数据，那些数据早就被刷入磁盘了，已经不在os cache里了，那么此时你就只能从磁盘上的文件里读取了，这个性能是比较差一些的。

### 什么时候会从os cache读？什么时候会从磁盘读？

如果你的消费者机器一直快速的在拉取和消费处理，紧紧的跟上了生产者写入broker的消息速率，那么你每次拉取几乎都是在拉取最近人家刚写入CommitLog的数据，那几乎都在os cache里。

如果broker的负载很高，导致你拉取消息的速度很慢，或者是你自己的消费者机器拉取到一批消息之后处理的时候性能很低，处理的速度很慢，这都会导致你跟不上生产者写入的速率,每次都在从磁盘里读取数据了！

### Master Broker什么时候会让你从Slave Broker拉取数据？

本质是对比你当前没有拉取消息的数量和大小，以及最多可以存放在os cache内存里的消息的大小，如果你没拉取的消息超过了最大能使用的内存的量，那么说明你后续会频繁从磁盘加载数据，此时就让你从slave broker去加载数据了！

## 25.RocketMQ通信

### Reactor主线程与长短连接

作为Broker而言，他会有一个Reactor主线程。

Broker里有这么一个名字的线程，而且这个线程是负责监听一个网络端口的，比如监听个2888，39150这样的端口。

一个Producer他现在想要跟Broker建立一个TCP长连接。TCP长连接，就是按照这个TCP协议建立的长连接。

### Producer和Broker建立一个长连接

比如有一个Producer他就要跟Broker建立一个TCP长连接了，此时Broker上的这个Reactor主线程，他会在端口上监听到这个Producer建立连接的请求.

接着这个Reactor主线程就专门会负责跟这个Producer按照TCP协议规定的一系列步骤和规范，建立好一个长连接。

在Broker里用SocketChannel跟Producer之间建立的这个长连接。Producer里面会有一个SocketChannel，Broker里也会有一个SocketChannel，这两个SocketChannel就代表了他们俩建立好的这个长连接。

### 基于Reactor线程池监听连接中的请求

Reactor线程池，默认是3个线程！

然后Reactor主线程建立好的每个连接SocketChannel，都会交给这个Reactor线程池里的其中一个线程去监听请求。

Producer发送请求过来了，他发送一个消息过来到达Broker里的SocketChannel，此时Reactor线程池里的一个线程会监听到这个SocketChannel中有请求到达了！

### 基于Worker线程池完成一系列准备工作

接着Reactor线程从SocketChannel中读取出来一个请求，这个请求在正式进行处理之前，必须就先要进行一些准备工作和预处理，比如SSL加密验证、编码解码、连接空闲检查、网络连接管理，诸如此类的一些事.

Worker线程池，他默认有8个线程，此时Reactor线程收到的这个请求会交给Worker线程池中的一个线程进行处理，会完成上述一系列的准备工作.

### 基于业务线程池完成请求的处理

Worker线程完成了一系列的预处理之后，接着就需要对这个请求进行正式的业务处理了！

接收到了消息，肯定是要写入CommitLog文件的，就得继续把经过一系列预处理之后的请求转交给业务线程池.

比如对于处理发送消息请求而言，就会把请求转交给SendMessage线程池。

### 总结

- Reactor主线程在端口上监听Producer建立连接的请求，建立长连接
- Reactor线程池并发的监听多个连接的请求是否到达
- Worker请求并发的对多个请求进行预处理
- 业务线程池并发的对多个请求进行磁盘读写业务操作

## 26.基于mmap内存映射实现磁盘文件的高性能读写

### 传统文件IO操作的多次数据拷贝问题

首先从磁盘上把数据读取到内核IO缓冲区里去，然后再从内核IO缓存区里读取到用户进程私有空间里去，然后我们才能拿到这个文件里的数据.

发生了两次数据拷贝，对磁盘读写性能是有影响的。

将一些数据写入到磁盘文件里，必须先把数据写入到用户进程私有空间里去，然后从这里再进入内核IO缓冲区，最后进入磁盘文件里去.

同样发生了两次数据拷贝。

### 基于mmap技术+page cache技术优化

RocketMQ底层对CommitLog、ConsumeQueue之类的磁盘文件的读写操作，基本上都会采用mmap技术来实现。

如果具体到代码层面，就是基于JDK NIO包下的MappedByteBuffer的map()函数，来先将一个磁盘文件（比如一个CommitLog文件，或者是一个ConsumeQueue文件）映射到内存里来.

刚开始你建立映射的时候，并没有任何的数据拷贝操作，其实磁盘文件还是停留在那里，只不过他把物理上的磁盘文件的一些地址和用户进程私有空间的一些虚拟内存地址进行了一个映射.

这个地址映射的过程，就是JDK NIO包下的MappedByteBuffer.map()函数干的事情，底层就是基于mmap技术实现的。

mmap技术在进行文件映射的时候，一般有大小限制，在1.5GB~2GB之间。所以RocketMQ才让CommitLog单个文件在1GB，ConsumeQueue文件在5.72MB，不会太大。这样限制了RocketMQ底层文件的大小，就可以在进行文件读写的时候，很方便的进行内存映射了。

### 基于mmap技术+pagecache技术实现高性能的文件读写

接下来就可以对这个已经映射到内存里的磁盘文件进行读写操作了，比如要写入消息到CommitLog文件，你先把一个CommitLog文件通过MappedByteBuffer的map()函数映射其地址到你的虚拟内存地址。

接着就可以对这个MappedByteBuffer执行写入操作了，写入的时候他会直接进入PageCache中，然后过一段时间之后，由os的线程异步刷入磁盘中。

似乎只有一次数据拷贝的过程，他就是从PageCache里拷贝到磁盘文件里而已！这个就是你使用mmap技术之后，相比于传统磁盘IO的一个性能优化。

如果我们要从磁盘文件里读取数据呢？

会判断一下，当前你要读取的数据是否在PageCache里？如果在的话，就可以直接从PageCache里读取了！如果PageCache里没有你要的数据，那么此时就会从磁盘文件里加载数据到PageCache中去。

而且PageCache技术在加载数据的时候**，**还会将**你加载的数据块的临近的其他数据块也一起加载到PageCache里去。**

### 预映射机制 + 文件预热机制

**（1）内存预映射机制**：Broker会针对磁盘上的各种CommitLog、ConsumeQueue文件预先分配好MappedFile，也就是提前对一些可能接下来要读写的磁盘文件，提前使用MappedByteBuffer执行map()函数完成映射，这样后续读写文件的时候，就可以直接执行了。

**（2）文件预热**：在提前对一些文件完成映射之后，因为映射不会直接将数据加载到内存里来，那么后续在读取尤其是CommitLog、ConsumeQueue的时候，其实有可能会频繁的从磁盘里加载数据到内存中去。

所以其实在执行完map()函数之后，会进行madvise系统调用，就是提前尽可能多的把磁盘文件加载到内存里去。

通过上述优化，才真正能实现一个效果，就是写磁盘文件的时候都是进入PageCache的，保证写入高性能；同时尽可能多的通过map + madvise的映射后预热机制，把磁盘文件里的数据尽可能多的加载到PageCache里来，后续对CosumeQueue、CommitLog进行读取的时候，才能尽可能从内存里读取数据。

## 27.事务消息机制

发送half消息到MQ去，试探一下MQ是否正常。

**half消息失败了**

说明现在肯定没法跟MQ通信了。这个时候你的订单系统就应该执行一系列的回滚操作。比如对订单状态做一个更新，让状态变成“关闭交易”，同时通知支付系统自动进行退款，交易失败。

**half消息成功了**

订单系统就应该在自己本地的数据库里执行一些增删改操作。

**订单系统的本地事务执行失败了**

让订单系统发送一个rollback请求给MQ，把之前发的half消息给删除掉。订单系统走后续的回退流程，就是通知支付系统退款。

**订单系统完成了本地事务**

完成了本地的事务操作，可以发送一个commit请求给MQ，要求让MQ对之前的half消息进行commit操作，让红包系统可以看见这个订单支付成功消息.

half状态的时候，红包系统是看不见他的，没法获取到这条消息，必须等到订单系统执行commit请求，消息被commit之后，红包系统才可以看到和获取这条消息进行后续处理。

**发送half消息成功了，但是没收到响应呢？**

RocketMQ这里有一个补偿流程，他会去扫描自己处于half状态的消息，如果我们一直没有对这个消息执行commit/rollback操作，超过了一定的时间，他就会回调你的订单系统的一个接口.

他会问问你：这个消息到底怎么回事？你到底是打算commit这个消息还是要rollback这个消息？

这个时候我们的订单系统就得去查一下数据库，看看这个订单当前的状态，一下发现订单状态是“已关闭”，此时就知道，你必然得发送rollback请求给MQ去删除之前那个half消息了！

**如果rollback或者commit发送失败了呢？**

因为MQ里的消息一直是half状态，所以说他过了一定的超时时间会发现这个half消息有问题，他会回调你的订单系统的接口.

你此时要判断一下，这个订单的状态如果更新为了“已完成”，那你就得再次执行commit请求，反之则再次执行rollback请求。

**没有执行rollback或者commit会怎么样？**

会在后台有定时任务，定时任务会去扫描RMQ_SYS_TRANS_HALF_TOPIC中的half消息，如果你超过一定时间还是half消息，他会回调订单系统的接口，让你判断这个half消息是要rollback还是commit.

**如果执行rollback操作的话，如何标记消息回滚？**

RocketMQ都是顺序把消息写入磁盘文件的，所以在这里如果你执行rollback，他的本质就是用一个OP操作来标记half消息的状态.

RocketMQ内部有一个OP_TOPIC，此时可以写一条rollback OP记录到这个Topic里，标记某个half消息是rollback了。

假设你一直没有执行commit/rollback，RocketMQ会回调订单系统的接口去判断half消息的状态，但是他最多就是回调15次，如果15次之后你都没法告知他half消息的状态，就自动把消息标记为rollback。

**如果执行commit操作，如何让消息对红包系统可见？**

你执行commit操作之后，RocketMQ就会在OP_TOPIC里写入一条记录，标记half消息已经是commit状态了。

接着需要把放在RMQ_SYS_TRANS_HALF_TOPIC中的half消息给写入到OrderPaySuccessTopic的ConsumeQueue里去，然后我们的红包系统可以就可以看到这条消息进行消费了。

### half 消息是如何对消费者不可见的？

RocketMQ一旦发现你发送的是一个half消息，他不会把这个half消息的offset写入OrderPaySuccessTopic的ConsumeQueue里去。

他会把这条half消息写入到自己内部的“RMQ_SYS_TRANS_HALF_TOPIC”这个Topic对应的一个ConsumeQueue里去.所以你的红包系统自然无法从OrderPaySuccessTopic的ConsumeQueue中看到这条half消息了.

### 什么情况下订单系统会收到half消息成功的响应？

必须要half消息进入到RocketMQ内部的RMQ_SYS_TRANS_HALF_TOPIC的ConsumeQueue文件了，此时就会认为half消息写入成功了，然后就会返回响应给订单系统。

### 重试机制行吗

先执行订单本地事务，接着发送消息到MQ，如果订单本地事务执行失败了，则不会继续发送消息到MQ了；

可能会有一个问题，假设你刚执行完成了订单本地事务了，结果还没等到你发送消息到MQ，结果你的订单系统突然崩溃了！

这就导致你的订单状态可能已经修改为了“已完成”，但是消息却没发送到MQ去！

把订单本地事务和重试发送MQ消息放到一个事务代码中行吗

```html
@Transactional
public void payOrderSuccess(){
    try{
        //执行订单本地事务
        orderService.finishOrderPay();
        //发送消息到MQ去
        producer.sendMessage();
    }catch(Exception e){
        //如果发送消息失败，重试
        for(int i=0;i<3;i++){
            //重试发送消息
        }
        //如果多次重试发送消息后还是不行，抛出异常，回滚本地事务
        throw new XXXException();
    }
}
```

假设用户支付成功了，然后支付系统回调通知你的订单系统说，有一笔订单已经支付成功了，这个时候你的订单系统卡在多次重试MQ的代码那里，可能耗时了好几秒种，此时回调通知你的系统早就等不及可能都超时异常了。

而且你把重试MQ的代码放在这个逻辑里，可能会导致订单系统的这个接口性能很差.

订单数据库的操作会回滚，但是Redis、Elasticsearch中的数据更新会自动回滚吗？

### 保证业务系统一致性的最佳方案：基于RocketMQ的事务消息机制

如果half消息发送就失败了，你就直接回滚整个流程。如果half消息发送成功了，后续的rollback或者commit发送失败了，你不需要自己去卡在那里反复重试，你直接让代码结束即可，因为后续MQ会过来回调你的接口让你判断再次rollback or commit的。

## 28.RocketMQ事物消息的代码实现细节

### 发送half事务消息出去

```html
//将消息作为half消息的模式发送出去
SendResult sendResult=producer.sendMessageInTransaction(mag,null);
```

### 假如half消息发送失败，或者没收到half消息响应怎么办？

会在执行“producer.sendMessageInTransaction(msg, null)”的时候，收到一个异常，发现消息发送失败了。

```html
try{
    SendResult sendResult=producer.sendMessageInTransaction(mag,null);
}catch(Exception e){
    //half消息发送失败，订单系统回滚，比如退款等
}
```

那如果一直没有收到half消息发送成功的通知呢？

针对这个问题，我们可以把发送出去的half消息放在内存里，或者写入本地磁盘文件，后台开启一个线程去检查，如果一个half消息超过比如10分钟都没有收到响应，那就自动触发回滚逻辑。

### 如果half消息成功了，如何执行订单本地事务？

代码里有一个TransactionListener，这个类也是我们自己定义的

```html
public class TransactionListenerImpl implements TransactionListener{
    //half消息发送成功，回调该函数，执行本地事务
    @Override
    public LocalTransactionState executeLocalTransaction(Message msg,Object arg){
        //执行本地事务，按照执行结果选择执行commit或rollback
        try{
            return LocalTransactionState.COMMIT_MESSAGE;
        }catch(Exception e){
            return LocalTransactionState.ROLLBACK_MESSAGE;
        }
    }
}
```

### 如果没有返回commit或者rollback，如何进行回调？

```html
@Override
    public LocalTransactionState checkLocalTransaction(MessageExt msg){
        //查询本地事务是否执行成功
        Integer status=localTrans.get(msg.getTransactionId());
        //根据本地事务的情况去选择执行commit或rollback
        if(null!=status){
            switch(status){
                case 0:return LocalTransactionState.UNKNOW;
                case 1:return LocalTransactionState.COMMIT_MESSAGE;
                case 2:return LocalTransactionState.ROLLBACK_MESSAGE;
            }
        }
        return LocalTransactionState.COMMIT_MESSAGE;
    }
```

## 29.Broker消息零丢失方案：同步刷盘 + Raft协议主从同步

即使你写入MQ成功了，这条消息也大概率是仅仅停留在MQ机器的os cache中，一旦机器宕机内存里的数据都会丢失，或者哪怕消息已经进入了MQ机器的磁盘文件里，但是磁盘一旦坏了，消息也会丢失。

在异步刷盘的模式下，消息只要进入os cache这个内存就可以了，写消息的性能就是写内存的性能，那每秒钟可以写入的消息数量肯定更多了，但是这个情况下，可能就会导致数据的丢失。

如果一定要确保数据零丢失的话，可以调整MQ的刷盘策略，我们需要调整broker的配置文件，将其中的flushDiskType配置设置为：SYNC_FLUSH，默认他的值是ASYNC_FLUSH，即默认是异步刷盘的。

如果调整为同步刷盘之后，我们写入MQ的每条消息，只要MQ告诉我们写入成功了，那么他们就是已经进入了磁盘文件了！

**如何避免磁盘故障导致的数据丢失？**

对Broker使用主从架构的模式.

必须让一个Master Broker有一个Slave Broker去同步他的数据，而且你一条消息写入成功，必须是让Slave Broker也写入成功，保证数据有多个副本的冗余。

## 30.Consumer消息零丢失方案：手动提交offset + 自动故障转移

如果红包系统已经拿到了这条消息，但是消息目前还在他的内存里，还没执行派发红包的逻辑，此时他就直接提交了这条消息的offset到broker去说自己已经处理过了，接着红包系统直接崩溃了，内存里的消息就没了，红包也没派发出去，结果Broker已经收到他提交的消息offset了，还以为他已经处理完这条消息了。等红包系统重启的时候，就不会再次消费这条消息了。

```html
consumer.registerMessageListener(new MessageListenerConcurrently(){
        @Override
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,ConsumeConcurrentlyContext context){
            //在这里对获取到的msgs订单消息进行处理
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
}
);
```

RocketMQ的消费者中会注册一个监听器，就是上面小块代码中的MessageListenerConcurrently这个东西，当你的消费者获取到一批消息之后，就会回调你的这个监听器函数，让你来处理这一批消息。

然后当你处理完毕之后，你才会返ConsumeConcurrentlyStatus.CONSUME_SUCCESS作为消费成功的示意，告诉RocketMQ，这批消息我已经处理完毕了。

如果你对一批消息都处理完毕了，然后再提交消息的offset给broker，接着红包系统崩溃了，此时是不会丢失消息的.

如果是红包系统获取到一批消息之后，还没处理完，也就没返回ConsumeConcurrentlyStatus.CONSUME_SUCCESS这个状态呢，自然没提交这批消息的offset给broker呢，此时红包系统突然挂了，会怎么样？

其实在这种情况下，你对一批消息都没提交他的offset给broker的话，broker不会认为你已经处理完了这批消息，此时你突然红包系统的一台机器宕机了，他其实会感知到你的红包系统的一台机器作为一个Consumer挂了。

接着他会把你没处理完的那批消息交给红包系统的其他机器去进行处理，所以在这种情况下，消息也绝对是不会丢失的.

### 不能异步消费消息

在默认的Consumer的消费模式之下，必须是你处理完一批消息了，才会返回ConsumeConcurrentlyStatus.CONSUME_SUCCESS这个状态标识消息都处理结束了，去提交offset到broker去。

在这种情况下，正常来说是不会丢失消息的，即使你一个Consumer宕机了，他会把你没处理完的消息交给其他Consumer去处理。

但是这里我们要警惕一点，就是**我们不能在代码中对消息进行异步的处理**。

如果要是用这种方式来处理消息的话，那可能就会出现你开启的子线程还没处理完消息呢，你已经返回ConsumeConcurrentlyStatus.CONSUME_SUCCESS状态，然后此时你红包系统突然宕机，必然会导致你的消息丢失了！

## 31.全链路消息零丢失方案总结

### 发送消息到MQ的零丢失：

方案一（同步发送消息 + 反复多次重试）

方案二（事务消息机制），两者都有保证消息发送零丢失的效果，但是经过分析，事务消息方案整体会更好一些

### MQ收到消息之后的零丢失：

开启同步刷盘策略 + 主从架构同步机制，只要让一个Broker收到消息之后同步写入磁盘，同时同步复制给其他Broker，然后再返回响应给生产者说写入成功，此时就可以保证MQ自己不会弄丢消息

### 消费消息的零丢失：

采用RocketMQ的消费者天然就可以保证你处理完消息之后，才会提交消息的offset到broker去，只要记住别采用多线程异步处理消息的方式即可.

### 消息零丢失方案弊端

发送一条消息，都至少要half + commit两次请求，会导致你的发送消息的性能大幅度下降，同时发送消息到MQ的吞吐量大幅度下降。

MQ的一台broker机器收到了消息之后，必然直接把消息刷入磁盘里。

接着你的这台broker机器还必须直接把消息复制给其他的broker，完成多副本的冗余，这个过程涉及到两台broker机器之间的网络通信。

为了保证数据不丢失，你必须是处理完一批消息再返回CONSUME_SUCCESS状态，那么此时你消费者处理消息的速度会降低，吞吐量 自然也会下降了！

### 消息零丢失方案到底适合什么场景？

对于跟金钱、交易以及核心数据相关的系统和核心链路，可以上这套消息零丢失方案。

对于其他大部分没那么核心的场景和系统，其实即使丢失一些数据，也不会导致太大的问题，此时可以不采取这些方案，或者说你可以在其他的场景里做一些简化。

比如你可以把事务消息方案退化成“**同步发送消息 + 反复重试几次**”的方案，如果发送消息失败，就重试几次，但是大部分时候可能不需要重试，那么也不会轻易的丢失消息的！最多在这个方案里，可能会出现一些数据不一致的问题。

## 32.消息重复发送

如果出现了接口超时等问题，可能会导致上游的支付系统重试调用订单系统的接口，进而导致订单系统对一个消息重复发送两条到MQ里去！

假设你发送了一条消息到MQ了，其实MQ是已经接收到这条消息了，结果MQ返回响应给你的时候，网络有问题超时了，就是你没能及时收到MQ返回给你的响应。这个时候，你的代码里可能会发现一个网络超时的异常，然后你就会进行重试再次发送这个消息到MQ去，然后MQ必然会收到一条一模一样的消息，进而导致你的消息重复发送了！

没来得及提交消息offset到broker，系统就进行了一次重启，因为你没提交这条消息的offset给broker，broker并不知道你已经处理完了这条消息，重启之后，broker就会再次把这条消息交给你，让你再一次进行处理。

## 33.幂等性机制，保证数据不会重复

幂等性机制，就是用来避免对同一个请求或者同一条消息进行重复处理的机制。

### 发送消息到MQ的时候如何保证幂等性？

第一个方案就是业务判断法，也就是说你的订单系统必须要知道自己到底是否发送过消息到MQ去，消息到底是否已经在MQ里了。

这种方案RocketMQ虽然是支持你查询某个消息是否存在的，但是性能也不是太好。

第二种方法，就是状态判断法，这个方法的核心在于，你需要引入一个Redis缓存来存储你是否发送过消息的状态，如果你成功发送了一个消息到MQ里去，你得在Redis缓存里写一条数据，标记这个消息已经发送过。

这种方案一般情况下是可以做到幂等性的，但是如果有时候你刚发送了消息到MQ，还没来得及写Redis，系统就挂了，之后你的接口被重试调用的时候，你查Redis还以为消息没发过，就会发送重复的消息到MQ去。没法100%保证幂等性。

在这里建议是不用在这个环节保证幂等性，也就是我们可以默许他可能会发送重复的消息到MQ里去。

### 优惠券系统如何保证消息处理的幂等性？

基于业务判断法就可以了，因为优惠券系统每次拿到一条消息后给用户发一张优惠券，实际上核心就是在数据库里给用户插入一条优惠券记录.只要先去优惠券数据库中查询一下，比如对订单id=1100的订单，是否已经发放过优惠券了，是否有优惠券记录，如果有的话，就不要重复发券了！

## 34.数据库宕机，会怎么样？

优惠券系统的数据库宕机了，就必然会导致我们从MQ里获取到消息之后是没办法进行处理的。

如果我们因为数据库宕机等问题，对这批消息的处理是异常的，此时没法处理这批消息，我们就应该返回一个RECONSUME_LATER状态.

意思是，我现在没法完成这批消息的处理，麻烦你稍后过段时间再次给我这批消息让我重新试一下！

如果你返回了RECONSUME_LATER状态，他会把你这批消息放到你这个消费组的重试队列中去.

比如你的消费组的名称是“VoucherConsumerGroup”，意思是优惠券系统的消费组，那么他会有一个“%RETRY%VoucherConsumerGroup”这个名字的重试队列。

然后过一段时间之后，重试队列中的消息会再次给我们，让我们进行处理。如果再次失败，又返回了RECONSUME_LATER，那么会再过一段时间让我们来进行处理，默认最多是重试16次！每次重试之间的间隔时间是不一样的，这个间隔时间可以如下进行配置：

messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h

上面这段配置的意思是，第一次重试是1秒后，第二次重试是5秒后，第三次重试是10秒后，第四次重试是30秒后，第五次重试是1分钟后，以此类推，最多重试16次！

### 如果连续重试16次还是无法处理消息，然后怎么办？

另外一个队列，叫做**死信队列**，所谓的死信队列，顾名思义，就是死掉的消息就放这个队列里。

什么叫死掉的消息呢？

其实就是一批消息交给你处理，你重试了16次还一直没处理成功，就不要继续重试这批消息了，你就认为他们死掉了就可以了。然后这批消息会自动进入死信队列。

死信队列的名字是“%DLQ%VoucherConsumerGroup”。

那么对死信队列中的消息我们怎么处理？

其实这个就看你的使用场景了，比如我们可以专门开一个后台线程，就是订阅“%DLQ%VoucherConsumerGroup”这个死信队列，对死信队列中的消息，还是一直不停的重试。

## 35.消息乱序

可以给每个Topic指定多个MessageQueue，然后你写入消息的时候，原本有顺序的消息，完全可能会分发到不同的MessageQueue中去，然后不同机器上部署的Consumer可能会用混乱的顺序从不同的MessageQueue里获取消息然后处理。

## 36.如何解决消息乱序问题？

要解决这个消息的乱序问题，最根本的方法其实非常简单，就是得想办法让一个订单的binlog进入到一个MessageQueue里去。

往MQ里发送binlog的时候，根据订单id来判断一下，如果订单id相同，你必须保证他进入同一个MessageQueue。

可以采用取模的方法，比如有一个订单id是1100，那么他可能有2个binlog，对这两个binlog，我们必须要用订单id=1100对MessageQueue的数量进行取模，比如MessageQueue一共有15个，那么此时订单id=1100对15取模，就是5.

也就是说，凡是订单id=1100的binlog，都应该进入位置为5的MessageQueue中去！

将binlog发送给MQ的时候，必须将一个订单的binlog都发送到一个MessageQueue里去，而且发送过去的时候，也必须是严格按照顺序来发送的.

### Consumer有序处理一个订单的binlog

一个Consumer可以处理多个MessageQueue的消息，但是一个MessageQueue只能交给一个Consumer来进行处理，所以一个订单的binlog只会有序的交给一个Consumer来进行处理！

### 万一消息处理失败了？

对于有序消息的方案中，如果你遇到消息处理失败的场景，就必须返回SUSPEND_CURRENT_QUEUE_A_MOMENT这个状态，意思是先等一会儿，一会儿再继续处理这批消息，而不能把这批消息放入重试队列去，然后直接处理下一批消息。

## 37.RocketMQ的顺序消息机制的代码实现

### 如何让一个订单的binlog进入一个MessageQueue？

```html
SendResult sendResult=producer.send(message,new MessageQueueSelector(){
    @Override
    public MessageQueue select(List<MessageQueue> mqs,Message msg,Object arg){
        Long orderId=(Long)arg;//根据订单id选择发送queue
        long index=id%mqs.size();//用订单id对MessageQueue数据取模
        return mqs.get((int)index);//返回一个MessageQueue
    }
},
    orderId//这里传入订单id
)
```

在上面的代码片段中，我们可以看到，关键因素就是两个，一个是发送消息的时候传入一个MessageQueueSelector，在里面你要根据订单id和MessageQueue数量去选择这个订单id的数据进入哪个MessageQueue。

同时在发送消息的时候除了带上消息自己以外，还要带上订单id，然后MessageQueueSelector就会根据订单id去选择一个MessageQueue发送过去，这样的话，就可以保证一个订单的多个binlog都会进入一个MessageQueue中去。

### 消费者如何保证按照顺序来获取一个MessageQueue中的消息？

```html
consumer.registerMessgeListener(new MessageListenerOrderly(){
    @Override
    public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs,ConsumeOrderlyContext context){
        context.setAutoCommit(true);
        try{
            for(MessageExt msg:msgs){
                //对有序的消息进行处理
            }
            return ConsumeOrderStatus.SUCCESS;
        }catch(Exception e){
            //如果消息处理有问题
            //返回一个状态，让他暂停一会再继续处理这批消息
            return SUSPEND_CURRENT_QUEUE_A_MOMENT;
        }
    }      
});
```

在上面的代码中，大家可以注意一下，我们使用的是MessageListenerOrderly这个东西，他里面有Orderly这个名称.

也就是说，Consumer会对每一个ConsumeQueue，都仅仅用一个线程来处理其中的消息。

## 38.数据过滤

假设我们的系统仅仅关注订单数据库中的表A的binlog，并不关注其他表的binlog，那么大数据系统可能需要在获取到所有表的binlog之后，对每条binlog判断一下，是否是表A的binlog？

如果不是表A的binlog，那么就直接丢弃不要处理；如果是表A的binlog，才会去进行处理！

但是这样的话，必然会导致大数据系统处理很多不关注的表的binlog，也会很浪费时间，降低消息的效率。

### 在发送消息的时候，给消息设置tag和属性

```html
Message msg=new Messge(
    "TopicOrderDbData",//这是我们
    "TableA",//这是这条数据的tag,可以是表的名字
    ("binlog").getBytes(RemotingHelper.DEFAULT_CHARSET)//这是一条binlog数据
);
//我们可以给一条消息设置一些属性
msg.putUserProperty("a",10);
msg.putUserProperty("b","abc");
```

上面的代码清晰的展示了我们发送消息的时候，其实是可以给消息设置tag、属性等多个附加的信息的。

### 在消费数据的时候根据tag和属性进行过滤

接着我们可以在消费的时候根据tag和属性进行过滤，比如我们可以通过下面的代码去指定，我们只要tag=TableA和tag=TableB的数据。

```html
consumer.subscribe("TopicOrderDbData","TableA||TableB");
```

或者我们也可以通过下面的语法去指定，我们要根据每条消息的属性的值进行过滤，此时可以支持一些语法，比如：

```html
consumer.subscribe("TopicOrderDbData",MessageSelector.bySql("a>5 AND b='abc'"));
//只要a大于5,b等于abc的数据
```

RocketMQ还是支持比较丰富的数据过滤语法的，如下所示：

（1）数值比较，比如：>，>=，<，<=，BETWEEN，=；

（2）字符比较，比如：=，<>，IN；

（3）IS NULL 或者 IS NOT NULL；

（4）逻辑符号 AND，OR，NOT；

（5）数值，比如：123，3.1415；

（6）字符，比如：'abc'，必须用单引号包裹起来；

（7）NULL，特殊的常量

（8）布尔值，TRUE 或 FALSE

## 39.延迟消息机制

一般订单系统都必须设置一个规则，当一个订单下单之后，超过比如30分钟没有支付，那么就必须订单系统自动关闭这个订单，后续你如果要购买这个订单里的商品，就得重新下订单了。

MQ里的**延迟消息**往往就会出场了，他是特别适合在这种场景里使用的.

所谓延迟消息，意思就是说，我们订单系统在创建了一个订单之后，可以发送一条消息到MQ里去，我们指定这条消息是延迟消息，比如要等待30分钟之后，才能被订单扫描服务给消费到.

这样当订单扫描服务在30分钟后消费到了一条消息之后，就可以针对这条消息的信息，去订单数据库里查询这个订单，看看他在创建过后都过了30分钟了，此时他是否还是未支付状态？如果此时订单还是未支付状态，那么就可以关闭他，否则订单如果已经支付了，就什么都不用做了.

## 40.延迟消息的代码实现

```html
public class ScheduledMessageProducer{
    public static void main(String[] args) throws Exception{
        //这是订单系统的生产者
        DefaultMQProducer producer=new DefaultMQProducer("OrderSystemProducerGroup");
        //启动生产者
        producer.start();
        Message message=new Message(
            "CreateOrderInformTopic",//这是创建订单通知的topic
            orderInfoJSON.getBytes()//这是订单信息的json串
        );
        //这里设置了消息为延迟消息,延迟级别为3
        message.setDelayTimeLevel(3);
        //发送消息
        producer.send(message);
    }
}
```

大家看上面的代码，其实发送延迟消息的核心，就是设置消息的**delayTimeLevel**，也就是延迟级别.

RocketMQ默认支持一些延迟级别如下：1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h

所以上面代码中设置延迟级别为3，意思就是延迟10s，你发送出去的消息，会过10s被消费者获取到。那么如果是订单延迟扫描场景，可以设置延迟级别为16，也就是对应上面的30分钟。

## 41.RocketMQ的生产实践中积累的各种一手经验总结

### 灵活的运用 tags来过滤数据

建议大家合理的规划Topic和里面的tags，一个Topic代表了一类业务消息数据，然后对于这类业务消息数据，如果你希望继续划分一些类别的话，可以在发送消息的时候设置tags。

### 基于消息key来定位消息是否丢失

怎么从MQ里查消息是否丢失呢？

可以基于消息key来实现，比如通过下面的方式设置一个消息的key为订单id：message.setKeys(orderId)，这样这个消息就具备一个key了。

接着这个消息到broker上，会基于key构建hash索引，这个hash索引就存放在IndexFile索引文件里。

然后后续我们可以通过MQ提供的命令去根据key查询这个消息，类似下面这样：mqadmin queryMsgByKey -n 127.0.0.1:9876 -t SCANRECORD -k orderId

### 消息零丢失方案的补充

一般假设MQ集群彻底崩溃了，你生产者就应该把消息写入到本地磁盘文件里去进行持久化，或者是写入数据库里去暂存起来，等待MQ恢复之后，然后再把持久化的消息继续投递到MQ里去。

### 提高消费者的吞吐量

如果消费的时候发现消费的比较慢，那么可以提高消费者的并行度，常见的就是部署更多的consumer机器.

但是这里要注意，你的Topic的MessageQueue得是有对应的增加，因为如果你的consumer机器有5台，然后MessageQueue只有4个，那么意味着有一个consumer机器是获取不到消息的。

然后就是可以增加consumer的线程数量，可以设置consumer端的参数：consumeThreadMin、consumeThreadMax，这样一台consumer机器上的消费线程越多，消费的速度就越快。

此外，还可以开启消费者的批量消费功能，就是设置consumeMessageBatchMaxSize参数，他默认是1，但是你可以设置的多一些，那么一次就会交给你的回调函数一批消息给你来处理了，此时你可以通过SQL语句一次性批量处理一些数据，比如：update xxx set xxx where id in (xx,xx,xx)。

通过批量处理消息的方式，也可以大幅度提升消息消费的速度。

### 要不要消费历史消息

其实consumer是支持设置从哪里开始消费消息的，常见的有两种：一个是从Topic的第一条数据开始消费，一个是从最后一次消费过的消息之后开始消费。对应的是：CONSUME_FROM_LAST_OFFSET，CONSUME_FROM_FIRST_OFFSET.

一般来说，我们都会选择CONSUME_FROM_FIRST_OFFSET，这样你刚开始就从Topic的第一条消息开始消费，但是以后每次重启，你都是从上一次消费到的位置继续往后进行消费的。

## 42.权限机制的控制？

规定好订单团队的用户，只能使用“OrderTopic”，然后商品团队的用户只能使用“ProductTopic”，大家互相之间不能混乱的使用别人的Topic。

首先我们需要在broker端放一个额外的ACL权限控制配置文件，里面需要规定好权限，包括什么用户对哪些Topic有什么操作权限，这样的话，各个Broker才知道你每个用户的权限。

首先在每个Broker的配置文件里需要设置aclEnable=true这个配置，开启权限控制.

其次，在每个Broker部署机器的${ROCKETMQ_HOME}/store/config目录下，可以放一个plain_acl.yml的配置文件，这个里面就可以进行权限配置，类似下面这样子：

```html
# 这个参数就是全局性的白名单
# 这里定义的ip地址，都是可以访问Topic的
globalWhiteRemoteAddresses:
- 13.21.33.*
- 192.168.0.*
# 这个accounts就是说，你在这里可以定义很多账号
# 每个账号都可以在这里配置对哪些Topic具有一些操作权限
accounts:
# 这个accessKey其实就是用户名的意思，比如我们这里叫做“订单技术团队”
- accessKey: OrderTeam
# 这个secretKey其实就是这个用户名的密码
  secretKey: 123456
# 下面这个是当前这个用户名下哪些机器要加入白名单的
  whiteRemoteAddress:
# admin指的是这个账号是不是管理员账号
  admin: false
# 这个指的是默认情况下这个账号的Topic权限和ConsumerGroup权限
  defaultTopicPerm: DENY
  defaultGroupPerm: SUB
# 这个就是这个账号具体的堆一些账号的权限
# 下面就是说当前这个账号对两个Topic，都具备PUB|SUB权限，就是发布和订阅的权限
# PUB就是发布消息的权限，SUB就是订阅消息的权限
# DENY就是拒绝你这个账号访问这个Topic
  topicPerms:
  - CreateOrderInformTopic=PUB|SUB
  - PaySuccessInformTopic=PUB|SUB
# 下面就是对ConsumerGroup的权限，也是同理的
  groupPerms:
  - groupA=DENY
  - groupB=PUB|SUB
  - groupC=SUB
# 下面就是另外一个账号了，比如是商品技术团队的账号
- accessKey: ProductTeam
  secretKey: 12345678
  whiteRemoteAddress: 192.168.1.*
  # 如果admin设置为true，就是具备一切权限
  admin: true
```

如果你一个账号没有对某个Topic显式的指定权限，那么就是会采用默认Topic权限。

接着我们看看在你的生产者和消费者里，如何指定你的团队分配到的RocketMQ的账号，当你使用一个账号的时候，就只能访问你有权限的Topic。

```html
DefaultMQProducer producer=new DefaultMQProducer(
    "OrderProducerGroup",
    new AclClientRPCHook(new SessionCredentials(OrderTeam,"123456"))
);
```

上面的代码中就是在创建Producer的时候后，传入进去一个AclClientRPCHook，里面就可以设置你这个Producer的账号密码，对于创建Consumer也是同理的。通过这样的方式，就可以在Broker端设置好每个账号对Topic的访问权限，然后你不同的技术团队就用不同的账号就可以了。

## 43.消息轨迹的追踪

对于一个消息，我想要知道，这个消息是什么时候从哪个Producer发送出来的？他在Broker端是进入到了哪个Topic里去的？他在消费者层面是被哪个Consumer什么时候消费出来的？

我们有时候对于一条消息的丢失，可能就想要了解到这样的一个消息轨迹，协助我们去进行线上问题的排查。

首先需要在broker的配置文件里开启**traceTopicEnable=true**这个选项，此时就会开启消息轨迹追踪的功能。

接着当我们开启了上述的选项之后，我们启动这个Broker的时候会自动创建出来一个内部的Topic，就是**RMQ_SYS_TRACE_TOPIC**，这个Topic就是用来存储所有的消息轨迹追踪的数据的。

接着做好上述这一切事情之后，我们需要在发送消息的时候开启消息轨迹，此时创建Producer的时候要用如下的方式，下面构造函数中的第二个参数，就是enableMsgTrace参数，他设置为true，就是说可以对消息开启轨迹追踪。

```html
DefaultMQProducer producer=new DefaultMQProducer("TestProducerGroup",true);
```

在订阅消息的时候，对于Consumer也是同理的，在构造函数的第二个参数设置为true，就是开启了消费时候的轨迹追踪。

一旦当我们在Broker、Producer、Consumer都配置好了轨迹追踪之后，其实Producer在发送消息的时候，就会上报这个消息的一些数据到内置的RMQ_SYS_TRACE_TOPIC里去.

此时会上报如下的一些数据：Producer的信息、发送消息的时间、消息是否发送成功、发送消息的耗时。

接着消息到Broker端之后，Broker端也会记录消息的轨迹数据，包括如下：消息存储的Topic、消息存储的位置、消息的key、消息的tags。

然后消息被消费到Consumer端之后，他也会上报一些轨迹数据到内置的RMQ_SYS_TRACE_TOPIC里去，包括如下一些东西：Consumer的信息、投递消息的时间、这是第几轮投递消息、消息消费是否成功、消费这条消息的耗时。

接着如果我们想要查询消息轨迹，也很简单，在RocketMQ控制台里，在导航栏里就有一个消息轨迹，在里面可以创建查询任务，你可以根据messageId、message key或者Topic来查询，查询任务执行完毕之后，就可以看到消息轨迹的界面了。

在消息轨迹的界面里就会展示出来刚才上面说的Producer、Broker、Consumer上报的一些轨迹数据了。

## 44.大量消息积压问题，应该如何处理？

### 方案一

如果这些消息你是允许丢失的，那么此时你就可以紧急修改消费者系统的代码，在代码里对所有的消息都获取到就直接丢弃，不做任何的处理，这样可以迅速的让积压在MQ里的百万消息被处理掉，只不过处理方式就是全部丢弃而已。

### 方案二

临时申请多台机器多部署消费者系统的实例，多个消费者系统同时消费，每个人消费一个MessageQueue的消息，消费的速度提高了，很快积压的百万消息都会被处理完毕。

### 方案三

如果你的Topic总共就只有4个MessageQueue，然后你就只有4个消费者系统呢？

往往是临时修改那4个消费者系统的代码，让他们获取到消息然后不写入NoSQL，而是直接把消息写入一个新的Topic，这个速度是很快的，因为仅仅是读写MQ而已。

然后新的Topic有20个MessageQueue，然后再部署20台临时增加的消费者系统，去消费新的Topic后写入数据到NoSQL里去，这样子也可以迅速的增加消费者系统的并行处理能力，使用一个新的Topic来允许更多的消费者系统并行处理。

## 45.集群崩溃设计高可用方案

通常都会在你发送消息到MQ的那个系统中设计高可用的降级方案，**这个降级方案通常的思路是，你需要在你发送消息到MQ代码里去try catch捕获异常，如果你发现发送消息到MQ有异常，此时你需要进行重试。**

如果你发现连续重试了比如超过3次还是失败，说明此时可能就是你的MQ集群彻底崩溃了，此时你必须把这条重要的消息写入到本地存储中去，可以是写入数据库里，也可以是写入到机器的本地磁盘文件里去，或者是NoSQL存储中去。

之后你要不停的尝试发送消息到MQ去，一旦发现MQ集群恢复了，你必须有一个后台线程可以把之前持久化存储的消息都查询出来，然后依次按照顺序发送到MQ集群里去，这样才能保证你的消息不会因为MQ彻底崩溃会丢失。

这里要有一个很关键的注意点，就是你把消息写入存储中暂存时，一定要保证他的顺序，比如按照顺序一条一条的写入本地磁盘文件去暂存消息。

而且一旦MQ集群故障了，你后续的所有写消息的代码必须严格的按照顺序把消息写入到本地磁盘文件里去暂存，这个顺序性是要严格保证的。

## 46.为什么要给RocketMQ增加消息限流功能保证其高可用性？

在接收消息这块，必须引入一个限流机制，也就是说要限制好，你这台机器每秒钟最多就只能处理比如3万条消息，根据你的MQ集群的压测结果来，你可以通过压测看看你的MQ最多可以抗多少QPS，然后就做好限流。

一般来说，限流算法可以采取**令牌桶算法**，也就是说你每秒钟就发放多少个令牌，然后只能允许多少个请求通过。

## 47. 设计一套Kafka到RocketMQ的双写+双读技术方案，实现无缝迁移！

首先你要做到双写，也就是说，在你所有的Producer系统中，要引入一个双写的代码，让他同时往Kafka和RocketMQ中去写入消息，然后多写几天，起码双写要持续个1周左右，因为MQ一般都是实时数据，里面数据也就最多保留一周。

当你的双写持续一周过后，你会发现你的Kafka和RocketMQ里的数据看起来是几乎一模一样了，因为MQ反正也就保留最近几天的数据，当你双写持续超过一周过后，你会发现Kafka和RocketMQ里的数据几乎一模一样了。

但是光是双写还是不够的，还需要同时进行双读，也就是说在你双写的同时，你所有的Consumer系统都需要同时从Kafka和RocketMQ里获取消息，分别都用一模一样的逻辑处理一遍。

只不过从Kafka里获取到的消息还是走核心逻辑去处理，然后可以落入数据库或者是别的存储什么的，但是对于RocketMQ里获取到的消息，你可以用一样的逻辑处理，但是不能把处理结果具体的落入数据库之类的地方。

你的Consumer系统在同时从Kafka和RocketMQ进行消息读取的时候，你需要统计每个MQ当日读取和处理的消息的数量，这点非常的重要，同时对于RocketMQ读取到的消息处理之后的结果，可以写入一个临时的存储中。

同时你要观察一段时间，当你发现持续双写和双读一段时间之后，如果所有的Consumer系统通过对比发现，从Kafka和RocketMQ读取和处理的消息数量一致，同时处理之后得到的结果也都是一致的，此时就可以判断说当前Kafka和RocketMQ里的消息是一致的，而且计算出来的结果也都是一致的。

这个时候就可以实施正式的切换了，你可以停机Producer系统，再重新修改后上线，全部修改为仅仅写RocketMQ，这个时候他数据不会丢，因为之前已经双写了一段时间了，然后所有的Consumer系统可以全部下线后修改代码再上线，全部基于RocketMQ来获取消息，计算和处理，结果写入存储中。

## 48.RocketMQ的源码目录结构

1. **broker**：顾名思义，这个里面存放的就是RocketMQ的Broker相关的代码，这里的代码可以用来启动Broker进程
2. **client**：顾名思义，这个里面就是RocketMQ的Producer、Consumer这些客户端的代码，生产消息、消费消息的代码都在里面
3. **common**：这里放的是一些公共的代码
4. **dev**：这里放的是开发相关的一些信息
5. **distribution**：这里放的就是用来部署RocketMQ的一些东西，比如bin目录 ，conf目录，等等
6. **example**：这里放的是RocketMQ的一些例子
7. **filter**：这里放的是RocketMQ的一些过滤器的东西
8. **logappender和logging**：这里放的是RocketMQ的日志打印相关的东西
9. **namesvr**：这里放的就是NameServer的源码
10. **openmessaging**：这是开放消息标准，这个可以先忽略
11. **remoting**：这个很重要，这里放的是RocketMQ的远程网络通信模块的代码，基于netty实现的
12. **srvutil**：这里放的是一些工具类
13. **store**：这个也很重要，这里放的是消息在Broker上进行存储相关的一些源码
14. **style、test、tools**：这里放的是checkstyle代码检查的东西，一些测试相关的类，还有就是tools里放的一些命令行监控工具类

## 49.启动NameServer

先在RocketMQ源码目录中找到namesvr这个工程，然后展开他的目录，找到NamesvrStartup.java这个类。

接着我们需要配置一个环境变量，就是ROCKETMQ_HOME，因为NameServer启动的时候他就是要求要有这个环境变量的，所以说我们需要在NamesvrStartup的配置编辑界面里给他加入一个环境变量的配置。

我们添加一个ROCKETMQ_HOME的环境变量，他的值你就输入一个新建的目录就好了，比如：xxx/rocketmq-nameserver，你填入你自己本地的目录。

接着我们就需要在rocketmq-nameserver运行目录中创建我们需要的目录结构，此时我们需要创建conf、logs、store三个文件夹，因为后续NameServer运行是需要使用一些目录的。

然后我们把RocketMQ源码目录中的distrbution目录下的broker.conf、logback_namesvr.xml两个配置文件拷贝到刚才新建的conf目录中去，接着就需要修改这两个配置文件。

首先修改logback_namesvr.xml这个文件，修改里面的日志的目录，修改为你的rocketmq运行目录中的logs目录。里面有很多的${user.home}，你直接把这些${user.home}全部替换为你的rocketmq运行目录就可以了。

接着就是修改broker.conf文件，改成如下所示：

```html
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
# 这是nameserver的地址
namesrvAddr=127.0.0.1:9876
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
# 这是存储路径，你设置为你的rocketmq运行目录的store子目录
storePathRootDir=你的rocketmq运行目录的store子目录
# 这是commitLog的存储路径
storePathCommitLog=你的rocketmq运行目录的store子目录/commitlog
# consume queue文件的存储路径
storePathConsumeQueue=你的rocketmq运行目录的store子目录/consumequeue
# 消息索引文件的存储路径
storePathIndex=你的rocketmq运行目录的store子目录/index
# checkpoint文件的存储路径
storeCheckpoint=你的rocketmq运行目录的store子目录/checkpoint
# abort文件的存储路径
abortFile=你的rocketmq运行目录/abort
```

着就可以右击NamesvrStartup类，选择Debug NamesvrStartup.main()了，就可以用debug模式去启动NameServer了，他会自动找到ROCKETMQ_HOME环境变量，这个目录就是你的运行目录，里面有conf、logs、store几个目录。

他会读取conf里的配置文件，所有的日志都会打印在logs目录里，然后数据都会写在store目录里，启动成功之后，在Intellij IDEA的命令行里就会看到下面的提示。

Connected to the target VM, address: '127.0.0.1:54473', transport: 'socket'

The Name Server boot success. serializeType=JSON

## 50.启动Broker

首先我们依然是在Intellij IDEA中找到broker模块，然后展开他的目录，就可以找到一个BrokerStartup类，这个类是用来启动Broker进程的.

然后我们选中这个BrokerStartup类，接着依然在Intellij IDEA的上面选择Edit Configuration，进入这个启动类的配置编辑界面，但是刚开始他会直接显示出来NamesvrStartup的配置编辑界面。

我们发现他的Main class指定的是：org.apache.rocketmq.namesrv.NamesrvStartup，这不是我们要的类，所以这个时候，我们得重新为BrokerStartup类新建一个配置模板。

这个配置模板此时是没有名字的，我们在Name中输入BrokerStartup，Main class可以选择broker模块下的BrokerStartup类，Use classpath of module中可以选择broker这个module。

首先在Program arguments里，我们需要输入下面的内容，给Broker启动的时候指定一个配置文件存放地址：-c 你的rocketmq运行目录/conf/broker.conf。

接着我们需要配置环境变量，也就是ROCKETMQ_HOME，此时我们可以在Environment Variables里面添加一个ROCKETMQ_HOME环境变量，他的值就是我们的rocketmq运行目录就可以了，就是里面有conf、store、logs几个目录的。

这些都配置好了之后，那么我们的BrokerStartup启动类就配置好了，因为这个时候Broker启动会收到一个-c以及配置文件的参数，而且他知道环境变量ROCKETMQ_HOME，知道运行目录是哪个，接着他就会基于这个配置文件来启动，同时在这个运行目录中存储数据，包括写入日志。

### broker的配置文件的内容

broker.conf里的内容，这里主要是配置了NameServer的地址，然后配置了Broker的数据存储路径，包括commitlog文件、consume queue文件、index文件、checkpoint文件的存储路径。

```html
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
# 这是nameserver的地址
namesrvAddr=127.0.0.1:9876
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
# 这是存储路径，你设置为你的rocketmq运行目录的store子目录
storePathRootDir=你的rocketmq运行目录/store
# 这是commitLog的存储路径
storePathCommitLog=你的rocketmq运行目录/store/commitlog
# consume queue文件的存储路径
storePathConsumeQueue=你的rocketmq运行目录/store/consumequeue
# 消息索引文件的存储路径
storePathIndex=你的rocketmq运行目录/store/index
# checkpoint文件的存储路径
storeCheckpoint=你的rocketmq运行目录/store/checkpoint
# abort文件的存储路径
abortFile=你的rocketmq运行目录/abort
# 设置topic会自动创建
autoCreateTopicEnable=true
```

所以只要我们基于上述的broker配置文件来启动broker，那么他就会跟指定的nameserver来进行通信，然后在指定的目录里存放各种数据文件，包括在运行目录的logs目录里写入他自己的日志。

同时我们别忘了，在rocketmq-master源码目录下的distribution里，有一个logback-broker.xml，需要把这个拷贝到运行目录的conf目录中去，然后修改里面的地址，把${user.hom}都修改为你的rocketmq运行目录。

看到如下的一段提示，就说明broker启动成功了：

Connected to the target VM, address: '127.0.0.1:55275', transport: 'socket'

The broker[broker-a, 192.168.3.9:10911] boot success. serializeType=JSON

然后我们在rocketmq运行目录下的logs中，会找到一个子目录是rocketmqlogs，里面有一个broker.log，就可以看到Broker的启动日志了。

## 51.如何基于本地运行的RocketMQ进行消息的生产与消费？

### 创建一个测试用的Topic出来

首先我们需要启动rocketmq-console工程，在启动之后，我们就可以看到集群里有一台broker机器，接着我们就进入Topic菜单，新建一个名称为TopicTest的Topic即可。

### 修改和运行RocketMQ自带的Producer示例程序

```html
public class Producer{
    public static void main(String[] args) throws MQClientException,InterruptedException{
        DefaultMQProducer producer=new DefaultMQProducer("please_rename_unique_group_name");
        //最核心的是修改这里，要设置下NameServer地址才行
        //要确保Producer可以找到NameServer去获取当前的Broker地址
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();
        //这里原本是发送1000条消息，改为发1条消息即可
        for(int i=0;i<1;i++){
            try{
                Message msg=new Message("TopicTest","TagA",("Hello RocketMQ"+i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                SendResult sendResult=producer.send(msg);
            }catch(Exception e){
                e.printStackTrance();
                Thread.sleep(1000);
            }
        }
        producer.shutdown();        
    }
}
```

### 修改和运行RocketMQ自带的Consume示例程序

```html
public class Consumer{
    public static void main(String[] args) throws MQClientException,InterruptedException{
        DefaultMQPushConsumer consumer=new DefaultMQPushConsumer("please_rename_unique_group_name_4");
        //最核心的是修改这里，要设置下NameServer地址才行
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
        consumer.subscribe("TopicTest","*");
        consumer.registerMessageListener(new MessageListenerConcurrently(){
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,ConsumeConcurrentlyContext context){       
                return ConsumeConcurrentlyStatus.SUCCESS;
            }  
        });
        consumer.start();        
    }
}
```

## 52.NameServer源码解析

### NameServer的启动脚本

NameServer启动的时候，基于rocketmq-master源码中的distribution/bin目录中的mqnamesrv这个脚本来启动的，在这个脚本中有极为关键的一行命令用于启动NameServer进程，如下。

sh ${ROCKETMQ_HOME}/bin/runserver.sh org.apache.rocketmq.namesrv.NamesrvStartup $@

上面那行命令中用sh命令执行了runserver.sh这个脚本，然后通过这个脚本去启动了NamesrvStartup这个Java类。

runserver.sh这个脚本中最为关键的启动NamesrvStartup类的命令简化一下就是类似这样的一行命令：

java -server -Xms4g -Xmx4g -Xmn2g org.apache.rocketmq.namesrv.NamesrvStartup

通过java命令 + 一个有main()方法的类，就是会启动一个JVM进程，通过这个JVM进程来执行NamesrvStartup类中的main()方法，这个main()方法里就完成了NameServer启动的所有流程和工作。

### 在启动的时候都会解析哪些配置信息？

NamesrvController到底是个什么东西？

NamesrvStartup这个类的main()方法会被执行，然后执行的时候实际上会执行一个main0()这么个方法。

```html
public static NamesrvController main0(String[] args){
    try{
        NamesrvController controller=createNamesrvController(args);
        start(controller);
        return controller
    }catch(Throwable e){
        e.printStackTrace();
        System.exit(-1);
    }
    return null;
}
```

NamesrvController这个组件，很可能就是NameServer中专门用来接受Broker和客户端的网络请求的一个组件！

NamesrvController是如何被创建出来的？

NamesrvController controller = createNamesrvController(args)

调用了一个createNamesrvController()方法，创建出来了NamesrvController这个关键组件！

在启动NameServer的时候，是使用mqnamesrv命令来启动的，启动的时候可能会在命令行里给他带入一些参数，解析一下我们传递进去的一些命令行参数。

非常核心的两个NameServer的配置类

createNamesrvController里面关键代码：

```html
final NamesrvConfig namesrvConfig=new NamesrvConfig();
final NettyServerConfig nettyServerConfig=new NettyServerConfig();
nettyServerConfig.setListenPort(9876);
```

创建了NamesrvConfig和NettyServerConfig两个关键的配置类！

NamesrvConfig包含的是NameServer自身运行的一些配置参数，NettyServerConfig包含的是用于接收网络请求的Netty服务器的配置参数。

NameServer在默认的9876这个端口上接收Broker和客户端的网络请求。

### 如何初始化基于Netty的网络通信架构的？

NamesrvController是如何被创建出来的？

createNamesrvController里面关键代码：

```html
public static NamesrvController createNamesrvController(String[] args) throws IOException,JoranException{
    final NamesrvController controller=new NamesrvController(namesrvConfig,nettyServerConfig);
    controller.getConfiguration().registerConfig(properties);
}
```

NameServer启动的时候，实际上会解析配置文件，然后初始化NamesrvConfig和NettyServerConfig两个核心配置类.

NamesrvController在启动时会干什么？

start(controller)里面关键代码：

```html
public static NamesrvController start(final NamesrvController controller) throws Exception{
    if(null==controller){
        throw new IllegalArgumentException("NamesrvController is null");
    }
    boolean initResult=controller.initialize();
    if(!initResult){
        controller.shutdown();
        System.exit(-3);
    }
}
```

对NamesrvController执行了initialize初始化的操作。

Netty服务器是如何初始化的？

controller.initilize()里面关键代码：

```html
public boolean initialize(){
    this.kvConfigManager.load();
    this.remotingServer=new NettyRemotingServer(this.nettyServerConfig,this.brokerHousekeepingService);
}
```

刚开始有一个kvConfigManager.load()推测可能就是在里面有一些kv配置数据。

构造了一个NettyRemotingServer，也就是Netty网络服务器。

在NamesrvController组件被构造好之后，接着进行初始化的时候，首先就是把核心的NettyRemotingServer网络服务器组件给构造了出来。

NettyRemotingServer是如何初始化的？

NettyRemotingServer初始化代码：

```html
public NettyRemotingServer(final NettyServerConfig nettyServerConfig,final ChannelEventListener channelEventListener){
    super(
        nettyServerConfig.getServerOnewaySemaphoreValue();
        nettyServerConfig.getServerAsyncSemaphoreValue();
    );
    this.serverBootstrap=new ServerBootstrap();
}
```

this.serverBootstrap = new ServerBootstrap()，这个ServerBootstrap，就是Netty里的一个核心的类，他就是代表了一个Netty网络服务器，通过这个东西，最终可以让Netty监听一个端口号上的网络请求。

NettyRemotingServer是一个RocketMQ自己开发的网络服务器组件，但是其实底层就是基于Netty的原始API实现的一个ServerBootstrap，是用作真正的网络服务器的。

### 最终是如何启动Netty网络通信服务器的？

回到start(controller)方法：

```html
Runtime.getRuntime().addShutdownHook(new ShutdownHookThread(log,new Callable<Void>(){
    @Override
    public void call() throws Exception{
        controller.shutdown();
        return null;
    }
}));
controller.start();
return controller;
```

通过Runtime类注册了一个JVM关闭时候的shutdown钩子，就是JVM关闭的时候会执行上述注册的回调函数。

那个回调函数里执行了NamesrvController.shutdown()方法，执行关闭Netty服务器的释放网络资源和线程资源的一些代码.

controller.start()，对NamesrvController组件做一个启动操作，这样的话，就可以把他内部的Netty服务器给启动了。

Netty服务器是如何启动的？

controller.start()方法：

```html
public void start() throw Exception{
    this.remotingServer.start();
    if(this.fileWatchService!=null){
        this.fileWatchService.start();
    }
}
```

NamesrvContorller启动，核心就是在启动NettyRemotingServer，也就是Netty服务器。

.localAddress(new InetSocketAddress(this.nettyServerConfig.getListenPort()))。这行代码，其实就是设置了Netty服务器要监听的端口号，默认就是9876。

### 总结

NameServer启动之后，就会有一个核心的NamesrvController组件，他就是用于控制NameServer的所有行为的，包括内部启动一个Netty服务器去监听一个9876端口号，然后接收处理Broker和客户端发送过来的请求。

## 53.Broker源码解析

### Broker启动的时候是如何初始化自己的核心配置的？

NameServer已经启动后

NameServer启动之后，其实他就是有一个Netty服务器监听了9876端口号，此时Broker、客户端这些就可以跟NameServer建立长连接和进行网络通信了！

BrokerStartup的入口源码分析

启动Broker的时候也是通过mqbroker这种脚本来实现的，最终脚本里一定会启动一个JVM进程，开始执行一个main class的代码。

实际上Broker的JVM进程启动之后，会执行BrokerStartup的main()方法，这个BrokerStartup类，就在rocketmq源码中的broker模块里。

进入这个BrokerStartup类，在里面可以看到一个main()方法。

```html
public static void main(String[] args){
    start(createBrokerController(args));
}
```

先创建了一个Controller核心组件，然后用start()方法去启动这个Controller组件！

BrokerController的创建过程

```html
//下面三个，是broker得核心配置类
/看起来分别是broker得配置，netty服务器的配置，netty客服端的配置
final BrokerConfig brokerConfig=new BrokerConfig();
final NettyServerConfig nettyServerConfig=new NettyServerConfig();
final NettyClientConfig nettyClientConfig=new NettyClientConfig();
//设置了netty客服端的一个是否使用Tls的配置
nettyClientConfig.setUseTLS(Boolean.parseBoolean(System.getProperty(TLS_ENABLE,String.valueOf(TlsSystemConfig.tlsMode==TlsMode.ENFORCING))));
//设置了netty服务器的监听端口号是10911
nettyServerConfig.setListenPort(10911);
//broker用来存储消息的一些配置信息
final MessageStoreConfig messageStoreConfig=new MessageStoreConfig();
如果当前broker是slave的话，要设置一个特殊的参数
if(BrokerRole.SLAVE==messageStoreConfig.getBrokerRole()){
    int ratio=messageStoreConfig.getAccessMessageInMemoryMaxRatio()-10;
    messageStoreConfig.setAccessMessageInMemoryMaxRatio(ratio);
}
```

broker在这里启动的时候也是先搞了几个核心的配置组件，包括了broker自己的配置、broker作为一个netty服务器的配置、broker作为一个netty客户端的配置、broker的消息存储的配置。

当你的客户端连接到broker上发送消息的时候，那么broker就是一个netty服务器，负责监听客户端的连接请求。

但是当你的broker跟nameserver建立连接的时候，你的broker又是一个netty客户端，他要跟nameserver的netty服务器建立连接。

为核心配置类解析和填充信息

在启动broker的时候，用了-c选项带了一个配置文件的地址，此时他会读取配置文件里的你自定义的一些配置的信息，然后读取出来覆盖到那4个核心配置类里去。

### BrokerController是如何构建出来的

BrokerController是在哪里创建出来的？

首先会执行一个关键的方法，就是createBrokerController().

准备好核心配置组件:NettyServerConfig，NettyClientConfig，BrokerConfig，MessageStoreConfig。

createBrokerController()方法：

```html
final BrokerController controller = new BrokerController(
   brokerConfig,
   nettyServerConfig,
   nettyClientConfig,
   messageStoreConfig); 
controller.getConfiguration().registerConfig(properties);
```

创建一个核心的BrokerController组件。

BrokerController

BrokerController，中文叫做“Broker管理控制组件”。

我们用mqbroker脚本启动的JVM进程，实际上你可以认为就是一个Broker，这里Broker实际上应该是代表了一个JVM进程的概念，而不是任何一个代码组件！

然后BrokerStartup作为一个main class，其实是属于一个代码组件，他的作用是准备好核心配置组件，然后就是创建、初始化以及启动BrokerController这个核心组件，也就是启动一个Broker管理控制组件，让BrokerController去控制和管理Broker这个JVM进程运行过程中的一切行为，包括接收网络请求、包括管理磁盘上的消息数据，以及一大堆的后台线程的运行。

### 初始化BrokerController的时候，都干了哪些事情？

BrokerController创建完之后是在哪里初始化的？

createBrokerController()方法：

```html
boolean initResult=controller.initialize();
if(!initResult){
    controller.shutdown();
    System.exit(-3);
}
```

BrokerController初始化的过程？

BrokerController里其实也会包含核心的Netty服务器，用来接收和处理Producer以及Consumer的请求。

初始化过程，创建一堆线程池，用来调度后面需要执行额一大堆的后台定时调度任务。后续Broker要处理一大堆的各种请求，那么不同的请求是不是要用不同的线程池里的线程来处理。

BrokerController一旦初始化完成过后，他其实就准备好了Netty服务器，可以用于接收网络请求，然后准备好了处理各种请求的线程池，准备好了各种执行后台定时调度任务的线程池。

### BrokerContorller在启动的时候，都干了哪些事儿？

BrokerStartup启动组件的main()方法中，完成BrokerContorller的初始化，接着就是执行了start()方法。

start()方法里面，最主要就是执行了BrokerContorller的start()方法。

start()方法里面，启动Netty服务器，可以接收网络请求了，然后还有一个BrokerOuterAPI组件是基于Netty客户端发送请求给别人的（比如broker发送请求到NameServer去注册以及心跳，都是通过这个组件），同时还启动一个线程去向NameServer注册。

（1）Broker启动了，必然要去注册自己到NameServer去，所以BrokerOuterAPI这个组件必须要.

（2）Broker启动之后，必然要有一个网络服务器去接收别人的请求，此时NettyServer这个组件是必须要知道的.

（3）当你的NettyServer接收到网络请求之后，需要有线程池来处理，你需要知道这里应该有一个处理各种请求的线程池.

（4）你处理请求的线程池在处理每个请求的时候，是不是需要各种核心功能组件的协调？比如写入消息到commitlog，然后写入索引到indexfile和consumer queue文件里去，此时你是不是需要对应的一些MessageStore之类的组件来配合你？

（5）除此之外，你是不是需要一些后台定时调度运行的线程来工作？比如定时发送心跳到NameServer去，类似这种事情。

### Broker是如何把自己注册到NameServer去的

Broker将自己注册到NameServer的入口

BrokerController.start()方法中

```html
BrokerController.this.registerBrokerAll(true, false, brokerConfig.isForceRegister());
```

只有完成了注册，NameServer才能知道集群里有哪些Broker，然后Producer和Consumer才能找NameServer去拉取路由数据，他们才知道集群里有哪些Broker，才能去跟Broker进行通信！

进入registerBrokerAll()方法

```html
//如果要进行注册的话，就调用了doRegisterBrokerAll（）这个方法，真正去注册
if(forceRegister||needRegister(
    this.brokerConfig.getBrokerClusterName(),
    this.getBrokerAddr(),
    this.brokerConfig.getBrokerName(),
    this.brokerConfig.getBrokerId,
    this.brokerConfig.getRegisterBrokerTimeoutMills())){
        doRegisterBrokerAll(checkOrderConfig,oneway,topicConfigWrapper);
    }
```

真正进行Broker注册的方法，也就是doRegisterBrokerAll()方法。

实际上就是通过BrokerOuterAPI去发送网络请求给所有的NameServer，把这个Broker注册了上去。

深入到网络请求级别的Broker注册

去看BrokerOuterAPI中的registerBrokerAll()方法。

通过请求头RequestHeader和请求体RequestBody构成了一个请求，然后会通过底层的NettyClient把这个请求发送到NameServer去进行注册.

请求头里会加入很多信息，比如broker的id和名称。

请求体会包含一些配置。

### BrokerOuter API是如何发送注册请求的？

真正的注册Broker的网络请求方法

```html
RegisterBrokerResult result = registerBroker(
namesrvAddr,oneway, timeoutMills,requestHeader,body);
```

进入这个方法之后，

发现最终的请求是基于NettyClient这个组件给发送出去的。

进入到NettyClient的网络请求方法中。

获取一个broker与NameServer之间的channel，通过这个Channel就可以发送实际的网络请求出去！

进入this.getAndCreateChannel(addr)。

```html
private Channel getAndCreateChannel(final String addr){
    if(null==addr){
        return getAndCreateNameServerChannel();
    }
    ChannelWrapper cw=this.channelTables.get(addr);
    if(cw!=null&&cw.isOk()){
        return cw.getChannel();
    }
    return this.createChannel(addr);
}
```

先从缓存里尝试获取连接，如果没有缓存的话，就创建一个连接。

进入this.createChannel(addr)方法。

基于Netty的BootStrap这个类的connect（）方法，构建出来一个真正的channel网络连接。

如何通过Channel网络连接发送请求？

核心入口。

```html
RemotingCommand response = this.invokeSyncImpl(channel, request, timeoutMillis - costTime);
```

基于Netty的Channel API，把注册的请求给发送到了NameServer就可以了。

### NameServer是如何处理Broker的注册请求的？

NamesrvController.initialize()方法。

```html
public boolean initialize(){
    this.kvConfigManager.load();
    //初始化Netty服务器
    this.remotingServer=new NettyRomtingServer(this.nettyServerConfig,this.brokerHouseKeepingService);
    this.remotingExecutor=Executors.newFixedThreadPool(nettyServerConfig.
getServerWorkerThreads(),new ThreadFactoryImpl("RemotingExecutorThread_")); 
    //注册Proccessor
    //这个Proccessor其实就是请求处理器，是NameServer用来处理网络请求的组件
    this.registerProcessor();    
}
```

registerProcessor()方法。

```html
private void registerProcessor(){
    if(namesrcConfig.isClusterTest()){
        this.remotingServer.registerDefaultProcessor(new ClusterTestRequestProcessor(this,namesrcConfig.getProductEnvName()),this.remotingExecutor);
    }else{
        this.remotingServer.registerDefaultProcessor(newrDefaultRequestProcessor(this),this.remotingExecutor);
    }
}
```

NettyServer是用于接收网络请求的，接收到的网络请求给其实就是给DefaultRequestProcessor这个请求处理组件来进行处理的。

Broker注册请求是如何处理的

RouteInfoManager这个路由数据管理组件，实际Broker注册就是通过他来做的。

最终把一个Broker机器的数据放入RouteInfoManager中维护的路由数据表里去的。

用一些Map类的数据结构，去存放你的Broker的路由数据就可以了，包括了Broker的clusterName、brokerId、brokerName这些核心数据。

而且在更新的时候，一定会基于Java并发包下的ReadWriteLock进行读写锁加锁，因为在这里更新那么多的内存Map数据结构，必须要加一个写锁，此时只能有一个线程来更新他们才行！

### Broker是如何发送定时心跳的，以及如何进行故障感知？

Broker中的发送注册请求给NameServer的一个源码入口，其实就是在BrokerController.start()方法中，在BrokerController启动的时候，他其实并不是仅仅发送一次注册请求，而是启动了一个定时任务，会每隔一段时间就发送一次注册请求。

```html
this.scheduleExecutorService.scheduleAtFixedRate(new Runnale(){
    @Override
    public void run(){
        try{
            BrokerController.this.registerBrokerAll(true,false,brokerConfig.isForceRegister());
        }catch(Throwable e){
        }
    }
},
1000*10,
Math.max(1000,Math.min(brokerConfig.getRegisterNameServerPeriod(),60000)),TimeUnit.MILLISECONDS
);
```

上面这块代码，其实是启动了一个定时调度的任务，他默认是每隔30s就会执行一次Broker注册的过程，上面的registerNameServerPeriod是一个配置，他默认的值就是30s一次。

第一次发送注册请求就是在进行注册，他会把Broker路由数据放入到NameServer的RouteInfoManager的路由数据表里去。后续每隔30s他都会发送一次注册请求，这些后续定时发送的注册请求，其实本质上就是Broker发送心跳给NameServer了。

那么后续每隔30s，Broker就发送一次注册请求，作为心跳来发送给NameServer的时候，NameServer对后续重复发送过来的注册请求（也就是心跳），是如何进行处理的呢？

RouteInfoManager注册Broker的源码。

每隔30s你发送注册请求作为心跳的时候，RouteInfoManager里会进行心跳时间刷新的处理。

假设Broker已经挂了，或者故障了，隔了很久都没有发送那个每隔30s一次的注册请求作为心跳，那么此时NameServer是如何感知到这个Broker已经挂掉的呢？

NamesrvController的initialize()方法里去，里面有一个代码是启动了RouteInfoManager中的一个定时扫描不活跃Broker的线程。

```html
this.scheduleExecutorService.scheduleAtFixedRate(new Runnable(){
    @Override
    public void run(){
        NamesrvController.this.routeInfoManager.scanNotActiveBroker();
    }
},5,10,TimeUnit.SECONDS);
```

上面这段代码，就是启动一个定时调度线程，每隔10s扫描一次目前不活跃的Broker，使用的是RouteInfoManager中的scanNotActiveBroke()方法.

扫描BrokerLiveInfo这个心跳数据结构，在里面遍历一下，就可以拿到每个Broker最近一次心跳刷新的BrokerLiveInfo，也就知道一个Broker最近一次发送心跳是什么时候。

判断当前时间距离上一次心跳时间，如果超过了broker心跳超时时间，默认是120s，继任为Broker已经死掉了。

### 总结

Broker启动之后，最核心的就是有一个BrokerController组件管控Broker的整体行为，包括初始化自己的Netty服务器用于接收客户端的网络请求，包括启动处理请求的线程池、执行定时任务的线程池，初始化核心功能组件，同时还会启动之后发送注册请求到NameServer去注册自己。

Broker启动之后进行注册以及定时发送注册请求作为心跳的机制，以及NameServer有一个后台进程定时检查每个Broker的最近一次心跳时间，如果长时间没心跳就认为Broker已经故障。

## 54.Producer源码解析

### Producer是如何创建出来的？

使用Producer发送消息到MQ的代码

```html
DefaultMQProducer producer = new DefaultMQProducer("order_producer_group");
producer.setNamesrvAddr("localhost:9876");
producer.start();
```

其实构造Producer很简单，就是创建一个DefaultMQProducer对象实例，在其中传入你所属的Producer分组，然后设置一下NameServer的地址，最后调用他的start()方法，启动这个Producer就可以了。

其实创建DefaultMQProducer对象实例是一个非常简单的过程，无非就是创建这么一个对象出来，然后保存一下他的Producer分组。设置NameServer地址也是一个很简单的过程，无非就是保存一下NameServer地址罢了。

其实最核心的还是调用了这个DefaultMQProducer的start()方法去启动了这个消息生产组件，那么这个start()都干了什么呢？

### 构建好的Producer是如何启动准备好相关资源的？

在构造Producer的时候，他内部构造了一个真正用于执行消息发送逻辑的组件，就是DefaultMQProducerImpl这个类的实例对象。

假设我们后续要通过Producer发送消息，必然会指定我们要往哪个Topic里发送消息。所以我们也知道，Producer必然是知道Topic的一些路由数据的，比如Topic有哪些MessageQueue，每个MessageQueue在哪些Broker上。

不可能是刚初始化启动的时候就拉取Topic的路由数据，因为你刚开始启动的时候，不知道要发送消息到哪个Topic去啊！

在你第一次发送消息到Topic的时候，才会去拉取一个Topic的路由数据，包括这个Topic有几个MessageQueue，每个MessageQueue在哪个Broker上，然后从中选择一个MessageQueue，跟那台Broker建立网络连接，发送消息过去。

Producer发送消息必然要跟Broker建立网络，这个是在Producer刚启动的时候就立马跟所有的Broker建立网络连接吗？

那必然也不是的，因为此时你也不知道你要跟哪个Broker进行通信。

所以其实很多核心的逻辑，包括Topic路有数据拉取，MessageQueue选择，以及跟Broker建立网络连接，通过网络连接发送消息到Broker去，这些逻辑都是在Producer发送消息的时候才会有。

### 当我们发送消息的时候，是如何从NameServer拉取Topic元数据的？

当你调用Producer的send()方法发送消息的时候，这个方法调用会一直到比较底层的逻辑里去，最终会调用到DefaultMQProducerImpl类的sendDefaultImpl()方法里去，在这个方法里，上来什么都没干，直接就有一行非常关键的代码，如下。

TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic());

其实看到这行代码，大家就什么都明白了，每次你发送消息的时候，他都会先去检查一下，这个你要发送消息的那个Topic的路由数据是否在你客户端本地.

如果不在的话，必然会发送请求到NameServer那里去拉取一下的，然后缓存在客户端本地。

进入this.tryToFindTopicPublishInfo(msg.getTopic())这个方法。

先检查了一下自己本地是否有这个Topic的路由数据的缓存，如果没有的话就发送网络请求到NameServer去拉取，如果有的话，就直接返回本地Topic路由数据缓存了。

Producer到底是如何发送网络请求到NameServer去拉取Topic路由数据的，

对应了tryToFindTopicPublishInfo()方法内的一行代码。

this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic);

通过这行代码，他就可以去从NameServer拉取某个Topic的路由数据，然后更新到自己本地的缓存里去了。

具体的发送请求到NameServer的拉取过程， 简单来说，就是封装一个Request请求对象，然后通过底层的Netty客户端发送请求到NameServer，接收到一个Response响应对象。

然后他就会从Response响应对象里取出来自己需要的Topic路由数据，更新到自己本地缓存里去，更新的时候会做一些判断，比如Topic路由数据是否有改变过，等等，然后把Topic路由数据放本地缓存就可以了。

### 对于一条消息，Producer是如何选择MessageQueue去发送的？

当你拿到了一个Topic的路由数据之后，其实接下来就应该选择要发送消息到这个Topic的哪一个MessageQueue上去了！

Topic是一个逻辑上的概念，一个Topic的数据往往是分布式存储在多台Broker机器上的，因此Topic本质是由多个MessageQueue组成的。

每个MessageQueue都可以在不同的Broker机器上，当然也可能一个Topic的多个MessageQueue在一个Broker机器上。

发送消息的核心源码是在DefaultMQProducerImpl.sendDefaultImpl()方法中的，在这个方法里，只要你获取到了Topic的路由数据，不管从本地缓存获取的，还是从NameServer拉取到的，接着就会执行下面的核心代码。

MessageQueue mqSelected = this.selectOneMessageQueue(topicPublishInfo, lastBrokerName);

这行代码其实就是在选择Topic中的一个MessageQueue，然后发送消息到这个MessageQueue去，在这行代码里面，实现了一些Broker故障自动回避机制。

最基本的选择MessageQueue的算法。

```html
int index=tyInfo.getSendWhichQueue().getAndIncrement();
for(int i=0;i<tpInfo.getMessageQueueList().size();i++){
    int pos=Math.abs(index++)%tyInfo.getMessageQueueList().size();
    if(pos<0){
        pos=0;
    }
    MessageQueue mq=tpInfo.getMessageQueueList().get(pos);
    if(latencyFaultTolerance.isAvailable(mq.getBrokerName())){
        if(null==lastBrokerName||mq.getBrokerName().equals(lastBrokerName))
        return mq;
    }
}
```

先获取到了一个自增长的index，用这个index对Topic的MessageQueue列表进行了取模操作，获取到了一个MessageQueue列表的位置，然后返回了这个位置的MessageQueue。

这种操作就是一种简单的负载均衡的算法，比如一个Topic有8个MessageQueue，那么可能第一次发送消息到MessageQueue01，第二次就发送消息到MessageQueue02，以此类推，就是轮询把消息发送到各个MessageQueue而已！

### 我们的系统与RocketMQ Broker之间是如何进行网络通信的？

Producer从Topic路由数据中选择一个MessageQueue出来之后，接着其实就应该要把消息投递到那个MessageQueue所在的Broker上去了.

Producer是如何把消息发送给Broker的呢？

这块代码就在DefaultMQProducerImpl.sendDefaultImpl()方法中，在这个方法里，先是获取到了MessageQueue所在的broker名称，如下源码片段：

```html
brokersSent[times]=mq.getBrokerName();
```

获取到了这个brokerName之后，接着其实就可以使用如下的代码把消息投递到那个Broker上去了，看下面的代码片段：

```html
sendResult=this.sendKernelImpl(msg,mq,communicationMode,sendCallback,topicPublishInfo,timeout-costTime);
```

这个代码里是如何把消息投递出去的。

```html
String brokerAddr=this.mQClientFactory.findBrokerAddressInPublish(mq.getBrokerName());
if(null==brokerAddr){
    tryToFindTopicPublishInfo(mq.getTopic());
    brokerAddr=this.mQClientFactory.findBrokerAddressInPublish(mq.getBrokerName());
}
```

通过brokerName去本地缓存找他的实际的地址，如果找不到，就去找NameServer拉取Topic的路由数据，然后再次在本地缓存获取broker的实际地址，你有这个地址了，才能给人家进行网络通信。

用自己的方式去封装了一个Request请求出来，这里涉及到了各种信息的封装，包括了请求头，还有一大堆所有你需要的数据，都封装在Request里了。

在这里做的事情，大体上包括了给消息分配全局唯一ID、对超过4KB的消息体进行压缩，在消息Request中包含了生产者组、Topic名称、Topic的MessageQueue数量、MessageQueue的ID、消息发送时间、消息的flag、消息扩展属性、消息重试次数、是否是批量发送的消息、如果是事务消息则带上prepared标记，等等。

总之，这里就是封装了很多很多的数据就对了，这些东西都封装到一个Request里去，然后在底层还是通过Netty把这个请求发送出去，发送到指定的Broker上去就可以了。

这里Producer和Broker之间都是通过Netty建立长连接，然后基于长连接进行持续的通信的。

## 55.当Broker获取到一条消息之后

### 如何存储这条消息的？

Broker通过Netty网络服务器获取到一条消息，接着就会把这条消息写入到一个CommitLog文件里去，一个Broker机器上就只有一个CommitLog文件，所有Topic的消息都会写入到一个文件里去。

然后同时还会以异步的方式把消息写入到ConsumeQueue文件里去，因为一个Topic有多个MessageQueue，任何一条消息都是写入一个MessageQueue的，那个MessageQueue其实就是对应了一个ConsumeQueue文件.

所以一条写入MessageQueue的消息，必然会异步进入对应的ConsumeQueue文件。

同时还会异步把消息写入一个IndexFile里，在里面主要就是把每条消息的key和消息在CommitLog中的offset偏移量做一个索引，这样后续如果要根据消息key从CommitLog文件里查询消息，就可以根据IndexFile的索引来了。

首先Broker收到一个消息之后，必然是先写入CommitLog文件的，那么这个CommitLog文件在磁盘上的目录结构大致如何呢？

CommitLog文件的存储目录是在${ROCKETMQ_HOME}/store/commitlog下的，里面会有很多的CommitLog文件，每个文件默认是1GB大小，一个文件写满了就创建一个新的文件，文件名的话，就是文件中的第一个偏移量，如下面所示。文件名如果不足20位的话，就用0来补齐就可以了。

00000000000000000000

000000000003052631924

在把消息写入CommitLog文件的时候，会申请一个putMessageLock锁.

也就是说，在Broker上写入消息到CommitLog文件的时候，都是串行的，不会让你并发的写入，并发写入文件必然会有数据错乱的问题。

```html
protected final PutMessageLock putMessageLock;
PutMessageLock.lock();
```

接着其实会对消息做出一通处理，包括设置消息的存储时间、创建全局唯一的消息ID、计算消息的总长度，然后会走一段很关键的源码，把消息写入到MappedFile里去。

```html
result=cb.doAppend(this.getFileFromOffset(),byteBuffer,this.fileSize-currentPos,(MessageExtBrokerInner)messageExt);
```

cb.doAppend()这行代码，这行代码其实是把消息追加到MappedFile映射的一块内存里去，并没有直接刷入磁盘中。

至于具体什么时候才会把内存里的数据刷入磁盘，其实要看我们配置的刷盘策略，另外就是不管是同步刷盘还是异步刷盘，假设你配置了主从同步，一旦你写入完消息到CommitLog之后，接下来都会进行主从同步复制的。

### 一条消息写入CommitLog文件之后，如何实时更新索引文件？

Broker收到一条消息之后，其实就会直接把消息写入到CommitLog里去，但是他写入刚开始仅仅是写入到MappedFile映射的一块内存里去。

这个消息写入CommitLog之后，然后消息是如何进入ConsumeQueue和IndexFile的。

实际上，Broker启动的时候会开启一个线程，ReputMessageService，他会把CommitLog更新事件转发出去，然后让任务处理器去更新ConsumeQueue和IndexFile。

在DefaultMessageStore的start()方法里，在里面就是启动了这个ReputMessageService线程。

这个DefaultMessageStore的start()方法就是在Broker启动的时候调用的，所以相当于是Broker启动就会启动这个线程。

```html
this.reputMessageService.setReputFromOffset(maxPhysicalPosInLogicQueue);
this.reputMessageService.start();
```

下面我们看这个ReputMessageService线程的运行逻辑

```html
@Override
public void run(){
    while(!this.isStopped()){
        try{
            Thread.sleep(1);
            this.doReput();
        }catch(Exception e){
            
        }
    }
}
```

也就是说，在这个线程里，每隔1毫秒，就会把最近写入CommitLog的消息进行一次转发，转发到ConsumeQueue和IndexFile里去，通过的是doReput()方法来实现的.

再看doReput()方法里的实现逻辑。

```html
DispatchRequest dispatchRequest=DefaultMessageStore.this.commitLog.checkMessageAndReturnSize(result.getByteBuffer(),false,false);
```

这段代码意思非常的清晰明了，就是从commitLog中去获取到一个DispatchRequest，拿到了一份需要进行转发的消息，也就是从CommitLog中读取的.

接着他就会通过下面的代码，调用doDispatch()方法去把消息进行转发，一个是转发到ConsumeQueue里去，一个是转发到IndexFile里去.

```html
public void doDispatch(DispatchRequest req){
    for(CommitLogDispatcher dispatcher:this.dispatcherList){
        dispatcher.dispatch(req);
    }
}
```

实际上正常来说这个CommitLogDispatcher的实现类有两个，分别是CommitLogDispatcherBuildConsumeQueue和CommitLogDispatcherBuildIndex，他们俩分别会负责把消息转发到ConsumeQueue和IndexFile.

接着我们看一下ConsumeQueueDispatche的源码实现逻辑，其实非常的简单，就是找到当前Topic的messageQueueId对应的一个ConsumeQueue文件.

一个MessageQueue会对应多个ConsumeQueue文件，找到一个即可，然后消息写入其中。

```html
public void putMessagePositionInfo(DispatchRequest dispatchRequest){
    ConsumeQueue cq=this.findConsumeQueue(dispatchRequest.getTopic(),dispatchRequest.getQueueId());
    cq.putMessagePositionInfoWrapper(dispatchRequest);
}
```

再来看看IndexFile的写入逻辑，其实也很简单，无非就是在IndexFile里去构建对应的索引罢了

```html
if(DefaultMessageStore.this.messageStoreConfig.isMessageIndexEnable()){
    DefaultMessageStore.this.indexService.buildIndex(request);
}
```

当我们把消息写入到CommitLog之后，有一个后台线程每隔1毫秒就会去拉取CommitLog中最新更新的一批消息，然后分别转发到ConsumeQueue和IndexFile里去，这就是他底层的实现原理。

### 总结

数据写入到Broker之后的存储流程，包括数据直接写入CommitLog，而且直接进入的是MappedFile映射的一块内存，不是直接进入磁盘，同时有一个后台线程会把CommitLog里更新的数据给写入到ConsumeQueue和IndexFile里去。

## 56.RocketMQ是如何实现同步刷盘以及异步刷盘两种策略的？

写入CommitLog的数据进入到MappedFile映射的一块内存里之后，后续会执行刷盘策略.

比如是同步刷盘还是异步刷盘，如果是同步刷盘，那么此时就会直接把内存里的数据写入磁盘文件，如果是异步刷盘，那么就是过一段时间之后，再把数据刷入磁盘文件里去。

往CommitLog里写数据的时候，是调用的CommitLog类的putMessage()这个方法。

```html
public PutMessageResult putMessage(final MessageExtBrokerInner msg){
    handleDiskFlush(result,putMessageResult,msg);
    handleHA(result,putMessageResult,msg);
    return putMessageResult;
}
```

末尾有两个方法调用，一个是handleDishFlush()，一个是handleHA().

顾名思义，一个就是用于决定如何进行刷盘的，一个是用于决定如何把消息同步给Slave Broker的。

### 同步刷盘的策略是如何处理的

构建了一个GroupCommitRequest，然后提交给了GroupCommitService去进行处理，然后调用request.waitForFlush()方法等待同步刷盘成功.

万一刷盘失败了，就打印日志。具体刷盘是由GroupCommitService执行的，他的doCommit()方法最终会执行同步刷盘的逻辑，里面有如下代码。

```html
CommitLog.this.mappedFileQueue.flush(0);
```

上面那行代码一层一层调用下去，最终刷盘其实是靠的MappedByteBuffer的force()方法.

```html
this.mappedByteBuffer.force();
```

这个MappedByteBuffer就是JDK NIO包下的API，他的force()方法就是强迫把你写入内存的数据刷入到磁盘文件里去，到此就是同步刷盘成功了。

### 异步刷盘

先看CommitLog.handleDiskFlush()里的的代码片段。

```html
if(!this.defaultMessageStore.getMessageStoreConfig().isTransientStorePoolEnable()){
    flushCommitLogService.wakeup();
}else{
    commitLogService.wakeup();
}
```

这里就是唤醒了一个flushCommitLogService组件.

FlushCommitLogService其实是一个线程，他是个抽象父类，他的子类是CommitRealTimeService，所以真正唤醒的是他的子类代表的线程。

具体在子类线程的run()方法里就有定时刷新的逻辑.

就是每隔一定时间执行一次刷盘，最大间隔是10s，所以一旦执行异步刷盘，那么最多就是10秒就会执行一次刷盘。

## 57.当Broker上的数据存储超过一定时间之后，磁盘数据是如何清理的？

默认broker会启动后台线程，这个后台线程会自动去检查CommitLog、ConsumeQueue文件，因为这些文件都是多个的，比如CommitLog会有多个，ConsumeQueue也会有多个。

然后如果是那种比较旧的超过72小时的文件，就会被删除掉，也就是说，默认来说，broker只会给你把数据保留3天而已，当然你也可以自己通过fileReservedTime来配置这个时间，要保留几天的时间。

这个定时检查过期数据文件的线程代码，在DefaultMessageStore这个类里，他的start()方法中会调用一个addScheduleTask()方法，里面会每隔10s定时调度执行一个后台检查任务。

```html
private void addScheduleTask(){
    this.scheduledExecutorService.scheduleAtFixedRate(new Runnable(){
        @Override
        public void run(){
            DefaultMessageStore.this.cleanFilesPeriodically();
        }
    },
    1000*60,
    this.messageStoreConfig.getCleanResourceInterval(),
    TimeUnit.MILLISECONDS
    );
}
```

这个调度任务里就会执行DefaultMessageStore.this.cleanFilesPeriodically()方法，其实就是会去周期性的清理掉磁盘上的数据文件，也就是超过72小时的CommitLog、ConsumeQueue文件。

接着我们具体看看这里的清理逻辑，他其实里面包含了清理CommitLog和ConsumeQueue的清理逻辑。

```html
private void cleanFilesPeriodically(){
    this.cleanCommitLogService.run();
    this.cleanConsumeQueueService.run();
}
```

在清理文件的时候，他会具体判断一下，如果当前时间是预先设置的凌晨4点，就会触发删除文件的逻辑，这个时间是默认的；或者是如果磁盘空间不足了，就是超过了85%的使用率了，立马会触发删除文件逻辑。

上面两个条件，第一个是说如果磁盘没有满 ，那么每天就默认一次会删除磁盘文件，默认就是凌晨4点执行，那个时候必然是业务低峰期，因为凌晨4点大部分人都睡觉了，无论什么业务都不会有太高业务量的。

第二个是说，如果磁盘使用率超过85%了，那么此时可以允许继续写入数据，但是此时会立马触发删除文件的逻辑；如果磁盘使用率超过90%了，那么此时不允许在磁盘里写入新数据，立马删除文件。这是因为，一旦磁盘满了，那么你写入磁盘会失败，此时你MQ就彻底故障了。

所以一旦磁盘满了，也会立马删除文件的。

在删除文件的时候，无非就是对文件进行遍历，如果一个文件超过72小时都没修改过了，此时就可以删除了，哪怕有的消息你可能还没消费过，但是此时也不会再让你消费了，就直接删除掉。

这就是RocketMQ的一整套文件删除的逻辑和机制。

## 58.源码解析：Consumer

### Consumer作为消费者是如何创建出来的？

我们平时创建的一般都是DefaultMQPushConsumerImpl，然后会调用他的start()方法来启动他。

首先在启动的时候，会看到如下一行源码片段：

```html
this.mQClientFactory=MQClientManager.getInstance().getOrCreateMQClientInstance(this.defaultMQPushConsumer,this.rpcHook);
```

这个Consumer一旦启动，必然是要跟Broker去建立长连接的，底层绝对也是基于Netty去做的，建立长连接之后，才能不停的通信拉取消息.

所以这个MQClientFactory底层直觉上就应该封装了Netty网络通信的东西.

一个叫做RebalanceImpl的东西，专门负责Consumer重平衡的。

假设你的ConsumerGroup里加入了一个新的Consumer，那么就会重新分配每个Consumer消费的MessageQueue，如果ConsumerGroup里某个Consumer宕机了，也会重新分配MessageQueue，这就是所谓的重平衡。

```html
this.pullAPIWrapper=new PullAPIWrapper(mQClientFactory,this.defaultMQPushConsumer.getConsumerGroup(),isUnitMode());
this.pullAPIWrapper.registerFilterMessageHook(filterMessageHookList);
```

PullAPIWrapper，用来拉取消息的。

下面还有一个OffsetStore，用来存储和管理Consumer消费进度offset的一个组件。

首先Consumer刚启动，必须依托Rebalancer组件，去进行一下重平衡，自己要分配一些MessageQueue去拉取消息。

接着拉取消息，必须要依托PullAPI组件通过底层网络通信去拉取。在拉取的过程中，必然要维护offset消费进度，此时就需要OffsetStore组件。万一要是ConsumerGroup里多了Consumer或者少了Consumer，又要依托Rebalancer组件进行重平衡了。

### 一个消费组中的多个Consumer是如何均匀分配消息队列的？

当你一个业务系统部署多台机器的时候，每个系统里都启动了一个Consumer，多个Consumer会组成一个ConsumerGroup，也就是消费组，此时就会有一个消费组内的多个Consumer同时消费一个Topic，而且这个Topic是有多个MessageQueue分布在多个Broker上的。

假设咱们一个业务系统部署在两台机器上，对应一个消费组里就有两个Consumer，那么现在一个Topic有三个MessageQueue，该怎么分配呢？

这就涉及到了**Consumer的负载均衡**的问题了。

RebalancerImpl重平衡组件是如何将多个MessageQueue均匀的分配给一个消费组内的多个Consumer的呢？

实际上，每个Consumer在启动之后，都会干一件事情，就是向所有的Broker进行注册，并且持续保持自己的心跳，让每个Broker都能感知到一个消费组内有哪些Consumer。

然后呢，每个Consumer在启动之后，其实重平衡组件都会随机挑选一个Broker，从里面获取到这个消费组里有哪些Consumer存在。

此时重平衡组件一旦知道了消费组内有哪些Consumer之后，接着就好办了，无非就是把Topic下的MessageQueue均匀的分配给这些Consumer了，这个时候其实有几种算法可以进行分配，但是比较常用的一种算法就是简单的平均分配。

比如现在一共有3个MessageQueue，然后有2个Consumer，那么此时就会给1个Consumer分配2个MessageQueue，同时给另外1个Consumer分配剩余的1个MessageQueue。

假设有4个MessageQueue的话，那么就可以2个Consumer每个人分配2个MessageQueue了.

总之，一切都是平均分配的，尽量保证每个Consumer的负载是差不多的。

这样的话，一旦MessageQueue负载确定了之后，下一步其实Consumer就知道自己要消费哪几个MessageQueue的消息了，就可以连接到那个Broker上去，从里面不停的拉取消息过来进行消费了。

### Consumer是如何从Broker上拉取一批消息过来处理的？

消费组

消费者组的意思，就是让你给一组消费者起一个名字。

比如说我们有一个Topic叫做“TopicOrderPaySuccess”，那么假设有库存系统、积分系统、营销系统、仓储系统他们都要去消费这个Topic中的数据。

此时我们应该给那四个系统分别起一个消费组的名字，比如说：stock_consumer_group，marketing_consumer_group，credie_consumer_group，wms_consumer_group。

设置消费组的方式是在代码里进行的。

```html
DefaultMQPushConsumer consumer=new DefaultMQPushConsumer("stock_consumer_group");
```

比如库存系统部署了4台机器，每台机器上的消费者组的名字都是“stock_consumer_group”，那么这4台机器就同属于一个消费者组，以此类推，每个系统的几台机器都是属于各自的消费者组的。

假设库存系统和营销系统作为两个消费者组，都订阅了“TopicOrderPaySuccess”这个订单支付成功消息的Topic，那么此时假设订单系统作为生产者发送了一条消息到这个Topic。

正常情况下来说，这条消息进入Broker之后，库存系统和营销系统作为两个消费组，每个组都会拉取到这条消息。

库存系统这个消费组里，他有两台机器，是两台机器都获取到这条消息？还是说只有一台机器会获取到这条消息？

正常情况下来说，库存系统的两台机器中只有一台机器会获取到这条消息，营销系统也是同理。

集群模式消费 vs 广播模式消费

默认情况下我们都是集群模式，也就是说，一个消费组获取到一条消息，只会交给组内的一台机器去处理，不是每台机器都可以获取到这条消息的。

但是我们可以通过如下设置来改变为广播模式：

consumer.setMessageModel(MessageModel.BROADCASTING);

如果修改为广播模式，那么对于消费组获取到的一条消息，组内每台机器都可以获取到这条消息。但是相对而言广播模式其实用的很少，常见基本上都是使用集群模式来进行消费的。

MessageQueue、CommitLog、ConsumeQueue之间的关系

Topic中的多个MessageQueue会分散在多个Broker上，在每个Broker机器上，一个MessageQueue就对应了一个ConsumeQueue，当然在物理磁盘上其实是对应了多个ConsumeQueue文件的。

但是对于一个Broker机器而言，存储在他上面的所有Topic以及MessageQueue的消息数据都是写入一个统一的CommitLog的。

然后对于Topic的各个MessageQueue而言，就是通过各个ConsumeQueue文件来存储属于MessageQueue的消息在CommitLog文件中的物理地址，就是一个offset偏移量。

MessageQueue与消费者的关系

大致可以认为一个Topic的多个MessageQueue会均匀分摊给消费组内的多个机器去消费，这里的一个原则就是，一个MessageQueue只能被一个消费机器去处理，但是一台消费者机器可以负责多个MessageQueue的消息处理。

Push模式 vs Pull模式

Push消费模式本质底层也是基于这种消费者主动拉取的模式来实现的，只不过他的名字叫做Push而已，意思是Broker会尽可能实时的把新消息交给消费者机器来进行处理，他的消息时效性会更好。

一般我们使用RocketMQ的时候，消费模式通常都是基于他的Push模式来做的，因为Pull模式的代码写起来更加的复杂和繁琐，而且Push模式底层本身就是基于消息拉取的方式来做的，只不过时效性更好而已。

Push模式的实现思路我们这里简单说一下：当消费者发送请求到Broker去拉取消息的时候，如果有新的消息可以消费那么就会立马返回一批消息到消费机器去处理，处理完之后会接着立刻发送请求到Broker机器去拉取下一批消息。

所以消费机器在Push模式下会处理完一批消息，立马发起请求拉取下一批消息，消息处理的时效性非常好，看起来就跟Broker一直不停的推送消息到消费机器一样。

另外Push模式下有一个请求挂起和长轮询的机制。

当你的请求发送到Broker，结果他发现没有新的消息给你处理的时候，就会让请求线程挂起，默认是挂起15秒，然后这个期间他会有后台线程每隔一会儿就去检查一下是否有的新的消息给你，另外如果在这个挂起过程中，如果有新的消息到达了会主动唤醒挂起的线程，然后把消息返回给你。

当然其实消费者进行消息拉取的底层源码是非常复杂的，涉及到大量的细节，但是他的核心思路大致就是如此，我们只要知道，其实哪怕是用常见的Push模式消费，本质也是消费者不停的发送请求到broker去拉取一批一批的消息就行了。

Broker是如何将消息读取出来返回给消费机器的？

假设一个消费者机器发送了拉取请求到Broker了，他说我这次要拉取MessageQueue0中的消息，然后我之前都没拉取过消息，所以就从这个MessageQueue0中的第一条消息开始拉取好了。

于是，Broker就会找到MessageQueue0对应的ConsumeQueue0，从里面找到第一条消息的offset。

接着Broker就需要根据ConsumeQueue0中找到的第一条消息的地址，去CommitLog中根据这个offset地址去读取出来这条消息的数据，然后把这条消息的数据返回给消费者机器。

所以其实消费消息的时候，本质就是根据你要消费的MessageQueue以及开始消费的位置，去找到对应的ConsumeQueue读取里面对应位置的消息在CommitLog中的物理offset偏移量，然后到CommitLog中根据offset读取消息数据，返回给消费者机器。

消费者机器如何处理消息、进行ACK以及提交消费进度？

消费者机器拉取到一批消息之后，就会将这批消息回调我们注册的一个函数。

当我们处理完这批消息之后，消费者机器就会提交我们目前的一个消费进度到Broker上去，然后Broker就会存储我们的消费进度，比如我们现在对ConsumeQueue0的消费进度假设就是在offset=1的位置，那么他会记录下来一个ConsumeOffset的东西去标记我们的消费进度。

那么下次这个消费组只要再次拉取这个ConsumeQueue的消息，就可以从Broker记录的消费位置开始继续拉取，不用重头开始拉取了。

如果消费组中出现机器宕机或者扩容加机器，会怎么处理？

这个时候其实会进入一个rabalance的环节，也就是说重新给各个消费机器分配他们要处理的MessageQueue。

给大家举个例子，比如现在机器01负责MessageQueue0和Message1，机器02负责MessageQueue2和MessageQueue3，现在机器02宕机了，那么机器01就会接管机器02之前负责的MessageQueue2和MessageQueue3。

或者如果此时消费组加入了一台机器03，此时就可以把机器02之前负责的MessageQueue3转移给机器03，然后机器01就仅仅负责一个MessageQueue2的消费了，这就是负载重平衡的概念。

拉取消息的源码入口

拉取消息的源码入口是在DefaultMQPushConsumerImpl类的pullMessage()方法中的，这个里面涉及到了拉取请求、消息流量控制、通过PullAPIWrapper与服务端进行网络交互、服务端根据ConsumeQueue文件拉取消息，等一系列的事情。

## 59.核心概念

RocketMQ是可以集群化部署的，可以部署在多台机器上。

发送消息到MQ的系统会把消息分散发送给多台不同的机器，每台机器上部署的RocketMQ进程一般称之为Broker，每个Broker都会收到不同的消息，然后就会把这批消息存储在自己本地的磁盘文件里。本质上RocketMQ存储海量消息的机制就是分布式的存储。

NameServer

负责去管理集群里所有Broker的信息，让使用MQ的系统可以通过他感知到集群里有哪些Broker。

也可以独立部署在不同机器上，然后所有的Broker都会把自己注册到NameServer上去，NameServer就知道集群里有哪些Broker了。

支持部署多台机器的，NameServer是可以集群化部署的。

Broker集群

必须得在多台机器上部署这么一个集群，而且还得用主从架构实现数据多副本存储和高可用。

Broker是有Master和Slave两种角色的，Master Broker收到消息之后会同步给Slave Broker，这样Slave Broker上就能有一模一样的一份副本数据！如果任何一个Master Broker出现故障，还有一个Slave Broker上有一份数据副本，可以保证数据不丢失，还能继续对外提供服务，保证了MQ的可靠性和高可用性。

每个Broker启动都得向所有的NameServer进行注册。

生产者

向MQ发送消息的那些系统了，如果系统要从Broker获取消息，会找NameServer获取路由信息，去找到对应的Broker获取消息。

消费者

从MQ获取消息的那些系统，如果他要发送消息到Broker，会找NameServer去获取路由信息，就是集群里有哪些Broker等信息。

RocketMQ中的生产者和消费者都是自己主动去NameServer拉取Broker信息的。

心跳机制

Broker会每隔30s给所有的NameServer发送心跳，告诉每个NameServer自己目前还活着。

每次NameServer收到一个Broker的心跳，就可以更新一下他的最近一次心跳的时间。

然后NameServer会每隔10s运行一个任务，去检查一下各个Broker的最近一次心跳时间，如果某个Broker超过120s都没发送心跳了，那么就认为这个Broker已经挂掉了。

## 60.生产者和消费者的客户端容错机制

### Broker的主从架构

Master Broker将消息同步给Slave Broker

一般都是得将Broker部署成Master-Slave模式的，也就是一个Master Broker对应一个Slave Broker。然后Master需要在接收到消息之后，将数据同步给Slave，这样一旦Master Broker挂了，还有Slave上有一份数据。

Slave Broker也会向所有的NameServer进行注册，也会向所有的NameServer每30s发送心跳。

RocketMQ的Master-Slave模式采取的是Slave Broker不停的发送请求到Master Broker去拉取消息。就是RocketMQ自身的Master-Slave模式采取的是**Pull模式**拉取消息。

Broker跟NameServer之间的通信在RocketMQ的实现中，采用的是**TCP长连接**进行通信。也就是说，Broker会跟每个NameServer都建立一个TCP长连接，然后定时通过TCP长连接发送心跳请求过去。

读写分离

消费者的系统在获取消息的时候，有可能从Master Broker获取消息，也有可能从Slave Broker获取消息。作为消费者的系统在获取消息的时候会先发送请求到Master Broker上去，请求获取一批消息，此时Master Broker是会返回一批消息给消费者系统的。然后Master Broker在返回消息给消费者系统的时候，会根据当时Master Broker的负载情况和Slave Broker的同步情况，向消费者系统建议下一次拉取消息的时候是从Master Broker拉取还是从Slave Broker拉取。

Slave Broke挂掉了

如果Slave Broker挂了，那么此时无论消息写入还是消息拉取，还是可以继续从Master Broke去走，对整体运行不影响。

Master Broker挂掉了

在RocketMQ 4.5版本之前，都是用Slave Broker同步数据，尽量保证数据不丢失，但是一旦Master故障了，Slave是没法自动切换成Master的。

所以在这种情况下，如果Master Broker宕机了，这时就得手动做一些运维操作，把Slave Broker重新修改一些配置，重启机器给调整为Master Broker，这是有点麻烦的，而且会导致中间一段时间不可用。

在RocketMQ 4.5之后，可以基于Dledger可以实现RocketMQ的高可用自动切换的效果。Dledger技术是要求至少得是一个Master带两个Slave。

把Dledger融入RocketMQ之后，就可以让一个Master Broker对应多个Slave Broker，此时一旦Master Broker宕机了，就可以在多个副本，也就是多个Slave中，通过Dledger技术和Raft协议算法进行leader选举，直接将一个Slave Broker选举为新的Master Broker，然后这个新的Master Broker就可以对外提供服务了。整个过程也许只要10秒或者几十秒的时间就可以完成，这样的话，就可以实现Master Broker挂掉之后，自动从多个Slave Broker中选举出来一个新的Master Broker，继续对外服务，一切都是自动的。

## 61.MQ的核心数据模型：Topic

Topic到底是什么？

比如你的订单系统需要往MQ里发送订单消息，那么此时你就应该建一个Topic，他的名字可以叫做：topic_order_info，也就是一个包含了订单信息的数据集合。然后你的订单系统投递的订单消息都是进入到这个“topic_order_info”里面去的，如果你的仓储系统要获取订单消息，那么他可以指定从“topic_order_info”这里面去获取消息，获取出来的都是他想要的订单消息了。

一句话：Topic其实就是一个数据集合的意思，不同类型的数据你得放不同的Topic里去。

你的系统如果要往MQ里写入消息或者获取消息，首先得创建一些Topic，作为数据集合存放不同类型的消息，比如说订单Topic，商品Topic，等等。

在Broker集群里如何存储

分布式存储。比如一个Topic里有1000万条数据，此时有2台Broker，那么就可以让每台Broker上都放500万条数据。

每个Broke在进行定时的心跳汇报给NameServer的时候，都会告诉NameServer自己当前的数据情况，比如有哪些Topic的哪些数据在自己这里，这些信息都是属于路由信息的一部分。

生产者系统是如何将消息发送给Broker

得先有一个Topic，然后在发送消息的时候你得指定你要发送到哪个Topic里面去。

接着既然你知道你要发送的Topic，那么就可以跟NameServer建立一个TCP长连接，然后定时从他那里拉取到最新的路由信息，包括集群里有哪些Broker，集群里有哪些Topic，每个Topic都存储在哪些Broker上。

然后生产者系统自然就可以通过路由信息找到自己要投递消息的Topic分布在哪几台Broker上，此时可以根据负载均衡算法，从里面选择一台Broke机器出来，比如round robine轮询算法，或者是hash算法，都可以。

选择一台Broker之后，就可以跟那个Broker也建立一个TCP长连接，然后通过长连接向Broker发送消息即可.

Broker收到消息之后就会存储在自己本地磁盘里去。

生产者一定是投递消息到Master Broker的，然后Master Broker会同步数据给他的Slave Brokers。

消费者是如何从Broker上拉取消息

消费者系统其实跟生产者系统原理是类似的，他们也会跟NameServer建立长连接，然后拉取路由信息，接着找到自己要获取消息的Topic在哪几台Broker上，就可以跟Broker建立长连接，从里面拉取消息了。

消费者系统可能会从Master Broker拉取消息，也可能从Slave Broker拉取消息，都有可能。

## 62.发送消息模式

### 同步发送

所谓同步，意思就是你通过这行代码发送消息到MQ去，SendResult sendResult = producer.send(msg)，然后你会卡在这里，代码不能往下走了.

你要一直等待MQ返回一个结果给你，你拿到了SendResult之后，接着你的代码才会继续往下走。

### 异步发送

首先在构造Producer的时候加入下面红框中的代码：

```html
//设置异步发送失败的时候重试次数为0
producer.setRetryTimesWhenSendAsyncFailed(0);
```

接着把发送消息的代码改成如下所示：

```html
producer.send(message,new SendCallback(){
    @override
    public void onSuccess(SendResult sendResult){
    
    }
    @override
    public void onException(Throwable e){
    
    }
})
```

你把消息发送出去，然后上面的代码就直接往下走了，不会卡在这里等待MQ返回结果给你！

然后当MQ返回结果给你的时候，Producer会回调你的SendCallback里的函数，如果发送成功了就回调onSuccess函数，如果发送失败了就回调onExceptino函数。

这个就是所谓的异步发送，异步的意思就是你发送消息的时候不会卡在上面那行代码等待MQ返回结果给你，会继续执行下面的别的代码，当MQ返回结果给你的时候，会回调你的函数！

发送单向消息到RocketMQ

### 发送单向消息

还有一种发送消息的方法，叫做发送单向消息，就是用下面的代码来发送消息：

```html
producer.sendOneway(msg);
```

这个sendOneway的意思，就是你发送一个消息给MQ，然后代码就往下走了，根本不会关注MQ有没有返回结果给你，你也不需要MQ返回的结果，无论发送的消息是成功还是失败，都不关你的事。

## 63.消费消息模式

Push消费模式

DefaultMQPushConsumer。

就是Broker会主动把消息发送给你的消费者，你的消费者是被动的接收Broker推送给过来的消息，然后进行处理。

这个就是所谓的Push模式，意思就是Broker主动推送消息给消费者。

Pull消费模式

使用的Consumer类是DefaultMQPullConsumer，从名字里就可以看到使用了Pull消费模式。

也就是说，Broker不会主动推送消息给Consumer，而是消费者主动发送请求到Broker去拉取消息过来。

## 64.MessageQueue

创建Topic的时候为何要指定MessageQueue数量？

**在创建Topic的时候需要指定一个很关键的参数，就是MessageQueue**。

简单来说，就是你要指定你的这个Topic对应了多少个队列，也就是多少个MessageQueue。

Topic、MessageQueue以及Broker之间到底是什么关系？

RocketMQ引入了MessageQueue的概念，本质上就是一个数据分片的机制。

假设你的Topic有1万条数据，然后你的Topic有4个MessageQueue，那么大致可以认为会在每个MessageQueue中放入2500条数据.

当然，这个不是绝对的，有可能有的MessageQueue的数据多，有的数据少，这个要根据你的消息写入MessageQueue的策略来定。

MessageQueue就是RocketMQ中非常关键的一个数据分片机制，他通过MessageQueue将一个Topic的数据拆分为了很多个数据分片，然后在每个Broker机器上都存储一些MessageQueue。通过这个方法，就可以实现Topic数据的分布式存储！

生产者发送消息的时候写入哪个MessageQueue？

生产者会跟NameServer进行通信获取Topic的路由数据。

所以生产者从NameServer中就会知道，一个Topic有几个MessageQueue，哪些MessageQueue在哪台Broker机器上，哪些MesssageQueue在另外一台Broker机器上，这些都会知道.

如果某个Broker出现故障该怎么办？

在Producer中开启一个开关，就是sendLatencyFaultEnable。

一旦打开了这个开关，那么他会有一个自动容错机制，比如如果某次访问一个Broker发现网络延迟有500ms，然后还无法访问，那么就会自动回避访问这个Broker一段时间，比如接下来3000ms内，就不会访问这个Broker了。

## 65.Broker是如何持久化存储消息

CommitLog消息顺序写入机制

当生产者的消息发送到一个Broker上的时候，他接收到了一条消息，接着他会对这个消息做什么事情？

首先第一步，他会把这个消息直接写入磁盘上的一个日志文件，叫做CommitLog，直接顺序写入这个文件。

这个CommitLog是很多磁盘文件，每个文件限定最多1GB，Broker收到消息之后就直接追加写入这个文件的末尾，如果一个CommitLog写满了1GB，就会创建一个新的CommitLog文件。

MessageQueue在数据存储中是体现在哪里呢？

在Broker中，对Topic下的每个MessageQueue都会有一系列的ConsumeQueue文件。

在Broker的磁盘上，会有下面这种格式的一系列文件：

$HOME/store/consumequeue/{topic}/{queueId}/{fileName}

上面那一串东西是什么意思？

我们之前说过，对每个Topic你不是在这台Broker上都会有一些MessageQueue吗？所以你会看到，{topic}指代的就是某个Topic，{queueId}指代的就是某个MessageQueue。

然后对存储在这台Broker机器上的Topic下的一个MessageQueue，他有很多的ConsumeQueue文件，这个ConsumeQueue文件里存储的是一条消息对应在CommitLog文件中的offset偏移量。

当你的Broker收到一条消息写入了CommitLog之后，其实他同时会将这条消息在CommitLog中的物理位置，也就是一个文件偏移量，就是一个offset，写入到这条消息所属的MessageQueue对应的ConsumeQueue文件中去。

所以实际上，ConsumeQueue0中存储的是一个一个消息在CommitLog文件中的物理位置，也就是offset。

实际上在ConsumeQueue中存储的每条数据不只是消息在CommitLog中的offset偏移量，还包含了消息的长度，以及tag hashcode，一条数据是20个字节，每个ConsumeQueue文件保存30万条数据，大概每个文件是5.72MB。

所以实际上Topic的每个MessageQueue都对应了Broker机器上的多个ConsumeQueue文件，保存了这个MessageQueue的所有消息在CommitLog文件中的物理位置，也就是offset偏移量。

如何让消息写入CommitLog文件近乎内存写性能

Broker是基于OS操作系统的**PageCache**和**顺序写**两个机制，来提升写入CommitLog文件的性能的。

首先Broker是以顺序的方式将消息写入CommitLog磁盘文件的，也就是每次写入就是在文件末尾追加一条数据就可以了，对文件进行顺序写的性能要比对文件随机写的性能提升很多。

另外，数据写入CommitLog文件的时候，其实不是直接写入底层的物理磁盘文件的，而是先进入OS的PageCache内存缓存中，然后后续由OS的后台线程选一个时间，异步化的将OS PageCache内存缓冲中的数据刷入底层的磁盘文件。

采用**磁盘文件顺序写+OS PageCache写入+OS异步刷盘的策略**，基本上可以让消息写入CommitLog的性能跟你直接写入内存里是差不多的。

同步刷盘与异步刷盘

在上述的异步刷盘模式下，生产者把消息发送给Broker，Broker将消息写入OS PageCache中，就直接返回ACK给生产者了。此时生产者就认为消息写入成功了.

如果生产者认为消息写入成功了，但是实际上那条消息此时是在Broker机器上的os cache中的，如果此时Broker直接宕机，那么是不是os cache中的这条数据就会丢失了。

所以异步刷盘的的策略下，可以让消息写入吞吐量非常高，但是可能会有数据丢失的风险，这个是大家需要清除的。

另外一种模式叫做同步刷盘，如果你使用同步刷盘模式的话，那么生产者发送一条消息出去，broker收到了消息，必须直接强制把这个消息刷入底层的物理磁盘文件中，然后才会返回ack给producer，此时你才知道消息写入成功了。

只要消息进入了物理磁盘上，那么除非是你的物理磁盘坏了导致数据丢失，否则正常来说数据就不会丢失了。

如果broker还没有来得及把数据同步刷入磁盘，然后他自己挂了，那么此时对producer来说会感知到消息发送失败了，然后你只要不停的重试发送就可以了，直到有slave broker切换成master broker重新让你可以写入消息，此时可以保证数据是不会丢的。

但是如果你强制每次消息写入都要直接进入磁盘中，必然导致每条消息写入性能急剧下降，导致消息写入吞吐量急剧下降，但是可以保证数据不会丢失。

## 66.基于DLedger技术的Broker主从同步原理

基于DLedger技术替换Broker的CommitLog

DLedger技术实际上首先他自己就有一个CommitLog机制，你把数据交给他，他会写入CommitLog磁盘文件里去，这是他能干的第一件事情。

如果基于DLedger技术来实现Broker高可用架构，实际上就是用DLedger先替换掉原来Broker自己管理的CommitLog，由DLedger来管理CommitLog.

我们需要使用DLedger来管理CommitLog，然后Broker还是可以基于DLedger管理的CommitLog去构建出来机器上的各个ConsumeQueue磁盘文件。

DLedger是如何基于Raft协议选举Leader Broker的

DLedger是基于Raft协议来进行Leader Broker选举的。

这需要发起一轮一轮的投票，通过三台机器互相投票选出来一个人作为Leader。

简单来说，三台Broker机器启动的时候，他们都会投票自己作为Leader，然后把这个投票发送给其他Broker。

我们举一个例子，Broker01是投票给自己的，Broker02是投票给自己的，Broker03是投票给自己的，他们都把自己的投票发送给了别人。

此时在第一轮选举中，Broker01会收到别人的投票，他发现自己是投票给自己，但是Broker02投票给Broker02自己，Broker03投票给Broker03自己，似乎每个人都很自私，都在投票给自己，所以第一轮选举是失败的。

因为大家都投票给自己，怎么选举出来一个Leader呢？

接着每个人会进入一个随机时间的休眠，比如说Broker01休眠3秒，Broker02休眠5秒，Broker03休眠4秒。

此时Broker01必然是先苏醒过来的，他苏醒过来之后，直接会继续尝试投票给自己，并且发送自己的选票给别人。

接着Broker03休眠4秒后苏醒过来，他发现Broker01已经发送来了一个选票是投给Broker01自己的，此时他自己因为没投票，所以会尊重别人的选择，就直接把票投给Broker01了，同时把自己的投票发送给别人。

接着Broker02苏醒了，他收到了Broker01投票给Broker01自己，收到了Broker03也投票给了Broker01，那么他此时自己是没投票的，直接就会尊重别人的选择，直接就投票给Broker01，并且把自己的投票发送给别人。

此时所有人都会收到三张投票，都是投给Broker01的，那么Broker01就会当选为Leader。

其实只要有（3台机器 / 2） + 1个人投票给某个人，就会选举他当Leader，这个（机器数量 / 2） + 1就是大多数的意思。

这就是Raft协议中选举leader算法的简单描述，简单来说，他确保有人可以成为Leader的核心机制就是一轮选举不出来Leader的话，就让大家随机休眠一下，先苏醒过来的人会投票给自己，其他人苏醒过后发现自己收到选票了，就会直接投票给那个人。

依靠这个随机休眠的机制，基本上几轮投票过后，一般都是可以快速选举出来一个Leader。

因此我们看下图，在三台Broker机器刚刚启动的时候，就是靠这个DLedger基于Raft协议实现的leader选举机制，互相投票选举出来一个Leader，其他人就是Follower，然后只有Leader可以接收数据写入，Follower只能接收Leader同步过来的数据。

DLedger是如何基于Raft协议进行多副本同步

DLedger在进行同步的时候是采用Raft协议进行多副本同步的。

数据同步会分为两个阶段，一个是uncommitted阶段，一个是commited阶段。

首先Leader Broker上的DLedger收到一条数据之后，会标记为uncommitted状态，然后他会通过自己的DLedgerServer组件把这个uncommitted数据发送给Follower Broker的DLedgerServer。

接着Follower Broker的DLedgerServer收到uncommitted消息之后，必须返回一个ack给Leader Broker的DLedgerServer，然后如果Leader Broker收到超过半数的Follower Broker返回ack之后，就会将消息标记为committed状态。

然后Leader Broker上的DLedgerServer就会发送commited消息给Follower Broker机器的DLedgerServer，让他们也把消息标记为comitted状态。

如果Leader Broker崩溃了怎么办

如果Leader Broker挂了，此时剩下的两个Follower Broker就会重新发起选举，他们会基于DLedger还是采用Raft协议的算法，去选举出来一个新的Leader Broker继续对外提供服务，而且会对没有完成的数据同步进行一些恢复性的操作，保证数据不会丢失。

## 67.消费者是如何获取消息处理以及进行ACK

消费者组的意思，就是让你给一组消费者起一个名字。比如我们有一个Topic叫“TopicOrderPaySuccess”，然后假设有库存系统、积分系统、营销系统、仓储系统他们都要去消费这个Topic中的数据。

不同的系统应该设置不同的消费组，如果不同的消费组订阅了同一个Topic，对Topic里的一条消息，每个消费组都会获取到这条消息。

库存系统这个消费组里有两台机器，正常情况下来说，库存系统的两台机器中只有一台机器会获取到这条消息。

集群模式消费 vs 广播模式消费

集群模式，也就是说，一个消费组获取到一条消息，只会交给组内的一台机器去处理，不是每台机器都可以获取到这条消息的。

但是我们可以通过如下设置来改变为广播模式：

consumer.setMessageModel(MessageModel.BROADCASTING);

如果修改为广播模式，那么对于消费组获取到的一条消息，组内每台机器都可以获取到这条消息。但是相对而言广播模式其实用的很少，常见基本上都是使用集群模式来进行消费的。

MessageQueue、CommitLog、ConsumeQueue之间的关系

Topic中的多个MessageQueue会分散在多个Broker上，在每个Broker机器上，一个MessageQueue就对应了一个ConsumeQueue，当然在物理磁盘上其实是对应了多个ConsumeQueue文件的。

但是对于一个Broker机器而言，存储在他上面的所有Topic以及MessageQueue的消息数据都是写入一个统一的CommitLog的.

然后对于Topic的各个MessageQueue而言，就是通过各个ConsumeQueue文件来存储属于MessageQueue的消息在CommitLog文件中的物理地址，就是一个offset偏移量。

MessageQueue与消费者的关系

大致可以认为一个Topic的多个MessageQueue会均匀分摊给消费组内的多个机器去消费，这里的一个原则就是，一个MessageQueue只能被一个消费机器去处理，但是一台消费者机器可以负责多个MessageQueue的消息处理。

Push模式 vs Pull模式

Push消费模式本质底层也是基于这种消费者主动拉取的模式来实现的，只不过他的名字叫做Push而已，意思是Broker会尽可能实时的把新消息交给消费者机器来进行处理，他的消息时效性会更好。

一般我们使用RocketMQ的时候，消费模式通常都是基于他的Push模式来做的，因为Pull模式的代码写起来更加的复杂和繁琐，而且Push模式底层本身就是基于消息拉取的方式来做的，只不过时效性更好而已。

Push模式的实现思路我这里简单说一下：当消费者发送请求到Broker去拉取消息的时候，如果有新的消息可以消费那么就会立马返回一批消息到消费机器去处理，处理完之后会接着立刻发送请求到Broker机器去拉取下一批消息。

所以消费机器在Push模式下会处理完一批消息，立马发起请求拉取下一批消息，消息处理的时效性非常好，看起来就跟Broker一直不停的推送消息到消费机器一样。

另外Push模式下有一个请求挂起和长轮询的机制，也要给大家简单介绍一下。

当你的请求发送到Broker，结果他发现没有新的消息给你处理的时候，就会让请求线程挂起，默认是挂起15秒，然后这个期间他会有后台线程每隔一会儿就去检查一下是否有的新的消息给你，另外如果在这个挂起过程中，如果有新的消息到达了会主动唤醒挂起的线程，然后把消息返回给你。

Broker是如何将消息读取出来返回给消费机器的

假设一个消费者机器发送了拉取请求到Broker了，他说我这次要拉取MessageQueue0中的消息，然后我之前都没拉取过消息，所以就从这个MessageQueue0中的第一条消息开始拉取好了。

于是，Broker就会找到MessageQueue0对应的ConsumeQueue0，从里面找到第一条消息的offset.

接着Broker就需要根据ConsumeQueue0中找到的第一条消息的地址，去CommitLog中根据这个offset地址去读取出来这条消息的数据，然后把这条消息的数据返回给消费者机器。

所以其实消费消息的时候，本质就是根据你要消费的MessageQueue以及开始消费的位置，去找到对应的ConsumeQueue读取里面对应位置的消息在CommitLog中的物理offset偏移量，然后到CommitLog中根据offset读取消息数据，返回给消费者机器。

消费者机器如何处理消息、进行ACK以及提交消费进度？

当我们处理完这批消息之后，消费者机器就会提交我们目前的一个消费进度到Broker上去，然后Broker就会存储我们的消费进度.

比如我们现在对ConsumeQueue0的消费进度假设就是在offset=1的位置，那么他会记录下来一个ConsumeOffset的东西去标记我们的消费进度。

那么下次这个消费组只要再次拉取这个ConsumeQueue的消息，就可以从Broker记录的消费位置开始继续拉取，不用重头开始拉取了。

如果消费组中出现机器宕机或者扩容加机器，会怎么处理？

会进入一个rabalance的环节，也就是说重新给各个消费机器分配他们要处理的MessageQueue。

给大家举个例子，比如现在机器01负责MessageQueue0和Message1，机器02负责MessageQueue2和MessageQueue3，现在机器02宕机了，那么机器01就会接管机器02之前负责的MessageQueue2和MessageQueue3。

或者如果此时消费组加入了一台机器03，此时就可以把机器02之前负责的MessageQueue3转移给机器03，然后机器01就仅仅负责一个MessageQueue2的消费了，这就是负载重平衡的概念。

## 68.消费者到底是根据什么策略从Master或Slave上拉取消息

ConsumeQueue文件也是基于os cache的

broker对ConsumeQueue文件同样也是基于os cache来进行优化的。

os自己有一个优化机制，就是读取一个磁盘文件的时候，他会自动把磁盘文件的一些数据缓存到os cache中。

ConsumeQueue文件主要是存放消息的offset，所以每个文件很小，30万条消息的offset就只有5.72MB而已。所以实际上ConsumeQueue文件们是不占用多少磁盘空间的，他们整体数据量很小，几乎可以完全被os缓存在内存cache里。

实际上在消费者机器拉取消息的时候，第一步大量的频繁读取ConsumeQueue文件，几乎可以说就是跟读内存里的数据的性能是一样的，通过这个就可以保证数据消费的高性能以及高吞吐.

CommitLog是基于os cache+磁盘一起读取的

因为CommitLog是用来存放消息的完整数据的，所以内容量是很大的，毕竟他一个文件就要1GB，所以整体完全有可能多达几个TB。

所以你思考一下，这么多的数据，可能都放在os cache里吗？

明显是不可能的，因为os cache用的也是机器的内存，一般多也就几十个GB而已，何况Broker自身的JVM也要用一些内存，留个os cache的内存只是一部分罢了，比如10GB~20GB的内存，所以os cache对于CommitLog而言，是无法把他全部数据都放在里面给你读取的！

也就是说，os cache对于CommitLog而言，主要是提升文件写入性能，当你不停的写入的时候，很多最新写入的数据都会先停留在os cache里，比如这可能有10GB~20GB的数据。

之后os会自动把cache里的比较旧的一些数据刷入磁盘里，腾出来空间给更新写入的数据放在os cache里，所以大部分数据可能多达几个TB都是在磁盘上的.

所以最终结论来了，当你拉取消息的时候，可以轻松从os cache里读取少量的ConsumeQueue文件里的offset，这个性能是极高的，但是当你去CommitLog文件里读取完整消息数据的时候，会有两种可能。

**第一种可能**，如果你读取的是那种刚刚写入CommitLog的数据，那么大概率他们还停留在os cache中，此时你可以顺利的直接从os cache里读取CommitLog中的数据，这个就是内存读取，性能是很高的。

**第二种可能**，你也许读取的是比较早之前写入CommitLog的数据，那些数据早就被刷入磁盘了，已经不在os cache里了，那么此时你就只能从磁盘上的文件里读取了，这个性能是比较差一些的。

什么时候会从os cache读？什么时候会从磁盘读？

如果你的消费者机器一直快速的在拉取和消费处理，紧紧的跟上了生产者写入broker的消息速率，那么你每次拉取几乎都是在拉取最近人家刚写入CommitLog的数据，那几乎都在os cache里。

但是如果broker的负载很高，导致你拉取消息的速度很慢，或者是你自己的消费者机器拉取到一批消息之后处理的时候性能很低，处理的速度很慢，这都会导致你跟不上生产者写入的速率。

比如人家都写入10万条数据了，结果你才拉取了2万条数据，此时有5万条最新的数据是在os cache里，有3万条你还没拉取的数据是在磁盘里，那么当后续你再拉取的时候，必然很大概率是从磁盘里读取早就刷入磁盘的3万条数据。

接着之前在os cache里的5万条数据可能又被刷入磁盘了，取而代之的是更新的几万条数据在os cache里，然后你再次拉取的时候，又会从磁盘里读取刷入磁盘里的5万条数据，相当于你每次都在从磁盘里读取数据了！

Master Broker什么时候会让你从Slave Broker拉取数据

对比你当前没有拉取消息的数量和大小，以及最多可以存放在os cache内存里的消息的大小，如果你没拉取的消息超过了最大能使用的内存的量，那么说明你后续会频繁从磁盘加载数据，此时就让你从slave broker去加载数据了！

## 69.RocketMQ 是如何基于Netty扩展出高性能网络通信架构的

Reactor主线程与长短连接

Broker里有这么一个名字的线程，Reactor主线程,而且这个线程是负责监听一个网络端口的，比如监听个2888，39150这样的端口。

短连接：如果你要给别人发送一个请求，必须要建立连接 -> 发送请求 -> 接收响应 -> 断开连接，下一次你要发送请求的时候，这个过程得重新来一遍.

每次建立一个连接之后，使用这个连接发送请求的时间是很短的，很快就会断开这个连接，所以他存在时间太短了，就是短连接。

长连接的话，就是反过来的意思，你建立一个连接 -> 发送请求 -> 接收响应 -> 发送请求 -> 接收响应 -> 发送请求 -> 接收响应.

大家会发现，当你建立好一个长连接之后，可以不停的发送请求和接收响应，连接不会断开，等你不需要的时候再断开就行了，这个连接会存在很长时间，所以是长连接。

那么TCP长连接是什么意思呢？

如果你对网络没太多的了解，简单理解为TCP就是一个协议，所谓协议的意思就是，按照TCP这个协议规定好的步骤建立连接，按照他规定好的步骤发送请求。

比如你要建立一个TCP连接，必须先给对方发送他规定好的几个数据，然后人家按照规定返回给你几个数据，你再给人家发送几个数据，一切都按TCP的规定来。按照规定来，大家就可以建立一个TCP连接。

所以TCP长连接，就是按照这个TCP协议建立的长连接。

Producer和Broker建立一个长连接

比如有一个Producer他就要跟Broker建立一个TCP长连接了，此时Broker上的这个Reactor主线程，他会在端口上监听到这个Producer建立连接的请求.

接着这个Reactor主线程就专门会负责跟这个Producer按照TCP协议规定的一系列步骤和规范，建立好一个长连接。

在Broker里用什么东西代表跟Producer之间建立的这个长连接呢？

**答案是：SocketChannel**

Producer里面会有一个SocketChannel，Broker里也会有一个SocketChannel，这两个SocketChannel就代表了他们俩建立好的这个长连接。

既然Producer和Broker之间已经通过SocketChannel维持了一个长连接了，接着Producer会通过这个SocketChannel去发送消息给Broker.

基于Reactor线程池监听连接中的请求

Reactor线程池，这个线程池里默认是3个线程,然后Reactor主线程建立好的每个连接SocketChannel，都会交给这个Reactor线程池里的其中一个线程去监听请求。

Producer发送请求过来了，他发送一个消息过来到达Broker里的SocketChannel，此时Reactor线程池里的一个线程会监听到这个SocketChannel中有请求到达了！

基于Worker线程池完成一系列准备工作

接着Reactor线程从SocketChannel中读取出来一个请求，这个请求在正式进行处理之前，必须就先要进行一些准备工作和预处理，比如SSL加密验证、编码解码、连接空闲检查、网络连接管理，诸如此类的一些事.

Worker线程池，他默认有8个线程，此时Reactor线程收到的这个请求会交给Worker线程池中的一个线程进行处理，会完成上述一系列的准备工作.

基于业务线程池完成请求的处理

如果Worker线程完成了一系列的预处理之后，比如SSL加密验证、编码解码、连接空闲检查、网络连接管理，等等，接着就需要对这个请求进行正式的业务处理了！

你接收到了消息，肯定是要写入CommitLog文件的，后续还有一些ConsumeQueue之类的事情需要处理，类似这种操作，就是业务处理逻辑。

总结

- Reactor主线程在端口上监听Producer建立连接的请求，建立长连接
- Reactor线程池并发的监听多个连接的请求是否到达
- Worker请求并发的对多个请求进行预处理
- 业务线程池并发的对多个请求进行磁盘读写业务操作

这些事情全部是利用不同的线程池并发执行的！任何一个环节在执行的时候，都不会影响其他线程池在其他环节进行请求的处理！

## 70.基于mmap内存映射实现磁盘文件的高性能读写

传统文件IO操作的多次数据拷贝问题

首先从磁盘上把数据读取到内核IO缓冲区里去，然后再从内核IO缓存区里读取到用户进程私有空间里去，然后我们才能拿到这个文件里的数据.发生了两次数据拷贝.

RocketMQ是如何基于mmap技术+page cache技术优化

RocketMQ底层对CommitLog、ConsumeQueue之类的磁盘文件的读写操作，基本上都会采用mmap技术来实现。

如果具体到代码层面，就是基于JDK NIO包下的MappedByteBuffer的map()函数，来先将一个磁盘文件（比如一个CommitLog文件，或者是一个ConsumeQueue文件）映射到内存里来.

内存映射.

**因为刚开始你建立映射的时候，并没有任何的数据拷贝操作，其实磁盘文件还是停留在那里**，只不过他把物理上的磁盘文件的一些地址和用户进程私有空间的一些虚拟内存地址进行了一个映射.

这个地址映射的过程，就是JDK NIO包下的MappedByteBuffer.map()函数干的事情，底层就是基于mmap技术实现的。

这个mmap技术在进行文件映射的时候，一般有大小限制，在1.5GB~2GB之间

所以RocketMQ才让CommitLog单个文件在1GB，ConsumeQueue文件在5.72MB，不会太大。

这样限制了RocketMQ底层文件的大小，就可以在进行文件读写的时候，很方便的进行内存映射了。

PageCache，实际上在这里就是对应于虚拟内存.

基于mmap技术+pagecache技术实现高性能的文件读写

接下来就可以对这个已经映射到内存里的磁盘文件进行读写操作了，比如要写入消息到CommitLog文件，你先把一个CommitLog文件通过MappedByteBuffer的map()函数映射其地址到你的虚拟内存地址。

接着就可以对这个MappedByteBuffer执行写入操作了，写入的时候他会直接进入PageCache中，然后过一段时间之后，由os的线程异步刷入磁盘中.

只有一次数据拷贝的过程，他就是从PageCache里拷贝到磁盘文件里而已！这个就是你使用mmap技术之后，相比于传统磁盘IO的一个性能优化。

如果我们要从磁盘文件里读取数据呢？

那么此时就会判断一下，当前你要读取的数据是否在PageCache里？如果在的话，就可以直接从PageCache里读取了！

比如刚写入CommitLog的数据还在PageCache里，此时你Consumer来消费肯定是从PageCache里读取数据的。

但是如果PageCache里没有你要的数据，那么此时就会从磁盘文件里加载数据到PageCache中去.

而且PageCache技术在加载数据的时候**，**还会将**你加载的数据块的临近的其他数据块也一起加载到PageCache里去。**

预映射机制 + 文件预热机制

**（1）内存预映射机制**：Broker会针对磁盘上的各种CommitLog、ConsumeQueue文件预先分配好MappedFile，也就是提前对一些可能接下来要读写的磁盘文件，提前使用MappedByteBuffer执行map()函数完成映射，这样后续读写文件的时候，就可以直接执行了。

**（2）文件预热**：在提前对一些文件完成映射之后，因为映射不会直接将数据加载到内存里来，那么后续在读取尤其是CommitLog、ConsumeQueue的时候，其实有可能会频繁的从磁盘里加载数据到内存中去。

所以其实在执行完map()函数之后，会进行madvise系统调用，就是提前尽可能多的把磁盘文件加载到内存里去。

通过上述优化，才真正能实现一个效果，就是写磁盘文件的时候都是进入PageCache的，保证写入高性能；同时尽可能多的通过map + madvise的映射后预热机制，把磁盘文件里的数据尽可能多的加载到PageCache里来，后续对CosumeQueue、CommitLog进行读取的时候，才能尽可能从内存里读取数据。

## 71.丢失消息

推送消息到MQ的过程会丢失消息吗？

比如订单系统在推送消息到RocketMQ的过程中，是通过网络去进行传输的，但是这个时候恰巧可能网络发生了抖动，也就是网络突然有点问题，导致这次网络通信失败了。

于是这个消息必然就没有成功投递给MQ.

比如MQ确实是收到消息了，但是他的网络通信模块的代码出现了异常，可能是他内部的网络通信的bug，导致消息没成功处理。

或者是你在写消息到RocketMQ的过程中，刚好遇到了某个Leader Broker自身故障，其他的Follower Broker正在尝试切换为Leader Broker，这个过程中也可能会有异常。类似的问题可能还有其他的。

消息到达MQ了，MQ自己会导致消息丢失吗？

你的消息写入MQ之后，其实MQ可能仅仅是把这个消息给写入到page cache里，也就是操作系统自己管理的一个缓冲区，这本质也是内存.

这个时候，假如要是出现了Broker机器的崩溃，大家思考一下，机器一旦宕机，是不是os cache内存中的数据就没了.

进入磁盘了会消息丢失吗？

如果消息进入了broker机器的磁盘之后，万一你实在是点儿背，赶上机器刚好磁盘坏了，可能上面的消息也就都丢失了.

红包系统拿到了消息，就一定不会丢失了吗？

假设红包系统已经获取到了消息1了，然后消息1此时就在他的内存里，正准备运行代码去派发现金红包呢，但是要注意，此时还没派发现金红包.

默认情况下，MQ的消费者有可能会自动提交已经消费的offset，那么如果此时你还没处理这个消息派发红包的情况下，MQ的消费者可能直接自动给你提交这个消息1的offset到broker去了，标识为你已经成功处理了这个消息.

## 72.发送消息零丢失方案

发送half消息到MQ去，试探一下MQ是否正常

在基于RocketMQ的事务消息机制中，我们首先要让订单系统去发送一条half消息到MQ去，这个half消息本质就是一个订单支付成功的消息，只不过你可以理解为他这个消息的状态是half状态，这个时候红包系统是看不见这个half消息的.

然后我们去等待接收这个half消息写入成功的响应通知.

half消息写入失败了呢？

你的half消息都没发送成功，总之你现在肯定没法跟MQ通信了。

这个时候你的订单系统就应该执行一系列的回滚操作，比如对订单状态做一个更新，让状态变成“关闭交易”，同时通知支付系统自动进行退款.

half消息成功之后，订单系统完成自己的任务

你的订单系统就应该在自己本地的数据库里执行一些增删改操作了，因为一旦half消息写成功了，就说明MQ肯定已经收到这条消息了，MQ还活着，而且目前你是可以跟MQ正常沟通的。

如果订单系统的本地事务执行失败了怎么办？

让订单系统发送一个rollback请求给MQ就可以了。这个意思就是说，你可以把之前我发给你的half消息给删除掉了.

当然你发送rollback请求给MQ删除那个half消息之后，你的订单系统就必须走后续的回退流程了，就是通知支付系统退款。

如果订单系统完成了本地事务之后，接着干什么？

如果订单系统成功完成了本地的事务操作，比如把订单状态都更新为“已完成”了，此时你就可以发送一个commit请求给MQ，要求让MQ对之前的half消息进行commit操作，让红包系统可以看见这个订单支付成功消息.

如果发送half消息成功了，但是没收到响应呢？

如果我们把half消息发送给MQ了，MQ给保存下来了，但是MQ返回给我们的响应我们没收到呢？此时会发生什么事情？

RocketMQ这里有一个补偿流程，他会去扫描自己处于half状态的消息，如果我们一直没有对这个消息执行commit/rollback操作，超过了一定的时间，他就会回调你的订单系统的一个接口.

这个时候我们的订单系统就得去查一下数据库，看看这个订单当前的状态，一下发现订单状态是“已关闭”，此时就知道，你必然得发送rollback请求给MQ去删除之前那个half消息了！

如果rollback或者commit发送失败了呢？

MQ里的消息一直是half状态，所以说他过了一定的超时时间会发现这个half消息有问题，他会回调你的订单系统的接口.

你此时要判断一下，这个订单的状态如果更新为了“已完成”，那你就得再次执行commit请求，反之则再次执行rollback请求。

## 73.事务消息机制的底层实现原理

half 消息是如何对消费者不可见的？

你写入一个Topic，最终是定位到这个Topic的某个MessageQueue，然后定位到一台Broker机器上去，然后写入的是Broker上的CommitLog文件，同时将消费索引写入MessageQueue对应的ConsumeQueue文件.

实际上红包系统却没法看到这条消息，其本质原因就是RocketMQ一旦发现你发送的是一个half消息，他不会把这个half消息的offset写入OrderPaySuccessTopic的ConsumeQueue里去。

他会把这条half消息写入到自己内部的“RMQ_SYS_TRANS_HALF_TOPIC”这个Topic对应的一个ConsumeQueue里去.

对于事务消息机制之下的half消息，RocketMQ是写入内部Topic的ConsumeQueue的，不是写入你指定的OrderPaySuccessTopic的ConsumeQueue的.

在什么情况下订单系统会收到half消息成功的响应？

half消息进入到RocketMQ内部的RMQ_SYS_TRANS_HALF_TOPIC的ConsumeQueue文件了，此时就会认为half消息写入成功了，然后就会返回响应给订单系统。

如果执行rollback操作的话，如何标记消息回滚？

RocketMQ会把这个half消息给删除，但是大家觉得删除消息是真的会在磁盘文件里删除吗？

显示不是的.

因为RocketMQ都是顺序把消息写入磁盘文件的，所以在这里如果你执行rollback，他的本质就是用一个OP操作来标记half消息的状态.

RocketMQ内部有一个OP_TOPIC，此时可以写一条rollback OP记录到这个Topic里，标记某个half消息是rollback了.

假设你一直没有执行commit/rollback，RocketMQ会回调订单系统的接口去判断half消息的状态，但是他最多就是回调15次，如果15次之后你都没法告知他half消息的状态，就自动把消息标记为rollback。

如果执行commit操作，如何让消息对红包系统可见？

你执行commit操作之后，RocketMQ就会在OP_TOPIC里写入一条记录，标记half消息已经是commit状态了。

接着需要把放在RMQ_SYS_TRANS_HALF_TOPIC中的half消息给写入到OrderPaySuccessTopic的ConsumeQueue里去.

### 为什么解决发送消息零丢失方案，一定要使用事务消息方案？

同步发消息 + 反复重试多次

完全寄希望于本地事务自动回滚是不现实的。Redis、Elasticsearch中的数据更新会自动回滚.

保证业务系统一致性的最佳方案：基于RocketMQ的事务消息机制

这个方案落地之后，他可以保证你的订单系统的本地事务一旦成功，那么必然会投递消息到MQ去，通知红包系统去派发红包，保证业务系统的数据是一致的。而且整个流程中，你没必要进行长时间的阻塞和重试。

如果half消息发送就失败了，你就直接回滚整个流程。如果half消息发送成功了，后续的rollback或者commit发送失败了，你不需要自己去卡在那里反复重试，你直接让代码结束即可，因为后续MQ会过来回调你的接口让你判断再次rollback or commit的。

## 74.Broker消息零丢失方案：同步刷盘 + Raft协议主从同步

用了事务消息机制，消息就一定不会丢了吗？

在异步刷盘的模式下，我们的写入消息的吞吐量肯定是极高的，毕竟消息只要进入os cache这个内存就可以了，写消息的性能就是写内存的性能，那每秒钟可以写入的消息数量肯定更多了，但是这个情况下，可能就会导致数据的丢失。

所以如果一定要确保数据零丢失的话，可以调整MQ的刷盘策略，我们需要调整broker的配置文件，将其中的flushDiskType配置设置为：SYNC_FLUSH，默认他的值是ASYNC_FLUSH，即默认是异步刷盘的。

如果调整为同步刷盘之后，我们写入MQ的每条消息，只要MQ告诉我们写入成功了，那么他们就是已经进入了磁盘文件了！

比如我们发送half消息的时候，只要MQ返回响应是half消息发送成功了，那么就说明消息已经进入磁盘文件了，不会停留在os cache里。

主从架构模式避免磁盘故障导致的数据丢失？

你一条消息但凡写入成功了，此时主从两个Broker上都有这条数据了，此时如果你的Master Broker的磁盘坏了，但是Slave Broker上至少还是有数据的，数据是不会因为磁盘故障而丢失的。

## 75.Consumer消息零丢失方案：手动提交offset + 自动故障转移

RocketMQ的消费者中会注册一个监听器，就是MessageListenerConcurrently这个东西，当你的消费者获取到一批消息之后，就会回调你的这个监听器函数，让你来处理这一批消息。

然后当你处理完毕之后，你才会返ConsumeConcurrentlyStatus.CONSUME_SUCCESS作为消费成功的示意，告诉RocketMQ，这批消息我已经处理完毕了。

所以对于RocketMQ而言，其实只要你的红包系统是在这个监听器的函数中先处理一批消息，基于这批消息都派发完了红包，然后返回了那个消费成功的状态，接着才会去提交这批消息的offset到broker去。

所以在这个情况下，如果你对一批消息都处理完毕了，然后再提交消息的offset给broker，接着红包系统崩溃了，此时是不会丢失消息的.

如果是红包系统获取到一批消息之后，还没处理完，也就没返回ConsumeConcurrentlyStatus.CONSUME_SUCCESS这个状态呢，自然没提交这批消息的offset给broker呢，此时红包系统突然挂了，会怎么样？

在这种情况下，你对一批消息都没提交他的offset给broker的话，broker不会认为你已经处理完了这批消息，此时你突然红包系统的一台机器宕机了，他其实会感知到你的红包系统的一台机器作为一个Consumer挂了。

接着他会把你没处理完的那批消息交给红包系统的其他机器去进行处理，所以在这种情况下，消息也绝对是不会丢失的.

不能异步消费消息

我们不能在代码中对消息进行异步的处理,比如我们开启了一个子线程去处理这批消息，然后启动线程之后，就直接返回ConsumeConcurrentlyStatus.CONSUME_SUCCESS状态了.

如果要是用这种方式来处理消息的话，那可能就会出现你开启的子线程还没处理完消息呢，你已经返回ConsumeConcurrentlyStatus.CONSUME_SUCCESS状态了，就可能提交这批消息的offset给broker了，认为已经处理结束了。

然后此时你红包系统突然宕机，必然会导致你的消息丢失了！

## 76.基于 RocketMQ 设计的全链路消息零丢失方案总结

1. 发送消息到MQ的零丢失：
2. 方案一（同步发送消息 + 反复多次重试）
3. 方案二（事务消息机制），两者都有保证消息发送零丢失的效果，但是经过分析，事务消息方案整体会更好一些
4. **MQ收到消息之后的零丢失**：开启同步刷盘策略 + 主从架构同步机制，只要让一个Broker收到消息之后同步写入磁盘，同时同步复制给其他Broker，然后再返回响应给生产者说写入成功，此时就可以保证MQ自己不会弄丢消息
5. **消费消息的零丢失**：采用RocketMQ的消费者天然就可以保证你处理完消息之后，才会提交消息的offset到broker去，只要记住别采用多线程异步处理消息的方式即可.

## 77.引入 幂等性机制，保证数据不会重复

什么是幂等性机制？

幂等性机制，其实就是用来避免对同一个请求或者同一条消息进行重复处理的机制，所谓的幂等，他的意思就是，比如你有一个接口，然后如果别人对一次请求重试了多次，来调用你的接口，你必须保证自己系统的数据是正常的，不能多出来一些重复的数据，这就是幂等性的意思。

发送消息到MQ的时候如何保证幂等性？

常见的方案有两种。

第一个方案就是业务判断法，也就是说你的订单系统必须要知道自己到底是否发送过消息到MQ去，消息到底是否已经在MQ里了。

当支付系统重试调用你的订单系统的接口时，你需要发送一个请求到MQ去，查询一下当前MQ里是否存在针对这个订单的支付消息？

如果MQ告诉你，针对id=1100这个订单的支付成功消息，在我这里已经有了，你之前已经写入进来了，那么订单系统就可以不要再次发送这条消息到MQ去了.

基于Redis缓存的幂等性机制

第二种方法，就是状态判断法.

这个方法的核心在于，你需要引入一个Redis缓存来存储你是否发送过消息的状态，如果你成功发送了一个消息到MQ里去，你得在Redis缓存里写一条数据，标记这个消息已经发送过.

当你的订单接口被重复调用的时候，你只要根据订单id去Redis缓存里查询一下，这个订单的支付消息是否已经发送给MQ了，如果发送过了，你就别再次发送了！

MQ消息幂等性的方案总结

对于MQ的重复消息问题而言，我们往MQ里重复发送一样的消息其实是还可以接收的，因为MQ里有多条重复消息，他不会对系统的核心数据直接造成影响，但是我们关键要保证的，是你从MQ里获取消息进行处理的时候，必须要保证消息不能重复处理。

这里的话，要保证消息的幂等性，我们优先推荐的其实还是业务判断法，直接根据你的数据存储中的记录来判断这个消息是否处理过，如果处理过了，那就别再次处理了。因为我们要知道，基于Redis的消息发送状态的方案，在一些极端情况下还是没法完全保证幂等性的。

## 78.如果优惠券系统的数据库宕机，如何用死信队列解决这种异常场景？

如果对消息的处理有异常，可以返回RECONSUME_LATER状态

RECONSUME_LATER状态意思是，我现在没法完成这批消息的处理，麻烦你稍后过段时间再次给我这批消息让我重新试一下！

RocketMQ是如何让你进行消费重试的？

RocketMQ会有一个针对你这个ConsumerGroup的重试队列。

如果你返回了RECONSUME_LATER状态，他会把你这批消息放到你这个消费组的重试队列中去.

过一段时间之后，重试队列中的消息会再次给我们，让我们进行处理。如果再次失败，又返回了RECONSUME_LATER，那么会再过一段时间让我们来进行处理，默认最多是重试16次！每次重试之间的间隔时间是不一样的，这个间隔时间可以如下进行配置：

messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h

上面这段配置的意思是，第一次重试是1秒后，第二次重试是5秒后，第三次重试是10秒后，第四次重试是30秒后，第五次重试是1分钟后，以此类推，最多重试16次！

如果连续重试16次还是无法处理消息，然后怎么办？

那么如果在16次重试范围内消息处理成功了，自然就没问题了，但是如果你对一批消息重试了16次还是无法成功处理呢？

这个时候就需要另外一个队列了，叫做**死信队列**，所谓的死信队列，顾名思义，就是死掉的消息就放这个队列里。

那么什么叫死掉的消息呢？

其实就是一批消息交给你处理，你重试了16次还一直没处理成功，就不要继续重试这批消息了，你就认为他们死掉了就可以了。然后这批消息会自动进入死信队列。

那么对死信队列中的消息我们怎么处理？

其实这个就看你的使用场景了，比如我们可以专门开一个后台线程，就是订阅“%DLQ%VoucherConsumerGroup”这个死信队列，对死信队列中的消息，还是一直不停的重试。

## 79.消息乱序

我们原本有顺序的消息，完全可能会分发到不同的MessageQueue中去，然后不同机器上部署的Consumer可能会用混乱的顺序从不同的MessageQueue里获取消息然后处理。

处理方法

一个Consumer可以处理多个MessageQueue的消息，但是一个MessageQueue只能交给一个Consumer来进行处理，所以一个订单的binlog只会有序的交给一个Consumer来进行处理！

这样的话一个大数据系统就可以获取到一个订单的有序的binlog，然后有序的根据binlog把数据还原到自己的存储中去。

对于有序消息的方案中，如果你遇到消息处理失败的场景，就必须返回SUSPEND_CURRENT_QUEUE_A_MOMENT这个状态，意思是先等一会儿，一会儿再继续处理这批消息，而不能把这批消息放入重试队列去，然后直接处理下一批消息。

如何让一个订单的binlog进入一个MessageQueue？

首先要实现消息顺序，必须让一个订单的binlog都进入一个MessageQueue中

关键因素就是两个，一个是发送消息的时候传入一个MessageQueueSelector，在里面你要根据订单id和MessageQueue数量去选择这个订单id的数据进入哪个MessageQueue。

同时在发送消息的时候除了带上消息自己以外，还要带上订单id，然后MessageQueueSelector就会根据订单id去选择一个MessageQueue发送过去，这样的话，就可以保证一个订单的多个binlog都会进入一个MessageQueue中去。

消费者如何保证按照顺序来获取一个MessageQueue中的消息？

Consumer会对每一个ConsumeQueue，都仅仅用一个线程来处理其中的消息。

比如对ConsumeQueue01中的订单id=1100的多个binlog，会交给一个线程来按照binlog顺序来依次处理。否则如果ConsumeQueue01中的订单id=1100的多个binlog交给Consumer中的多个线程来处理的话，那还是会有消息乱序的问题。

基于RocketMQ的数据过滤机制，提升订单数据库同步的处理效率

在发送消息的时候，给消息设置tag和属性

可以采用RocketMQ支持的数据过滤机制，来让大数据系统仅仅关注他想要的表的binlog数据即可。

首先，我们在发送消息的时候，可以给消息设置tag和属性.

在消费数据的时候根据tag和属性进行过滤

接着我们可以在消费的时候根据tag和属性进行过滤

## 80.基于延迟消息机制优化大量订单的定时退款扫描问题！

延迟消息，意思就是说，我们订单系统在创建了一个订单之后，可以发送一条消息到MQ里去，我们指定这条消息是延迟消息，比如要等待30分钟之后，才能被订单扫描服务给消费到.

这样当订单扫描服务在30分钟后消费到了一条消息之后，就可以针对这条消息的信息，去订单数据库里查询这个订单，看看他在创建过后都过了30分钟了，此时他是否还是未支付状态？

如果此时订单还是未支付状态，那么就可以关闭他，否则订单如果已经支付了，就什么都不用做了.

这种方式就比你用后台线程扫描订单的方式要好的多了，一个是对每个订单你只会在他创建30分钟后查询他一次而已，不会反复扫描订单多次。

另外就是如果你的订单数量很多，你完全可以让订单扫描服务多部署几台机器，然后对于MQ中的Topic可以多指定一个MessageQueue，这样每个订单扫描服务的机器作为一个Consumer都会处理一部分订单的查询任务。

发送延迟消息的核心，就是设置消息的**delayTimeLevel**，也就是延迟级别.

RocketMQ默认支持一些延迟级别如下：1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h

所以上面代码中设置延迟级别为3，意思就是延迟10s，你发送出去的消息，会过10s被消费者获取到。那么如果是订单延迟扫描场景，可以设置延迟级别为16，也就是对应上面的30分钟。

## 81.经验总结

（1）灵活的运用 tags来过滤数据

合理的规划Topic和里面的tags，一个Topic代表了一类业务消息数据，然后对于这类业务消息数据，如果你希望继续划分一些类别的话，可以在发送消息的时候设置tags。

（2）基于消息key来定位消息是否丢失

可以基于消息key来实现，比如通过下面的方式设置一个消息的key为订单id：message.setKeys(orderId)，这样这个消息就具备一个key了。

接着这个消息到broker上，会基于key构建hash索引，这个hash索引就存放在IndexFile索引文件里。

然后后续我们可以通过MQ提供的命令去根据key查询这个消息，类似下面这样：mqadmin queryMsgByKey -n 127.0.0.1:9876 -t SCANRECORD -k orderId

（3）消息零丢失方案的补充

一般假设MQ集群彻底崩溃了，你生产者就应该把消息写入到本地磁盘文件里去进行持久化，或者是写入数据库里去暂存起来，等待MQ恢复之后，然后再把持久化的消息继续投递到MQ里去。

（4）提高消费者的吞吐量

如果消费的时候发现消费的比较慢，那么可以提高消费者的并行度，常见的就是部署更多的consumer机器.

但是这里要注意，你的Topic的MessageQueue得是有对应的增加，因为如果你的consumer机器有5台，然后MessageQueue只有4个，那么意味着有一个consumer机器是获取不到消息的。

然后就是可以增加consumer的线程数量，可以设置consumer端的参数：consumeThreadMin、consumeThreadMax，这样一台consumer机器上的消费线程越多，消费的速度就越快。

此外，还可以开启消费者的批量消费功能，就是设置consumeMessageBatchMaxSize参数，他默认是1，但是你可以设置的多一些，那么一次就会交给你的回调函数一批消息给你来处理了，此时你可以通过SQL语句一次性批量处理一些数据，比如：update xxx set xxx where id in (xx,xx,xx)。

通过批量处理消息的方式，也可以大幅度提升消息消费的速度。

（5）要不要消费历史消息

其实consumer是支持设置从哪里开始消费消息的，常见的有两种：一个是从Topic的第一条数据开始消费，一个是从最后一次消费过的消息之后开始消费。对应的是：CONSUME_FROM_LAST_OFFSET，CONSUME_FROM_FIRST_OFFSET.

一般来说，我们都会选择CONSUME_FROM_FIRST_OFFSET，这样你刚开始就从Topic的第一条消息开始消费，但是以后每次重启，你都是从上一次消费到的位置继续往后进行消费的。

## 82.百万消息积压问题，应该如何处理？

方法一

可以临时申请16台机器多部署16个消费者系统的实例，然后20个消费者系统同时消费，每个人消费一个MessageQueue的消息，此时你会发现你消费的速度提高了5倍，很快积压的百万消息都会被处理完毕。

方法二

如果你的Topic总共就只有4个MessageQueue，然后你就只有4个消费者系统呢？

这个时候就没办法扩容消费者系统了，因为你加再多的消费者系统，还是只有4个MessageQueue，没法并行消费。

所以此时往往是临时修改那4个消费者系统的代码，让他们获取到消息然后不写入NoSQL，而是直接把消息写入一个新的Topic，这个速度是很快的，因为仅仅是读写MQ而已。

然后新的Topic有20个MessageQueue，然后再部署20台临时增加的消费者系统，去消费新的Topic后写入数据到NoSQL里去，这样子也可以迅速的增加消费者系统的并行处理能力，使用一个新的Topic来允许更多的消费者系统并行处理。

## 83.为什么要给RocketMQ增加消息限流功能保证其高可用性？

在接收消息这块，必须引入一个限流机制，也就是说要限制好，你这台机器每秒钟最多就只能处理比如3万条消息，根据你的MQ集群的压测结果来，你可以通过压测看看你的MQ最多可以抗多少QPS，然后就做好限流。

一般来说，限流算法可以采取**令牌桶算法**，也就是说你每秒钟就发放多少个令牌，然后只能允许多少个请求通过。

很多互联网大厂其实都会改造开源MQ的内核源码，引入限流机制，然后只能允许指定范围内的消息被在一秒内被处理，避免因为一些异常的情况，导致MQ集群挂掉。

## 84.设计一套Kafka到RocketMQ的双写+双读技术方案，实现无缝迁移！

假设你们公司本来线上的MQ用的主要是Kafka，现在要从Kafka迁移到RocketMQ去，那么这个迁移的过程应该怎么做呢？应该采用什么样的技术方案来做迁移呢？

首先你要做到双写，也就是说，在你所有的Producer系统中，要引入一个双写的代码，让他同时往Kafka和RocketMQ中去写入消息，然后多写几天，起码双写要持续个1周左右，因为MQ一般都是实时数据，里面数据也就最多保留一周。

当你的双写持续一周过后，你会发现你的Kafka和RocketMQ里的数据看起来是几乎一模一样了，因为MQ反正也就保留最近几天的数据，当你双写持续超过一周过后，你会发现Kafka和RocketMQ里的数据几乎一模一样了。

但是光是双写还是不够的，还需要同时进行双读，也就是说在你双写的同时，你所有的Consumer系统都需要同时从Kafka和RocketMQ里获取消息，分别都用一模一样的逻辑处理一遍。

只不过从Kafka里获取到的消息还是走核心逻辑去处理，然后可以落入数据库或者是别的存储什么的，但是对于RocketMQ里获取到的消息，你可以用一样的逻辑处理，但是不能把处理结果具体的落入数据库之类的地方。

你的Consumer系统在同时从Kafka和RocketMQ进行消息读取的时候，你需要统计每个MQ当日读取和处理的消息的数量，这点非常的重要，同时对于RocketMQ读取到的消息处理之后的结果，可以写入一个临时的存储中。

同时你要观察一段时间，当你发现持续双写和双读一段时间之后，如果所有的Consumer系统通过对比发现，从Kafka和RocketMQ读取和处理的消息数量一致，同时处理之后得到的结果也都是一致的，此时就可以判断说当前Kafka和RocketMQ里的消息是一致的，而且计算出来的结果也都是一致的。

这个时候就可以实施正式的切换了，你可以停机Producer系统，再重新修改后上线，全部修改为仅仅写RocketMQ，这个时候他数据不会丢，因为之前已经双写了一段时间了，然后所有的Consumer系统可以全部下线后修改代码再上线，全部基于RocketMQ来获取消息，计算和处理，结果写入存储中。

基本上对于类似的一些重要中间件的迁移，往往都会采取双写的方法，双写一段时间，然后观察两个方案的结果都一致了，你再正式 下线旧的一套东西。

## 85.**RocketMQ组成部分有哪些？**

**Nameserver** 无状态，动态列表；这也是和zookeeper的重要区别之一。zookeeper是有状态的。

**Producer** 消息生产者，负责发消息到Broker。

**Broker** 就是MQ本身，负责收发消息、持久化消息等。

**Consumer** 消息消费者，负责从Broker上拉取消息进行消费，消费完进行ack。

### 86.**消息重复消费如何解决？**

**出现原因** 正常情况下在consumer真正消费完消息后应该发送ack，通知broker该消息已正常消费，从queue中剔除 当ack因为网络原因无法发送到broker，broker会认为词条消息没有被消费，此后会开启消息重投机制把消息再次投递到consumer。

消费模式：在`CLUSTERING`模式下，消息在broker中会保证相同group的consumer消费一次，但是针对不同group的consumer会推送多次

**解决方案**

- 数据库表：处理消息前，使用消息主键在表中带有约束的字段中insert
- Map：单机时可以使用map做限制，消费时查询当前消息id是不是已经存在
- Redis：使用分布式锁。

### 87.**RocketMQ如何保证消息的顺序消费？**

首先多个queue只能保证单个queue里的顺序，queue是典型的FIFO，天然顺序。多个queue同时消费是无法绝对保证消息的有序性的。 可以使用同一`topic`，同一个QUEUE，发消息的时候一个线程去发送消息，消费的时候 一个线程去消费一个queue里的消息。

## 88.**RocketMQ如何保证消息不丢失？**

**Producer端** 采取`send()`同步发消息，发送结果是同步感知的。 发送失败后可以重试，设置重试次数。默认3次。

**Broker端** 修改刷盘策略为同步刷盘。默认情况下是异步刷盘的。 集群部署

**Consumer端** 完全消费正常后在进行手动ack确认

## 89.**RocketMQ如何实现分布式事务？**

1、生产者向MQ服务器发送half消息。 2、half消息发送成功后，MQ服务器返回确认消息给生产者。 3、生产者开始执行本地事务。 4、根据本地事务执行的结果（`UNKNOW`、`commit`、`rollback`）向MQ Server发送提交或回滚消息。 5、如果错过了（可能因为网络异常、生产者突然宕机等导致的异常情况）提交/回滚消息，则MQ服务器将向同一组中的每个生产者发送回查消息以获取事务状态。 6、回查生产者本地事物状态。 7、生产者根据本地事务状态发送`提交/回滚`消息。 8、MQ服务器将丢弃回滚的消息，但已提交（进行过二次确认的half消息）的消息将投递给消费者进行消费。

**Half Message**：预处理消息，当broker收到此类消息后，会存储到`RMQ_SYS_TRANS_HALF_TOPIC`的消息消费队列中

**检查事务状态**：Broker会开启一个定时任务，消费`RMQ_SYS_TRANS_HALF_TOPIC`队列中的消息，每次执行任务会向消息发送者确认事务执行状态（提交、回滚、未知），如果是未知，Broker会定时去回调在重新检查。

超时：如果超过回查次数，默认回滚消息。 也就是他并未真正进入Topic的queue，而是用了临时queue来放所谓的`half message`，等提交事务后才会真正的将half message转移到topic下的queue。

## 90.**RocketMQ的消息堆积如何处理？**

1、如果可以添加消费者解决，就添加消费者的数据量

2、如果出现了queue，但是消费者多的情况。可以使用准备一个临时的topic，同时创建一些queue，在临时创建一个消费者来把这些消息转移到topic中，让消费者消费。

## 压测

### 1.压测目的

平时做压测，主要关注的还是要压测出来一个最合适的最高负载。

什么叫最合适的最高负载呢？

意思就是**在RocketMQ的TPS和机器的资源使用率和负载之间取得一个平衡。**

比如RocketMQ集群在机器资源使用率极高的极端情况下可以扛到10万TPS，但是当他仅仅抗下8万TPS的时候，你会发现cpu负载、内存使用率、IO负载和网卡流量，都负载较高，但是可以接受，机器比较安全，不至于宕机。

那么这个8万TPS实际上就是最合适的一个最高负载，也就是说，哪怕生产环境中极端情况下，RocketMQ的TPS飙升到8万TPS，你知道机器资源也是大致可以抗下来的，不至于出现机器宕机的情况。

所以我们做压测，其实最主要的是综合TPS以及机器负载，尽量找到一个最高的TPS同时机器的各项负载在可承受范围之内，这才是压测的目的。

### 2.模拟压测

（1）RocketMQ的TPS和消息延时

我们让两个Producer不停的往RocketMQ集群发送消息，每个Producer所在机器启动了80个线程，相当于每台机器有80个线程并发的往RocketMQ集群写入消息。

而RocketMQ集群是1主2从组成的一个dledger模式的高可用集群，只有一个Master Broker会接收消息的写入。

然后有2个Cosumer不停的从RocketMQ集群消费数据。

每条数据的大小是500个字节，**这个非常关键**，大家一定要牢记这个数字，因为这个数字是跟后续的网卡流量有关的。

我们发现，一条消息从Producer生产出来到经过RocketMQ的Broker存储下来，再到被Consumer消费，基本上这个时间跨度不会超过1秒钟，这些这个性能是正常而且可以接受的。

同时在RocketMQ的管理工作台中可以看到，Master Broker的TPS（也就是每秒处理消息的数量），可以稳定的达到7万左右，也就是每秒可以稳定处理7万消息。

（2）cpu负载情况

其次我们检查了一下Broker机器上的CPU负载，可以通过top、uptime等命令来查看.

比如执行top命令就可以看到cpu load和cpu使用率，这就代表了cpu的负载情况。

在你执行了top命令之后，往往可以看到如下一行信息：

load average：12.03，12.05，12.08

类似上面那行信息代表的是cpu在1分钟、5分钟和15分钟内的cpu负载情况

比如我们一台机器是24核的，那么上面的12意思就是有12个核在使用中。换言之就是还有12个核其实还没使用，cpu还是有很大余力的。

这个cpu负载其实是比较好的，因为并没有让cpu负载达到极限。

（3）内存使用率

使用free命令就可以查看到内存的使用率，根据当时的测试结果，机器上48G的内存，仅仅使用了一部分，还剩下很大一部分内存都是空闲可用的，或者是被RocketMQ用来进行磁盘数据缓存了。

所以内存负载是很低的。

（4）JVM GC频率

使用jstat命令就可以查看RocketMQ的JVM的GC频率，基本上新生代每隔几十秒会垃圾回收一次，每次回收过后存活的对象很少，几乎不进入老年代.

因此测试过程中，Full GC几乎一次都没有。

（5）磁盘IO负载

接着可以检查一下磁盘IO的负载情况。

首先可以用top命令查看一下IO等待占用CPU时间的百分比，你执行top命令之后，会看到一行类似下面的东西：

Cpu(s): 0.3% us, 0.3% sy, 0.0% ni, 76.7% id, 13.2% wa, 0.0% hi, 0.0% si。

在这里的13.2% wa，说的就是磁盘IO等待在CPU执行时间中的百分比.

如果这个比例太高，说明CPU执行的时候大部分时间都在等待执行IO，也就说明IO负载很高，导致大量的IO等待。

这个当时我们压测的时候，是在40%左右，说明IO等待时间占用CPU执行时间的比例在40%左右，这是相对高一些，但还是可以接受的，只不过如果继续让这个比例提高上去，就很不靠谱了，因为说明磁盘IO负载可能过高了。

（6）网卡流量

使用如下命令可以查看服务器的网卡流量：

sar -n DEV 1 2

通过这个命令就可以看到每秒钟网卡读写数据量了。当时我们的服务器使用的是千兆网卡，千兆网卡的理论上限是每秒传输128M数据，但是一般实际最大值是每秒传输100M数据。

因此当时我们发现的一个问题就是，在RocketMQ处理到每秒7万消息的时候，每条消息500字节左右的大小的情况下，每秒网卡传输数据量已经达到100M了，就是已经达到了网卡的一个极限值了。

因为一个Master Broker服务器，每秒不光是通过网络接收你写入的数据，还要把数据同步给两个Slave Broker，还有别的一些网络通信开销。

因此实际压测发现，每条消息500字节，每秒7万消息的时候，服务器的网卡就几乎打满了，无法承载更多的消息了。

（7）针对压测的一点小总结

最后针对本次压测做一点小的总结，实际上经过压测，最终发现我们的服务器的性能瓶颈在网卡上，因为网卡每秒能传输的数据是有限的.

因此当我们使用平均大小为500字节的消息时，最多就是做到RocketMQ单台服务器每秒7万的TPS，而且这个时候cpu负载、内存负载、jvm gc负载、磁盘io负载，基本都还在正常范围内。

只不过这个时候网卡流量基本已经打满了，无法再提升TPS了。

因此在这样的一个机器配置下，RocketMQ一个比较靠谱的TPS就是7万左右。

总结

1. 到底应该如何压测：应该在TPS和机器的cpu负载、内存使用率、jvm gc频率、磁盘io负载、网络流量负载之间取得一个平衡，尽量让TPS尽可能的提高，同时让机器的各项资源负载不要太高。
2. 实际压测过程：采用几台机器开启大量线程并发读写消息，然后观察TPS、cpu load（使用top命令）、内存使用率（使用free命令）、jvm gc频率（使用jstat命令）、磁盘io负载（使用top命令）、网卡流量负载（使用sar命令），不断增加机器和线程，让TPS不断提升上去，同时观察各项资源负载是否过高。
3. 生产集群规划：根据公司的后台整体QPS来定，稍微多冗余部署一些机器即可，实际部署生产环境的集群时，使用高配置物理机，同时合理调整os内核参数、jvm参数、中间件核心参数，如此即可