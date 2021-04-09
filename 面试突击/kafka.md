# kafka

## 1.kafka

Kafka是一种高吞吐、分布式、基于发布和订阅模型的消息系统，最初住Linkedln 公司开发，使用Scala编写，目前是Apache的开源项目。kafka用于离线和在线消息的消费。kafka将消息数据按顺序保存在磁盘上，并在集群内以副本的形式存储以防止数据丢失。

Kafka依赖ZooKeeper进行集群的管理，kafka与Storm、Spark能够非常友好地集成，用于实时流式计算。

1.broker：Kafka服务器，负责消息存储和转发。

一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。

2.topic：消息类别，Kafka按照topic来分类消息，可以理解为一个队列。

3.partition：topic的分区，一个topic可以包含多个partition，topic消息保存在各个partition上。

一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序。

4.offset：消息在日志中的位置，可以理解是消息在partition上的偏移量，也是代表该消息的唯一序号。

5.Producer：消息生产者，就是向kafka broker发消息的客户端。

6.Consumer：消息消费者，向kafka broker取消息的客户端。

7.Consumer Group：消费者分组，每个Consumer必须属于一个group。

这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制-给consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic。

8.Zookeeper：保存着集群broker、topic、partition等meta数据；另外，还负责broker故障发现，partition leader选举，负载均衡等功能。

## 2.Kafka的组成

Kafka的核心概念有Producer、Consumer、Broker和Topic。其中，Producer为消息生产者；Consumer为消息消费者；Broker为Kafka的消息服务端，负责消息的存储和转发；Topic为消息类别，Kafka按照Topic来对消息分类。

为了提高集群的并发度，Kafka还设计了Partition用于Topic上数据的分区，一个Topic数据可以分为多个Partition，每个Partition都负责保存和处理其中一部分消息数据。Partition的个数对应了消费者和生产者的并发度，比如Partition的个数为3，则集群中最多同时有3个线程的消费者并发处理数据。

Consumer Group为消费者组，每个Consumer都必须属于同一个Group，同一个Group内的Consumer可以并发地消费消息。

ZooKeeper为Kafka提供集群的管理。它保存着集群的Broker、Topic、Partition等元数据，还负责Broker故障发现、Leader选举、负载均衡等。

## 3.Kafka的数据存储设计

### Partition数据文件

Partition中的每条Message都包含3个属性：Offset、MessageSize、Data。其中，Offset表示Message在这个Partition中的偏移量，它在逻辑上是一个值，唯一确定了Partition中的一条Message;MessageSize表示消息内容Data的大小；Data为Message的具体内容。

### 数据文件分段segment（顺序读写、分段命令、二分查找）

Partition在物理上由多个Segment数据文件组成，每个Segment数据文件都大小相等、按顺序读写。每个Segment数据文件都以该段中最小的Offset命名，文件扩展名为.log。这样在查找指定Offset的Message的时候，用二分查找就可以定位到该Message在哪个Segment数据文件中。

Segment数据文件首先会被存储在内存中，当Segment上的消息条数达到配置值或消息发送时间超过阈值时，其上的消息会被Flush到磁盘，只有被Flush到磁盘的消息才能被消费者消费到。

Segment达到一定的大小（可以通过配置文件设定，默认为1GB）后将不会再往该Segment中写数据，Broker会创建新的Segment。

### 数据文件索引（分段索引、稀疏存储）

Kafka为每个Segment数据文件都建立了索引文件以方便数据寻址，索引文件的文件名与数据文件的文件名一致，不同的是索引文件的扩展名为.index。Kafka的索引文件并不会为数据文件中的每条Message都建立索引，而是采用稀疏索引的方式，每隔一定字节建立一条索引。这样可以有效地降低索引文件的大小，方便将索引文件加载到内存中以提高集群的吞吐量。索引文件中的第一位表示索引对应的Message的编号，第二位表示索引对应的Message的数据位置。

## 4.生产者并发设计

### 负载均衡（partition会均衡分布到不同broker上）

Kafka将一个Topic分为多个Partition，每个Partition上的数据都均衡分布在不同的Broker上，这样一个Topic上的数据就可以被多个Broker并发地接收或发送。

在实际应用过程中，为了提高消息的吞吐量，应用程序可以将Topic的Partition设置为多个（Partition的个数也不宜太多，一般依据集群大小和Topic上的数据量来决定，Partition的个数不能超过Broker节点的个数）。Producer可以通过随机或者Hash等方式将消息平均发送到多个Partition上以实现负载均衡。Kafka Partition多个Producer并发生产消息。

### 批量发送消息

批量发送消息是提高吞吐量的重要方式，Producer端可以在内存中合并多条消息后，以一次请求的方式发送批量的消息给Broker，从而大大减少Broker存储消息的I/O操作次数。但批量发送的时间应该在业务能够接受的延迟时间范围内。

### 压缩消息

Producer端可以通过gzip或Snappy格式对消息集合进行压缩。消息在Producer端进行压缩，在Consumer端进行解压。压缩的好处就是减少网络传输的数据量，减轻对网络带宽传输的压力。在实时处理海量数据的集群环境下，系统瓶颈往往体现在网络I/O和带宽上，因为内存和CPU对数据的处理效率常常是I/O的几十倍甚至上百倍。

## 5.消费者并发设计

### 多个Consumer并发消费消息

Topic的消息以Partition的形式存在于多个Broker上，应用程序可以启动多个Consumer并行地消费Topic上的数据以提高消息的处理效率。需要注意的是，一个Partition上的消息是时间有序的，多个Partition之间的顺序无法保证。Kafka多个Consumer并发消费消息。

### Consumer Group的概念和特性

Consumer Group是一个消费者组，同一个Consumer Group中的多个Consumer 线程可以并发地消费Topic上的消息，Consumer的线程并发数一般等于Partition的个数。同一个Consumer Group中的多个Consumer不能同时消费同一个Partition上的数据。不同Consumer Group 中的Consumer在同一个Topic上的数据消费互不影响。Consumer Group和 Consumer以group.id和client.id唯一标识。每个Consumer的每条消费记录都以Offset的形式提交到Kafka集群的Broker上，用以记录消息消费的位置。

同一个Partition内的消息是有序的，多个Partition上的数据无法保证时间的有序性。Consumer通过Pull方式消费消息。Kafka不删除已消费的消息。在Partition内部，Kafka消息按顺序读写磁盘数据，以时间复杂度O(1)的方式提供消息的持久化功能.

## 6.Kafka写入方式

producer采用推（push）模式将消息发布到broker，每条消息都被追加（append）到分区（patition）中，属于顺序写磁盘（顺序写磁盘效率比随机写内存要高，保障kafka吞吐率）。

## 7.分区（Partition）

Kafka集群有多个消息代理服务器（broker-server）组成，发布到Kafka集群的每条消息都有一个类别，用主题（topic）来表示。通常，不同应用产生不同类型的数据，可以设置不同的主题。一个主题一般会有多个消息的订阅者，当生产者发布消息到某个主题时，订阅了这个主题的消费者都可以接收到生成者写入的新消息。

Kafka集群为每个主题维护了分布式的分区（partition）日志文件，物理意义上可以把主题（topic）看作进行了分区的日志文件（partition log）。主题的每个分区都是一个有序的、不可变的记录序列，新的消息会不断追加到日志中。分区中的每条消息都会按照时间顺序分配到一个单调递增的顺序编号，叫做偏移量（offset），这个偏移量能够唯一地定位当前分区中的每一条消息。

消息发送时都被发送到一个topic，其本质就是一个目录，而topic是由一些Partition Logs(分区日志)组成。

每个Partition中的消息都是有序的，生产的消息被不断追加到Partition log上，其中的每一个消息都被赋予了一个唯一的offset值。

发布到Kafka主题的每条消息包括键值和时间戳。消息到达服务器端的指定分区后，都会分配到一个自增的偏移量。原始的消息内容和分配的偏移量以及其他一些元数据信息最后都会存储到分区日志文件中。消息的键也可以不用设置，这种情况下消息会均衡地分布到不同的分区。

分区的原因

1.方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了。

2.可以提高并发，因为可以以Partition为单位读写了。

Kafka比传统消息系统有更强的顺序性保证，它使用主题的分区作为消息处理的并行单元。Kafka以分区作为最小的粒度，将每个分区分配给消费者组中不同的而且是唯一的消费者，并确保一个分区只属于一个消费者，即这个消费者就是这个分区的唯一读取线程。那么，只要分区的消息是有序的，消费者处理的消息顺序就有保证。每个主题有多个分区，不同的消费者处理不同的分区，所以Kafka不仅保证了消息的有序性，也做到了消费者的负载均衡。

分区的原则

（1）指定了patition，则直接使用；

（2）未指定patition但指定key，通过对key的value进行hash出一个patition

（3）patition和key都未指定，使用轮询选出一个patition。

## 8.副本（Replication）

同一个partition可能会有多个replication（对应 server.properties 配置中的 default.replication.factor=N）。没有replication的情况下，一旦broker 宕机，其上所有 patition 的数据都不可被消费，同时producer也不能再将数据存于其上的patition。引入replication之后，同一个partition可能会有多个replication，而这时需要在这些replication之间选出一个leader，producer和consumer只与这个leader交互，其它replication作为follower从leader 中复制数据。

## 9.写入流程

1）producer先从zookeeper的 "/brokers/.../state"节点找到该partition的leader

2）producer将消息发送给该leader

3）leader将消息写入本地log

4）followers从leader pull消息，写入本地log后向leader发送ACK

5）leader收到所有ISR中的replication的ACK后，增加HW（high watermark，最后commit 的offset）并向producer发送ACK

## 10.存储策略

无论消息是否被消费，kafka都会保留所有消息。有两种策略可以删除旧数据：

1）基于时间：log.retention.hours=168

2）基于大小：log.retention.bytes=1073741824

需要注意的是，因为Kafka读取特定消息的时间复杂度为O(1)，即与文件大小无关，所以这里删除过期文件与提高 Kafka 性能无关。

## 11.消费模型

消息由生产者发布到Kafka集群后，会被消费者消费。消息的消费模型有两种：推送模型（push）和拉取模型（pull）。

基于推送模型（push）的消息系统，由消息代理记录消费者的消费状态。消息代理在将消息推送到消费者后，标记这条消息为已消费，但这种方式无法很好地保证消息被处理。

Kafka采用拉取模型，由消费者自己记录消费状态，每个消费者互相独立地顺序读取每个分区的消息。这种由消费者控制偏移量的优点是：消费者可以按照任意的顺序消费消息。

## 12.Kafka producer拦截器(interceptor)

Producer拦截器(interceptor)是在Kafka 0.10版本被引入的，主要用于实现clients端的定制化控制逻辑。

对于producer而言，interceptor使得用户在消息发送前以及producer回调逻辑前有机会对消息做一些定制化需求，比如修改消息等。同时，producer允许用户指定多个interceptor按序作用于同一条消息从而形成一个拦截链(interceptor chain)。Intercetpor的实现接口是org.apache.kafka.clients.producer.

ProducerInterceptor，其定义的方法包括：

（1）configure(configs)

获取配置信息和初始化数据时调用。

（2）onSend(ProducerRecord)：

该方法封装进KafkaProducer.send方法中，即它运行在用户主线程中。Producer确保在消息被序列化以及计算分区前调用该方法。用户可以在该方法中对消息做任何操作，但最好保证不要修改消息所属的topic和分区，否则会影响目标分区的计算

（3）onAcknowledgement(RecordMetadata, Exception)：

该方法会在消息被应答或消息发送失败时调用，并且通常都是在producer回调逻辑触发之前。onAcknowledgement运行在producer的IO线程中，因此不要在该方法中放入很重的逻辑，否则会拖慢producer的消息发送效率

（4）close：

关闭interceptor，主要用于执行一些资源清理工作

如前所述，interceptor可能被运行在多个线程中，因此在具体实现时用户需要自行确保线程安全。另外倘若指定了多个interceptor，则producer将按照指定顺序调用它们，并仅仅是捕获每个interceptor可能抛出的异常记录到错误日志中而非在向上传递。这在使用过程中要特别留意。

## 13.Kafka 的高可用性

Kafka 一个最基本的架构认识：由多个 broker 组成，每个 broker 是一个节点；你创建一个 topic，这个 topic 可以划分为多个 partition，每个 partition 可以存在于不同的 broker 上，每个 partition 就放一部分数据。

这就是**天然的分布式消息队列**，就是说一个 topic 的数据，是**分散放在多个机器上的，每个机器就放一部分数据**。

实际上 RabbitMQ 之类的，并不是分布式消息队列，它就是传统的消息队列，只不过提供了一些集群、HA(High Availability, 高可用性) 的机制而已，因为无论怎么玩儿，RabbitMQ 一个 queue 的数据都是放在一个节点里的，镜像集群下，也是每个节点都放这个 queue 的完整数据。

Kafka 0.8 以后，提供了 HA 机制，就是 replica（复制品） 副本机制。每个 partition 的数据都会同步到其它机器上，形成自己的多个 replica 副本。所有 replica 会选举一个 leader 出来，那么生产和消费都跟这个 leader 打交道，然后其他 replica 就是 follower。写的时候，leader 会负责把数据同步到所有 follower 上去，读的时候就直接读 leader 上的数据即可。只能读写 leader？很简单，**要是你可以随意读写每个 follower，那么就要 care 数据一致性的问题**，系统复杂度太高，很容易出问题。Kafka 会均匀地将一个 partition 的所有 replica 分布在不同的机器上，这样才可以提高容错性。

如果某个 broker 宕机了，没事儿，那个 broker 上面的 partition 在其他机器上都有副本的。如果这个宕机的 broker 上面有某个 partition 的 leader，那么此时会从 follower 中**重新选举**一个新的 leader 出来，大家继续读写那个新的 leader 即可。这就有所谓的高可用性了。

**写数据**的时候，生产者就写 leader，然后 leader 将数据落地写本地磁盘，接着其他 follower 自己主动从 leader 来 pull 数据。一旦所有 follower 同步好数据了，就会发送 ack 给 leader，leader 收到所有 follower 的 ack 之后，就会返回写成功的消息给生产者。（当然，这只是其中一种模式，还可以适当调整这个行为）

**消费**的时候，只会从 leader 去读，但是只有当一个消息已经被所有 follower 都同步成功返回 ack 的时候，这个消息才会被消费者读到。

## 14.如何保证消息消费的幂等性？

Kafka 实际上有个 offset 的概念，就是每个消息写进去，都有一个 offset，代表消息的序号，然后 consumer 消费了数据之后，**每隔一段时间**（定时定期），会把自己消费过的消息的 offset 提交一下，表示“我已经消费过了，下次我要是重启啥的，你就让我继续从上次消费到的 offset 来继续消费吧”。

但是凡事总有意外，比如我们之前生产经常遇到的，就是你有时候重启系统，看你怎么重启了，如果碰到点着急的，直接 kill 进程了，再重启。这会导致 consumer 有些消息处理了，但是没来得及提交 offset，尴尬了。重启之后，少数消息会再次消费一次。

注意：新版的 Kafka 已经将 offset 的存储从 Zookeeper 转移至 Kafka brokers，并使用内部位移主题 `__consumer_offsets` 进行存储。

一条数据重复出现两次，数据库里只有一条数据，这就保证了系统的幂等性。

幂等性，通俗点说，就一个数据，或者一个请求，给你重复来多次，你得确保对应的数据是不会改变的，**不能出错**。

怎么保证消息队列消费的幂等性？其实还是得结合业务来思考，我这里给几个思路：

- 比如你拿个数据要写库，你先根据主键查一下，如果这数据都有了，你就别插入了，update 一下好吧。
- 比如你是写 Redis，那没问题了，反正每次都是 set，天然幂等性。
- 比如你不是上面两个场景，那做的稍微复杂一点，你需要让生产者发送每条数据的时候，里面加一个全局唯一的 id，类似订单 id 之类的东西，然后你这里消费到了之后，先根据这个 id 去比如 Redis 里查一下，之前消费过吗？如果没有消费过，你就处理，然后这个 id 写 Redis。如果消费过了，那你就别处理了，保证别重复处理相同的消息即可。
- 比如基于数据库的唯一键来保证重复数据不会重复插入多条。因为有唯一键约束了，重复数据插入只会报错，不会导致数据库中出现脏数据。

## 15.如何保证消息的可靠性传输

### 消费端弄丢了数据

唯一可能导致消费者弄丢数据的情况，就是说，你消费到了这个消息，然后消费者那边**自动提交了 offset**，让 Kafka 以为你已经消费好了这个消息，但其实你才刚准备处理这个消息，你还没处理，你自己就挂了，此时这条消息就丢咯。

