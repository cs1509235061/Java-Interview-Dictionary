# ZooKeeper

## 1.Zookeeper概念

ZooKeeper是一个分布式协调服务，其设计的初衷是为分布式软件提供一致性服务。Zoo Keeper提供了一个类似Linux文件系统的树形结构，ZooKeeper的每个节点既可以是目录也可以是数据，同时，ZooKeeper提供了对每个节点的监控与通知机制基于ZooKeeper一致性服务，可以方便地实现分布式锁、分布式选举、服务发现和监控、配置中心等功能。

ZooKeeper是以 Fast Paxos 算法为基础的， Paxos 算法存在活锁的问题，即当有多个 proposer 交错提交时，有可能互相排斥导致没有一个 proposer 能提交成功，而 Fast Paxos 作了一些优化，通过选举产生一个 leader ( 领导者 ))，只有 leader 才能提交 proposer 。

ZooKeeper的基本运转流程 :1 、选举 Leader 。 2 、同步数据。 3 、选举 Leader 过程中算法有很多，但要达到的选举标准是一致的。 4 、 Leader 要具有最高的执行 ID ，类似 root 权限。 5 、集群中大多数的机器得到响应并 follow选出的Leader 。

## 2.ZooKeeper的角色

ZooKeeper是一个基于主从复制的高可用集群，ZooKeeper的角色包括Leader、Follower、Observer。Leader是集群主节点，主要负责管理集群状态和接收用户的写请求；Follower是从节点，主要负责集群选举投票和接收用户的读请求；Observer的功能与Follower类似，只是没有投票权，主要用于分担Follower的读请求，降低集群的负载。

### Leader

一个运行中的ZooKeeper集群只有一个Leader服务，Leader服务主要有两个职责：一是负责集群数据的写操作；二是发起并维护各个Follower及Observer之间的心跳以监控集群的运行状态。在ZooKeeper集群中，所有的写操作都必须经过Leader，只有Leader写操作完成后，才将写操作广播到其他Follower。只有超过半数节点（不包括Observer节点）写入成功时，该写请求才算写成功。

### Follower

一个ZooKeeper集群可以有多个Follower，Follower通过心跳和Leader保持连接。Follower服务主要有两个职责：一是负责集群数据的读操作，二是参与集群的Leader选举。Follower在接收到一个客户端请求后会先判断该请求是读请求是写请求，如果是读请求，则Follower从本地节点上读取数据并返回给客户端；如果是写请求，则Follower会将写请求转发给Leader来处理；同时，在Leader失效后，Follower需要在集群选举时进行投票。

### Observer

一个ZooKeeper集群可以有多个Observer，Observer主要职责是负责集群数据的读操作。在ZooKeeper的设计上，Observer的功能与Follower类似，主要的差别是Observer无投票权。ZooKeeper集群在运行过程中要支持更多的客户端并发的操作，就需要增加更多的服务实例，而过多的服务实例会使集群的投票阶段变得复杂，集群选主时间过长，不利于集群故障的快速恢复。因此，ZooKeeper引入了Observer角色，Observer不参与投票，只用来接收客户端的连接并响应客户端的读请求，将写请求转发给Leader节点。加入更多的Observer节点，不仅提高了ZooKeeper集群的吞吐量，也保障了系统的稳定性。

## 3.ZAB协议