大家都知道 Kafka 会自动提交 offset，那么只要**关闭自动提交** offset，在处理完之后自己手动提交 offset，就可以保证数据不会丢。但是此时确实还是**可能会有重复消费**，比如你刚处理完，还没提交 offset，结果自己挂了，此时肯定会重复消费一次，自己保证幂等性就好了。

生产环境碰到的一个问题，就是说我们的 Kafka 消费者消费到了数据之后是写到一个内存的 queue 里先缓冲一下，结果有的时候，你刚把消息写入内存 queue，然后消费者会自动提交 offset。然后此时我们重启了系统，就会导致内存 queue 里还没来得及处理的数据就丢失了。

### Kafka 弄丢了数据

这块比较常见的一个场景，就是 Kafka 某个 broker 宕机，然后重新选举 partition 的 leader。大家想想，要是此时其他的 follower 刚好还有些数据没有同步，结果此时 leader 挂了，然后选举某个 follower 成 leader 之后，不就少了一些数据？这就丢了一些数据啊。

生产环境也遇到过，我们也是，之前 Kafka 的 leader 机器宕机了，将 follower 切换为 leader 之后，就会发现说这个数据就丢了。

所以此时一般是要求起码设置如下 4 个参数：

- 给 topic 设置 `replication.factor` 参数：这个值必须大于 1，要求每个 partition 必须有至少 2 个副本。
- 在 Kafka 服务端设置 `min.insync.replicas` 参数：这个值必须大于 1，这个是要求一个 leader 至少感知到有至少一个 follower 还跟自己保持联系，没掉队，这样才能确保 leader 挂了还有一个 follower 吧。
- 在 producer 端设置 `acks=all` ：这个是要求每条数据，必须是**写入所有 replica 之后，才能认为是写成功了**。
- 在 producer 端设置 `retries=MAX` （很大很大很大的一个值，无限次重试的意思）：这个是**要求一旦写入失败，就无限重试**，卡在这里了。

我们生产环境就是按照上述要求配置的，这样配置之后，至少在 Kafka broker 端就可以保证在 leader 所在 broker 发生故障，进行 leader 切换时，数据不会丢失。

### 生产者会不会弄丢数据？

如果按照上述的思路设置了 `acks=all` ，一定不会丢，要求是，你的 leader 接收到消息，所有的 follower 都同步到了消息之后，才认为本次写成功了。如果没满足这个条件，生产者会自动不断的重试，重试无限次。

## 16.如何保证消息的顺序性？

比如说我们建了一个 topic，有三个 partition。生产者在写的时候，其实可以指定一个 key，比如说我们指定了某个订单 id 作为 key，那么这个订单相关的数据，一定会被分发到同一个 partition 中去，而且这个 partition 中的数据一定是有顺序的。 消费者从 partition 中取出来数据的时候，也一定是有顺序的。到这里，顺序还是 ok 的，没有错乱。接着，我们在消费者里可能会搞**多个线程来并发处理消息**。因为如果消费者是单线程消费处理，而处理比较耗时的话，比如处理一条消息耗时几十 ms，那么 1 秒钟只能处理几十条消息，这吞吐量太低了。而多个线程并发跑的话，顺序可能就乱掉了。

### 解决方案

- 一个 topic，一个 partition，一个 consumer，内部单线程消费，单线程吞吐量太低，一般不会用这个。
- 写 N 个内存队列queue，具有相同 key 的数据都到同一个内存 queue；然后对于 N 个线程，每个线程分别消费一个内存 queue 即可，这样就能保证顺序性。

## 17.Consumer Group

同一 Consumer Group 中的多个 Consumer 实例，不同时消费同一个 partition，等效于队列模式。 partition 内消息是有序的， Consumer 通过 pull 方式消费消息。 Kafka 不 删除已消费的消息对于 partition，顺序读写磁盘数据，以时间复杂度 O(1)方式提供消息持久化能力。

## 18.consumer 是推还是拉？

Kafka 最初考虑的问题是，customer 应该从 brokes 拉取消息还是 brokers 将消息推送到 consumer，也就是 pull 还 push。在这方面，Kafka 遵循了一种大部分消息系统共同 的传统的设计：producer 将消息推送到 broker，consumer 从broker 拉取消息。 一些消息系统比如 Scribe 和 Apache Flume 采用了 push 模式，将消息推送到下游的 consumer。这样做有好处也有坏处：由 broker 决定消息推送的速率，对于不同消费速率 的 consumer 就不太好处理了。消息系统都致力于让 consumer 以最大的速率最快速的消费消息，但不幸的是，push 模式下，当 broker 推送的速率远大于 consumer 消费的 速率时，consumer 恐怕就要崩溃了。最终 Kafka 还是选取了传统的 pull 模式。 Pull 模式的另外一个好处是 consumer 可以自主决定是否批量的从 broker 拉取数据。Push 模式必须在不知道下游 consumer 消费能力和消费策略的情况下决定是立即推送每条 消息还是缓存之后批量推送。如果为了避免 consumer 崩溃而采用较低的推送速率，将可能导致一次只推送较少的消息而造成浪费。Pull 模式下， consumer 就可以根据自己的消费能力去决定这些策略。 Pull 有个缺点是，如果 broker 没有可供消费的消息，将导致 consumer 不断在循环中轮询，直到新消息到 t 达。为了避免这点，Kafka 有个参数可以让 consumer阻塞知道新消 息到达(当然也可以阻塞知道消息的数量达到某个特定的量这样就可以批量发送)。

## 19.讲讲 kafka 维护消费状态跟踪的方法

大部分消息系统在 broker 端的维护消息被消费的记录：一个消息被分发到consumer 后 broker 就马上进行标记或者等待 customer 的通知后进行标记。这样也可以在消息在消 费后立马就删除以减少空间占用。 但是这样会不会有什么问题呢？如果一条消息发送出去之后就立即被标记为消费过的，一旦 consumer 处理消息时失败了（比如程序崩溃）消息就丢失了。为了解决这个问题， 很多消息系统提供了另外一个个功能：当消息被发送出去之后仅仅被标记为已发送状态，当接到 consumer 已经消费成功的通知后才标记为已被消费的状态。这虽然解决了消息 丢失的问题，但产生了新问题，首先如果 consumer处理消息成功了但是向 broker 发送响应时失败了，这条消息将被消费两次。第二个问题时，broker 必须维护每条消息的状 态，并且每次都要先锁住消息然后更改状态然后释放锁。这样麻烦又来了，且不说要维护大量的状态数据，比如如果消息发送出去但没有收到消费成功的通知，这条消息将一直 处于被锁定的状态，Kafka 采用了不同的策略。Topic 被分成了若干分区，每个分区在同一时间只被一个 consumer 消费。这意味着每个分区被消费的消息在日志中的位置仅仅是 一个简单的整数：offset。这样就很容易标记每个分区消费状态就很容易了，仅仅需要一个整数而已。这样消费状态的跟踪就很简单了。这带来了另外一个好处：consumer 可以 把 offset 调成一个较老的值，去重新消费老的消息。这对传统的消息系统来说看起来有些不可思议，但确实是非常有用的，谁规定了一条消息只能被消费一次呢？

## 20.Zookeeper 对于 Kafka 的作用是什么？

Zookeeper 是一个开放源码的、高性能的协调服务，它用于 Kafka 的分布式应用。Zookeeper 主要用于在集群中不同节点之间进行通信 在 Kafka 中，它被用于提交偏移量，因此如果节点在任何情况下都失败了，它都可以从之前提交的偏移量中获取 除此之外，它还执行其他活动，如: leader 检测、分布式同步、配置管理、识别新节点何时离开或连接、集群、节点实时状态等等。

## 21.Kafka 判断一个节点是否还活着有那两个条件？

（1）节点必须可以维护和 ZooKeeper 的连接，Zookeeper 通过心跳机制检查每个节点的连接 （2）如果节点是个 follower,他必须能及时的同步 leader 的写操作，延时不能太久

## 22.Kafka 与传统 MQ 消息系统之间有三个关键区别

(1).Kafka 持久化日志，这些日志可以被重复读取和无限期保留 (2).Kafka 是一个分布式系统：它以集群的方式运行，可以灵活伸缩，在内部通过复制数据提升容错能力和高可用性 (3).Kafka 支持实时的流式处理

## 23.讲一讲 kafka 的 ack 的三种机制

request.required.acks 有三个值 0 1 -1(all) 0:生产者不会等待 broker 的 ack，这个延迟最低但是存储的保证最弱当 server 挂掉的时候就会丢数据。 1：服务端会等待 ack 值 leader 副本确认接收到消息后发送 ack 但是如果 leader挂掉后他不确保是否复制完成新 leader 也会导致数据丢失。 -1(all)：服务端会等所有的 follower 的副本受到数据后才会受到 leader 发出的ack，这样数据不会丢失

## 24.消费者故障，出现活锁问题如何解决？

出现 “活锁 ” 的情 况， 是它 持续 的发 送心 跳， 但是 没有 处理 。为 了预 防消 费者 在这种 情况 下一 直持 有分 区，我们 使用 max.poll.interval.ms 活跃 检测 机制 。 在此基础 上， 如果 你调 用的 poll 的频 率大 于最 大间 隔， 则客 户端 将主 动地 离开 组， 以便其 他消 费者 接管 该分 区。 发生 这种 情况 时， 你会 看到 offset 提交 失败 （调 用 commitSync（） 引发 的 CommitFailedException）。 这是 一种 安全 机制 ，保 障只有 活动 成员 能够 提交 offset。所 以要 留在 组中 ，你 必须 持续 调用 poll。 消费者提供两个配置设置来控制 poll 循环： max.poll.interval.ms：增大 poll 的间 隔 ，可以 为消 费者 提供 更多 的时 间去 处理 返回的 消息（调用 poll(long)返回 的消 息，通常 返回 的消 息都 是一 批）。缺点 是此 值越 大 将会 延迟 组重 新平 衡。 max.poll.records：此 设置 限制 每次 调用 poll 返回 的消 息数 ，这 样可 以更 容易 的预测 每次 poll 间隔 要处 理的 最大 值。通过 调整 此值 ，可以 减少 poll 间隔 ，减少 重新 平 衡分 组的 对于 消息 处理 时间 不可 预测 地的 情况 ，这些 选项 是不 够的 。 处理 这种 情况 的推 荐方法 是将 消息 处理 移到 另一 个线 程中 ，让消 费者 继续 调用 poll。 但是 必须 注意 确保已 提交 的 offset 不超 过实 际位 置。 另外 ，你 必须 禁用 自动 提交 ，并 只有 在线 程完成 处理 后才 为记 录手 动提 交偏 移量（取决 于你 ）。 还要 注意 ，你需 要 pause 暂停分 区， 不会 从 poll 接收 到新 消息 ，让 线程 处理 完之 前返 回的 消息 （如 果你 的处理能 力比 拉取 消息 的慢 ，那 创建 新线 程将 导致 你机 器内 存溢 出） 。

## 25.kafka 分布式（不是单机）的情况下，如何保证消息的顺序消费?

Kafka 分布式的单位是 partition，同一个 partition 用一个 write ahead log 组织，所以可以保证 FIFO 的顺序。不同 partition 之间不能保证顺序。但是绝大多数用户都可以通过 message key 来定义，因为同一个 key 的 Message 可以保证只发送到同一个 partition。 Kafka 中发送 1 条消息的时候，可以指定(topic, partition, key) 3 个参数。partiton 和 key 是可选的。如果你指定了 partition，那就是所有消息发往同 1个 partition，就是有序 的。并且在消费端，Kafka 保证，1 个 partition 只能被1 个 consumer 消费。或者你指定 key（比如 order id），具有同 1 个 key 的所有消息，会发往同 1 个 partition。

## 26.核心 API

Kafka 有四个核心API，它们分别是： Producer API，它允许应用程序向一个或多个 topics 上发送消息记录 Consumer API，允许应用程序订阅一个或多个 topics 并处理为其生成的记录流 Streams API，它允许应用程序作为流处理器，从一个或多个主题中消费输入流并为其生成输出流，有效的将输入流转换为输出 流。 Connector API，它允许构建和运行将 Kafka 主题连接到现有应用程序或数据系统的可用生产者和消费者。例如，关系数据库的连 接器可能会捕获对表的所有更改

## 27.**kafka follower如何与leader同步数据?**

Kafka的复制机制既不是完全的同步复制，也不是单纯的异步复制。 完全同步复制要求`All Alive Follower`都复制完，这条消息才会被认为commit，这种复制方式极大的影响了吞吐率。 异步复制方式下，Follower异步的从Leader复制数据，数据只要被Leader写入log就被认为已经commit，这种情况下，如果leader挂掉，会丢失数据； kafka使用`ISR`的方式很好的均衡了确保数据不丢失以及吞吐率。Follower可以批量的从Leader复制数据，而且Leader充分利用磁盘顺序读以及`send file(zero copy)`机制，这样极大的提高复制性能，内部批量写磁盘，大幅减少了Follower与Leader的消息量差。

## 28.**kafka 为什么那么快？**

- `Cache Filesystem Cache PageCache`缓存
- `顺序写`：由于现代的操作系统提供了预读和写技术，磁盘的顺序写大多数情况下比随机写内存还要快。
- `Zero-copy`：零拷技术减少拷贝次数
- `Batching of Messages`：批量量处理。合并小的请求，然后以流的方式进行交互，直顶网络上限。
- `Pull 拉模式`：使用拉模式进行消息的获取消费，与消费端处理能力相符。

## 29.**kafka producer如何优化打入速度？**

- 增加线程
- 提高 `batch.size`
- 增加更多 `producer` 实例
- 增加 `partition` 数
- 设置 `acks=-1` 时，如果延迟增大：可以增大 `num.replica.fetchers`（follower 同步数据的线程数）来调解；
- 跨数据中心的传输：增加 `socket` 缓冲区设置以及 `OS tcp` 缓冲区设置。

## 30.**kafka producer发送数据，ack为0，1，-1分别是什么意思？**

- `1`（默认） 数据发送到Kafka后，经过leader成功接收消息的的确认，就算是发送成功了。在这种情况下，如果leader宕机了，则会丢失数据。
- `0` 生产者将数据发送出去就不管了，不去等待任何返回。这种情况下数据传输效率最高，但是数据可靠性确是最低的。
- `-1`producer需要等待ISR中的所有follower都确认接收到数据后才算一次发送完成，可靠性最高。当ISR中所有Replica都向Leader发送ACK时，leader才commit，这时候producer才能认为一个请求中的消息都commit了。

## 31.**kafka的message格式是什么样的？**

一个Kafka的Message由一个`固定长度的header`和一个`变长的消息体body`组成

- header部分由一个字节的`magic`(文件格式)和四个字节的`CRC32`(用于判断body消息体是否正常)构成。 当magic的值为1的时候，会在magic和crc32之间多一个字节的数据：`attributes`(保存一些相关属性， 比如是否压缩、压缩格式等等);如果magic的值为0，那么不存在attributes属性
- body是由N个字节构成的一个消息体，包含了具体的`key/value`消息

## 32.**kafka中consumer group 是什么概念？**

同样是逻辑上的概念，是Kafka实现单播和广播两种消息模型的手段。 同一个topic的数据，会广播给不同的group； 同一个group中的worker，只有一个worker能拿到这个数据。 换句话说，对于同一个topic，每个group都可以拿到同样的所有数据，但是数据进入group后只能被其中的一个worker消费。group内的worker可以使用多线程或多进程来实现，也可以将进程分散在多台机器上，worker的数量通常不超过partition的数量，且二者最好保持整数倍关系，因为Kafka在设计时假定了一个partition只能被一个worker消费（同一group内）。

## 33.**Kafka中的消息是否会丢失和重复消费？**

**消息发送** Kafka消息发送有两种方式：同步（sync）和异步（async）， 默认是同步方式，可通过`producer.type`属性进行配置。 Kafka通过配置`request.required.acks`属性来确认消息的生产

- 0---表示不进行消息接收是否成功的确认；
- 1---表示当Leader接收成功时确认；
- -1---表示Leader和Follower都接收成功时确认；

综上所述，有6种消息生产的情况，消息丢失的场景：

- acks=0，不和Kafka集群进行消息接收确认，则当网络异常、缓冲区满了等情况时，消息可能丢失；
- acks=1、同步模式下，只有Leader确认接收成功后但挂掉了，副本没有同步，数据可能丢失；

**消息消费** Kafka消息消费有两个consumer接口，`Low-level API`和`High-level API`：

- Low-level API：消费者自己维护offset等值，可以实现对Kafka的完全控制；
- High-level API：封装了对parition和offset的管理，使用简单；

如果使用高级接口High-level API，可能存在一个问题就是当消息消费者从集群中把消息取出来、并提交了新的消息offset值后，还没来得及消费就挂掉了，那么下次再消费时之前没消费成功的消息就“诡异”的消失了；

解决办法： **针对消息丢失**：同步模式下，确认机制设置为-1，即让消息写入Leader和Follower之后再确认消息发送成功；异步模式下，为防止缓冲区满，可以在配置文件设置不限制阻塞超时时间，当缓冲区满时让生产者一直处于阻塞状态；

**针对消息重复**：将消息的唯一标识保存到外部介质中，每次消费时判断是否处理过即可。

## 34.**为什么Kafka不支持读写分离？**

在 Kafka 中，生产者写入消息、消费者读取消息的操作都是与 leader 副本进行交互的，从 而实现的是一种主写主读的生产消费模型。

Kafka 并不支持主写从读，因为主写从读有 2 个很明 显的缺点:

- **数据一致性问题**。数据从主节点转到从节点必然会有一个延时的时间窗口，这个时间 窗口会导致主从节点之间的数据不一致。某一时刻，在主节点和从节点中 A 数据的值都为 X， 之后将主节点中 A 的值修改为 Y，那么在这个变更通知到从节点之前，应用读取从节点中的 A 数据的值并不为最新的 Y，由此便产生了数据不一致的问题。
- **延时问题**。类似 Redis 这种组件，数据从写入主节点到同步至从节点中的过程需要经历`网络→主节点内存→网络→从节点内存`这几个阶段，整个过程会耗费一定的时间。而在 Kafka 中，主从同步会比 Redis 更加耗时，它需要经历`网络→主节点内存→主节点磁盘→网络→从节点内存→从节点磁盘`这几个阶段。对延时敏感的应用而言，主写从读的功能并不太适用。

## 35.kafka 如何不消费重复数据？比如扣款，我们不能重复的扣。

其实还是得结合业务来思考，我这里给几个思路： 比如你拿个数据要写库，你先根据主键查一下，如果这数据都有了，你就别插入了，update 一下好 吧。 比如你是写 Redis，那没问题了，反正每次都是 set，天然幂等性。 比如你不是上面两个场景，那做的稍微复杂一点，你需要让生产者发送每条数据的时候，里面加一个全 局唯一的 id，类似订单 id 之类的东西，然后你这里消费到了之后，先根据这个 id 去比如 Redis 里查一 下，之前消费过吗？如果没有消费过，你就处理，然后这个 id 写 Redis。如果消费过了，那你就别处理 了，保证别重复处理相同的消息即可。 比如基于数据库的唯一键来保证重复数据不会重复插入多条。因为有唯一键约束了，重复数据插入只会 报错，不会导致数据库中出现脏数据。

## kafka

### 1.吞吐量，延迟

写数据请求发送给kafka一直到他处理成功，你认为写请求成功，假设是1毫秒，这个就说明性能很高，延迟

kafka，每毫秒可以处理1条数据，每秒可以处理1000条数据，这个单位时间内可以处理多少条数据，就叫做吞吐量，1000条数据，每条数据10kb，10mb，吞吐量相当于是每秒处理10mb的数据

### 2.Kafka是如何利用顺序磁盘写机制实现单机每秒几十万消息写入的？

直接写入os的page cache中，文件，kafka仅仅是追加数据到文件末尾，磁盘顺序写，性能极高，几乎跟写内存是一样高的。磁盘随机写，你要随机在文件的某个位置修改数据，这个叫做磁盘随机写，性能是很低的，磁盘顺序写，仅仅追加数据到文件末尾

而且写磁盘的方式是顺序写，不是随机写，性能跟内存写几乎一样。就是仅仅在磁盘文件的末尾追加写，不能在文件随机位置写入

假设基于上面说的os cache写 + 磁盘顺序写，0.01毫秒，低延迟，高吞吐，每毫秒可以处理100条数据，每秒可以处理10万条数据，不需要依托类似spark straeming那种batch微批处理的机制

正是依靠了这个超高的写入性能，单物理机可以做到每秒几十万条消息写入Kafka

这种方式让kafka的写性能极高，最大程度减少了每条数据处理的时间开销，反过来就大幅度提升了每秒处理数据的吞吐量，一般kafka部署在物理机上，单机每秒写入几万到几十万条消息是没问题的

这种方式是不是就兼顾了低延迟和高吞吐两个要求，尽量把每条消息的写入性能压榨到极致，就可以实现低延迟的写入，同时对应的每秒的吞吐量自然就提升了

所以这是kafka非常核心的一个底层机制

而且这里很关键的一点，比如rabbitmq这种消息中间件，他会先把数据写入内存里，然后到了一定时候再把数据一次性从内存写入磁盘里，但是kafka不是这种机制，他收到数据直接写磁盘

只不过是写的page cache，而且是磁盘顺序写，所以写入的性能非常高，而且这样不需要让kafka自身的jvm进程占用过多内存，可以更多的把内存空间留给os的page cache来缓存磁盘文件的数据

只要能让更多的磁盘数据缓存在os cache里，那么后续消费数据从磁盘读的时候，就可以直接走os cache读数据了，性能是非常高的

### 3.Kafka是如何利用零拷贝和页缓存技术实现高性能读取的？

那么在消费数据的时候，需要从磁盘文件里读取数据后通过网络发送出去，这个时候怎么提升性能呢？

首先就是利用了page cache技术，之前说过，kafka写入数据到磁盘文件的时候，实际上是写入page cache的，没有直接发生磁盘IO，所以写入的数据大部分都是停留在os层的page cache里的

这个本质其实跟elasticsearch的实现原理是类似的

然后在读取的时候，如果正常情况下从磁盘读取数据，先尝试从page cache读，读不到才从磁盘IO读，读到数据以后先会放在os层的一个page cache里，接着会发生上下文切换到系统那边，把os的读缓存数据拷贝到应用缓存里

接着再次发生上下文二切换到os层，把应用缓存的数据拷贝到os的socket缓存中，最后数据再发送到网卡上

这个过程里，发生了好几次上下文切换，而且还涉及到了好几次数据拷贝，如果不考虑跟硬件之间的交互，起码是从os cache到用户缓存，从用户缓存到socket缓存，有两次拷贝是绝对没必要的

但是如果用零拷贝技术，就是linux的sendfile，就可以直接把操作交给os，os看page cache里是否有数据，如果没有就从磁盘上读取，如果有的话直接把os cache里的数据拷贝给网卡了，中间不用走那么多步骤了

对比一下，是不是所谓的零考贝了？

所以呢，通过零拷贝技术来读取磁盘上的数据，还有page cahce的帮助，这个性能就非常高了

### 4.Kafka的底层数据存储结构：日志文件以及offset

基本上可以认为每个partition就是一个日志文件，存在于某台Kafka服务器上，然后这个日志里写入了很多消息，每个消息在partition日志文件里都有一个序号，叫做offset，代表这个消息是日志文件里的第几条消息

但是在消费消息的时候也有一个所谓的offset，这个offset是代表消费者目前在partition日志文件里消费到了第几条消息，是两回事儿

### 5.Kafka是如何通过精心设计消息格式节约磁盘空间占用开销的？

kafka的消息格式如下：

crc32，magic，attribute，时间戳，key长度，key，value长度，value

kafka是直接通过NIO的ByteBuffer以二进制的方式来保存消息的，这种二级制紧凑保存格式可以比使用Java对象保存消息要节约40%的内存空间

然后这个消息实际上是封装在一个log entry里的，你可以认为是一个日志条目吧，在kafka里认为每个partition实际上就是一个磁盘上的日志文件，写到parttion里去的消息就是一个日志，所以log entry就是一个日志

这个日志条目包含了一个offset，一个消息的大小，然后是消息自身，就是上面那个数据结构，但是这里要注意的一点，就是这个message里可能会包含多条消息压缩在一起，所以可能找一条消息，需要从这个压缩数据里遍历搜索

而且这里还有一个概念就是消息集合，一个消息集合里包含多个日志，最新名称叫做RecordBatch

后来消息格式演化为了如下所示：

（1）消息总长度

（2）属性：废弃了，已经不用

（3）时间戳增量：跟RecordBatch的时间戳的增量差值

（4）offset增量：跟RecordBatch的offset的增量差值

（5）key长度

（6）key

（7）value长度

（8）value

（9）header个数

（10）header：自定义的消息元数据，key-value对

通过时间戳、offset、key长度等都用可变长度来尽可能减少空间占用，v2版本的数据格式比v1版本的数据格式要节约很多磁盘开销

### 6.如何实现TB量级的数据在Kafka集群中分布式的存储？

有一个很大的问题，就是不可能说把TB量级的数据都放在一台Kafka服务器上吧？这样肯定会遇到容量有限的问题，所以Kafka是支持分布式存储的，也就是说你的一个topic，代表了逻辑上的一个数据集

你大概可以认为一个业务上的数据集合吧，比如说用户行为日志都走一个topic，数据库里的每个表的数据分别是一个topic，订单表的增删改的变更记录进入一个topic，促销表的增删改的变更记录进入一个topic

每个topic都有很多个partition，你认为是数据分区，或者是数据分片，大概这些意思都可以，就是说这个topic假设有10TB的数据量需要存储在磁盘上，此时你给他分配了5个partition，那么每个partition都可以存放2TB的数据

然后每个partition不就可以放在一台机器上，通过这个方式就可以实现数据的分布式存储了，每台机器上都运行一个Kafka的进程，叫做Broker，以后大家记住，borker就是一个kafka进程，在一台服务器上就可以了

### 7.如何基于多副本冗余机制保证Kafka宕机时还具备高可用性？

这里就有一个问题了，如果此时Kafka某台机器宕机了，那么一个topic就丢失了一个partition的数据，此时不就导致数据丢失了吗？所以啊，所以对数据做多副本冗余，也就是每个parttion都有副本

比如最基本的就是每个partition做一个副本，副本放在另外一台机器上

然后呢kafka自动从一个partition的多个副本中选举出来一个leader partition，这个leader partition就负责对外提供这个partiton的数据读写，接收到写过来的数据，就可以把数据复制到副本partition上去

这个时候如果说某台机器宕机了，上面的leader partition没了，此时怎么办呢？通过zookeeper来维持跟每个kafka的会话，如果一个kafka进程宕机了，此时kafka集群就会重新选举一个leader partition，就是用他的某个副本partition即可

通过副本partition可以继续体统这个partition的数据写入和读取，这样就可以实现容错了，这个副本partition的专业术语叫做follower partition，所以每个partitino都有多个副本，其中一个是leader，是选举出来的，其他的都是follower partition

多副本冗余的机制，就可以实现Kafka高可用架构

### 8.保证写入Kafka的数据不丢失：ISR机制到底是什么意思？

光是依靠多副本机制能保证Kafka的高可用性，但是能保证数据不丢失吗？不行，因为如果leader宕机，但是leader的数据还没同步到follower上去，此时即使选举了follower作为新的leader，当时刚才的数据已经丢失了

ISR是：in-sync replica，就是跟leader partition保持同步的follower partition的数量，只有处于ISR列表中的follower才可以在leader宕机之后被选举为新的leader，因为在这个ISR列表里代表他的数据跟leader是同步的

如果要保证写入kafka的数据不丢失，首先需要保证ISR中至少有一个follower，其次就是在一条数据写入了leader partition之后，要求必须复制给ISR中所有的follower partition，才能说代表这条数据已提交，绝对不会丢失，这是Kafka给出的承诺

### 9.如何让Kafka集群处理请求的时候实现负载均衡的效果？

假如说很多partition的leader都在一台机器上，那么不就会导致大量的客户端都请求那一台机器？这样是不对的，kafka集群会自动实现负载均衡的算法，尽量把leader partition均匀分布在集群各个机器上

然后客户端在请求的时候，就会尽可能均匀的请求到kafka集群的每一台机器上去了，假如出现了partition leader的变动，那么客户端会感知到，然后下次就可以就可以请求最新的那个leader partition了

### 10.Partition的几个核心offset：高水位offset、LEO代表了什么？

实际上来说，每次leader接收到一条消息，都会更新自己的LEO，也就是log end offset，把最后一位offset + 1，这个大家都能理解吧？接着各个follower会从leader请求同步数据，这是持续进行的

offset = 0 ~ offset = 4，LEO = 5，代表了最后一条数据后面的offset，下一次将要写入的数据的offset，LEO，你一定要明白他的名词

然后follower同步到数据之后，就会更新自己的LEO

并不是leader主动推送数据给follower，他实际上是follower主动向leader尝试获取数据，不断的发送请求到leader来fetch最新的数据

然后对于接收到的某一条数据，所有follower的LEO都更新之后，leader才会把自己的HW（High Water Mark）高水位offset + 1，这个高水位offset表示的就是最新的一条所有follower都同步完成的消息

partition中最开始的一条数据的offset是base offset

LEO和HW分别是干什么的呢？

LEO很重要的一个功能，是负责用来更新HW的，就是如果leader和follower的LEO同步了，此时HW就可以更新

所有对于消费者来说，他只能看到base offset到HW offset之间的数据因为只有这之间的数据才表明是所有follower都同步完成的，这些数据叫做“已提交”的，也就是committed，是可以被消费到的

HW offset到LEO之间的数据，是“未提交的”，这时候消费者是看不到的

HW offset表示的是当前已经提交的数据offset，LEO表示的是下一个要写入的数据的offset

### 11.Leader与Follower上的LEO是如何更新的？

首先leader接收到数据字后就会更新自己的LEO值

接着follower会不断的向leader发送fetch请求同步数据，然后每次一条数据同步到follower之后，他的LEO就会更新，同时leader发送数据给follower的时候，在leader端会维护所有follower的LEO值

follower发送fetch请求给leader的时候会带上自己的LEO值，然后leader每次收到一个fetch请求就会更新自己维护的每个follower的LEO值

所以这里大家要知道的是，leader上是会保存所有follower的LEO值的，这个是非常关键和核心的一点

### 12.Leader与Follower上的高水位offset是如何更新的？

每次leader发送数据给follower的时候，都会发送自己的HW值，然后follower获取到leader HW之后，就会跟自己的LEO比较一下，取里面小的那个值作为自己的HW值，换句话说，如果follower的LEO比leader HW大了，那么follower的HW就是leader HW

但是如果follower的LEO比leader HW小，说明自己明显落后于leader，那么follower的HW就是自己的LEO值

然后leader上的HW就很明显了，那就是主要是他在接收follower的fetch请求的时候，就会在更新自己维护的所有follower的LEO之后，判断一下当前自己的LEO是否跟所有follower都保持一致，那么就会自动更新自己的HW值

这个leader的HW值就是partition的HW值，代表了从这个partition的哪个offset之前可以被消费数据

### 13.Leader与Follower的LEO与高水位如何更新

假设leader收到第一条数据，此时leader LEO = 1，HW = 0，因为他发现其他follower的LEO也是0，所以HW必须是0

接着follower来发送fetch请求给leader同步数据，带过去follower的LEO = 0，所以leader上维护的follower LEO = 0，更新了一下，此时发现follower的LEO还是0，所以leader的HW继续是0

接着leader发送一条数据给follower，这里带上了leader的HW = 0，因为发现leader的HW = 0，此时follower LEO更新为1，但是follower HW = 0，取leader HW

接着下次follower再次发送fetch请求给leader的时候，就会带上自己的LEO = 1，leader更新自己维护的follower LEO = 1，此时发现follower跟自己的LEO同步了，那么leader的HW更新为1

接着leader发送给follower的数据里包含了HW = 1，此时follower发现leader HW = 1，自己的LEO = 1，此时follower的HW有更新为1

5个数据：全部都要往前推进更新，需要2次请求，第一次请求是仅仅是更新两边的LEO，第二次请求是更新另外leader管理的follower LEO，以及两个HW

### 14.高水位机制可能导致leader切换时发生数据丢失问题

基于之前说的高水位机制，可能会导致一些问题，比如数据丢失

假如说生产者的min.insync.replicas设置为1，这个就会导致说生产者发送消息给leader，leader写入log成功后，生产者就会认为写成功了，此时假设生产者发送了两条数据给leader，leader写成功了

此时leader的LEO = 1，HW = 0，因为follower还没同步，HW肯定是0