ZAB ( ZooKeeper Atomic Broadcast）即ZooKeeper原子消息广播协议。该协议主要通过唯一的事务编号Zxid( ZooKeeper Transaction id）保障集群状态的唯一性。Zxid与RDBMS中的事务id类似，用于标识一次提议（Proposal）的id；为了保证顺序性，Zxid必须单调递增。

( 1 ) Epoch: Epoch指当前集群的周期号（年代号），集群的每次Leader变更都会产生一个新的周期号，周期号的产生规则是在上一个周期号的基础上加1，这样当之前的Leader崩溃恢复后会发现自己的周期号比当前的周期号小，说明此时集群已经产生了新的Leader，旧的Leader会再次以Follower的角色加入集群。

( 2) Zxid: Zxid指ZAB协议的事务编号，它是一个64位的数字，其中低32位存储的是一个简单的单调递增的计数器，针对客户端的每一个事务请求，计数器都加1。高32位存储的是Leader的周期号Epoch。每次选举产生一个新的Leader时，该Leader都会从当前服务器的日志中取出最大事务的Zxid，获取其中高32位的Epoch值并加1，以作为新的Epoch，并将低32位从0开始重新计数.

ZA协议有两种模式，分别是恢复模式（集群选主）和广播模式（数据同步）。

( 1 ）恢复模式：当集群启动、集群重启或者Leader崩溃后，集群将开始选主，该过程为恢复模式。

( 2 ）广播模式：当Leader被选举出来后，Leader将最新的集群状态广播给其他Follower，该过程为广播模式。在半数以上的Follower完成与Leader的状态同步后，广播模式结束。

### ZAB协议的4个阶段

( 1 ）选举阶段（Leader Election ）：在集群选举开始时，所有节点都处于选举阶段。当某一个节点的票数超过半数节点后，该节点将被推选为准Leader。选举阶段的目的就是产生一个准Leader。只有到达广播阶段（Broadcast）后，准Leader才会成为真正的Leader。

( 2 ）发现阶段（Discovery）： 在发现阶段，各个Follower开始和准Leader进行通信，同步Follower最近接收的事务提议。这时，准Leader会产生一个新的Epoch，并尝试让其他Follower接收该Epoch后再更新到本地。发现阶段的一个Follower只会连接一个Leader，如果节点1认为节点2是Leader，则当节点1尝试连接节点2时，如果连接被拒绝，则集群会进入重新选举阶段。发现阶段的主要目的是发现当前大多数节点接收的最新提议。

( 3 ）同步阶段（Synchronizatation）：同步阶段主要是将Leader在前一阶段获得的最新提议信息同步到集群中所有的副本，只有当半数以上的节点都同步完成时，准Leader才会成为真正的Leader。Follower只会接收Zxid比自己的lastZxid大的提议。同步阶段完成后集群选主的操作才完成，新的Leader将产生。

( 4 ）广播阶段（Broadcast）：在广播阶段，ZooKeeper集群开始正式对外提供事务服务，这时Leader进行消息广播，将其上的状态通知到其他Follower，如果后续有新的节点加入，则Leader会对新节点进行状态的同步。

## 3.ZooKeeper的选举机制和流程

ZooKeeper的选举机制被定义为：每个Server首先都提议自己是Leader，并为自己投票，然后将投票结果与其他Server的选票进行对比，权重大的胜出，使用权重较大的选票更新自身的选票箱，具体选举过程如下。

( 1 ）每个Server启动以后都询问其他Server给谁投票，其他Server根据自己的状态回复自己推荐的Leader并返回对应的Leader id和Zxid。在集群初次启动时，每个Server都会推荐自己为Leader。

( 2 ）当Server收到所有其他Server的回复后，计算出Zxid最大的Server，并将该Server设置成下一次要投票推荐的Server。

( 3 ）计算过程中票数最多的Server将成为获胜者，如果获胜者的票数超过集群个数的一半，则该Server将被推选为Leader，否则，继续投票，直到Leader被选举出来。

( 4 ) Leader等待其他Server连接。

( 5 ) Follower连接Leader，将最大的Zxid发送给Leader。

( 6) Leader根据Follower的Zxid确定同步点，至此，选举阶段完成。

( 7) 在选举阶段完成后，Leader通知其他Follower集群已经成为Uptodate状态，

( 8) Follower收到Uptodate消息后，接收Client的请求并开始对外提供服务。

下面以5台服务器Server1、Server2、Server3、Server4、Servers5的选主为例，假设它们的编号分别是1、2、3、4、5，按编号依次启动，它们的选举过程。

( 1 ) Server1启动：Server1提议自己为Leader并为自己投票，然后将投票结果发送给其他服务器，由于其他服务器还未启动，因此收不到任何反馈信息.此时Server1处于Looking状态（Looking状态表示当前集群正处于选举状态）。

( 2) Server2启动：Server2提议自己为Leader为自己投票，然后与Server1交换投票结果，由于Server2的编号大于Server1，因此Server2胜出。同时，由于投票未过半，两个服务器均处于Looking状态。

( 3 ) Server3启动：Server3提议自己为Leader并为自己投票，然后与Server1、Server2交换投票结果，由于Server3编号最大，因此Server3胜出。此时Server3票数大于半数集群，因此Server3成为Leader，Server1、Server2成为Follower。

( 4) Server4启动：Server4提议自己为Leader并为自己投票，然后与Server1、Server2、Server3交换投票结果，发现Server3已经成为Leader，因此Server4成为Follower。

( 5 ) Servers5启动：首先给自己投票，然后与其他服务器交换信息，发现Server3已经成为Leader，因此Server5成为Follower。

## 4.Zookeeper工作原理（原子广播）

1.Zookeeper的核心是原子广播，这个机制保证了各个server之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式和广播模式。

2.当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数server的完成了和leader的状态同步以后，恢复模式就结束了。

3.状态同步保证了leader和server具有相同的系统状态.

4.一旦leader已经和多数的follower进行了状态同步后，他就可以开始广播消息了，即进入广播状态。这时候当一个server加入zookeeper服务中，它会在恢复模式下启动，发现leader，并和leader进行状态同步。待到同步结束，它也参与消息广播。Zookeeper服务一直维持在Broadcast状态，直到leader崩溃了或者leader失去了大部分的followers支持。

5.广播模式需要保证proposal被按顺序处理，因此zk采用了递增的事务id号(zxid)来保证。所有的提议(proposal)都在被提出的时候加上了zxid。

6.实现中zxid是一个64为的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch。低32位是个递增计数。

7.当leader崩溃或者leader失去大多数的follower，这时候zk进入恢复模式，恢复模式需要重新选举出一个新的leader，让所有的server都恢复到一个正确的状态。

## 5.Znode有四种形式的目录节点

1.PERSISTENT：持久的节点。

所谓持久节点，是指在节点创建后，就一直存在，直到有删除操作来主动清除这个节点。否则不会因为创建该节点的客户端会话失效而消失。

2.EPHEMERAL：暂时的节点。

和持久节点不同的是，临时节点的生命周期和客户端会话绑定。也就是说，如果客户端会话失效，那么这个节点就会自动被清除掉。注意，这里提到的是会话失效，而非连接断开。另外，在临时节点下面不能创建子节点。

这里还要注意一件事，就是当你客户端会话失效后，所产生的节点也不是一下子就消失了，也要过一段时间，大概是10 秒以内，可以试一下，本机操作生成节点，在服务器端用命令来查看当前的节点数目，你会发现客户端已经stop，但是产生的节点还在。

3.PERSISTENT_SEQUENTIAL：持久化顺序编号目录节点。

这类节点的基本特性和上面的节点类型是一致的。额外的特性是，在ZK 中，每个父节点会为他的第一级子节点维护一份时序，会记录每个子节点创建的先后顺序。基于这个特性，在创建子节点的时候，可以设置这个属性，那么在创建节点过程中，ZK 会自动为给定节点名加上一个数字后缀，作为新的节点名。这个数字后缀的范围是整型的最大值。 在创建节点的时候只需要传入节点 “/test*”，这样之后，zookeeper 自动会给”test*”后面补充数字。

4.EPHEMERAL_SEQUENTIAL：暂时化顺序编号目录节点。

此节点是属于临时节点，不过带有顺序，客户端会话结束节点就消失。

## 6.ZooKeeper的数据模型

ZooKeeper使用一个树形结构的命名空间来表示其数据结构，在结构上与标准文件系统非常相似，ZooKeeper树中的每个节点都被称为一个Znode。ZooKeeper的数据结构类似文件系统的目录树，ZooKeeper树中的每个节点都可以拥有子节点；与文件系统不同是，ZooKeeper的每个节点都存储了数据信息，同时提供对节点信息的监控（Watch）等操作。

### Znode的数据模型

ZooKeeper命名空间中的Znode兼具文件和目录两种特点。它既能像文件一样保存和维护数据，又能像目录一样作为路径标识的一部分。每个Znode都由3部分组成.

( 1 ) Stat：状态信息，用于存储该Znode的版本、权限、时间戳等信息。

( 2 ) Data: Znode上具体存储的数据。

( 3 ) Children: Znode子节点的信息描述。

Znode节点虽然可以存储数据，但它并不能像数据库那样存储大量的数据。设计Znode的初衷是存储分布式应用中的配置文件、集群状态等元数据信息。

### Znode的控制访问

(1) ACL：每一个Znode节点都拥有一个访问控制列表（Access Control List, ACL ) , 该列表规定了用户对节点的访问权限，应用程序可以根据需求将用户分为只读、只写和读写用户。

(2 ）原子操作：每一个Znode节点上的数据都具有原子操作的特性，读操作将获取与节点相关的数据，写操作将替换节点上的数据。

### Znode的节点类型

ZooKepe中的节点有两种，分别为临时节点和永久节点。节点的类型在创建时被确定并且不能改变。

( 1 ）临时节点：临时节点的生命周期取决于过期时间，当临时节点过期后系统会自除该节点，此外，ZooKeeper的临时节点不允许拥有子节点。临时节点常用于心跳监控，例如，可以设置过期时间为30s，要求各个子节点对应的服务端每5s发送一次心跳到ZooKeeer集群，当有服务端连续30s没有向ZooKeeper汇报心跳信息（连续6次未收到心跳信息，6次×5s=30s），就可以认为该节点宕机，将其从服务列表中移除。

( 2 ）永久节点：永久节点的数据会一直存储，直到用户调用接口将其数据删除。该节点一般用于存储一些永久性的配置信息。

### Znode的节点Watch

ZooKeeper的每个节点上都有Watch用于监控节点数据的变化，当节点状态发生改变时（Znode新增、删除、修改）将会触发Watch所对应的操作。在Watch被触发时，Zoo Keeper会向监控该节点的客户端发送一条通知说明节点的变化情况

## 7.Zookeeper的应用场景

ZooKeeper可用于统一命名服务、配置管理、集群管理、分布式通知协调、分布式锁等应用场景。

### 统一命名服务

在分布式环境下，应用程序经常需要对服务进行统一命名，以便识别不同的服务和快速获取服务列表，应用程序可以将服务名称和服务地址信息维护在ZooKeeper上，客户端通过ZooKeeper获取可用服务列表。

### 配置管理

在分布式环境下，应用程序可以将配置文件统一在ZooKeeper上管理。配置信息可以按照系统配置、告警配置、业务开关配置、业务阈值配置等分类存储在不同的Znode上。 各个服务在启动的时候从ZooKeeper上读取配置，同时监听各个节点的Znode数据，一且Znode中的配置被修改，ZooKeeper就将通知各个服务然后在线更新配置。

使用ZooKeeper做统一配置管理，不但避免了维护散落在各个服务器上的配置文件的复杂性，同时在配置信息变化的时候能及时通知各个服务在线更新配置，而不用重启服务。

### 集群管理

在分布式环境下，实时管理每个服务的状态是ZooKeeper使用最广的场景，常见的HBase、Kafka、Storm、HDFS等集群都依赖ZooKeeper做统一的状态管理。

### 分布式通知协调

基于Znode的临时节点和Watch特性，应用程序可以很容易地实现一个分布式通知协调系统。比如在集群中为每个服务都创建一个周期为30s的临时节点作为服务状态监控，要求各个服务每10s定时向ZooKeeper汇报监控状态。当ZooKeeper连续30s未收到服务的状态反馈时，则可以认为该服务异常，将其从服务列表移除，同时将该结果通知到他监控节点状态的服务。

### 分布式锁

由于ZooKeeper是强一致性的，多个客户端同时在ZooKeeper上创建相同的Znode时，只有一个能创建成功。基于该机制，应用程序可以实现锁的独占性，当多个客户端同时在ZooKeeper上创建相同的Znode时，创建成功的那个客户端将得到锁，其他客户端则等待。

同时将锁节点设置为EPHEMERAL_SEQUENTIAL，这样该Znode可掌握全局锁的访问时序。

## 8.什么是Znode？

在Zookeeper 中，znode 是一个跟Unix 文件系统路径相似的节点，可以往这个节点存储或获取数据。 Zookeeper 底层是一套数据结构。这个存储结构是一个树形结构，其上的每一个节点，我们称之为“znode”。 zookeeper 中的数据是按照“树”结构进行存储的。而且znode 节点还分为4 中不同的类型。 每一个znode 默认能够存储1MB 的数据（对于记录状态性质的数据来说，够了）。

可以使用zkCli 命令，登录到zookeeper 上，并通过ls、create、delete、get、set等命令操作这些znode 节点。

## 9.zk到底通过什么协议在集群间进行数据一致性同步？

在整个zk的架构和工作原理中，有一个非常关键的环节，就是zk集群的数据同步是用什么协议做的？其实用的是特别设计的ZAB协议，ZooKeeper Atomic Broadcast，就是ZooKeeper原子广播协议。

通过这个协议来进行zk集群间的数据同步，保证数据的强一致性。

## 10.ZAB消息广播流程

每一个消息广播的时候，都是2PC思想走的，先是发起事务Proposal的广播，就是事务提议，仅仅只是个提议而已，各个follower返回ack，过半follower都ack了，就直接发起commit消息到全部follower上去，让大家提交。

发起一个事务proposal之前，leader会分配一个全局唯一递增的事务id，zxid，通过这个可以严格保证顺序。

leader会为每个follower创建一个队列，里面放入要发送给follower的事务proposal，这是保证了一个同步的顺序性。

每个follower收到一个事务proposal之后，就需要立即写入本地磁盘日志中，写入成功之后就可以保证数据不会丢失了，然后返回一个ack给leader，然后过半follower都返回了ack，leader推送commit消息给全部follower。

leader自己也会进行commit操作。

commit之后，就意味这个数据可以被读取到了。

## 11.ZooKeeper到底是强一致性还是最终一致性？

强一致性：只要写入一条数据，立马无论从zk哪台机器上都可以立马读到这条数据，强一致性，你的写入操作卡住，直到leader和全部follower都进行了commit之后，才能让写入操作返回，认为写入成功了。

此时只要写入成功，无论你从哪个zk机器查询，都是能查到的，强一致性。

最终一致性：写入一条数据，方法返回，告诉你写入成功了，此时有可能你立马去其他zk机器上查是查不到的，短暂时间是不一致的，但是过一会儿，最终一定会让其他机器同步这条数据，最终一定是可以查到的。

研究了ZooKeeper的ZAB协议之后，你会发现，其实过半follower对事务proposal返回ack，就会发送commit给所有follower了，只要follower或者leader进行了commit，这个数据就会被客户端读取到了。

那么有没有可能，此时有的follower已经commit了，但是有的follower还没有commit？绝对会的，所以有可能其实某个客户端连接到follower01，可以读取到刚commit的数据，但是有的客户端连接到follower02在这个时间还没法读取到。

所以zk不是强一致的，不是说leader必须保证一条数据被全部follower都commit了才会让你读取到数据，而是过程中可能你会在不同的follower上读取到不一致的数据，但是最终一定会全部commit后一致，让你读到一致的数据的。

zk官方给自己的定义：顺序一致性。

因此zk是最终一致性的，但是其实他比最终一致性更好一点，因为leader一定会保证所有的proposal同步到follower上都是按照顺序来走的，起码顺序不会乱。

但是全部follower的数据一致确实是最终才能实现一致的。

如果要求强一致性，可以手动调用zk的sync()操作。

## 12.ZooKeeper特性

集群模式部署 :一般奇数节点，因为你5台机器可以挂2台，6台机器也是挂2台，不能超过一半的机器挂掉，所以5台和6台效果一致，那奇数节点可以减少机器开销，小集群部署，读多写少。

主从架构：Leader、Follower、Observer

内存数据模型：znode，多种节点类型。

客户端跟zk进行长连接，TCP，心跳，维持session。

zxid，高32位，低32位。

ZAB协议，2PC，过半ack + 磁盘日志写，commit + 写内存数据结构。

支持Watcher机制，监听回调通知。

顺序一致性：消息按顺序同步，但是最终才会一致，不是强一致。

高性能，2PC中的过半写机制，纯内存的数据结构，znode。

高可用，follower宕机没影响，leader宕机有数据不一致问题，新选举的leader会自动处理，正常运行，但是在恢复模式期间，可能有一小段时间是没法写入zk的。

高并发，单机leader写，Observer可以线性扩展读QPS。

### 13.zookeeper 有几种部署模式？

zookeeper 有三种部署模式： 单机部署：一台集群上运行； 集群部署：多台集群运行； 伪集群部署：一台集群启动多个 zookeeper 实例运行。

### 14.说一下 zookeeper 的通知机制？

客户端端会对某个 znode 建立一个 watcher 事件，当该 znode 发生变化时，这些客户端会收到zookeeper 的通知，然后客户端可以根据 znode 变化来做出业务上的改变。

## 15.服务提供者能实现失效踢出是什么原理？

基于zookeeper的临时节点原理 持久节点 所谓持久节点,是指在节点创建后,就一直存在,直到有删除操作来主动清除这个节点,也就是说不会因为创建该节点的客户端会话失效而消失 临时节点 临时节点的生命周期和客户端会话绑定,也就是说,如果客户端会话失效,那么这个节点就会自动被清除掉

## 16.Zookeeper 文件系统

Zookeeper 提供一个多层级的节点命名空间（节点称为znode）。与文件系统不同的是， 这些节点都可以设置关联的数据，而文件系统中只有文件节点可以存放 数据而目录节点不行。 Zookeeper 为了保证高吞吐和低延迟， 在内存中维护了这个树状的目录结构， 这种特性使得Zookeeper 不能用于存放大量的数据， 每个节点的存放数据上限为1M。

## 17.分布式一致性特性

ZooKeeper 是一个开放源码的分布式协调服务， 它是集群的管理者，监视着集群中各个节点的状态根据节点提交的反馈进行下一步合理操作。最终， 将简单易用 的接口和性能高效、功能稳定的系统提供给用户。 分布式应用程序可以基于Zookeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等能。 Zookeeper 保证了如下分布式一致性特性：

1、顺序一致性 2、原子性 3、单一视图 4、可靠性 5、实时性（最终一致性） 客户端的读请求可以被集群中的任意一台机器处理，如果读请求在节点上注册了监听器，这个监听器也是由所连接的zookeeper 机器来处理。对于写请求，这些请求会同时发给其他zookeeper 机器并且达成一致后，请求才会返回成功。因此，随着zookeeper 的集群机器增多，读请求的吞吐会提高但是写请求的吞吐会下降。 有序性是zookeeper 中非常重要的一个特性，所有的更新都是全局有序的，每个更新都有一个唯一的时间戳，这个时间戳称为zxid（Zookeeper Transaction Id）。而读请求只会相对于更新有序，也就是读请求的返回结果中会带有这个zookeeper 最新的zxid。

## 18.Zookeeper Watcher 机制-- 数据变更通知

Zookeeper 允许客户端向服务端的某个Znode 注册一个Watcher 监听，当服务端的一些指定事件触发了这个Watcher，服务端会向指定客户端发送一个事件通知来实现分布式的通知功能，然后客户端根据Watcher 通知状态和事件类型做出业务上的改变。 工作机制： 1、客户端注册watcher 2、服务端处理watcher 3、客户端回调watcher Watcher 特性总结： 1、一次性 无论是服务端还是客户端，一旦一个Watcher 被触发，Zookeeper 都会将其从相应的存储中移除。这样的设计有效的减轻了服务端的压力， 不然对于更新非常频繁的节点，服务端会不断的向客户端发送事件通知，无论对于网络还是服务端的压力都非常大。 2、客户端串行执行 客户端Watcher 回调的过程是一个串行同步的过程。 3、轻量

3.1、Watcher 通知非常简单，只会告诉客户端发生了事件，而不会说明事件的具体内容。 3.2、客户端向服务端注册Watcher 的时候， 并不会把客户端真实的Watcher 对象实体传递到服务端，仅仅是在客户端请求中使用boolean 类型属性进行了标记。 4、watcher event 异步发送watcher 的通知事件从server 发送到client 是异步的，这就存在一个问题，不同的客户端和服务器之间通过socket 进行通信，由于 网络延迟或其他因素导致客户端在不通的时刻监听到事件，由于Zookeeper 本身提供了ordering guarantee，即客户端监听事件后，才会感知它所监视znode发生了变化。所以我们使用Zookeeper 不能期望能够监控到节点每次的变化。Zookeeper 只能保证最终的一致性，而无法保证强一致性。 5、注册watcher getData、exists、getChildren 6、触发watcher create、delete、setData 7、当一个客户端连接到一个新的服务器上时，watch 将会被以任意会话事件触发。当与一个服务器失去连接的时候，是无法接收到watch 的。而当client 重新连接时， 如果需要的话，所有先前注册过的watch，都会被重新注册。通常这是完全透明的。只有在一个特殊情况下，watch 可能会丢失： 对于一个未创建的znode的exist watch，如果在客户端断开连接期间被创建了，并且随后在客户端连接上之前又删除了，这种情况下，这个watch 事件可能会被丢失。

## 19.客户端注册Watcher 实现

1、调用getData()/getChildren()/exist()三个API，传入Watcher 对象 2、标记请求request，封装Watcher 到WatchRegistration 3、封装成Packet 对象，发服务端发送request 4、收到服务端响应后，将Watcher 注册到ZKWatcherManager 中进行管理 5、请求返回，完成注册。

## 20.服务端处理Watcher 实现

1、服务端接收Watcher 并存储 接收到客户端请求，处理请求判断是否需要注册Watcher，需要的话将数据节点的节点路径和ServerCnxn（ ServerCnxn 代表一个客户端和服务端的连接，实现了Watcher 的process 接口，此时可以看成一个Watcher 对象）存储在WatcherManager 的WatchTable 和watch2Paths 中去。 2、Watcher 触发 以服务端接收到setData() 事务请求触发NodeDataChanged 事件为例： 2.1 封装WatchedEvent 将通知状态（ SyncConnected）、事件类型（ NodeDataChanged）以及节点路径封装成一个WatchedEvent 对象 2.2 查询Watcher 从WatchTable 中根据节点路径查找Watcher 2.3 没找到； 说明没有客户端在该数据节点上注册过Watcher 2.4 找到；提取并从WatchTable 和Watch2Paths 中删除对应Watcher（ 从这里可以看出Watcher 在服务端是一次性的， 触发一次就失效了） 3、调用process 方法来触发Watcher 这里process 主要就是通过ServerCnxn 对应的TCP 连接发送Watcher 事件通知。

## 21.客户端回调Watcher

客户端SendThread 线程接收事件通知，交由EventThread 线程回调Watcher。

客户端的Watcher 机制同样是一次性的， 一旦被触发后，该Watcher 就失效了。

## 22.Zookeeper 下Server 工作状态

服务器具有四种状态，分别是LOOKING、FOLLOWING、LEADING、OBSERVING。

1、LOOKING：寻找Leader 状态。当服务器处于该状态时，它会认为当前集群中没有Leader，因此需要进入Leader 选举状态。 2、FOLLOWING：跟随者状态。表明当前服务器角色是Follower。 3、LEADING：领导者状态。表明当前服务器角色是Leader。 4、OBSERVING：观察者状态。表明当前服务器角色是Observer。

## 23.数据同步

Zookeeper 的数据同步通常分为四类： 1、直接差异化同步（ DIFF 同步） 2、先回滚再差异化同步（ TRUNC+DIFF 同步） 3、仅回滚同步（ TRUNC 同步） 4、全量同步（ SNAP 同步）

进行数据同步前，Leader 服务器会完成数据同步初始化： peerLastZxid：

从learner 服务器注册时发送的ACKEPOCH 消息中提取lastZxid（该Learner 服务器最后处理的ZXID）

minCommittedLog： Leader 服务器Proposal 缓存队列committedLog 中最小ZXID

maxCommittedLog： Leader 服务器Proposal 缓存队列committedLog 中最大ZXID

直接差异化同步（ DIFF 同步）

场景：peerLastZxid 介于minCommittedLog 和maxCommittedLog之间

先回滚再差异化同步（ TRUNC+DIFF 同步）

场景：当新的Leader 服务器发现某个Learner 服务器包含了一条自己没有的事务记录，那么就需要让该Learner 服务器进行事务回滚--回滚到Leader服务器上存在的，同时也是最接近于peerLastZxid 的ZXID

仅回滚同步（ TRUNC 同步）

场景：peerLastZxid 大于maxCommittedLog

全量同步（SNAP 同步）

场景一：peerLastZxid 小于minCommittedLog 场景二：Leader 服务器上没有Proposal 缓存队列且peerLastZxid 不等于lastProcessZxid

## 24.如何保证事务的顺序一致性的？

zookeeper 采用了全局递增的事务Id 来标识， 所有的proposal（提议） 都在被提出的时候加上了zxid，zxid 实际上是一个64 位的数字，高32 位是epoch（ 时期; 纪元; 世; 新时代）用来标识leader 周期，如果有新的leader 产生出来，epoch会自增， 低32 位用来递增计数。当新产生proposal 的时候， 会依据数据库的两阶段过程，首先会向其他的server 发出事务执行请求，如果超过半数的机器都能执行并且能够成功，那么就会开始执行。

## 25.zk 节点宕机如何处理？

Zookeeper 本身也是集群，推荐配置不少于3 个服务器。Zookeeper 自身也要保证当一个节点宕机时，其他节点会继续提供服务。 如果是一个Follower 宕机，还有2 台服务器提供访问，因为Zookeeper 上的数据是有多个副本的，数据并不会丢失； 如果是一个Leader 宕机，Zookeeper 会选举出新的Leader。 ZK 集群的机制是只要超过半数的节点正常， 集群就能正常提供服务。只有在ZK节点挂得太多，只剩一半或不到一半节点能工作， 集群才失效。 所以 3 个节点的cluster 可以挂掉1 个节点(leader 可以得到2 票>1.5) 2 个节点的cluster 就不能挂掉任何1 个节点了(leader 可以得到1 票<=1)

## 26.Zookeeper 对节点的watch 监听通知是永久的吗？为什么不是永久的?

不是。官方声明：一个Watch 事件是一个一次性的触发器，当被设置了Watch的数据发生了改变的时候，则服务器将这个改变发送给设置了Watch 的客户端，以便通知它们。 为什么不是永久的，举个例子，如果服务端变动频繁， 而监听的客户端很多情况下， 每次变动都要通知到所有的客户端， 给网络和服务器造成很大压力。 一般是客户端执行getData(“ /节点A” ,true)，如果节点A 发生了变更或删除，客户端会得到它的watch 事件，但是在之后节点A 又发生了变更，而客户端又没有设置watch 事件，就不再给客户端发送。 在实际应用中，很多情况下，我们的客户端不需要知道服务端的每一次变动， 我只要最新的数据即可。

## 27.ACL 权限控制机制

UGO（User/Group/Others） 目前在 Linux/Unix 文件系统中使用，也是使用最广泛的权限控制方式。是一种粗粒度的文件系统权限控制模式。 ACL（Access Control List）访问控制列表包括三个方面： 权限模式（Scheme） 1、IP：从 IP 地址粒度进行权限控制 2、Digest：最常用，用类似于 username:password 的权限标识来进行权限配置，便于区分不同应用来进行权限控制 3、World：最开放的权限控制方式，是一种特殊的 digest 模式，只有一个权限标识“world:anyone” 4、Super：超级用户 授权对象 授权对象指的是权限赋予的用户或一个指定实体，例如 IP 地址或是机器灯。 权限 Permission 1、CREATE：数据节点创建权限，允许授权对象在该 Znode 下创建子节点 2、DELETE：子节点删除权限，允许授权对象删除该数据节点的子节点 3、READ：数据节点的读取权限，允许授权对象访问该数据节点并读取其数据内容或子节点列表等 4、WRITE：数据节点更新权限，允许授权对象对该数据节点进行更新操作 5、ADMIN：数据节点管理权限，允许授权对象对该数据节点进行 ACL 相关设置操作

## 28.会话管理

分桶策略：将类似的会话放在同一区块中进行管理，以便于 Zookeeper 对会话进行不同区块的隔离处理以及同一区块的统一处理。 分配原则：每个会话的“下次超时时间点”（ExpirationTime）

## 29.集群支持动态添加机器吗？

其实就是水平扩容了，Zookeeper 在这方面不太好。两种方式： 全部重启：关闭所有 Zookeeper 服务，修改配置之后启动。不影响之前客户端的会话。 逐个重启：在过半存活即可用的原则下，一台机器重启不影响整个集群对外提供服务。这是比较常用的方式。 3.5 版本开始支持动态扩容

## 30.zk 的命名服务

命名服务是指通过指定的名字来获取资源或者服务的地址，利用 zk 创建一个全局的路径，这个路径就可以作为一个名字，指向集群中的集群，提供的服务的地址，或者一个远程的对象等等。

## 31.分布式通知和协调

对于系统调度来说：操作人员发送通知实际是通过控制台改变某个节点的状态，然后 zk 将这些变化发送给注册了这个节点的 watcher 的所有客户端。 对于执行情况汇报：每个 工作进程都在某个目录下创建一个临时节点。并携带工作的进度数据，这样汇总的进程可以监控目录子节点的变化获得工作进度的实时的全局情况。

## 32.事务编号 Zxid（事务请求计数器+ epoch）

在 ZAB ( ZooKeeper Atomic Broadcast , ZooKeeper 原子消息广播协议） 协议的事务编号 Zxid设计中， Zxid 是一个 64 位的数字，其中低 32 位是一个简单的单调递增的计数 器， 针对客户端每一个事务请求，计数器加 1；而高 32 位则代表 Leader 周期 epoch 的编号， 每个当选产生一个新的 Leader 服务器，就会从这个 Leader 服务器上取出其本地 日志中最大事务的 ZXID，并从中读取epoch 值，然后加 1，以此作为新的 epoch，并将低 32 位从 0 开始计数。Zxid（Transaction id） 类似于 RDBMS 中的事务 ID，用于标识 一次更新操作的 Proposal（提议） ID。为了保证顺序性，该 zkid 必须单调递增。

## 33.zk 的配置管理（文件系统、通知机制）

程序分布式的部署在不同的机器上，将程序的配置信息放在 zk 的 znode 下，当有配置发生改变时，也就是 znode 发生变化时，可以通过改变 zk 中某个目录节点的内容，利用 watcher 通知给各个客户端，从而更改配置。

## 34.Zookeeper 集群管理（文件系统、通知机制）

所谓集群管理无在乎两点：是否有机器退出和加入、选举 master。对于第一点，所有机器约定在父目录下创建临时目录节点，然后监听父目录节点的子节点变化消息。一旦有机 器挂掉，该机器与 zookeeper 的连接断开，其所创建的临时目录节点被删除，所有其他机器都收到通知：某个兄弟目录被删除，于是，所有人都知道：它上船了。 新机器加入也是类似，所有机器收到通知：新兄弟目录加入，highcount 又有了，对于第二点，我们稍微改变一下，所有机器创建临时顺序编号目录节点，每次选取编号最小的 机器作为 master 就好。

## 35.Zookeeper 分布式锁（文件系统、通知机制）

有了 zookeeper 的一致性文件系统，锁的问题变得容易。锁服务可以分为两类，一个是保持独占，另一个是控制时序 对于第一类，我们将 zookeeper 上的一个 znode 看作是一 把锁，通过 createznode的方式来实现。所有客户端都去创建 /distribute_lock 节点，最终成功创建的那个客户端也即拥有了这把锁。用完删除掉自己创建的 distribute_lock 节 点就释放出锁。 对于第二类， /distribute_lock 已经预先存在，所有客户端在它下面创建临时顺序编号目录节点，和选 master 一样，编号最小的获得锁，用完删除，依次方便。

### 35.获取分布式锁的流程

在获取分布式锁的时候在locker节点下创建临时顺序节点，释放锁的时候删除该临时节点。客⼾端调⽤createNode⽅法在locker 下创建临时顺序节点，然后调⽤getChildren(“locker”)来获取locker下⾯的所有⼦节点，注意此时不⽤设置任何Watcher。 客⼾端获取到所有的⼦节点path之后，如果发现⾃⼰创建的节点在所有创建的⼦节点序号最⼩，那么就认为该客⼾端获取到了 锁。如果发现⾃⼰创建的节点并⾮locker所有⼦节点中最⼩的，说明⾃⼰还没有获取到锁，此时客⼾端需要找到⽐⾃⼰⼩的那个 节点，然后对其调⽤exist()⽅法，同时对其注册事件监听器。 之后，让这个被关注的节点删除，则客⼾端的Watcher会收到相应通知，此时再次判断⾃⼰创建的节点是否是locker⼦节点中序 号最⼩的，如果是则获取到了锁，如果不是则重复以上步骤继续获取到⽐⾃⼰⼩的⼀个节点并注册监听。当前这个过程中还需要 许多的逻辑判断。

代码的实现主要是基于互斥锁，获取分布式锁的重点逻辑在于BaseDistributedLock，实现了基于Zookeeper实现分布式锁的细 节。

## 36.Zookeeper 队列管理（文件系统、通知机制）

两种类型的队列： 1、同步队列，当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达。 2、队列按照 FIFO 方式进行入队和出队操作。 第一类，在约定目录下创建临时目录节点，监听节点数目是否是我们要求的数目。 第二类，和分布式锁服务中的控制时序场景基本原理一致，入列有编号，出列按编号。在特定的目录下创建 PERSISTENT_SEQUENTIAL 节点，创建成功时Watcher 通知等待的队 列，队列删除序列号最小的节点用以消费。此场景下Zookeeper 的 znode 用于消息存储，znode 存储的数据就是消息队列中的消息内容，SEQUENTIAL 序列号就是消息的编 号，按序取出即可。由于创建的节点是持久化的，所以不必担心队列消息的丢失问题。

## 37.Zab 协议有两种模式-恢复模式（选主）、广播模式（同步）

Zab 协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步） 。当服务启动或者在领导者崩溃后， Zab 就进入了恢复模式，当领导者被选举出来，且大多数 Server 完成了和 leader 的状态同步以后，恢复模式就结束了。状态同步保证了 leader 和 Server 具有相同的系统状态。

## 38.**Zookeeper对节点的 watch监听通知是永久的吗?为什么不是永久的?**

不是永久的。 官方声明：一个 Watch事件是一个次性的触发器，当被设置了 Watch的数据发生了改变的时候，则服务器将这个改变发送给设置了 Watch的客户端，以便通知它们。

原因： 如果服务端变动频繁，而监听的客户端很多情况下，每次变动都要通知到所有的客户端，给网络和服务器造成很大压力。一般是客户端执行`getData`（“/节点”，true），如果节点A发生了变更或删除，客户端会得到它的 watch事件，但是在之后节点A又发生了变更，而客户端又没有设置 watch事件，就不再给客户端发送。

> 在实际应用中，很多情况下，我们的客户端不需要知道服务端的每一次变动，我只要最新的数据即可。

## 39.**Zookeeper 下 Server工作状态有哪些？**

1、`LOOKING`：寻找Leader状态。当服务器处于该状态时，它会认为当前集群中没有Leader，因此需要进入Leader选举状态。 2、`FOLLOWING`：跟随者状态。表明当前服务器角色是Follower。 3、`LEADING`：领导者状态。表明当前服务器角色是Leader。 4、`OBSERVING`：观察者状态。表明当前服务器角色是Observer。

## 40.ACL 权限控制机制

UGO（User/Group/Others） 目前在 Linux/Unix 文件系统中使用，也是使用最广泛的权限控制方式。是一种粗粒度的文件系统权限 控制模式。 ACL（Access Control List）访问控制列表 包括三个方面： 权限模式（Scheme） （1）IP：从 IP 地址粒度进行权限控制 （2）Digest：最常用，用类似于 username:password 的权限标识来进行权限配置，便于区分不同应 用来进行权限控制 （3）World：最开放的权限控制方式，是一种特殊的 digest 模式，只有一个权限标识“world:anyone” （4）Super：超级用户 授权对象 授权对象指的是权限赋予的用户或一个指定实体，例如 IP 地址或是机器灯。 权限 Permission （1）CREATE：数据节点创建权限，允许授权对象在该 Znode 下创建子节点 （2）DELETE：子节点删除权限，允许授权对象删除该数据节点的子节点 （3）READ：数据节点的读取权限，允许授权对象访问该数据节点并读取其数据内容或子节点列表等 （4）WRITE：数据节点更新权限，允许授权对象对该数据节点进行更新操作 （5）ADMIN：数据节点管理权限，允许授权对象对该数据节点进行 ACL 相关设置操作

## 41.服务器角色

Leader （1）事务请求的唯一调度和处理者，保证集群事务处理的顺序性 （2）集群内部各服务的调度者 Follower （1）处理客户端的非事务请求，转发事务请求给 Leader 服务器 （2）参与事务请求 Proposal 的投票 （3）参与 Leader 选举投票 Observer （1）3.0 版本以后引入的一个服务器角色，在不影响集群事务处理能力的基础上提升集群的非事务处理 能力 （2）处理客户端的非事务请求，转发事务请求给 Leader 服务器 （3）不参与任何形式的投票

## 42.zookeeper 负载均衡和 nginx 负载均衡区别

zk 的负载均衡是可以调控，nginx 只是能调权重，其他需要可控的都需要自己写插件；但是 nginx 的吞 吐量比 zk 大很多，应该说按业务选择用哪种方式。

## 43.Zookeeper 的典型应用场景

Zookeeper 是一个典型的发布/订阅模式的分布式数据管理与协调框架，开发人员可以使用它来进行分 布式数据的发布和订阅。 通过对 Zookeeper 中丰富的数据节点进行交叉使用，配合 Watcher 事件通知机制，可以非常方便的构 建一系列分布式应用中年都会涉及的核心功能，如： （1）数据发布/订阅 （2）负载均衡 （3）命名服务 （4）分布式协调/通知 （5）集群管理 （6）Master 选举 （7）分布式锁 （8）分布式队列 数据发布/订阅 介绍 数据发布/订阅系统，即所谓的配置中心，顾名思义就是发布者发布数据供订阅者进行数据订阅。 目的 动态获取数据（配置信息） 实现数据（配置信息）的集中式管理和数据的动态更新 设计模式 Push 模式

Pull 模式 数据（配置信息）特性 （1）数据量通常比较小 （2）数据内容在运行时会发生动态更新 （3）集群中各机器共享，配置一致 如：机器列表信息、运行时开关配置、数据库配置信息等 基于 Zookeeper 的实现方式 · 数据存储：将数据（配置信息）存储到 Zookeeper 上的一个数据节点 · 数据获取：应用在启动初始化节点从 Zookeeper 数据节点读取数据，并在该节点上注册一个数据变更 Watcher · 数据变更：当变更数据时，更新 Zookeeper 对应节点数据，Zookeeper会将数据变更通知发到各客户 端，客户端接到通知后重新读取变更后的数据即可。 负载均衡 zk 的命名服务 命名服务是指通过指定的名字来获取资源或者服务的地址，利用 zk 创建一个全局的路径，这个路径就 可以作为一个名字，指向集群中的集群，提供的服务的地址，或者一个远程的对象等等。 分布式通知和协调 对于系统调度来说：操作人员发送通知实际是通过控制台改变某个节点的状态，然后 zk 将这些变化发 送给注册了这个节点的 watcher 的所有客户端。 对于执行情况汇报：每个工作进程都在某个目录下创建一个临时节点。并携带工作的进度数据，这样汇 总的进程可以监控目录子节点的变化获得工作进度的实时的全局情况。 zk 的命名服务（文件系统） 命名服务是指通过指定的名字来获取资源或者服务的地址，利用 zk 创建一个全局的路径，即是唯一的 路径，这个路径就可以作为一个名字，指向集群中的集群，提供的服务的地址，或者一个远程的对象等 等。 zk 的配置管理（文件系统、通知机制） 程序分布式的部署在不同的机器上，将程序的配置信息放在 zk 的 znode 下，当有配置发生改变时，也 就是 znode 发生变化时，可以通过改变 zk 中某个目录节点的内容，利用 watcher 通知给各个客户端， 从而更改配置。 Zookeeper 集群管理（文件系统、通知机制） 所谓集群管理无在乎两点：是否有机器退出和加入、选举 master。 对于第一点，所有机器约定在父目录下创建临时目录节点，然后监听父目录节点 的子节点变化消息。一旦有机器挂掉，该机器与 zookeeper 的连接断开，其所创建的临时目录节点被 删除，所有其他机器都收到通知：某个兄弟目录被删除，于是，所有人都知道：它上船了。 新机器加入也是类似，所有机器收到通知：新兄弟目录加入，highcount 又有了，对于第二点，我们稍 微改变一下，所有机器创建临时顺序编号目录节点，每次选取编号最小的机器作为 master 就好。 Zookeeper 分布式锁（文件系统、通知机制）

有了 zookeeper 的一致性文件系统，锁的问题变得容易。锁服务可以分为两类，一个是保持独占，另 一个是控制时序。 对于第一类，我们将 zookeeper 上的一个 znode 看作是一把锁，通过 createznode的方式来实现。所 有客户端都去创建 /distribute_lock 节点，最终成功创建的那个客户端也即拥有了这把锁。用完删除掉 自己创建的 distribute_lock 节点就释放出锁。 对于第二类， /distribute_lock 已经预先存在，所有客户端在它下面创建临时顺序编号目录节点，和选 master 一样，编号最小的获得锁，用完删除，依次方便。 Zookeeper 队列管理（文件系统、通知机制） 两种类型的队列： （1）同步队列，当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达。 （2）队列按照 FIFO 方式进行入队和出队操作。 第一类，在约定目录下创建临时目录节点，监听节点数目是否是我们要求的数目。 第二类，和分布式锁服务中的控制时序场景基本原理一致，入列有编号，出列按编号。在特定的目录下 创建 PERSISTENT_SEQUENTIAL 节点，创建成功时Watcher 通知等待的队列，队列删除序列号最小的 节点用以消费。此场景下Zookeeper 的 znode 用于消息存储，znode 存储的数据就是消息队列中的消 息内容，SEQUENTIAL 序列号就是消息的编号，按序取出即可。由于创建的节点是持久化的，所以不必 担心队列消息的丢失问题。

## 44.Zookeeper数据复制

Zookeeper作为⼀个集群提供⼀致的数据服务，⾃然，它要在所有机器间做数据复制。数据复制的好处： 容错：⼀个节点出错，不致于让整个系统停⽌⼯作，别的节点可以接管它的⼯作； 提⾼系统的扩展能⼒ ：把负载分布到多个节点上，或者增加节点来提⾼系统的负载能⼒； 提⾼性能：让客⼾端本地访问就近的节点，提⾼⽤⼾访问速度。 从客⼾端读写访问的透明度来看，数据复制集群系统分下⾯两种： 写主(WriteMaster) ：对数据的修改提交给指定的节点。读⽆此限制，可以读取任何⼀个节点。这种情况下客⼾端需要对读 与写进⾏区别，俗称读写分离； 写任意(Write Any)：对数据的修改可提交给任意的节点，跟读⼀样。这种情况下，客⼾端对集群节点的⻆⾊与变化透明。 对zookeeper来说，它采⽤的⽅式是写任意。通过增加机器，它的读吞吐能⼒和响应能⼒扩展性⾮常好，⽽写，随着机器的增多 吞吐能⼒肯定下降（这也是它建⽴observer的原因），⽽响应能⼒则取决于具体实现⽅式，是延迟复制保持最终⼀致性，还是⽴ 即复制快速响应。

## 45.zookeeper是如何选取主leader的？

当leader崩溃或者leader失去⼤多数的follower，这时zk进⼊恢复模式，恢复模式需要重新选举出⼀个新的leader，让所有的 Server都恢复到⼀个正确的状态。Zk的选举算法有两种：⼀种是基于basic paxos实现的，另外⼀种是基于fast paxos算法实现 的。系统默认的选举算法为fast paxos。 1、Zookeeper选主流程(basic paxos) （1）选举线程由当前Server发起选举的线程担任，其主要功能是对投票结果进⾏统计，并选出推荐的Server； （2）选举线程⾸先向所有Server发起⼀次询问(包括⾃⼰)； （3）选举线程收到回复后，验证是否是⾃⼰发起的询问(验证zxid是否⼀致)，然后获取对⽅的id(myid)，并存储到当前询问对象 列表中，最后获取对⽅提议的leader相关信息(id,zxid)，并将这些信息存储到当次选举的投票记录表中； （4）收到所有Server回复以后，就计算出zxid最⼤的那个Server，并将这个Server相关信息设置成下⼀次要投票的Server； （5）线程将当前zxid最⼤的Server设置为当前Server要推荐的Leader，如果此时获胜的Server获得n/2 + 1的Server票数，设置 当前推荐的leader为获胜的Server，将根据获胜的Server相关信息设置⾃⼰的状态，否则，继续这个过程，直到leader被选举出 来。通过流程分析我们可以得出：要使Leader获得多数Server的⽀持，则Server总数必须是奇数2n+1，且存活的Server的数⽬ 不得少于n+1. 每个Server启动后都会重复以上流程。在恢复模式下，如果是刚从崩溃状态恢复的或者刚启动的server还会从磁盘 快照中恢复数据和会话信息，zk会记录事务⽇志并定期进⾏快照，⽅便在恢复时进⾏状态恢复。

2、Zookeeper选主流程(basic paxos) fast paxos流程是在选举过程中，某Server⾸先向所有Server提议⾃⼰要成为leader，当其它Server收到提议以后，解决epoch 和 zxid的冲突，并接受对⽅的提议，然后向对⽅发送接受提议完成的消息，重复这个流程，最后⼀定能选举出Leader。

## 46.Zookeeper同步流程

选完Leader以后，zk就进⼊状态同步过程。 Leader等待server连接； Follower连接leader，将最⼤的zxid发送给leader； Leader根据follower的zxid确定同步点； 完成同步后通知follower 已经成为uptodate状态； Follower收到uptodate消息后，⼜可以重新接受client的请求进⾏服务了。

## 47.zookeeper watch机制

Watch机制官⽅声明：⼀个Watch事件是⼀个⼀次性的触发器，当被设置了Watch的数据发⽣了改变的时候，则服务器将这个改 变发送给设置了Watch的客⼾端，以便通知它们。 Zookeeper机制的特点： 1、⼀次性触发数据发⽣改变时，⼀个watcher event会被发送到client，但是client只会收到⼀次这样的信息。 2、watcher event异步发送watcher的通知事件从server发送到client是异步的，这就存在⼀个问题，不同的客⼾端和服务器之 间通过socket进⾏通信，由于⽹络延迟或其他因素导致客⼾端在不通的时刻监听到事件，由于Zookeeper本⾝提供了ordering guarantee，即客⼾端监听事件后，才会感知它所监视znode发⽣了变化。所以我们使⽤Zookeeper不能期望能够监控到节点每 次的变化。Zookeeper只能保证最终的⼀致性，⽽⽆法保证强⼀致性。 3、数据监视Zookeeper有数据监视和⼦数据监视getdata() and exists()设置数据监视，getchildren()设置了⼦节点监视。 4、注册watcher getData、exists、getChildren 5、触发watcher create、delete、setData 6、setData()会触发znode上设置的data watch（如果set成功的话）。⼀个成功的create() 操作会触发被创建的znode上的数 据watch，以及其⽗节点上的child watch。⽽⼀个成功的delete()操作将会同时触发⼀个znode的data watch和child watch（因为这样就没有⼦节点了），同时也会触发其⽗节点的child watch。 7、当⼀个客⼾端连接到⼀个新的服务器上时，watch将会被以任意会话事件触发。当与⼀个服务器失去连接的时候，是⽆法接收 到watch的。⽽当client重新连接时，如果需要的话，所有先前注册过的watch，都会被重新注册。通常这是完全透明的。只有在 ⼀个特殊情况下，watch可能会丢失：对于⼀个未创建的znode的exist watch，如果在客⼾端断开连接期间被创建了，并且随后 在客⼾端连接上之前⼜删除了，这种情况下，这个watch事件可能会被丢失。 8、Watch是轻量级的，其实就是本地JVM的Callback，服务器端只是存了是否有设置了Watcher的布尔类型