接着follower发送fetch请求，此时leader发现follower LEO = 0，所以HW还是0，给follower带回去的HW也是0，然后follower开始同步数据也写入了两条数据，自己的LEO = 1，但是HW = 0，因为leader HW为0

接着follower再次发送fetch请求过来，自己的LEO = 1，leader发现自己LEO = 1，follower LEO = 1，所以HW更新为1，同时会把HW = 1带回给follower，但是此时follower还没更新HW的时候，HW还是0

这个时候假如说follower机器宕机了，重启机器之后，follower的LEO会自动被调整为0，因为会依据HW来调整LEO，而且自己的那两条数据会被从日志文件里删除，数据就没了

这个时候如果leader宕机，就会选举follower为leader，此时HW = 0，接着leader那台机器被重启后作为follower，这个follower会从leader同步HW是0，此时会截断自己的日志，删除两条数据

这种场景就会导致数据的丢失

非常极端的一个场景，数据可能会莫名其妙的丢失

### 15.高水位机制可能导致leader切换时发生数据不一致问题

假设min.insync.replicas = 1，那么只要leader写入成功，生产者而就会认为写入成功

如果leader写入了两条数据，但是follower才同步了一条数据，第二条数据还没同步，假设这个时候leader HW = 2，follower HW = 1，因为follower LEO小于leader HW，所以follower HW取自己的LEO

这个时候如果leader挂掉，切换follower变成leader，此时HW = 1，就一条数据，然后生产者又发了一条数据给新leader，此时HW变为2，但是第二条数据是新的数据。接着老leader重启变为follower，这个时候发现两者的HW都是2

所以他们俩就会继续运行了

这个时候他们俩数据是不一致的，本来合理的应该是新的follower要删掉自己原来的第二条数据，跟新leader同步的，让他们俩的数据一致，但是因为依赖HW发现一样，所以就不会截断数据了

### 16.Kafka为Partition维护ISR列表的底层机制是如何设计的？

很多公司比较常用的一个kafka的版本，是0.8.2.x系列，这个系列的版本是非常经典的，在过去几年相当大比例的公司都是用这个版本的kafka。当然，现在很多公司开始用更新版本的kafka了，就是0.9.x或者是1.x系列的

我们先说说在0.9.x之前的版本里，这个kafka到底是如何维护ISR列表的，什么样的follower才有资格放到ISR列表里呢？

在之前的版本里，有一个核心的参数：replica.lag.max.messages。这个参数就规定了follower如果落后leader的消息数量超过了这个参数指定的数量之后，就会认为follower是out-of-sync，就会从ISR列表里移除了

咱们来举个例子好了，假设一个partition有3个副本，其中一个leader，两个follower，然后replica.lag.max.messages = 3，刚开始的时候leader和follower都有3条数据，此时HW和LEO都是offset = 2的位置，大家都同步上来了

现在来了一条数据，leader和其中一个follower都写入了，但是另外一个follower因为自身所在机器性能突然降低，导致没及时去同步数据，follower所在机器的网络负载、内存负载、磁盘负载过高，导致整体性能下降了，此时leader partition的HW还是offset = 2的位置，没动，但是LEO变成了offset = 3的位置

依托LEO来更新ISR的话，在每个follower不断的发送Fetch请求过来的时候，就会判断leader和follower的LEO相差了多少，如果差的数量超过了replica.lag.max.messages参数设置的一个阈值之后，就会把follower给踢出ISR列表

但是这个时候第二个follower的LEO就落后了leader才1个offset，还没到replica.lag.max.messages = 3，所以第二个follower实际上还在ISR列表里，只不过刚才那条消息没有算“提交的”，在HW外面，所以消费者是读不到的

而且这个时候，生产者写数据的时候，如果默认值是要求必须同步所有follower才算写成功的，可能这个时候会导致生产者一直卡在那儿，认为自己还没写成功，这个是有可能的

一共有3个副本，1个leaderr，2个是follower，此时其中一个follower落后，被ISR踢掉了，ISR里还有2个副本，此时一个leader和另外一个follower都同步成功了，此时就可以让那些卡住的生产者就可以返回，认为写数据就成功了

min.sync.replicas = 2，ack = -1，生产者要求你必须要有2个副本在isr里，才可以写，此外，必须isr里的副本全部都接受到数据，才可以算写入成功了，一旦说你的isr副本里面少于2了，其实还是可能会导致你生产数据被卡住的

假设这个时候，第二个follower fullgc持续了几百毫秒然后结束了，接着从leader同步了那条数据，此时大家LEO都一样，而且leader发现所有follower都同步了这条数据，leader就会把HW推进一位，HW变成offset = 3

这个时候，消费者就可以读到这条在HW范围内的数据了，而且生产者认为写成功了

但是要是此时follower fullgc一直持续了好几秒钟，此时其他的生产者一直在发送数据过来，leader和第一个follower的LEO又推进了2位，LEO offset = 5，但是HW还是停留在offset = 2，这个时候HW后面的数据都是消费不了的，而且HW后面的那几条数据的生产者可能都会认为写未成功

现在导致第二个follower的LEO跟leader的LEO差距超过3了，此时触发阈值，follower认为是out-of-sync，就会从ISR列表里移除了

一旦第二个follower从ISR列表里移除了，世界清静了，此时ISR列表里就leader和第一个follower两个副本了，此时leader和第一个follower的LEO都是offset = 5，是同步的，leader就会把HW推进到offset = 5，此时消费者就可以消费全部数据了，生产者也认为他们的写操作成功了

那如果第二个follower后来他的fullgc结束了，开始大力追赶leader的数据，慢慢LEO又控制在replica.lag.max.messages限定的范围内了，此时follower会重新加回到ISR列表里去

上面就是ISR的工作原理和机制，一般导致follower跟不上的情况主要就是以下三种：

（1）follower所在机器的性能变差，比如说网络负载过高，IO负载过高，CPU负载过高，机器负载过高，都可能导致机器性能变差，同步 过慢，这个时候就可能导致某个follower的LEO一直跟不上leader，就从ISR列表里移除了

我们生产环境遇到的一些问题，kafka，机器层面，某台机器磁盘坏了，物理机的磁盘有故障，写入性能特别差，此时就会导致follower，CPU负载太高了，线程间的切换太频繁了，CPU忙不过来了，网卡被其他的程序给打满了，就导致网络传输的速度特别慢

（2）follower所在的broker进程卡顿，常见的就是fullgc问题

kafka自己本身对jvm的使用是很有限的，生产集群部署的时候，他主要是接收到数据直接写本地磁盘，写入os cache，他一般不怎么在自己的内存里维护过多的数据，主要是依托os cache（缓存）来提高读和写的性能的

（3）kafka是支持动态调节副本数量的，如果动态增加了partition的副本，就会增加新的follower，此时新的follower会拼命从leader上同步数据，但是这个是需要过程的，所以此时需要等待一段时间才能跟leader同步

replica.lag.max.messages主要是解决第一种情况的，还有一个replica.lag.time.max.ms是解决第二种情况的，比如设置为500ms，那么如果在500ms内，follower没法送请求找leader来同步数据，说明他可能在fullgc，此时就会从ISR里移除

### 17.Kafka在磁盘上是如何采用分段机制保存日志的？

每个分区对应的目录，就是“topic-分区号”的格式，比如说有个topic叫做“order-topic”，那么假设他有3个分区，每个分区在一台机器上，那么3台机器上分别会有3个目录，“order-topic-0”，“order-topic-1”，“order-topic-2”

每个分区里面就是很多的log segment file，也就是日志段文件，每个分区的数据会被拆分为多个段，放在多个文件里，每个文件还有自己的索引文件，大概格式可能如下所示：

00000000000000000000.index

00000000000000000000.log

00000000000000000000.timeindex



00000000000005367851.index

00000000000005367851.log

00000000000005367851.timeindex



00000000000009936472.index

00000000000009936472.log

00000000000009936472.timeindex

这个9936472之类的数字，就是代表了这个日志段文件里包含的起始offset，也就说明这个分区里至少都写入了接近1000万条数据了

kafka broker有一个参数，log.segment.bytes，限定了每个日志段文件的大小，最大就是1GB，一个日志段文件满了，就自动开一个新的日志段文件来写入，避免单个文件过大，影响文件的读写性能，这个过程叫做log rolling

正在被写入的那个日志段文件，叫做active log segment

### 18.引入索引文件之后如何基于二分查找快速定位数据？

日志段文件，.log文件会对应一个.index和.timeindex两个索引文件

kafka在写入日志文件的时候，同时会写索引文件，就是.index和.timeindex，一个是位移索引，一个是时间戳索引，是两种索引

默认情况下，有个参数log.index.interval.bytes限定了在日志文件写入多少数据，就要在索引文件写一条索引，默认是4KB，写4kb的数据然后在索引里写一条索引，所以索引本身是稀疏格式的索引，不是每条数据对应一条索引的

而且索引文件里的数据是按照位移和时间戳升序排序的，所以kafka在查找索引的时候，会用二分查找，时间复杂度是O(logN)，找到索引，就可以在.log文件里定位到数据了

.index

44576 物理文件（.log位置）

57976 物理文件（.log位置）

64352 物理文件（.log位置）

offset = 58892 => 57976这条数据对应的.log文件的位置

接着就可以从.log文件里的57976这条数对应的位置开始查找，去找offset = 58892这条数据在.log里的完整数据

.timeindex是时间戳索引文件，如果要查找某段时间范围内的时间，先在这个文件里二分查找找到offset，然后再去.index里根据offset二分查找找对应的.log文件里的位置，最后就去.log文件里查找对应的数据

### 19.磁盘上的日志文件是按照什么策略定期清理腾出空间的？

大家可以想，不可能说每天涌入的数据都一直留存在磁盘上，本质kafka是一个流式数据的中间件，不需要跟离线存储系统一样保存全量的大数据，所以kafka是会定期清理掉数据的，这里有几个清理策略

kafka默认是保留最近7天的数据，每天都会把7天以前的数据给清理掉，包括.log、.index和.timeindex几个文件，log.retention.hours参数，可以自己设置数据要保留多少天，你可以根据自己线上的场景来判断一下

只要你的数据保留在kafka里，你随时可以通过offset的指定，随时可以从kafka楼出来几天之前的数据，数据回放一遍，下游的数据，有多么的重要，如果是特别核心的数据，在kafka这个层面，可以保留7天，甚至是15天的数据

下游的消费者消费了数据之后，数据丢失了，你需要从kafka里楼出来3天前的数据，重新来回放处理一遍

在大数据的实时分析的项目里，其实就会涉及到这个东西的一个使用，如果你今天实时分析的一些数据出错了，此时你就需要把过去几天的数据重新楼出来回放一遍，重新来算一遍。实时数据分析的结果和hadoop离线分析的结果做一个比对

你每天都会从kafka里搂出来几天前的数据，算一下，跟离线数据的结果做一个比对

kafka broker会在后台启动线程异步的进行日志清理的工作

### 20.Kafka是如何自定义TCP之上的通信协议以及使用长连接通信的

kafka的通信主要发生于生产端和broker之间，broker和消费端之间，broker和broker之间，这些通信都是基于TCP协议进行的，大家自己看看网络课程，底层基于TCP连接和传输数据，应用层的协议，是Kafka自己自定义的

所谓自定义协议，就是定好传输数据的格式，请求格式、响应格式，这样大家就可以统一按照规定好的格式来封装、传输和解析数据了

生产端发送数据到kafka broker来，此时发送的数据是这样子的：

sent data: 一大串数据

kafka broker直接就从sent data:截取一大段数据就可以用了，如果你没有自定义一套完整的协议，是没办法进行通信的

http协议，生产端也可以发送http协议的数据给kafka broker，http请求，http响应。应用层的协议，规定了数据请求和响应的种种复杂的格式，大家全部按照这个格式和规范来走，不要乱来

request v1.1

isCache: true

一大串数据

对于生产端和broker，消费端和broker来说，还会基于TCP建立长连接（具体见网络课程），也就是维护一批长连接，然后通过固定的连接不断的传输数据，避免频繁的创建连接和销毁连接的开销

broker端会构造一个请求队列，然后不停的获取请求放入队列，后台再搞一堆的线程来获取请求进行处理

### 21.roker是如何基于Reactor模式进行多路复用请求处理的？

每个broker上都有一个acceptor线程和很多个processor线程，可以用num.network.threads参数设置processor线程的数量，默认是3，client跟一个broker之间只会创建一个socket长连接，他会复用

然后broker就用一个acceptor来监听每个socket连接的接入，分配这个socket连接给一个processor线程，processor线程负责处理这个socket连接，监听socket连接的数据传输以及客户端发送过来的请求，acceptor线程会不停的轮询各个processor来分配接入的socket连接

proessor需要处理多个客户端的socket连接，就是通过java nio的selector多路复用思想来实现的，用一个selector监听各个socket连接，看其是否有请求发送过来，这样一个processor就可以处理多个客户端的socket连接了

processor线程会负责把请求放入一个broker全局唯一的请求队列，默认大小是500，是queued.max.requests参数控制的，所以那几个processor会不停的把请求放入这个请求队列中

接着就是一个KafkaRequestHandler线程池负责不停的从请求队列中获取请求来处理，这个线程池大小默认是8个，由num.io.threads参数来控制，处理完请求后的响应，会放入每个processor自己的响应队列里

每个processor其实就是负责对多个socket连接不停的监听其传入的请求，放入请求队列让KafkaRequestHandler来处理，然后会监听自己的响应队列，把响应拿出来通过socket连接发送回客户端

### 22.如何对Kafka集群进行整体控制：Controller是什么东西？

不知道大家有没有思考过一个问题，就是Kafka集群中某个broker宕机之后，是谁负责感知到他的宕机，以及负责进行Leader Partition的选举？如果你在Kafka集群里新加入了一些机器，此时谁来负责把集群里的数据进行负载均衡的迁移？

包括你的kafka集群的各种元数据，比如说每台机器上有哪些partition，谁是leader，谁是follower，是谁来管理的？如果你要删除一个topic，那么背后的各种partition如何删除，是谁来控制？

还有就是比如kafka集群扩容加入一个新的broker，是谁负责监听这个broker的加入？如果某个broker崩溃了，是谁负责监听这个broker崩溃？

这里就需要一个kafka集群的总控组件，Controller。他负责管理整个kafka集群范围内的各种东西

### 23.如何基于Zookeeper实现Controller的选举以及故障转移

在kafka集群启动的时候，会自动选举一台broker出来承担controller的责任，然后负责管理整个集群，这个过程就是说集群中每个broker都会尝试在zk上创建一个/controller临时节点

zk的一些基础知识和临时节点是什么，百度一下zookeeper入门

但是zk会保证只有一个人可以创建成功，这个人就是所谓controller角色

一旦controller所在broker宕机了，此时临时节点消失，集群里其他broker会一直监听这个临时节点，发现临时节点消失了，就争抢再次创建临时节点，保证有一台新的broker会成为controller角色

### 24.创建Topic时Kafka Controller是如何完成Leader选举的呢？

如果你现在创建一个Topic，肯定会分配几个Partition，每个partition还会指定几个副本，这个时候创建的过程中就会在zookeeper中注册对应的topic的元数据，包括他有几个partition，每个partition有几个副本，每个partition副本的状态，此时状态都是：NonExistentReplica

然后Kafka Controller本质其实是会监听zk上的数据变更的，所以此时就会感知到topic变动，接着会从zk中加载所有partition副本到内存里，把这些partition副本状态变更为：NewReplica，然后选择的第一个副本作为leader，其他都是follower，并且把他们都放到partition的ISR列表中

比如说你创建一topic，order_topic，3个partition，每个partition有2个副本，写入zk里去

/topics/order_topic

partitions = 3, replica_factor = 2

[partition0_1, partition0_2]

[partition1_1, partition1_2]

[partition2_1, partition2_2]

从每个parititon的副本列表中取出来第一个作为leader，其他的就是follower，把这些东西给放到partition对应的ISR列表里去

每个partition的副本在哪台机器上呢？会做一个均匀的分配，把partition分散在各个机器上面，通过算法来保证，尽可能把每个leader partition均匀分配在各个机器上，读写请求流量都是打在leader partition上的

同时还会设置整个Partition的状态：OnlinePartition

接着Controller会把这个partition和副本所有的信息（包括谁是leader，谁是follower，ISR列表），都发送给所有broker让他们知晓，在kafka集群里，controller负责集群的整体控制，但是每个broker都有一份元数据

### 25.删除Topic时又是如何通过Kafka Controller控制数据清理？

如果你要是删除某个Topic的话，Controller会发送请求给这个Topic所有Partition所在的broker机器，通知设置所有Partition副本的状态为：OfflineReplica，也就是让副本全部下线，接着Controller接续将全部副本状态变为：ReplicaDeletionStarted

然后Controller还要发送请求给broker，把各个partition副本的数据给删了，其实对应的就是删除磁盘上的那些文件，删除成功之后，副本状态变为：ReplicaDeletionSuccessful，接着再变为NonExistentReplica

而且还会设置分区状态为：Offline

### 26.每日10亿数据会对Kafka集群造成多大压力

电商平台假设有每日10亿数据，此时我们的生产环境的kafka集群应该如何来规划和部署

10亿数据，他对kafka集群造成的并发请求会有多少，24小时，电商平台，其中有12个小时，是承担了大部分的流量，比如说晚上12点过后，到第二天的早上8点，这之间可能没什么流量过来

16小时 -> 有数据的，但是其中高峰期，可能就20%的时间，`16 * 0.2 = 3小时

8亿，80%的数据会集中在16小时内涌入，8亿时间里的80%的请求会集中在3小时的高峰时间段，8 * 0.8 = 6.4亿数据

大概是每秒钟有6万请求，会涌入你的kafka集群，整体kafka的集群需要抗住高峰期每秒6万的QPS

kafka集群默认是保留最近7天的数据，假设我们就用默认的配置

每天是进来10亿条数据，但是每条数据做几个副本？每个topic都可以设置副本因子，平均都是2个副本，每天在kafka磁盘上需要保留10亿* 2 = 20亿条数据，集群磁盘空间需要保留最近7天的20亿 * 7 = 140亿条数据

每条数据大概有多大？算大点，平均每条数据是1kb，140亿kb -> 整个kafka集群的磁盘空间至少需要容纳13TB的数据，就是最近7天的数据量

### 27.Kafka集群生产规划：需要部署几台物理机？

10亿数据，高峰期6万QPS，集群容纳的数据量是13TB

部署Kafka，Hadoop，MySQL，大数据核心分布式系统，一般建议大家直接采用物理机，不建议用一些低配置的虚拟机自己瞎搞

QPS这个东西，不可能是说，你只要支撑6万QPS，你的集群就刚好支撑6万QPS就可以了。假如说你只要支撑6w QPS，2台物理机绝对绝对够了，单台物理机部署kafka支撑个几万QPS是没问题的

但是这里有一个问题，我们通常是建议，公司预算充足，尽量是让高峰QPS控制在集群能承载的总QPS的30%左右

你的kafka集群能承载的总QPS给他搞成20万~30万，是非常安全的

大体上来说，需要5~7台物理机来部署，基本上就很安全了，每台物理机要求吞吐量在每秒几万条数据就可以了，物理机的配置和性能也不需要特别高

硬盘容量这块，每台物理机硬盘空间起码有个几个TB，所以如果是6台物理机的话，起码可以支撑个十几TB的数据，你到底需要不需要保留最近7天的数据，也可以根据你的需求场景减小这个时间间隔

保留最近3天的数据，6TB左右

如果你kafka集群的硬盘容量总共加起来有个十几TB，或者是20TB，都可以来支撑

### 28.Kafka集群生产规划：到底该不该用SSD？

10亿，6w/s的吞吐量，12TB的数据量，6台物理机，2块1TB的SAS盘

CPU、内存、硬盘、网路、参数，各种环节应该如何来规划

SSD固态硬盘，还是普通机械硬盘

比如说我们在规划和部署线上系统的MySQL集群的时候，一般来说必须用SSD性能可以提高很多，MySQL可以承载的并发请求量也会高很多，而且SQL语句执行的性能也会提高很多

SSD，固态硬盘，快，比机械硬盘要快，到底是快在哪里呢？快主要是快在磁盘随机读写，就要对磁盘上的随机位置来读写的时候，SSD比机械硬盘要快。像比如说是MySQL这种系统，就应该使用SSD了

Kafka集群，物理机是用昂贵的SSD呢？还是用普通的机械硬盘呢？写磁盘的时候，他是怎么来写的？磁盘顺序写的，压测，机械硬盘顺序写的性能机会跟内存读写的性能是差不多的，SSD做顺序写的性能

SSD主要是随机读写的性能要比机械硬盘随机读写的性能高很多倍

他在使用硬盘的时候，最核心的一个思想就是仅仅允许追加数据在每个日志段文件的末尾，不支持在磁盘文件里随机写，磁盘随机写

其实就没必要使用SSD了，会贵很多，增加很大的机器成本；顺序写，每台物理机给配置个常规的2块机械硬盘就可以，就足够了

每天多少条消息，每条消息几个备份，保留几天的数据，总共需要多大容量，对应到每台机器需要多大容量

### 29.Kafka集群生产规划：硬盘空间应该给多大？

10亿数据，6w QPS，12TB，6台物理机，2块1TB的机械硬盘

最近7天一共是12TB的数据量的角度而言，大概就是每台物理机搞2块机械硬盘，每块盘就是1TB好像就够了

很多是从mysql同步的binlog数据过来的，那种数据很多是到不了1kb的，几十个字节，几百个字节，也算是比较大了，用户行为日志，也不一定就可以到达1kb一条，一两百个字节，具体看你们公司采集的数据，看一看，估算一下平均每条数据有多大

有一个经验值，1亿条用户行为日志，也就几个GB，几个TB，也就差不多了

未来给机器扩容硬盘也是可以的，继续加硬盘，或者把小容量的硬盘换成大容量的盘，运维工程师会协助你来做的，偏硬件一些的，给服务器加硬盘，挂载，等等，诸如此类的一些事情

很多公司，每天其实就几千万条数据采集到大数据平台里来的话，根本不算是什么大数据，几百MB，1GB以内，没多少数据量

### 30.Kafka集群生产规划：充分利用os cache提高性能

10亿数据，6w QPS，12TB，6台物理机，2块1TB的机械硬盘

接下来看看，内存这块到底应该怎么来弄，kafka部署的时候内存的规划是很有讲究的，你一定要明白他底层的原理，才能知道应该如何来规划kafka的内存空间

kafka写数据基本上优先让数据写入os cache，后面来消费数据的时候，就可以从os cache里读取数据出来，写和读基本上都是依托于操作系统的缓存来执行的，我们在规划这个内存空间的时候

kafka本身是scala语言开发的，是基于jvm的，jvm代表了进程使用的内存空间，是多少，jvm堆内存，机器上留给os cache的内存应该有多大。kafka jvm堆内存，你觉得需要很大吗？你其实听懂了kafka原理之后

kafka并没有在自己的jvm堆内存里放入过多的数据，rabbitmq是不一样的，数据过来优先写入jvm堆内存里去缓冲一下，一定时间之后，rabbitmq再一次性把jvm堆内存里的缓冲一批数据给刷入磁盘中

导致jvm堆内存中存放大量的数据，需要给jvm堆内存开辟比较大的空间了

但是kafka正好相反的，他接收到了这个数据之后不是直接写jvm堆内存的，而是采用自己的二进制紧凑的数据格式，给写入到磁盘文件里去，是先写入os cache（操作系统管理的一块内存缓冲空间）

kafka并没有使用过多的jvm堆内存空间，不需要给kafka jvm堆内存分配过大的空间，基本上来说几个G就够了

在kafka部署的机器上，你需要比较大的空间应该是os cache这块，如果这块空间越大，那么你其实就可以在os cache内存里存放越多的数据，就可以保证说，人家在消费的时候，就可以从os cache里去读取数据了

os cache这块空间起码要有多大，在底层磁盘文件这块，其实是每个分区的数据是分段存的，每个分区的数据有多个分段日志文件，人家消费的时候比较频繁的需要读取的，一般是最新的这个正在写入的分段日志文件

起码你应该让这台机器上的broker管理的每个parition的最新正在写的日志段文件的数据都可以驻留在os cache中，这样可以保证每个parition正在写的数据，最有可能被消费的数据，就可以直接从os cache里来读了

假设100个topic，每个topic是6个partition，每个partiton是2个副本，一共100 * 6 =600个leader partition，平均到6台机器上去，假设每台机器是放100个partition，每个partition的最新正在写的日志段文件的大小是默认的1GB

所以说单台机器上，最新的正在写的日志段文件的大小100个partition * 1GB的日志段文件 = 600GB。如果最佳的情况，单台机器可以有600GB的内存的给os cache的话，就可以每个partiton最新的数据都在os cache里，比如说一个日志段文件有1GB，可能都对应几千万条数据了

没必要说一定要几千万的数据都在os cache里才可以，日志段文件3000万条数据，其实人家消费比较频繁的，就是最近的10万条数据，2GB的数据可以保留在os cache中，最新写入的2GB的数据驻留在os cache中，就可以让人家从内存里消费到最新的数据了

给os cache可以分配的内存空间绝对是不止2GB，几十个GB，每个partition的最新日志段文件里的最新写入的10%的数据，都可以驻留在os cache中，让人家对最新的数据都可以从os cache里读取到

你必须保证最新的10%的数据都在os cache里，人家消费大部分的消费请求可以走os cache来读取就可以了

### 31.Kafka集群生产规划：内存空间如何规划？

kafka自身的jvm是用不了过多堆内存的，因为kafka设计就是规避掉用jvm对象来保存数据，避免频繁fullgc导致的问题，所以一般kafka自身的jvm堆内存，分配个6G左右就够了，剩下的内存全部留给os cache

32G以上的内存，尽量是64G~128G的内存空间

比如一台物理机，有64GB的内存空间，那么50多G都是给os cache的

然后要看看每个日志段的大小，每个分区都有一个日志段是当前正在写入的，这个大家都懂了，所以此时这个正在写入日志段的数据，就是消费者读取最频繁的数据，主要让这台机器上每个partition副本的正在写的日志段，都可以放在os cache里

那么基本上80%~90%的消费行为，都可以直接从os cache里读到数据了，如果你真的按照课程的思路和讲解，来推算和规划你的kafka生产集群，把的内存规划设置的很好的话，基本上可以做到

整个人家kafka的读写吞吐量都很大，性能很高，延迟很低

### 32.Kafka集群生产规划：为什么需要16核CPU？

10亿数据，6w QPS，12TB，6台物理机，2块1TB的机械硬盘，64GB内存（6GB给JVM，剩余给os cache），16核CPU

CPU规划，主要是看你的这个进程里会有多少个线程，线程主要是依托多核CPU来执行的，如果你的线程特别多，但是CPU核很少，就会导致你的CPU负载很高，会导致你的整体工作线程执行的效率不太高

acceptor线程负责去接入客户端的连接请求，但是他接入了之后其实就会把连接分配给多个processor，默认是3个，但是说实话一般生产环境的话呢 ，建议大家还是多加几个，整体可以提升kafka的吞吐量

比如说你可以增加到6个，或者是9个

另外就是负责处理请求的线程，是一个线程池，默认是8个线程，在生产集群里，建议大家可以把这块的线程数量稍微多加个2倍~3倍，其实都正常，比如说搞个16个工作线程，24个工作线程

他后台会有很多的其他的一些线程，比如说定期清理7天前数据的线程，Controller负责感知和管控整个集群的线程，后台线程在工作

每个broker可能都会有上百个起码，一两百个线程繁忙的在工作

CPU给到4核，一般来说几十个线程，在高峰期CPU几乎都快打满了，8核，也就能够比较宽裕的支撑几十个线程繁忙的工作，一般是建议16核，靠谱，基本上可以hold住一两百线程的工作

broker一般都会启动几十个甚至上百个线程，大家看过broker端的原理了，各种处理请求的线程，后台线程，几十个线程频繁工作，一般都建议是16核CPU，甚至32核CPU

### 33.Kafka集群生产规划：千兆网卡还是万兆网卡？

10亿数据，6w QPS，12TB，6台物理机，2块1TB的机械硬盘，64GB内存（6GB给JVM，剩余给os cache），16核CPU，千兆网卡

生产环境的机器规划，无论是MySQL、Redis、RocketMQ、Elasticsearch、Kafka、Hadoop、Flink，Yarn，规划的思路都是类似的，从技术本质和底层的原理出发来考虑，请求量有多大、数据量有多大、内存应该分配多大（底层工作机制）、线程数量有多少、网络数据传输量有多大

现在一般就是千兆网卡（1GB / s），还有万兆网卡（10GB / s）

你要计算一下，kafka集群之间，broker和broker之间是会做数据同步的，因为leader要同步数据到follower上去，他们是在不同的broker机器上的，broker机器之间会进行频繁的数据同步，传输大量的数据

每秒两台broker机器之间大概会传输多大的数据量？高峰期每秒大概会涌入6万条数据，每秒就是60mb/s的数据量，他会在broker之间来传输，你就算多估算一些，每秒大概也就传输个几百mb/s的数据，就足够了

针对千兆网卡来算一下，假如每台物理机的网卡带宽都是1GB / s

实际上每台机器能用的网卡的带宽还达不到极限，因为kafka只能用其中一部分的带宽资源，比如700mb / s，但是一般不能允许kafka占用这么多带宽，因为避免说占用这么多带宽，万一再多一点，就容易把网卡打满

所以说，一般限制kafka每秒带宽资源就是300mb / s，如果你给物理机使用的千兆网卡，那么其实他每秒最多传输数据是几百mb，是够了，几十mb，或者一两百mb，基本上都够了，可以足够传输

假如说现在每秒要传输7万条数据，每条数据是平均1kb，那么就是70mb / s，这样的话，是可以轻松抗住的

如果是万兆网卡，那么就更加轻松了

### 34.Kafka集群生产规划：线上机器典型配置

10亿数据，6w/s的吞吐量，12TB的数据量

6台物理机

硬盘：2块机械硬盘，每块1TB，7200转

内存：64GB，JVM分配6G，剩余的给os cache

CPU：16核

网络：千兆网卡

### 35.生产环境中的Kafka集群参数设置：内核参数

自己可能都或多或少哪怕是虚拟机环境的kafka都搭建过

broker.id=0，这个是每个broker都必须自己设置的一个唯一id

log.dirs，这个极为重要，kafka的所有数据就是写入这个目录下的磁盘文件中的，如果说机器上有多块物理硬盘，那么可以把多个目录挂载到不同的物理硬盘上，然后这里可以设置多个目录，这样kafka可以数据分散到多块物理硬盘，多个硬盘的磁头可以并行写，这样可以提升吞吐量

zookeeper.connect，同样很重要，这个是连接kafka底层的zookeeper集群的

listeners，这是broker监听客户端发起请求的端口号

unclean.leader.election.enable，默认是false，意思就是只能选举ISR列表里的follower成为新的leader，1.0版本后才设为false，之前都是true，允许非ISR列表的follower选举为新的leader

delete.topic.enable，默认true，允许删除topic

log.retention.hours，可以设置一下，要保留数据多少个小时，这个就是底层的磁盘文件，默认保留7天的数据，根据自己的需求来就行了

log.retention.bytes，这个设置如果分区的数据量超过了这个限制，就会自动清理数据，默认-1不按照这个策略来清理，这个一般不常用

min.insync.replicas，这个跟acks=-1配合起来使用，意思就是说必须要求ISR列表里有几个follower，然后acks=-1就是写入数据的时候，必须写入这个数量指定的follower才可以，一般是可以考虑如果一个leader两个follower，三个副本，那么这个设置为2，可以容忍一台机器宕机，保证高可用，但是写入数据的时候必须ISR里有2个副本，而且必须副本都写入才算成功

如果你就一个leader和follower，双副本，min.insync.replicas = 2。如果有一台机器宕机，导致follower没了，此时ISR列表里就一个leader的话，你就不能写入了，如果你只有一个副本了，此时还可以写入数据的话，就会导致写入到leader之后，万一leader也宕机了，此时数据必然丢失

一般来说为了避免数据占用磁盘空间过大，一般都是kafka设置双副本，一个leader和一个follower就够了，设置的三副本的话，数据在集群里乘以3倍的空间来存放，非常耗费磁盘空间，对你的整体集群的性能消耗也会更大

min.insync.replicas=1，acks=-1（一条数据必须写入ISR里所有副本才算成功），你写一条数据只要写入leader就算成功了，不需要等待同步到follower才算写成功。但是此时如果一个follower宕机了，你写一条数据到leader之后，leader也宕机，会导致数据的丢失。

一般来说双副本的场景下，我们为了避免数据丢失，min.insync.replicas=2，acks=-1，每次写入必须保证ISR里2个副本都写成功才可以，如果其中一个副本没了，会导致你就没法写入，必须阻塞等待kafka恢复

这样就可以保证数据的不丢失

你是要计算为客户准备的财务数据报表，非常严格的数据必须精准的话，精准到每一分钱，你还是得设置成2

num.network.threads，这个是负责转发请求给实际工作线程的网络请求处理线程的数量，默认是3，高负载场景下可以设置大一些

num.io.threads，这个是控制实际处理请求的线程数量，默认是8，高负载场景下可以设置大一些

要自己做一些压测，在不同的线程数量的条件下，对整体的生产和消费的吞吐量分别是多少，还得看一下线程越多的时候，对CPU的复杂有多大，整体看一下，如果增加一些线程，吞吐量提升很多，而且CPU负载还能接受

message.max.bytes，这个是broker默认能接受的消息的最大大小，默认是977kb，太小了，可以设置的大一些，一般建议设置大一些，很多大消息可能大到几mb，这个设置个10mb，可以考虑

log.flush.interval.messages

log.flush.interval.ms

线上的常用的一个配合，高负载高吞吐的情况下，建议设置为1分钟

### 36.生产环境中的Kafka集群参数设置：JVM以及GC参数

修改bin/kafka-start-server.sh中的jvm设置

export KAFKA_HEAP_OPTS=”-Xmx6g -Xms6g -XX:MetaspaceSize=96m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80”

### 37.生产环境中的Kafka集群参数设置：操作系统参数

文件描述符限制：kafka会频繁的创建和修改文件，大约是分区数量 * （分区总大小 / 日志段大小） * 3，比如一个broker上大概有100个分区，每个分区大概是10G的数据，日志段大小是默认的1G，那么就是100 * 10（分区里的日志段文件数量） * 3 = 3000个文件

ulimit -n 100000，这个可以设置很大的描述符限制，允许创建大量的文件

磁盘flush时间：默认是5秒，os cache刷入磁盘，可以设置为几分钟，比如1分钟才刷入磁盘，这样可以大幅度提升单机的吞吐量

sysctl -a | grep dirty

vm.dirty_background_bytes = 0

vm.dirty_background_ratio = 5

vm.dirty_bytes = 0

vm.dirty_expire_centisecs = 3000

vm.dirty_ratio = 10

vm.dirty_writeback_centisecs = 500

vm.dirty_writeback_centisecs：每隔5秒唤醒一次刷磁盘的线程

vm.dirty_expire_centisecs：os cache里的数据在3秒后会被刷入磁盘

可以设置大一些，1分钟~2分钟都可以，特别大规模的企业和公司，如果你可以接收机器宕机的时候，数据适当可以丢失一些，kafka里的数据可以适当丢失一些，但是为了提升集群的吞吐量的话

大数据分析类的应用的话，主要是出一些数据报表，不要求数据100%精准的话，允许适当丢一些数据，此时的话，这里给他调大一些是没问题的

### 38. 在生产环境中如何正确的启动和关闭Kafka集群？

如果你是作为一个大数据平台工程师，你负责在生产环境部署一套kafka集群，选择对应的物理机，在物理机上部署kafka就可以了，接下来其实你作为一个平台工程师，你要做到五大块的事情：

（1）你需要去支持业务团队的需求：Topic的创建、管理、扩容

（2）对kafka集群进行监控：机器监控->broker监控->JVM监控，方方面面的监控

（3）日常运维和管理的：集群扩容、版本升级、管理工作

（4）从生产消息->broker处理消息->消费消息，整个链路都可能会出问题，异常报错，kafka技术大牛，精通这个技术的人，你需要能够处理kafka相关的所有的异常报错

（5）对常见一些生产技术方案，解决一些需求和问题：事务、消息幂等、顺序、0丢失、回溯、消息积压

集群如何启动，如何关闭，小的细节需要注意一下的，需要在每台机器上都依次去执行一个脚本来启动这台机器的broker，他启动之后，自然而然就会去注册自己到zk，controller就会发现这个broker

controller会负责去把最新的集群的元数据同步给所有的broker

JMX_PORT=9997 bin/kafka-server-start.sh -daemon /server.properties

一定要用后台方式来启动kafka，这里可以在启动的时候设置好JMX端口号，这个是用来后面对Kafka的数据进行监控的

bin/kafka-server-stop.sh

这个是关闭kafka的脚本，需要使用这个脚本来关闭

### 39.作为Kafka集群管理员如何对根据业务需求管理Topic？

手头已经有了一套生产集群了，都部署好了，压力测试过后集群整体抗个每秒几十万的请求都可以做到了，支撑几十TB的数据量都可以了

平时作为kafka平台工程师，你需要协助业务团队来维护topic，各个业务团队，小公司，主要就是一个后端技术团队，或者是大数据技术团队，你自己本身也是大数据技术团队里面的一个人，只不过你的leader专门让你来负责维护kafka集群

比如说你们团队里，有的哥儿们是做实时计算这块的，他需要把实时的数据流引入到kafka里去，接着他会需要去使用flink、spark streaming、storm，这种实时计算技术，来从kafka里消费数据，接着继续去运算，做实时的一些分析

风控技术团队，推荐技术团队，他们可能也需要将一些实时的数据流引入到kafka，或者是人家直接从你这里获取实时的数据流，进行风控，或者是实时的个性化推荐，可能性都有，所以他们就是你的业务方

bin/kafka-topics.sh --create --zookeeper localhost:2181 --partitions 6 --replication-factor 2 --topic test01

创建topic其实主要是指定这个topic的分区数量，具体来说你要综合考虑Kafka集群的整体规模，有几台机器，还有就是这个topic最近7天的数据量会有多大。举个例子，某个topic每天要放用户行为日志

每天大概是1GB的数据量，最近7天就需要保留7GB的数据量，还有两个副本因子，那么就是一共14GB的数据量。现在假设Kafka集群有4台物理机，你觉得应该给这个Topic分配多少个分区？

很明显了，就4个分区就足够了，每台机器会放一个leader partition，然后都有一个follower partition，这样这个topic的数据会均匀分散在4台机器上，而且对这个topic的读写请求都均匀分散在4台机器上了

假设说你对这个topic每秒写入并发是10万条，4个分区，写入的时候会写入4个分区，在4台机器上有leader partition，均匀的分散在4台机器上，每台机器其实就是大概每秒写入2.5万个请求

那如果你的Kafka物理集群有20台机器呢？这个时候初始分区数量可以设置为7，把7个分区分散在其中的7台机器上，这样的好处，就是对这个Topic的读写请求会均匀分散在7台机器上上，对Topic的读写吞吐量是不是会更高？

所以在创建Topic的时候，大家一定要考虑好这一点，至于说副本因子的话，其实一般就是2就足够了，因为双副本可以保证一定的数据容错性，哪怕一台机器宕机了，还有其他副本保证数据不丢失，可以继续使用

如果你设置为3副本，那当然更好了，最多允许2台机器宕机不会丢数据，但是问题是你的数据存储空间会triple一下，那你的机器资源就会耗费更多了，所以这里要权衡一下。如果希望数据严格不丢失

那么min.insync.replicas，那个参数可以设置为2，要求比如ISR里有2个副本，才能写入成功，acks=-1，就是必须写入2个副本才行，这样可以保证写成功了一条数据，绝对一般是数据不会丢的

对于大数据的场景，要求的是吞吐量，而不是数据不丢失

但是如果要求更高的吞吐量，那么就设置min.insync.replicas，这个参数就是设置为1就可以了，只要有1个副本就可以写入，写入1个副本就算写成功，也就是写入leader就够了，可能数据会丢失，但是吞吐量很高，因为写入不需要等待副本同步成功

通常情况下建议采取高吞吐的策略，不要为了数据0丢失牺牲掉吞吐量，除非是极为核心的业务数据，比如说广告计费的数据

bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test01，可以删除topic

如果要删除topic，需要设置delete.topic.enable为true，允许用户删除topic，删除之后是后台异步执行的，要过很长时间才能删除掉topic

bin/kafka-topics.sh --zookeeper localhost:2181 --list，可以查看topic列表，通过这个命令可以快速查看当前有哪些topic，还有就是可以看看要删除的topic删除掉了没有，平时这个命令管理员用的是很多的

bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test01，可以查看topic详情，这个命令其实相当的实用，因为平时肯定要经常看看一个topic的具体情况，每个partition的leader在哪台机器上，副本在哪些机器上，ISR列表

### 40.某个Topic的数据量太大了需要扩容partition怎么做

如果说某个topic一开始数据量很小，比如每天就几百MB的数据，那你不需要给他分配过多的分区，因为你要是搞10个分区，分散在10台机器上，每台机器就几十MB的数据，有什么意义呢

所以一般建议生产环境采取的策略是刚开始就是按照预估的数据量给合理的分区数即可，通常来说，你可以限制每个分区在几个GB到10个GB，或者最多20个GB左右，都可以，具体要看你们公司的Kafka集群的机器资源情况

那么如果后来业务不停的发展，发现topic数据量太大了，当前的几个分区所在机器有点压力了，需要扩容增加更多的topic呢？这个时候你就可以动态扩容分区了，这样就可以让新的数据慢慢分散到更多的分区，更多的机器上去

bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 10 --topic test-topic

### 41.如果某台broker机器承载了过多leader partition怎么办

现在各个业务方可以自行申请创建topic，分区数量都是自动分配和后续动态调整的，kafka本身会自动把leader partition均匀分散在各个机器上，这样可以保证每台机器的读写吞吐量都是均匀的

但是也有例外，那就是如果某些broker宕机，会导致leader partition过于集中在其他少部分几台broker上，这会导致少数几台broker的读写请求压力过高，其他宕机的broker重启之后都是follower partition，读写请求很低，造成集群负载不均衡

有一个参数，auto.leader.rebalance.enable，默认是true，每隔300秒（leader.imbalance.check.interval.seconds）会执行一次preferred leader选举，如果一台broker上的不均衡的leader超过了10%，leader.imbalance.per.broker.percentage，就会对这个broker进行选举

也可以手动执行，bin/kafka-preferred-replica-election.sh，但是不建议手动执行，让他自动执行就好了

topic创建自助的，分区扩容自动的，leader partition均匀分散也是自动的

### 42.应该如何对你的Kafka集群设计生产级监控方案？

如何支撑业务这块，基本上你就已经思路比较清晰了，其实接下来这个业务方就会大量的写入数据到kafka以及从kafka消费数据出去，对你来说，你是kafka数据平台的工程师，你需要干的事情就是监控好这个集群

作为一个kafka管理员，在部署好了生产集群之后，要干的一个非常核心的事情，就是对集群设计一整套监控方案，基于开源工具完成部署，每天都对集群进行监控，所以这里就要先搞明白需要监控哪些东西

（1）Broker集群的运行状态：包括每个broker上管理了多少个partition，有多大的数据量，磁盘使用情况，物理机的CPU、内存、磁盘、网络的监控，比如CPU负载，内存使用率，磁盘IO负载，网络IO负载，JVM GC的情况

（2）ZooKeeper集群的运行状态：同理，跟上面是类似的，主要是磁盘使用率，然后物理机的CPU、内存、磁盘、网络的监控，这个东西一般来说可以先排除在外，可能不是你同时在维护的zk集群

（3）Topic的状态：这个你需要每天知道集群里有多少Topic，每个Topic有多少分区，有多大数据量，每个分区的副本在哪些机器上，ISR列表，等等

（4）后台定是运行的作业：比如说partition负载均衡自动迁移数据，Leader partition的重新选举，等等

（5）Kafka、生产、消费、集群是否有异常报错日志

### 43.如何对Kafka Broker所在机器的CPU、内存、网络进行监控

一般来说如果你要对类似kafka这种集群进行监控，建议采用开源的工具，kafka-manager，kafka eagle，都可以使用，人家都做好了会通过图表的形式来展现出来你的broker所在机器的CPU、网络、磁盘、内存的一些负载

就是一个小的公司，也可以直接基于linux的命令进行监控

使用top命令就可以查看CPU的负载，使用free命令就可以查看内存的使用情况，使用iostat -d -x -k 1 5来查看磁盘的使用率

### 44.Kafka Broker有哪些核心的运行指标是值得去监控的？

下面的JMX指标，都可以基于JConsole去连接到kafka的JMX端口来监控查看，也可以基于kafka自己提供的一个工具来监控

bin/kafka-run-class.sh kafka.tools.JmxTool --object-name kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec --jmx-url service:jmx:rmi:///jndi/rmi://:9997/jmxrmi --date-format “YYYY-MM-dd HH:mm:ss” --attributes FifteenMinuteRate --reporting-interval 5000

上面的命令就是每隔5秒去监控过于15分钟的消息接收速率，kafka的JMX的指标，特别特别的多，官网去查一下，成百上千个JMX指标

（1）消息接收/发送速率：监控每个broker的负载压力

这个就是说leader broker接收消息的速率，以及follower broker接收消息的速率；还有就是leader broker发送给follower broker的速率

kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec,topic=test

（2）controller是否存活：监控controller是否存活

kafka.controller:type=KafkaController,name=ActiveControllerCount

（3）副本不足的分区数：某个分区的副本不足了

kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions

（4）leader parttion在各个broker上的数量：确认集群的各个broker负载是均衡的

kafka.server:type=ReplicaManager,name=LeaderCount

（5）ISR变化速率：监控ISR不能变化太快

kafka.server:type=ReplicaManager,name=isExpandsPerSec

kafka.server:type=ReplicaManager,name=IsrShrinksPerSec

（6）Broker IO工作线程空闲率

kafka.server:type=KafkaRequestHandlerPool,name=RequestHandlerAvgIdlePercent

（7）Broker网络处理线程空闲率

kafka.network:type=SocketServer,name=NetworkProcessorAvgIdlePercent

这个JMX指标是非常非常多的，大家可以自己上官网去找，平时作为一个运维人员，需要经常对核心的一些指标看一看，这样你可以对kafka集群当前的运行情况了解的很清晰，特别是在一些故障发生的时候

### 45.如何基于JMX对Kafka集群的JVM和GC进行监控？

kafka他的设计理念就是不要过度的依赖于通过jvm内存来进行数据管理，避免说过多的数据驻留在kafka jvm内存里面，会导致过多的gc

java.lang:type=GarbageCollector,name=G1 Old Generation

java.lang:type=GarbageCollector,name=G1 Young Generation

就是对垃圾回收进行监控

### 46.集群、业务、监控

（1）监控资源的使用率：对于kafka来说，就是磁盘资源的使用率，当前集群保存了多大的数据量；每台机器的吞吐量，JMX来看到，broker接收数据的速率，每秒接收多少条消息，接收多少字节的数据

如果说你本来一个集群有4台机器，总共可以放20TB的数据，随着磁盘资源的使用率的攀升，可能你们的业务在不断的增长，会可能导致你们的机器就不够了，因为马上20TB的磁盘资源就要用满了

就需要进行broker机器的扩容，多增加一些broker机器，集群有更多的磁盘空间

每个broker的吞吐量，如果最多每台机器每秒可以接收10万条消息，每秒接收500mb的数据，现在发现每台broker在高峰期的时候，吞吐量几乎都应快要打了，此时就说明你需要扩容了

就需要多增加一些broker机器，让每个broker所在的节点承载更小的并发请求量，保证集群的健康稳定的运行

（2）集群里的负载倾斜：如果说某几台broker承载的数据量特别大，或者是承载的吞吐量特别大，此时你就应该手动进行一下数据的迁移，可以把这些broker上的partition迁移一些到负载比较轻的机器上去

（3）异常报错、JVM GC频繁：解决报错的问题，那是非常考验你的技术功底的，Kafka源码有比较深入的研究，你后续才能在他就是说有报错的时候，可以去通过源码的分析，看他到底为什么报错

### 47.如何对Kafka集群进行动态扩容加入更多的Broker节点？

之前大家了解过原理了，现在都知道，其实扩容增加broker是很容易的，只要配置好这个broker，然后穷，他会自动在zk里加入自己的节点，然后这个时候Controller会感知到他的加入，同步给其他所有的broker

而且Controller还会自动把集群元数据同步给这个新的broker，就是有哪些topic，每个topic有多少partitioin，每个partition有几个副本，以及对应的ISR

但是刚启动的broker不会自动被负载均衡迁移一些partiton，需要手动进行partiton在集群里的负载均衡，这样让每台broker机器管理均匀的数据量，同时分配leader partition承载均匀的读写请求流量

比如说现在你有3个topic，每个topic有5个partition，每个partitino有2个副本，一共有partition副本数量有30个，比如原来有3台broker，每台broker有10个partition副本，现在假如了第四台broker了

partition概念，资源都集中在partition里面了，数据是在partition作为单位来存放的，读写负载也是针对leader partition来进行的

### 48.Kafka Producer怎么把消息发送给Broker集群的？

需要指定把消息发送到哪个topic去

首先需要选择一个topic的分区，默认是轮询来负载均衡，但是如果指定了一个分区key，那么根据这个key的hash值来分发到指定的分区，这样可以让相同的key分发到同一个分区里去，还可以自定义partitioner来实现分区策略

producer.send(msg); // 用类似这样的方式去发送消息，就会把消息给你均匀的分布到各个分区上去

producer.send(key, msg); // 订单id，或者是用户id，他会根据这个key的hash值去分发到某个分区上去，他可以保证相同的key会路由分发到同一个分区上去

知道要发送到哪个分区之后，还得找到这个分区的leader副本所在的机器，然后跟那个机器上的Broker通过Socket建立连接来进行通信，发送Kafka自定义协议格式的请求过去，把消息就带过去了

如果找到了partition的leader所在的broker之后，就可以通过socket跟那台broker建立连接，接着发送消息过去

Producer（生产者客户端），起码要知道两个元数据，每个topic有几个分区，每个分区的leader是在哪台broker上，会自己从broker上拉取kafka集群的元数据，缓存在自己client本地客户端上

kafka核心原理、集群部署、运维管理 -> 初步玩儿起来一套kafka集群

kafka使用者的层面来考虑一下，我如果要把数据写入kafka集群，应该如何来做，怎么把数据写入kafka集群，以及他背后的一些原理还有使用过程中需要设置的一些参数，到底应该怎么来弄

### 49.Producer发送消息的内部实现原理

每次发送消息都必须先把数据封装成一个ProducerRecord对象，里面包含了要发送的topic，具体在哪个分区，分区key，消息内容，timestamp时间戳，然后这个对象交给序列化器，变成自定义协议格式的数据

接着把数据交给partitioner分区器，对这个数据选择合适的分区，默认就轮询所有分区，或者根据key来hash路由到某个分区，这个topic的分区信息，都是在客户端会有缓存的，当然会提前跟broker去获取

接着这个数据会被发送到producer内部的一块缓冲区里

然后producer内部有一个Sender线程，会从缓冲区里提取消息封装成一个一个的batch，然后每个batch发送给分区的leader副本所在的broker

### 50.发送消息的缓冲区应该如何优化来提升发送的吞吐量？

buffer.memory：设置发送消息的缓冲区，默认值是33554432，就是32MB

如果发送消息出去的速度小于写入消息进去的速度，就会导致缓冲区写满，此时生产消息就会阻塞住，所以说这里就应该多做一些压测，尽可能保证说这块缓冲区不会被写满导致生产行为被阻塞住

compression.type，默认是none，不压缩，但是也可以使用lz4压缩，效率还是不错的，压缩之后可以减小数据量，提升吞吐量，但是会加大producer端的cpu开销

### 51.消息批量发送的核心参数batch.size是如何优化吞吐量？

batch.size，设置meigebatch的大小，如果batch太小，会导致频繁网络请求，吞吐量下降；如果batch太大，会导致一条消息需要等待很久才能被发送出去，而且会让内存缓冲区有很大压力，过多数据缓冲在内存里

默认值是：16384，就是16kb，也就是一个batch满了16kb就发送出去，一般在实际生产环境，这个batch的值可以增大一些来提升吞吐量，可以自己压测一下

还有一个参数，linger.ms，这个值默认是0，意思就是消息必须立即被发送，但是这是不对的，一般设置一个100毫秒之类的，这样的话就是说，这个消息被发送出去后进入一个batch，如果100毫秒内，这个batch满了16kb，自然就会发送出去

但是如果100毫秒内，batch没满，那么也必须把消息发送出去了，不能让消息的发送延迟时间太长，也避免给内存造成过大的一个压力

### 52.如何根据业务场景对消息大小以及请求超时进行合理的设置？

max.request.size：这个参数用来控制发送出去的消息的大小，默认是1048576字节，也就1mb，这个一般太小了，很多消息可能都会超过1mb的大小，所以需要自己优化调整，把他设置更大一些

你发送出去的一条大数据，超大的JSON串，超过1MB，就不让你发了

request.timeout.ms：这个就是说发送一个请求出去之后，他有一个超时的时间限制，默认是30秒，如果30秒都收不到响应，那么就会认为异常，会抛出一个TimeoutException来让我们进行处理

### 53.acks参数到底是干嘛的

acks参数，其实是控制发送出去的消息的持久化机制的

如果acks=0，那么producer根本不管写入broker的消息到底成功没有，发送一条消息出去，立马就可以发送下一条消息，这是吞吐量最高的方式，但是可能消息都丢失了，你也不知道的，但是说实话，你如果真是那种实时数据流分析的业务和场景，就是仅仅分析一些数据报表，丢几条数据影响不大的

会让你的发送吞吐量会提升很多，你发送弄一个batch出，不需要等待人家leader写成功，直接就可以发送下一个batch了，吞吐量很大的，哪怕是偶尔丢一点点数据，实时报表，折线图，饼图

acks=all，或者acks=-1：这个leader写入成功以后，必须等待其他ISR中的副本都写入成功，才可以返回响应说这条消息写入成功了，此时你会收到一个回调通知

min.insync.replicas = 2，ISR里必须有2个副本，一个leader和一个follower，最最起码的一个，不能只有一个leader存活，连一个follower都没有了

acks = -1，每次写成功一定是leader和follower都成功才可以算做成功，leader挂了，follower上是一定有这条数据，不会丢失

retries = Integer.MAX_VALUE，无限重试，如果上述两个条件不满足，写入一直失败，就会无限次重试，保证说数据必须成功的发送给两个副本，如果做不到，就不停的重试，除非是面向金融级的场景，面向企业大客户，或者是广告计费，跟钱的计算相关的场景下，才会通过严格配置保证数据绝对不丢失

acks=1：只要leader写入成功，就认为消息成功了，默认给这个其实就比较合适的，还是可能会导致数据丢失的，如果刚写入leader，leader就挂了，此时数据必然丢了，其他的follower没收到数据副本，变成leader

### 54.基于Consumer Group的消费者组的模型

每个consumer都要属于一个consumer.group，就是一个消费组，topic的一个分区只会分配给一个消费组下的一个consumer来处理，每个consumer可能会分配多个分区，也有可能某个consumer没有分配到任何分区

分区内的数据是保证顺序性的

group.id = “membership-consumer-group”

如果你希望实现一个广播的效果，你的每台机器都要消费到所有的数据，每台机器启动的时候，group.id可以是一个随机生成的UUID也可以，你只要让不同的机器的KafkaConsumer的group.id是不一样的

如果consumer group中某个消费者挂了，此时会自动把分配给他的分区交给其他的消费者，如果他又重启了，那么又会把一些分区重新交还给他，这个就是所谓的消费者rebalance的过程

### 55. 消费者offset的记录方式以及基于内部topic的提交模式

每个consumer内存里数据结构保存对每个topic的每个分区的消费offset，定期会提交offset，老版本是写入zk，但是那样高并发请求zk是不合理的架构设计，zk是做分布式系统的协调的，轻量级的元数据存储，不能负责高并发读写，作为数据存储

所以后来就是提交offset发送给内部topic：__consumer_offsets，提交过去的时候，key是group.id+topic+分区号，value就是当前offset的值，每隔一段时间，kafka内部会对这个topic进行compact

也就是每个group.id+topic+分区号就保留最新的那条数据即可

而且因为这个__consumer_offsets可能会接收高并发的请求，所以默认分区50个，这样如果你的kafka部署了一个大的集群，比如有50台机器，就可以用50台机器来抗offset提交的请求压力，就好很多

### 56.Kafka感知消费者故障是通过哪三个参数来实现的？

heartbeat.interval.ms：consumer心跳时间，必须得保持心跳才能知道consumer是否故障了，然后如果故障之后，就会通过心跳下发rebalance的指令给其他的consumer通知他们进行rebalance的操作

session.timeout.ms：kafka多长时间感知不到一个consumer就认为他故障了，默认是10秒

max.poll.interval.ms：如果在两次poll操作之间，超过了这个时间，那么就会认为这个consume处理能力太弱了，会被踢出消费组，分区分配给别人去消费，一遍来说结合你自己的业务处理的性能来设置就可以了

### 57.对消息进行消费时有哪几个参数需要注意以及设置呢？

fetch.max.bytes：获取一条消息最大的字节数，一般建议设置大一些

max.poll.records：一次poll返回消息的最大条数，默认是500条

connection.max.idle.ms：consumer跟broker的socket连接如果空闲超过了一定的时间，此时就会自动回收连接，但是下次消费就要重新建立socket连接，这个建议设置为-1，不要去回收

### 58.消费者offset相关的参数设置会对运行产生什么样的影响？

auto.offset.reset：这个参数的意思是，如果下次重启，发现要消费的offset不在分区的范围内，就会重头开始消费；但是如果正常情况下会接着上次的offset继续消费的

enable.auto.commit：这个就是开启自动提交唯一

### 59.Group Coordinator是什么以及主要负责什么？

生产者和消费者的基本原理，使用，核心参数，这一周的课，我们就是对kafka的生产者、消费者、协议，底层的原理做一些更加深入的理解

每个consumer group都会选择一个broker作为自己的coordinator，他是负责监控这个消费组里的各个消费者的心跳，以及判断是否宕机，然后开启rebalance的，那么这个如何选择呢？

就是根据group.id来进行选择，他有内部的一个选择机制，会给你挑选一个对应的Broker，总会把你的各个消费组均匀分配给各个Broker作为coordinator来进行管理的

他负责的事情只要就是rebalance，说白了你的consumer group中的每个consumer刚刚启动就会跟选举出来的这个consumer group对应的coordinator所在的broker进行通信，然后由coordinator分配分区给你的这个consumer来进行消费

coordinator会尽可能均匀的分配分区给各个consumer来消费

### 60. 为消费者选择Coordinator的算法是如何实现的？

首先对groupId进行hash，接着对**consumer_offsets的分区数量取模，默认是50，可以通过offsets.topic.num.partitions来设置，找到你的这个consumer group的offset要提交到**consumer_offsets的哪个分区

比如说：groupId，“membership-consumer-group” -> hash值（数字）-> 对50取模 -> 就知道这个consumer group下的所有的消费者提交offset的时候是往哪个分区去提交offset，大家可以找到__consumer_offsets的一个分区

__consumer_offset的分区的副本数量默认来说1，只有一个leader

然后对这个分区找到对应的leader所在的broker，这个broker就是这个consumer group的coordinator了，接着就会维护一个Socket连接跟这个Broker进行通信

### 61.Coordinator和Consume Leader如何协作制定分区方案？

每个consumer都发送JoinGroup请求到Coordinator，然后Coordinator从一个consumer group中选择一个consumer作为leader，把consumer group情况发送给这个leader，接着这个leader会负责制定分区方案，通过SyncGroup发给Coordinator

接着Coordinator就把分区方案下发给各个consumer，他们会从指定的分区的leader broker开始进行socket连接以及消费消息

### 62.rebalance的三种策略分别有哪些优劣势？

这里有三种rebalance的策略：range、round-robin、sticky

0~8

order-topic-0

order-topic-1

order-topic-2

range策略就是按照partiton的序号范围，比如partitioin02给一个consumer，partition35给一个consumer，partition6~8给一个consumer，默认就是这个策略；

round-robin策略，就是轮询分配，比如partiton0、3、6给一个consumer，partition1、4、7给一个consumer，partition2、5、8给一个consumer

但是上述的问题就在于说，可能在rebalance的时候会导致分区被频繁的重新分配，比如说挂了一个consumer，然后就会导致partition0-4分配给第一个consumer，partition5~8分配给第二个consumer

这样的话，原本是第二个consumer消费的partition3~4就给了第一个consumer，实际上来说未必就很好

最新的一个sticky策略，就是说尽可能保证在rebalance的时候，让原本属于这个consumer的分区还是属于他们，然后把多余的分区再均匀分配过去，这样尽可能维持原来的分区分配的策略

consumer1：0~2 + 6~7

consumer2：3~5 + 8

### 63.Consumer内部单线程处理一切事务的核心设计思想

其实就是在一个while循环里不停的去调用poll()方法，其实是我们自己的一个线程，就是我们自己的这个线程就是唯一的KafkaConsumer的工作线程，新版本的kafka api，简化，减少了线程数量

Consumer自己内部就一个后台线程，定时发送心跳给broker；但是其实负责进行拉取消息、缓存消息、在内存里更新offset、每隔一段时间提交offset、执行rebalance这些任务的就一个线程，其实就是我们调用Consumer.poll()方法的那个线程

就一个线程调用进去，会负责把所有的事情都干了

为什么叫做poll呢？因为就是你可以监听N多个Topic的消息，此时会跟集群里很多Kafka Broker维护一个Socket连接，然后每一次线程调用poll()，就会监听多个socket是否有消息传递过来

可能一个consumer会消费很多个partition，每个partition其实都是leader可能在不同的broker上，那么如果consumer要拉取多个partition的数据，就需要跟多个broker进行通信，维护socket

每个socket就会跟一个broker进行通信

每个Consumer内部会维护多个Socket，负责跟多个Broker进行通信，我们就一个工作线程每次调用poll()的时候，他其实会监听多个socket跟broker的通信，是否有新的数据可以去拉取

### 64.自动提交offset的语义以及导致消息丢失和重复消费的问题

默认是自动提交

auto.commit.inetrval.ms：5000，默认是5秒提交一次

如果你提交了消费到的offset之后，人家kafka broker就可以感知到了，比如你消费到了offset = 56987，下次你的consumer再次重启的时候，就会自动从kafka broker感知到说自己上一次消费到的offset = 56987

这次重启之后，就继续从offset = 56987这个位置继续往后去消费就可以了

他的语义是一旦消息给你poll到了之后，这些消息就认为处理完了，后续就可以提交了，所以这里有两种问题：

第一，消息丢失，如果你刚poll到消息，然后还没来得及处理，结果人家已经提交你的offset了，此时你如果consumer宕机，再次重启，数据丢失，因为上一次消费的那批数据其实你没处理，结果人家认为你处理了

poll到了一批数据，offset = 65510~65532，人家刚好就是到了时间提交了offset，offset = 65532这个地方已经提交给了kafka broker，接着你准备对这批数据进行消费，但是不巧的是，你刚要消费就直接宕机了

其实你消费到的数据是没处理的，但是消费offset已经提交给kafka了，下次你重启的时候，offset = 65533这个位置开始消费的，之前的一批数据就丢失了

第二，重复消费，如果你poll到消息，都处理完毕了，此时还没来得及提交offset，你的consumer就宕机了，再次重启会重新消费到这一批消息，再次处理一遍，那么就是有消息重复消费的问题

poll到了一批数据，offset = 65510~65532，你很快的处理完了，都写入数据库了，结果还没来得及提交offset就宕机了，上一次提交的offset = 65509，重启，他会再次让你消费offset = 65510~65532，一样的数据再次重复消费了一遍，写入数据库

重启kafka consumer，修改了他的代码

### 65.什么情况下会导致Consumer Group之间的重平衡？

其实最主要的就是consumer所在服务比如重启了，或者宕机了，或者新部署了，此时coordinator感知到了consumer group内的变化，就会触发rebalance的操作，或者是如果动态给topic增加了分区，也会触发rebalance

也或者是这个消费组订阅了更多的topic，也会触发rebalance进行新订阅topic的分区分配给各个consumer来处理

### 66.如何实现Consumer Group的状态机流转机制？

刚开始Consumer Group状态是：Empty

接着如果部分consumer发送了JoinGroup请求，会进入：PreparingRebalance的状态，等待一段时间其他成员加入，这个时间现在默认就是max.poll.interval.ms来指定的，所以这个时间间隔一般可以稍微大一点

接着如果所有成员都加入组了，就会进入AwaitingSync状态，这个时候就不能允许任何一个consumer提交offset了，因为马上要rebalance了，进行重新分配了，这个时候就会选择一个leader consumer，由他来制定分区方案

然后leader consumer制定好了分区方案，SyncGroup请求发送给coordinator，他再下发方案给所有的consumer成员，此时进入stable状态，都可以正常基于poll来消费了

所以如果说在stable状态下，有consumer进入组或者离开崩溃了，那么都会重新进入PreparingRebalance状态，重新看看当前组里有谁，如果剩下的组员都在，那么就进入AwaitingSync状态

leader consumer重新制定方案，然后再下发

### 67.rebalance分代机制可以有什么作用？

大家设想一个场景，在rebalance的时候，可能你本来消费了partition3的数据，结果有些数据消费了还没提交offset，结果此时rebalance，把partition3分配给了另外一个cnosumer了，此时你如果提交partition3的数据的offset，能行吗？

必然不行，所以每次rebalance会触发一次consumer group generation，分代，每次分代会加1，然后你提交上一个分代的offset是不行的，那个partiton可能已经不属于你了，大家全部按照新的partiton分配方案重新消费数据

consumer group generation = 1

consumer group generation = 2

### 68.Producer的缓冲区内部数据结构是什么样子的？

producer会创建一个accumulator缓冲区，他里面是一个HashMap数据结构，每个分区都会对应一个batch队列，因为你打包成出来的batch，那必须是这个batch都是发往同一个分区的，这样才能发送一个batch到这个分区的leader broker

{

“order-topic-0” -> [batch1, batch2],

“order-topic-1” -> [batch3]

}

batch.size

每个batch包含三个东西，一个是compressor，这是负责追加写入batch的组件；第二个是batch缓冲区，就是写入数据的地方；第三个是thunks，就是每个消息都有一个回调Callback匿名内部类的对象，对应batch里每个消息的回调函数

每次写入一条数据都对应一个Callback回调函数的

### 69.消息缓冲区满的时候是阻塞住还是抛出异常？

max.block.ms，其实就是说如果写缓冲区满了，此时是阻塞住一段时间，然后什么时候抛异常，默认是60000，也就是60秒

### 70.负责IO请求的Sender线程是如何基于缓冲区发送数据的？

Sender线程会不停的轮询缓冲区内的HashMap，看batch是否满了，或者是看linger.ms时间是不是到了，然后就得发送数据去，发送的时候会根据各个batch的目标leader broker来进行分组

因为可能不同的batch是对应不同的分区，但是不同的分区的Leader是在一个broker上的，<Node, List<ProducerBatch>>，接着会进一步封装为<Node, Request>，每个broker一次就是一个请求，但是这里可能包含很多个batch，接着就是将分组好的batch发送给leader broker，并且处理response，来反过来调用每个batch的callback函数

发送出去的Request会被放入InFlighRequests里面去保存，Map<NodeId, Deque<Request>>，这里就代表了发送出去的请求，但是还没接收到响应的

### 71.同时可以接受有几个发送到Broker的请求没收到响应？

Map<NodeId, Deque<Request>> => 给这个broker发送了哪些请求过去了？

max.in.flight.requests.per.connection：5

这个参数默认值是5，默认情况下，每个Broker最多只能有5个请求是发送出去但是还没接收到响应的，所以这种情况下是有可能导致顺序错乱的，大家一定要搞清楚这一点，先发送的请求可能后续要重发

### 72.盘点一下在Broker内部有哪些不同场景下会有延时任务？

比如说acks=-1，那么必须等待leader和follower都写完才能返回响应，而且有一个超时时间，默认是30秒，也就是request.timeout.ms，那么在写入一条数据到leader磁盘之后，就必须有一个延时任务，到期时间是30秒

延时任务会被放到DelayedOperationPurgatory，延时操作管理器中

这个延时任务如果因为所有follower都写入副本到本地磁盘了，那么就会被自动触发苏醒，那么就可以返回响应结果给客户端了，否则的话，这个延时任务自己指定了最多是30秒到期，如果到了超时时间都没等到，那么就直接超时返回异常了

还有一种是延时拉取任务，也就是说follower往leader拉取消息的时候，如果发现是空的，那么此时会创建一个延时拉取任务，然后延时时间到了之后，就会再次读取一次消息，如果过程中leader写入了消息那么也会自动执行这个拉取任务

### 73.Broker端

kafka全链路架构，broker端、生产端、消费端

生产集群规划和部署，监控和管理，生产服务，消费服务，各个端的参数应该如何来设置，我们的一个驱动的场景，电商场景驱动，数据量，10亿消息量的场景下，要存储多大量的数据，要支撑多高的并发

broker端机器数量和配置，生产端的机器数量和配置，消费端的机器数量和配置

（1）集群架构：Controller -> zk -> broker

（2）元数据管理：Controller，Topic -> Partition -> Leader & Follower -> Broker

（3）分布式架构：Topic逻辑数据集合 -> 物理上的多个Partition -> 分布在多台Broker

（4）broker端的多路复用模型的请求处理架构，自定义二进制协议

（5）磁盘存储，os cache，零拷贝，索引文件，日志文件，定期清理文件

（6）高可用架构：多副本冗余机制，Leader & Follower角色分配，leader -> follower的副本同步机制，LEO & HW的更新，ISR机制，Broker宕机后如何由Controller感知以及触发Leader重新选举

（7）负载均衡架构：所有partition物理的均匀分散在broker集群，leader partition也是均匀分散在broker集群

（8）延时处理机制：时间轮实现

### 74.生产端

（1）元数据拉取机制：Topic -> Partition -> Leader & Follower -> Broker

（2）分区机制，默认的分区策略，hash分区策略

（3）缓冲区：数据结构，每个分区有多个batch

（4）Sender线程 + batch批量发送，按照broker来聚合多个batch作为request

（5）同步和异步

（6）核心参数：batch.size、缓冲区大小、acks、超时时间

### 75.消费端的架构原理

（1）消费模型：consumer group，每个分区就给一个consumer，queue & PUB/SUB

（2）分区分配机制：coordinator -> __consumer_offsets，JoinGroup -> leader consumer -> 下发分区方案 -> 状态机，三种分区策略

（2）消费方式：poll()调用，单线程干所有的事情

（3）offset内存数据结构更新，定时提交offset，内部topic（__consumer_offsets）

（4）故障感知机制：核心参数，心跳参数、会话超时、消费过慢

（5）老版本api的high-level和low-level，简单的认知就可以了，新版本

### 76.Kafka集群部署

需求场景 -> 请求量 -> 数据量 -> 高峰并发量

kafka集群部署：多少台物理机、磁盘规划（容量和类型）、内存规划（os cache & jvm）、CPU、网卡，10亿请求的场景，需要一个多大规模的kafka集群

kafka运维管理：支撑业务（topic创建和维护、可视化平台）、集群监控（可视化平台、开源工具）、资源使用率 -> 集群扩容 / topic扩容、机器负载率 -> 负载均衡、异常监控（报错 / jvm fullgc）、运维管理（多集群同步、安全认证、版本升级）

### 77.生产服务部署

生产端机器部署：可能是你的java系统在生产消息，也可能是flume日志采集，canal数据库binlog采集，爬虫，如果你的机器就是每天1000万~2000万，数据往kafka里写入，没其他的数据库访问或者是别的线程和操作，4核8G的虚拟机要2台正好，高峰期的时候，CPU和内存的负载都到50%了，10亿 -> 40台4核8G虚拟机 -> 10台~20台左右16核32G的高配置虚拟机，也可以抗住每天生产10亿条数据的负载

### 78.消费服务部署

消费端机器部署：flink、spark streaming、自己写的java系统，4核8G虚拟机，单线程消费出来 -> 线程池来处理（60），每秒钟消费吞吐量大概可以到个几百，一两千~两三千，对应的是说，他需要对数据库进行很多增删改查的操作，16核32G，每秒消费和处理5000条数据，10多台机器，spark streaming他每秒要消费和处理几万条数据，那么需要多少的资源，cpu core、内存，集群是运行在多少台机器上的集群，spark streaming，集群规模有30，480核 + 640G，160核 + 320G

### 79.Kafka集群参数

kafka集群部署好，生产者代码写好 + 部署好，消费者代码写好 + 部署好，基本上已经可以run起来了，可以抗住每秒几万的并发请求，一天10亿的数据量流过kafka在处理，对其中的一些细节进行参数设置

数据保留几天，默认是7天，min.insync.replicas配合ISR + 数据不丢失，topic默认的分区数量，分区默认的副本数量，__consumer_offsets的分区数量和副本数量，processor线程数量，Handler线程池的大小，jvm参数的设置（堆大小 & G1垃圾回收器）

### 80.生产服务参数

生产端参数：缓冲区、batch、超时时间、acks

### 81.消费服务参数

消费端参数：offset提交、poll拉取消息、心跳、超时、拉取过慢

### 82.问题

Kafka全链路数据丢失风险分析

万一在producer缓冲区的时候，直接就生产服务就挂了，此时你写出去的数据不就丢失了吗？

acks = 1，那么只要leader写入成功了，就可以认为是成功，但是如果数据刚刚写入leader成功，客户端也认为成功了，但是follower还没同步数据，此时leader宕机了，必然会导致数据的丢失

acks = -1，leader + follower，leader和follower都写成功了，才算是成功，此时任何一个副本宕机，都不会导致数据的丢失；如果min.insync.replicas = 1，此时如果follower先宕机了，导致ISR里就一个leader了

此时acks = -1，ISR里的副本都写入成功，leader写入成功，就算成功了，结果leader也挂了，此时数据还是会丢失的

如果你消费到数据，poll到了数据，还没来得及处理，人家就自动提交offset了，此时kafka就认为你已经成功处理了这批数据，但是此时你consumer直接宕机了，数据丢失了，但是offset已经提交了

就是按照这个kafka课程，你如果上了生产，其实会经常发现丢数据的情况的，订单系统支付成功以后就会发送订单到kafka，会员系统需要来增加积分，如果丢失一些订单，会导致很多人支付以后，会员积分没有累加

为什么线上数据总是莫名其妙出现重复

数据重复这个问题其实也是挺正常，全链路都有可能会导致数据重复

生产端是有重试机制，发现你有一些网络抖动，在底层的网络环节，其实消息发送出去了，结果收到了一个NetworkException，此时会重发消息

消费端，重复的问题，很频繁，如果你是用自动offset提交，一定会每次重启consumer服务的时候，一定会重复消费消息，刚刚接收到一批消息，poll出来的，处理完毕了，自动提交offset还没来得及执行，你就重启了

会从上一次提交offset的地方重新拉取消息再次执行一遍，消息重复

kafka，同步订单，结果每天都发现有重复的订单同步的现象，支付了一个订单，应该累加积分是1000，结果订单重复同步了一次，导致人家积分增加了2000

MySQL binlog数据同步顺序为何不能出错

消息可能会乱序的一个问题

导致乱序的原因还是重试，<NodeId, Deque<Request>>，最多允许同时发送出去5个请求，忍受5个请求都发出去，但是没有收到响应，如果1和2两个请求都成功了，3这个请求失败了，4和5也成功了

对请求3重试，此时就算成功，他的数据也会排在4和5请求的后面了

消费者，一个consumer对应一个或者多个分区，分区内的消息一定是有顺序的，单线程消费各个分区的数据，所以单线程消费到的消息一定都是在分区内有顺序的，消费到的时候还是有顺序的

如果你把消息，每个消息提交一个任务给线程池异步处理，消息1~10分别分配给10个线程来并发的处理，但是此时可能先处理完消息10，然后处理完消息6，最后是处理完消息2，你的消息被处理的顺序完全不是按照顺序来的

比如你要从业务数据库，监听他的mysql binlog的变化，增删改，INSERT、UPDATE、DELETE相关的语句，对同一条数据的三个mysql bilog是：insert、update、delete。结果这个消息写入kafka是有顺序的

消费到了3条消息，提交给3个线程并发处理，他需要把这些操作同步到es里去，先在es里插入一条数据，然后更新，接着是删除，先执行delete，然后是update，接着是insert，最后反而es里有一条数据

线上消息队列百万消息积压怎么处理

比较致命的一个问题

（1）下游的消费者，比如说他是依赖于MySQL、NoSQL之类的系统，但是现在就是说MySQL他的性能突然出现了很大的问题，比如说一个表有几千万条数据，有个同学在高峰期做了一个DDL，导致对数据库的增删改查非常的慢，性能急剧下降几十倍，瞬间导致kafka里大量的积压了很多很多的消息，几百万条消息积压在里面

（2）依赖的ES突然宕机了，ES集群突然故障了，无法访问了，机房故障，机房的冷却系统坏了，温度过高，机器都故障了，下游的消费者就不工作了，积压几百万的消息

（3）你怎么来处理，应对

如何配合分布式事务方式实现消息事务支持

可靠消息最终一致性的事务方案，基于MQ来实现的，可靠消息服务，上游服务的本地事务成功了，必须保证消息投递出去到MQ上去，可以由MQ自己来支持和实现，RocketMQ就实现了一套机制

提供了事务相关的支持，数据库事务 + 消息，封装在一个事务里，数据库事务成功了，消息也必须投递成功，如果要跟分布式事务来整合的话，一般用的不是Kafka，RocketMQ来支持就最好

消息的过期时间（TTL）如何实现

有的时候其实是希望对消息设置一个过期时间，TTL，就是如果在kafka里积压了一段时间，TTL过期时间，还没被消费和处理，那么就直接过期掉他，就不要继续去处理了，可以跟百万消息积压的方案结合起来来使用

比如说现在kafka里积压了100万消息，TTL（30秒），此时如果下游的消费者恢复了，可以继续消费了，此时他发现这些消息都到TTL了，就可以直接丢弃掉这些消息，不要慢吞吞的来处理

你可以把TTL + 死信队列来实现一套生产方案，过期的消息直接放到另外一个独立的特殊的队列里去，那个队列专门是用来处理一些长时间过期的消息的，有专门的后台的一套系统和机制来处理

如何实现延迟队列的效果

举这么一个场景，你在生产消息的时候，如果说发现某个队列现在压力很大，或者是下游消费者压力很大，你是不是可以不要继续把消息拼命的往下游发送去执行，你可以不可以让消息进入特殊的延迟队列

你进入kafka之后，我指定你延迟30秒后才能被消费到，DelayQueue

不同的消息如何实现多优先级队列

消息写入kafka的时候，可以指定消息的优先级，就是优先级越高的数据应该越是先被消费出来，消息可以按照优先级来排序，优先级高的先被消费出来，你可能在方那哦数据过去的时候，有些数据是大客户的数据，优先级可以指定的高一些

针对链路故障的死信队列应该如何实现

如果你的某个消息此时发送到kafka之后，比如说他在正常的topic里无法被处理，不管怎么说，下游的消费者整个就挂掉了，此时这些消息可以进入一个队列，特殊的队列，死信队列，处理失败的消息，放入特殊的队列中，后续可以用特殊的机制来进行处理

假设我们给消息设置了TTL，然后消息进入了kafka topic，结果下游消费者故障，导致百万消息积压了，消费者一旦恢复，对这些过期的消息，直接就不处理，放入到一个死信队列里去

后面可以用一些特殊的机制对死信队列中的消息进行处理

消费服务故障场景下的重试队列

下游消费者整个的就是故障了，没办法继续消费了，此时可以结合TTL，死信队列，针对一些异常故障的场景来进行处理，下游消费者是正常的，结果呢，不知道为什么，在消费以及处理某一条消息的时候，就是失败了

比如说消费服务在对MySQL操作的时候，突然出现一个dead lock，抛异常，此时就可以把这个消息放入一个特殊的队列，叫做重试队列，后面可以重新消费一遍再次处理

下游数据计算错误时如何进行消息回溯

比如说消费者服务里的一些代码逻辑有bug，他对消息的处理是错误的，此时如果你发现这个问题之后，首先应该把错误的数据给删除掉，对那些数据，是不是要进行回溯，从kafka里重新读取某个offset~某个offset的一段数据，重新这个流程来一遍

就是重新对这些数据做一遍处理

不同的业务数据如何实现消息路由

topic里，可能也细分为不同的数据，同步订单

有的订单是正常情况下支付的，就是按照正常的流程来增加积分就可以了

但是有的订单是采用的是特殊的折扣优惠的场景，说明了是无积分累加的，此时对这种特殊的订单是否可以打一个标签

特殊订单和正常订单都在一个topic里，但是下游的消费服务获取到了消息，可以进行路由，也就是仅仅路由出来正常的订单给后续的逻辑来处理，特殊订单就是不予以处理就可以了，对消息打上不同的标签，消息路由

如何对消息流转链路进行消息轨迹监控

很多常见的一些应用，实时数据仓库，可能是从kafka里消费到数据，处理一下，写入到下一个kafka topic，再有人消费出来，再处理一下，再写入到下一个kafka topic，他是一个链路的场景

消息流转链路，面向B端的场景里，就是说你有一个链路，比如在哪个程序里消费到了一条消息，如何如何处理，写入到了下一个kafka topic去，又是哪一个程序获取到了这条消息，如何如何处理，再次写入到了下一个kafka topic

程序A -> Topic A -> 程序B -> Topic B -> 程序C -> Topic C

对消息质量是否正常进行监控