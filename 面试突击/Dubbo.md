# Dubbo

### 1.什么是Dubbo？

Dubbo 是阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和Spring 框架无缝集成。Dubbo 框架，是基于容器运行的.。容器是Spring。

其核心部分包含:

1. 远程通讯: 提供对多种基于长连接的NIO 框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式。
2. 集群容错: 提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
3. 自动发现: 基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

Dubbo 能做什么？ 1.透明化的远程方法调用，就像调用本地方法一样调用远程方法，只需简单配置，没有任何API 侵入。 2.软负载均衡及容错机制，可在内网替代F5 等硬件负载均衡器，降低成本，减少单点。

3.服务自动注册与发现，不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的IP 地址，并且能够平滑添加或删除服务提供者。 Dubbo 的存在简单来说就是要减小service 层的压力。

### 2.什么是RPC 远程过程调用？

远程过程调用协议，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC 协议假定某些传输协议的存在，如TCP 或UDP，为通信程序之间携带信息数据。在OSI 网络通信模型中，RPC 跨越了传输层和应用层。RPC 使得开发包括网络分布式多程序在内的应用程序更加容易。

### 3.Dubbo 集群容错策略

Failover Cluster 模式

失败自动切换，自动重试其他机器，**默认**就是这个，常见于读操作。（失败重试其它机器）

可以通过以下几种方式配置重试次数：

```html
<dubbo:service retries="2" />
```

或者

```html
<dubbo:reference retries="2" />
```

或者

```html
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```

Failfast Cluster 模式

一次调用失败就立即失败，常见于非幂等性的写操作，比如新增一条记录（调用失败就立即失败）

Failsafe Cluster 模式

出现异常时忽略掉，常用于不重要的接口调用，比如记录日志。

配置示例如下：

```html
<dubbo:service cluster="failsafe" />
```

或者

```html
<dubbo:reference cluster="failsafe" />
```

Failback Cluster 模式

失败了后台自动记录请求，然后定时重发，比较适合于写消息队列这种。

Forking Cluster 模式

**并行调用**多个 provider，只要一个成功就立即返回。常用于实时性要求比较高的读操作，但是会浪费更多的服务资源，可通过 `forks="2"` 来设置最大并行数。

Broadcast Cluster 模式

逐个调用所有的 provider。任何一个 provider 出错则报错（从 `2.1.0` 版本开始支持）。通常用于通知所有提供者更新缓存或日志等本地资源信息。

总结：

在实际应用中查询语句容错策略建议使用默认 Failover Cluster ，而增删改建议使用 Failfast Cluster 或者 使用 Failover Cluster retries= 0 策略 防止出现数据 重复添加等等其它问题！建议在设计接口时候把查询接口方法单独做一个接口 提供查询。

### 4.Dubbo 的连接方式有哪些？

Dubbo的客户端和服务端有三种连接方式，分别是：广播，直连和使用 zookeeper 注册中心。

Dubbo广播

官方入门程序所使用的连接方式，但是这种方式有很多问题。在企业开发中，不使用广播的方式。

服务端配置：

```html
使用 multicast 广播暴露服务地址
<dubbo:registry address="multicast://224.5.6.7:1234"/>
```

客户端配置：

```html
使用 multicast 广播暴露服务地址
<dubbo:registry address="multicast://224.5.6.7:1234"/>
```

Dubbo直连

这种方式在企业中一般在开发中环境中使用，但是生产环境很少使用，因为服务是直接调用，没有使用注册中心，很难对服务进行管理。 Dubbo 直连，首先要取消广播，然后客户端直接到指定需要的服务的 url 获取服务即可。

服务端配置：取消广播，注册中心地址为 N/A

```html
使用 multicast 广播暴露服务地址
<dubbo:registry address="N/A"/>
```

客户端配置：取消广播，从指定的 url 中获取服务

```html
使用 multicast 广播暴露服务地址
<dubbo:registry address="multicast://224.5.6.7:1234"/>
```

zookeeper注册中心

Dubbo注册中心和广播注册中心配置类似，不过需要指定注册中心类型和注册中心地址，这个时候就不是把服务信息进行广播了，而是告诉给注册中心进行管理，这个时候我们就需要有一个注册中心。

Zookeeper 介绍

zookeeper在 dubbo 所处的位置：

1）Provider: 暴露服务的服务提供方。 2）Consumer: 调用远程服务的服务消费方。 3）Registry: 服务注册与发现的注册中心。 4）Monitor: 统计服务的调用次调和调用时间的监控中心。 5）Container: 服务运行容器。

调用关系说明： 1)服务容器负责启动，加载，运行服务提供者。 2)服务提供者在启动时，向注册中心注册自己提供的服务。 3)服务消费者在启动时，向注册中心订阅自己所需的服务。 4)注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。 5)服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。 6)服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

### 5.Dubbo 中有哪些角色？

registry

注册中心. 是用于发布和订阅服务的一个平台.用于替代SOA 结构体系框架中的ESB服务总线的。

发布

开发服务端代码完毕后, 将服务信息发布出去. 实现一个服务的公开.

订阅

客户端程序,从注册中心下载服务内容 这个过程是订阅.订阅服务的时候, 会将发布的服务所有信息,一次性下载到客户端.客户端也可以自定义, 修改部分服务配置信息. 如: 超时的时长, 调用的重试次数等.

Consumer

服务的消费者, 就是服务的客户端.消费者必须使用Dubbo 技术开发部分代码. 基本上都是配置文件定义.

provider

服务的提供者, 就是服务端.服务端必须使用Dubbo 技术开发部分代码. 以配置文件为主.

container

容器. Dubbo 技术的服务端(Provider), 在启动执行的时候, 必须依赖容器才能正常启动. 默认依赖的就是spring 容器. 且Dubbo 技术不能脱离spring 框架. 在2.5.3 版本的dubbo 中, 默认依赖的是spring2.5 版本技术. 可以选用spring4.5 以下版本. 在2.5.7 版本的dubbo 中, 默认依赖的是spring4.3.10 版本技术. 可以选择任意的spring 版本.

monitor

监控中心. 是Dubbo 提供的一个jar 工程.主要功能是监控服务端(Provider)和消费端(Consumer)的使用数据的. 如: 服务端是什么,有多少接口,多少方法, 调用次数, 压力信息等. 客户端有多少, 调用过哪些服务端,调用了多少次等.

### 6.Dubbo 执行流程什么是？

0、start: 启动Spring 容器时,自动启动Dubbo 的Provider。 1、register: Dubbo 的Provider 在启动后自动会去注册中心注册内容.注册的内容包括: 1.1 Provider 的 IP 1.2 Provider 的端口. 1.3 Provider 对外提供的接口列表.哪些方法.哪些接口类 1.4 Dubbo 的版本. 1.5 访问Provider 的协议. 2、subscribe: 订阅.当Consumer 启动时,自动去Registry 获取到所已注册的服务的信息. 3、notify: 通知.当Provider 的信息发生变化时, 自动由Registry 向Consumer 推送通知. 4、invoke: 调用. Consumer 调用Provider 中方法

4.1 同步请求.消耗一定性能.但是必须是同步请求,因为需要接收调用方法后的结果. 5、count:次数. 每隔2 分钟,provoider 和consumer 自动向Monitor 发送访问次数.Monitor 进行统计.

### 7.Dubbo 支持的协议有哪些？

1、Dubbo 协议(官方推荐协议)dubbo://

**默认**就是走 dubbo 协议，单一长连接，进行的是 NIO 异步通信，基于 hessian 作为序列化协议。使用的场景是：传输数据量小（每次请求在 100kb 以内），但是并发量很高，以及服务消费者机器数远大于服务提供者机器数的情况。

为了要支持高并发场景，一般是服务提供者就几台机器，但是服务消费者有上百台，可能每天调用量达到上亿次！此时用长连接是最合适的，就是跟每个服务消费者维持一个长连接就可以，可能总共就 100 个连接。然后后面直接基于长连接 NIO 异步通信，可以支撑高并发请求。

长连接，通俗点说，就是建立连接过后可以持续发送请求，无须再建立连接。

而短连接，每次要发送请求之前，需要先重新建立一次连接。

优点：

采用NIO 复用单一长连接，并使用线程池并发处理请求，减少握手和加大并发效率，性能较好（推荐使用）

缺点：

大文件上传时,可能出现问题(不使用Dubbo 文件上传)

2、RMI(Remote Method Invocation)协议rmi://

RMI 协议采用 JDK 标准的 java.rmi.* 实现，采用阻塞式短连接和 JDK 标准序列化方式。多个短连接，适合消费者和提供者数量差不多的情况，适用于文件的传输，一般较少用。

优点:

JDK 自带的能力。可与原生RMI 互操作，基于TCP 协议

缺点:

偶尔连接失败.

3、Hessian 协议hessian://

Hessian 协议用于集成 Hessian 的服务，Hessian 底层采用 Http 通讯，采用 Servlet 暴露服务，Dubbo 缺省内嵌 Jetty 作为服务器实现。走 hessian 序列化协议，多个短连接，适用于提供者数量比消费者数量还多的情况，适用于文件的传输，一般较少用。

优点:

可与原生Hessian 互操作，基于HTTP 协议

缺点:

需hessian.jar 支持，http 短连接的开销大

4、http 协议 `http://`

基于 HTTP 表单的远程调用协议，采用 Spring 的 HttpInvoker 实现。走表单序列化。

5、thrift 协议 `thrift://`

当前 dubbo 支持的 thrift 协议是对 thrift 原生协议的扩展，在原生协议的基础上添加了一些额外的头信息，比如 service name，magic number 等。

6、webservice `webservice://`

基于 WebService 的远程调用协议，基于 Apache CXF 的 frontend-simple 和 transports-http 实现。走 SOAP 文本序列化。

7、memcached 协议 `memcached://`

基于 memcached 实现的 RPC 协议。

8、redis 协议 `redis://`

基于 Redis 实现的 RPC 协议。

9、rest 协议 `rest://`

基于标准的 Java REST API——JAX-RS 2.0（Java API for RESTful Web Services 的简写）实现的 REST 调用支持。

10、gPRC 协议 `grpc://`

Dubbo 自 2.7.5 版本开始支持 gRPC 协议，对于计划使用 HTTP/2 通信，或者想利用 gRPC 带来的 Stream、反压、Reactive 编程等能力的开发者来说， 都可以考虑启用 gRPC 协议。

### 8.Dubbo 支持的注册中心有哪些？

1、Zookeeper(官方推荐)

优点:支持分布式.很多周边产品. 缺点: 受限于Zookeeper 软件的稳定性.Zookeeper 专门分布式辅助软件,稳定较优

2、Multicast

优点:去中心化,不需要单独安装软件. 缺点:Provider 和Consumer 和Registry 不能跨机房(路由)

3、Redis

优点:支持集群,性能高 缺点:要求服务器时间同步.否则可能出现集群失败问题.

4、Simple

优点: 标准RPC 服务.没有兼容问题 缺点: 不支持集群.

### 9.dubbo 工作原理

- 第一层：service 层，接口层，给服务提供者和消费者来实现的

该层与业务逻辑相关，根据provider 和consumer 的业务设计对应的接口和实现

- 第二层：config 层，配置层，主要是对 dubbo 进行各种配置的

对外配置接口，以ServiceConfig 和ReferenceConfig 为中心

- 第三层：proxy 层，服务代理层，无论是 consumer 还是 provider，dubbo 都会给你生成代理，代理之间进行网络通信

服务接口透明代理，生成服务的客户端Stub 和服务端的Skeleton，以ServiceProxy 为中心， 扩展接口为ProxyFactory

- 第四层：registry 层，服务注册层，负责服务的注册与发现

封装服务地址的注册和发现， 以服务URL 为中心，扩展接口为RegistryFactory、Registry、RegistryService

- 第五层：cluster 层，集群层，封装多个服务提供者的路由以及负载均衡，将多个实例组合成一个服务

封装多个提供者的路由和负载均衡，并桥接注册中心， 以Invoker 为中心， 扩展接口为Cluster、Directory、Router 和LoadBlancce

- 第六层：monitor 层，监控层，对 rpc 接口的调用次数和调用时间进行监控

RPC 调用次数和调用时间监控，以Statistics 为中心，扩展接口为MonitorFactory、Monitor 和MonitorService

- 第七层：protocal 层，远程调用层，封装 rpc 调用

封装RPC 调用，以Invocation 和Result 为中心，扩展接口为Protocal、Invoker 和Exporter

- 第八层：exchange 层，信息交换层，封装请求响应模式，同步转异步

封装请求响应模式，同步转异步。以Request 和Response 为中心，扩展接口为Exchanger、ExchangeChannel、ExchangeClient 和ExchangeServer

- 第九层：transport 层，网络传输层，抽象 mina 和 netty 为统一接口

抽象mina 和netty 为统一接口，以Message 为中心，扩展接口为Channel、Transporter、Client、Server 和Codec

- 第十层：serialize 层，数据序列化层

可复用的一些工具， 扩展接口为Serialization、ObjectInput、ObjectOutput 和ThreadPool

### 10.工作流程

- 第一步：provider 向注册中心去注册
- 第二步：consumer 从注册中心订阅服务，注册中心会通知 consumer 注册好的服务
- 第三步：consumer 调用 provider
- 第四步：consumer 和 provider 都异步通知监控中心

### 11.注册中心挂了可以继续通信吗？

可以，因为刚开始初始化的时候，消费者会将提供者的地址等信息**拉取到本地缓存**，所以注册中心挂了可以继续通信。

### 12.dubbo 负载均衡策略？

RandomLoadBalance

默认情况下，dubbo 是 RandomLoadBalance ，即**随机**调用实现负载均衡，可以对 provider 不同实例**设置不同的权重**，会按照权重来负载均衡，权重越大分配流量越高，一般就用这个默认的就可以了。

算法思想很简单。假设有一组服务器 servers = `[A, B, C]`，他们对应的权重为 weights = `[5, 3, 2]`，权重总和为 10。现在把这些权重值平铺在一维坐标值上，`[0, 5)` 区间属于服务器 A，`[5, 8)` 区间属于服务器 B，`[8, 10)` 区间属于服务器 C。接下来通过随机数生成器生成一个范围在 `[0, 10)` 之间的随机数，然后计算这个随机数会落到哪个区间上。比如数字 3 会落到服务器 A 对应的区间上，此时返回服务器 A 即可。权重越大的机器，在坐标轴上对应的区间范围就越大，因此随机数生成器生成的数字就会有更大的概率落到此区间内。只要随机数生成器产生的随机数分布性很好，在经过多次选择后，每个服务器被选中的次数比例接近其权重比例。比如，经过一万次选择后，服务器 A 被选中的次数大约为 5000 次，服务器 B 被选中的次数约为 3000 次，服务器 C 被选中的次数约为 2000 次。

RoundRobinLoadBalance

这个的话默认就是均匀地将流量打到各个机器上去，但是如果各个机器的性能不一样，容易导致性能差的机器负载过高。所以此时需要调整权重，让性能差的机器承载权重小一些，流量少一些。

举个栗子。

跟运维同学申请机器，有的时候，我们运气好，正好公司资源比较充足，刚刚有一批热气腾腾、刚刚做好的虚拟机新鲜出炉，配置都比较高：8 核 + 16G 机器，申请到 2 台。过了一段时间，我们感觉 2 台机器有点不太够，我就去找运维同学说，“哥儿们，你能不能再给我一台机器”，但是这时只剩下一台 4 核 + 8G 的机器。我要还是得要。

这个时候，可以给两台 8 核 16G 的机器设置权重 4，给剩余 1 台 4 核 8G 的机器设置权重 2。

LeastActiveLoadBalance

官网对 `LeastActiveLoadBalance` 的解释是“**最小活跃数负载均衡**”，活跃调用数越小，表明该服务提供者效率越高，单位时间内可处理更多的请求，那么此时请求会优先分配给该服务提供者。

最小活跃数负载均衡算法的基本思想是这样的：

每个服务提供者会对应着一个活跃数 `active`。初始情况下，所有服务提供者的 `active` 均为 0。每当收到一个请求，对应的服务提供者的 `active` 会加 1，处理完请求后，`active` 会减 1。所以，如果服务提供者性能较好，处理请求的效率就越高，那么 `active` 也会下降的越快。因此可以给这样的服务提供者优先分配请求。

当然，除了最小活跃数，`LeastActiveLoadBalance` 在实现上还引入了权重值。所以准确的来说，`LeastActiveLoadBalance` 是基于加权最小活跃数算法实现的。

ConsistentHashLoadBalance

一致性 Hash 算法，相同参数的请求一定分发到一个 provider 上去，provider 挂掉的时候，会基于虚拟节点均匀分配剩余的流量，抖动不会太大。**如果你需要的不是随机负载均衡**，是要一类请求都到一个节点，那就走这个一致性 Hash 策略。

### 13.dubbo 的 spi 思想是什么？

spi 是啥？

spi，简单来说，就是 `service provider interface` ，说白了是什么意思呢，比如你有个接口，现在这个接口有 3 个实现类，那么在系统运行的时候对这个接口到底选择哪个实现类呢？这就需要 spi 了，需要**根据指定的配置**或者是**默认的配置**，去**找到对应的实现类**加载进来，然后用这个实现类的实例对象。

举个栗子。

你有一个接口 A。A1/A2/A3 分别是接口 A 的不同实现。你通过配置 `接口 A = 实现 A2` ，那么在系统实际运行的时候，会加载你的配置，用实现 A2 实例化一个对象来提供服务。

spi 机制一般用在哪儿？**插件扩展的场景**，比如说你开发了一个给别人使用的开源框架，如果你想让别人自己写个插件，插到你的开源框架里面，从而扩展某个功能，这个时候 spi 思想就用上了。

Java spi 思想的体现

spi 经典的思想体现，大家平时都在用，比如说 jdbc。

Java 定义了一套 jdbc 的接口，但是 Java 并没有提供 jdbc 的实现类。

但是实际上项目跑的时候，要使用 jdbc 接口的哪些实现类呢？一般来说，我们要**根据自己使用的数据库**，比如 mysql，你就将 `mysql-jdbc-connector.jar` 引入进来；oracle，你就将 `oracle-jdbc-connector.jar` 引入进来。

在系统跑的时候，碰到你使用 jdbc 的接口，他会在底层使用你引入的那个 jar 中提供的实现类。

dubbo 的 spi 思想

dubbo 也用了 spi 思想，不过没有用 jdk 的 spi 机制，是自己实现的一套 spi 机制。

```html
Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
```

Protocol 接口，在系统运行的时候，，dubbo 会判断一下应该选用这个 Protocol 接口的哪个实现类来实例化对象来使用。

它会去找一个你配置的 Protocol，将你配置的 Protocol 实现类，加载到 jvm 中来，然后实例化对象，就用你的那个 Protocol 实现类就可以了。

在 dubbo 自己的 jar 里，在 `/META_INF/dubbo/internal/com.alibaba.dubbo.rpc.Protocol` 文件中：

```html
dubbo=com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol
http=com.alibaba.dubbo.rpc.protocol.http.HttpProtocol
hessian=com.alibaba.dubbo.rpc.protocol.hessian.HessianProtocol
```

所以说，这就看到了 dubbo 的 spi 机制默认是怎么玩儿的了，其实就是 Protocol 接口， `@SPI("dubbo")` 说的是，通过 SPI 机制来提供实现类，实现类是通过 dubbo 作为默认 key 去配置文件里找到的，配置文件名称与接口全限定名一样的，通过 dubbo 作为 key 可以找到默认的实现类就是 `com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol` 。

如果想要动态替换掉默认的实现类，需要使用 `@Adaptive` 接口，Protocol 接口中，有两个方法加了 `@Adaptive` 注解，就是说那俩接口会被代理实现。

啥意思呢？

比如这个 Protocol 接口搞了俩 `@Adaptive` 注解标注了方法，在运行的时候会针对 Protocol 生成代理类，这个代理类的那俩方法里面会有代理代码，代理代码会在运行的时候动态根据 url 中的 protocol 来获取那个 key，默认是 dubbo，你也可以自己指定，你如果指定了别的 key，那么就会获取别的实现类的实例了。

### 14.如何基于 dubbo 进行服务治理

\1. 调用链路自动生成

一个大型的分布式系统，或者说是用现在流行的微服务架构来说吧，**分布式系统由大量的服务组成**。那么这些服务之间互相是如何调用的？调用链路是啥？说实话，几乎到后面没人搞的清楚了，因为服务实在太多了，可能几百个甚至几千个服务。

那就需要基于 dubbo 做的分布式系统中，对各个服务之间的调用自动记录下来，然后自动将**各个服务之间的依赖关系和调用链路生成出来**，做成一张图，显示出来，大家才可以看到对吧。

\2. 服务访问压力以及时长统计

需要自动统计**各个接口和服务之间的调用次数以及访问延时**，而且要分成两个级别。

- 一个级别是接口粒度，就是每个服务的每个接口每天被调用多少次，TP50/TP90/TP99，三个档次的请求延时分别是多少；
- 第二个级别是从源头入口开始，一个完整的请求链路经过几十个服务之后，完成一次请求，每天全链路走多少次，全链路请求延时的 TP50/TP90/TP99，分别是多少。

这些东西都搞定了之后，后面才可以来看当前系统的压力主要在哪里，如何来扩容和优化啊。

\3. 其它

- 服务分层（避免循环依赖）
- 调用链路失败监控和报警
- 服务鉴权
- 每个服务的可用性的监控（接口调用成功率？几个 9？99.99%，99.9%，99%）

### 14.如何基于 dubbo 进行服务降级

比如说服务 A 调用服务 B，结果服务 B 挂掉了，服务 A 重试几次调用服务 B，还是不行，那么直接降级，走一个备用的逻辑，给用户返回响应。

我们调用接口失败的时候，可以通过 `mock` 统一返回 null。

mock 的值也可以修改为 true，然后再跟接口同一个路径下实现一个 Mock 类，命名规则是 “接口名称+ `Mock` ” 后缀。然后在 Mock 类里实现自己的降级逻辑。

### 15.如何基于 dubbo 进行失败重试和超时重试

所谓失败重试，就是 consumer 调用 provider 要是失败了，比如抛异常了，此时应该是可以重试的，或者调用超时了也可以重试。配置如下：

```html
<dubbo:reference id="xxxx" interface="xx" check="true" async="false" retries="3" timeout="2000"/>
```

可以结合你们公司具体的场景来说说你是怎么设置这些参数的：

- `timeout` ：一般设置为 `200ms` ，我们认为不能超过 `200ms` 还没返回。
- `retries` ：设置 retries，一般是在读请求的时候，比如你要查询个数据，你可以设置个 retries，如果第一次没读到，报错，重试指定的次数，尝试再次读取。

### 16.接口请求的顺序性如何保证？

系统最好是不需要这种顺序性的保证，因为一旦引入顺序性保障，比如使用**分布式锁**，会**导致系统复杂度上升**，而且会带来**效率低下**，热点数据压力过大等问题。

首先你得用 Dubbo 的一致性 hash 负载均衡策略，将比如某一个订单 id 对应的请求都给分发到某个机器上去，接着就是在那个机器上，因为可能还是多线程并发执行的，你可能得立即将某个订单 id 对应的请求扔一个**内存队列**里去，强制排队，这样来确保他们的顺序性。

最好是比如说刚才那种，一个订单的插入和删除操作，能不能合并成一个操作，就是一个删除，或者是其它什么，避免这种问题的产生。

### 17.如何自己设计一个类似 Dubbo 的 RPC 框架？

- 得有个注册中心，保留各个服务的信息，可以用 zookeeper 来做。
- 然后你的消费者需要去注册中心拿对应的服务信息吧，对吧，而且每个服务可能会存在于多台机器上。
- 接着你就该发起一次请求了，咋发起？当然是基于动态代理了，你面向接口获取到一个动态代理，这个动态代理就是接口在本地的一个代理，然后这个代理会找到服务对应的机器地址。
- 然后找哪个机器发送请求？那肯定得有个负载均衡算法了，比如最简单的可以随机轮询是不是。
- 接着找到一台机器，就可以跟它发送请求了，第一个问题咋发送？你可以说用 netty 了，nio 方式；第二个问题发送啥格式数据？你可以说用 hessian 序列化协议了，或者是别的，对吧。然后请求过去了。
- 服务器那边一样的，需要针对你自己的服务生成一个动态代理，监听某个网络端口了，然后代理你本地的服务代码。接收到请求的时候，就调用对应的服务代码，对吧。

### 18.Dubbo 用到哪些设计模式？

Dubbo 框架在初始化和通信过程中使用了多种设计模式， 可灵活控制类加载、权限控制等功能。

工厂模式

Provider 在export 服务时，会调用ServiceConfig 的export 方法。ServiceConfig中有个字段： private static final Protocol protocol =ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension(); Dubbo 里有很多这种代码。这也是一种工厂模式， 只是实现类的获取采用了JDK SPI 的机制。这么实现的优点是可扩展性强， 想要扩展实现，只需要在classpath 下增加个文件就可以了，代码零侵入。另外，像上面的Adaptive 实现，可以做到调用时动态决定调用哪个实现，但是由于这种实现采用了动态代理， 会造成代码 调试比较麻烦，需要分析出实际调用的实现类。

装饰器模式

Dubbo 在启动和调用阶段都大量使用了装饰器模式。以Provider 提供的调用链为例， 具体的调用链代码是在ProtocolFilterWrapper 的buildInvokerChain 完成 的， 具体是将注解中含有group=provider 的Filter 实现，按照order 排序，最后的调用顺序是：

EchoFilter -> ClassLoaderFilter -> GenericFilter -> ContextFilter ->ExecuteLimitFilter -> TraceFilter -> TimeoutFilter -> MonitorFilter ->ExceptionFilter 更确切地说，这里是装饰器和责任链模式的混合使用。例如，EchoFilter 的作用是判断是否是回声测试请求，是的话直接返回内容， 这是一种责任链的体现。而像 ClassLoaderFilter 则只是在主功能上添加了功能，更改当前线程的ClassLoader，这是典型的装饰器模式。

观察者模式

Dubbo 的Provider 启动时，需要与注册中心交互，先注册自己的服务，再订阅自己的服务，订阅时，采用了观察者模式，开启一个listener。注册中心会每5 秒定时检查是否有服务更新，如果有更新，向该服务的提供者发送一个notify 消息，provider 接受到notify 消息后， 即运行NotifyListener 的notify 方法，执行监听器方法。

动态代理模式

Dubbo 扩展JDK SPI 的类ExtensionLoader 的Adaptive 实现是典型的动态代理实现。Dubbo 需要灵活地控制实现类，即在调用阶段动态地根据参数决定调用哪个实现类，所以采用先生成代理类的方法，能够做到灵活的调用。生成代理类的代码是ExtensionLoader 的createAdaptiveExtensionClassCode 方法。代理类的主要逻辑是，获取URL 参数中指定参数的值作为获取实现类的key。

### 19.Dubbo 可以对结果进行缓存吗？

为了提高数据访问的速度。Dubbo 提供了声明式缓存，以减少用户加缓存的工作量 <dubbo:reference cache="true" /> 其实比普通的配置文件就多了一个标签cache="true"

### 20.Dubbo 服务降级，失败重试怎么做？

可以通过 dubbo:reference 中设置 mock="return null"。mock 的值也可以修改为 true，然后再跟接口同一个路径下实现一个 Mock 类， 命名规则是 “接口名称+Mock” 后缀。然后在 Mock 类里实现自己的降级逻辑

### 21.Dubbo 使用过程中都遇到了些什么问题？

在注册中心找不到对应的服务,检查 service 实现类是否添加了@service 注解无法连接到注册中心,检查配置文件中的对应的测试 ip 是否正确

### 22.Dubbo Monitor 实现原理？

Consumer 端在发起调用之前会先走 filter 链；provider 端在接收到请求时也是先走 filter 链，然后才进行真正的业务逻辑处理。 默认情况下，在 consumer 和 provider 的 filter 链中都会有 Monitorfilter。 1、MonitorFilter 向 DubboMonitor 发送数据 2、DubboMonitor 将数据进行聚合后（默认聚合 1min 中的统计数据）暂存到ConcurrentMap<Statistics, AtomicReference> statisticsMap，然后使用一个含有 3 个线程（线程名字：DubboMonitorSendTimer）的线程池每隔 1min 钟，调用 SimpleMonitorService 遍历发送 statisticsMap 中的统计数据，每发送完毕一个，就重置当前的 Statistics 的 AtomicReference 3、SimpleMonitorService 将这些聚合数据塞入 BlockingQueue queue 中（队列大写为 100000） 4、SimpleMonitorService 使用一个后台线程（线程名为：DubboMonitorAsyncWriteLogThread）将 queue 中的数据写入文件（该线程 以死循环的形式来写） 5、SimpleMonitorService 还会使用一个含有 1 个线程（线程名字：DubboMonitorTimer）的线程池每隔 5min 钟，将文件中的统计数据 画成图表

### 23.Dubbo 配置文件是如何加载到 Spring 中的？

Spring 容器在启动的时候，会读取到 Spring 默认的一些 schema 以及 Dubbo 自定义的 schema，每个 schema 都会对应一个自己的 NamespaceHandler，NamespaceHandler 里面通过 BeanDefinitionParser 来解析配置信息并转化为需要加载的 bean 对象!

### 24.Dubbo 如何优雅停机？

Dubbo 是通过 JDK 的 ShutdownHook 来完成优雅停机的，所以如果使用kill -9 PID 等强制关闭指令，是不会执行优雅停机的，只有通过 kill PID 时，才会执行。

### 25.Dubbo有哪几种配置方式？

1）Spring 配置方式 2）Java API 配置方式

### 26.在 Provider 上可以配置的 Consumer 端的属性有哪些？

1）timeout：方法调用超时 2）retries：失败重试次数，默认重试 2 次 3）loadbalance：负载均衡算法，默认随机 4）actives 消费者端，最大并发调用限制

### 27.Dubbo服务之间的调用是阻塞的吗？

默认是同步等待结果阻塞的，支持异步调用。 Dubbo 是基于 NIO 的非阻塞实现并行调用，客户端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小，异步调用会 返回一个 Future 对象。

### 28.服务读写推荐的容错策略是怎样的？

读操作建议使用 Failover 失败自动切换，默认重试两次其他服务器。 写操作建议使用 Failfast 快速失败，发一次调用失败就立即报错。

### 29.说说 Dubbo 服务暴露的过程。

Dubbo 会在 Spring 实例化完 bean 之后，在刷新容器最后一步发布 ContextRefreshEvent 事件的时候，通知实现了 ApplicationListener 的 ServiceBean 类进行回调 onApplicationEvent 事件方法，Dubbo 会在这个方法中调用 ServiceBean 父类 ServiceConfig 的 export 方 法，而该方法真正实现了服务的（异步或者非异步）发布。

### 30.注册中心

注册中心作用

1. 动态加入,服务提供者通过注册中心动态的把自己暴露给消费者，无需消费者逐个更新配置文件。
2. 动态发现服务，消费者可以动态发现新的服务，无需重启生效。
3. 统一配置，避免本地配置导致每个服务配置不一致。
4. 动态调整，注册中心支持参数动态调整，新参数自动更新到所有相关的服务节点。
5. 统一管理，依靠注册中心数据，可以统一管理配置服务节点。

注册中心工作流程

1. 服务提供者启动之后，会将服务注册到注册中心。
2. 消费者启动之后主动订阅注册中心上提供者服务，从而获取到当前所有可用服务，同时留下一个回调函数。
3. 若服务提供者新增或下线，注册中心将通过第二步的注册的回调函数通知消费者。
4. dubbo-admin(服务治理中心)将会会订阅服务提供者以及消费者，从而可以在控制台管理所有服务提供者以及消费者。

AbstractRegistry 缓存实现

如果每次服务调用都需要调用注册中心实时查询可用服务列表，不但会让注册中心承受巨大的流量压力，还会产生额外的网络请求，导致系统性能下降。 其次注册中心需要非强依赖，其宕机不能影响正常的服务调用。 基于以上几点，注册中心模块在 AbstractRegistry 类中实现通用的缓存机制。这里的缓存可以分为两类，内存服务缓存以及磁盘文件缓存。

内存服务缓存

内存服务缓存很好理解也最容易实现， AbstractRegistry 使用一个 ConcurrentMap 保存相关信息。

这个集合中 key 为消费者的 URL，而 value 为一个 Map 集合。这个内层 Map 集合使用服务目录作为 key，分别为 providers，routers，configurators，consumers 四类，value 则是对应服务列表集合。

磁盘文件缓存

由于服务重启就会导致内存缓存消失，所以额外增加磁盘文件缓存。

缓存文件的加载

dubbo 程序初始化的时候， AbstractRegistry 构造函数将会从本地磁盘文件中将数据读取到 Properties 对象实例中，后续都将会先写入 Properties ，最后再将里面信息再写入文件。

缓存文件的保存与更新 缓存文件将会通过 AbstractRegistry#notify 方法保存或更新。客户端第一次订阅服务获取的全量数据，或者后续回调中获取到新数据，都将会调用 AbstractRegistry#notify 方法，用来更新内存缓存以及文件缓存。

FailbackRegistry 重试机制

FailbackRegistry 继承 AbstractRegistry ，实现了 register ， subscribe 等通用法，并增加 doRegister ， doSubscribe等模板方法，交由子类实现。 如果 doRegister 等模板方法发生异常，会将失败任务放入集合，然后定时再次调用模板方法。

以 subscribe 方法为例，这里将会调用这些 doSubscribe 的模板方法。如果发生异常将会读取缓存文件中内容，然后加载服务。最后新建异步定时任务加入重试集合中，然后由定时器去重试这些任务。

### 31.说说Dubbo暴露过程？

1、ServiceConfig 类拿到对外提供服务的实际类 ref(如：HelloWorldImpl) 2、通过 ProxyFactory 类的 getInvoker 方法使用 ref 生成一个 AbstractProxyInvoker 实例，到这一步就完成具体服务到 Invoker 的转化 3、接下来就是 Invoker 转换到 Exporter 的过程：

- ProxyFactory 是动态代理，用来创建 Invoker 对象，实现代理使用 JavassistProxyFactory和 JdkProxyFactory。
- Invoker 是一个服务对象实例，Dubbo 框架的实体域。它可以是一个本地的实现，一个远程的实现或一个集群的实现，可以向它发起 Invoker 调用。
- Protocol 是服务域，负责 Invoker 的生命周期管理，是 Invoker 暴露和引用的主要功能入口，对应该类的 export和 refer方法。
- Exporter 是根据不同协议暴露 Invoker 进行封装的类，它会根据不同的协议头进行识别（比如：registry://和 dubbo://），调用对应 XXXProtocol的 export()方法。

### 32.**Dubbo有哪些均衡负载的方法？**

1、`随机模式`。按权重设置随机概率。在一个截面上碰撞的概率较高，但调用越大分布越均匀 2、`轮询模式`。按公约后的权重设置轮询比例。但存在响应慢的服务提供者会累积请求 3、`最少活跃调用数`。响应快的提供者接受越多请求，响应慢的接受越少请求 4、`一致hash`。根据服务提供者ip设置hash环，携带相同的参数总是发送的同一个服务提供者，若服务挂了，则会基于虚拟节点平摊到其他提供者上

### 33.**Dubbo有哪些集群容错方案？**

1、`Failover Cluster：失败重试` 当服务消费方调用服务提供者失败后自动切换到其他服务提供者服务器进行重试 2、`Failfast Cluster：快速失败` 当服务消费方调用服务提供者失败后，立即报错，也就是只调用一次。通常这种模式用于非幂等性的写操作。 3、`Failsafe Cluster：失败安全` 当服务消费者调用服务出现异常时，直接忽略异常。这种模式通常用于写入审计日志等操作。 4、`Failback Cluster：失败自动恢复` 当服务消费端用服务出现异常后，在后台记录失败的请求，并按照一定的策略后期再进行重试。这种模式通常用于消息通知操作。 5、`Forking Cluster：并行调用` 当消费方调用一个接口方法后，Dubbo Client会并行调用多个服务提供者的服务，只要一个成功即返回。这种模式通常用于实时性要求较高的读操作，但需要浪费更多服务资源 6、`Broadcast Cluster：广播调用` 当消费者调用一个接口方法后，Dubbo Client会逐个调用所有服务提供者，任意一台调用异常则这次调用就标志失败。这种模式通常用于通知所有提供者更新缓存或日志等本地资源信息。

### 34.**Dubbo服务注册与发现的流程？**

- Provider(提供者)绑定指定端口并启动服务
- 指供者连接注册中心，并发本机IP、端口、应用信息和提供服务信息发送至注册中心存储
- Consumer(消费者），连接注册中心 ，并发送应用信息、所求服务信息至注册中心
- 注册中心根据消费者所求服务信息匹配对应的提供者列表发送至Consumer 应用缓存。
- Consumer 在发起远程调用时基于缓存的消费者列表择其一发起调用。
- Provider 状态变更会实时通知注册中心、在由注册中心实时推送至Consumer

### 35.Dubbo 调用流程

- Provider
  - 第 0 步，start 启动服务。
  - 第 1 步，register 注册服务到注册中心。
- Consumer
  - 第 2 步，subscribe 向注册中心订阅服务。
    - 注意，只订阅使用到的服务。
    - 再注意，首次会拉取订阅的服务列表，缓存在本地。
  - 【异步】第 3 步，notify 当服务发生变化时，获取最新的服务列表，更新本地缓存。
- invoke 调用
  - Consumer 直接发起对 Provider 的调用，无需经过注册中心。而对多个 Provider 的负载均衡，Consumer 通过 **cluster** 组件实现。
- count 监控
  - 【异步】Consumer 和 Provider 都异步通知监控中心。

### 36.Dubbo 在 Zookeeper 存储了哪些信息？

流程说明：

- **服务提供者**启动时: 向 `/dubbo/com.foo.BarService/providers` 目录下写入自己的 URL 地址
- **服务消费者**启动时: 订阅 `/dubbo/com.foo.BarService/providers` 目录下的提供者 URL 地址。并向 `/dubbo/com.foo.BarService/consumers` 目录下写入自己的 URL 地址
- **监控中心**启动时: 订阅 `/dubbo/com.foo.BarService` 目录下的所有提供者和消费者 URL 地址。

Zookeeper 的节点层级，自上而下是：

- **Root** 层：根目录，可通过 `<dubbo:registry group="dubbo" />` 的 `"group"` 设置 Zookeeper 的根节点，缺省使用 `"dubbo"` 。
- **Service** 层：服务接口全名。
- **Type** 层：分类。目前除了我们在图中看到的 `"providers"`( 服务提供者列表 ) `"consumers"`( 服务消费者列表 ) 外，还有 [`"routes"`](https://dubbo.gitbooks.io/dubbo-user-book/demos/routing-rule.html)( 路由规则列表 ) 和 [`"configurations"`](https://dubbo.gitbooks.io/dubbo-user-book/demos/config-rule.html)( 配置规则列表 )。
- **URL** 层：URL ，根据不同 Type 目录，下面可以是服务提供者 URL 、服务消费者 URL 、路由规则 URL 、配置规则 URL 。
- 实际上 URL 上带有 `"category"` 参数，已经能判断每个 URL 的分类，但是 Zookeeper 是基于节点目录订阅的，所以增加了 **Type** 层。

实际上，**服务消费者**启动后，不仅仅订阅了 `"providers"` 分类，也订阅了 `"routes"` `"configurations"` 分类。

### 37.Dubbo Provider 如何实现优雅停机？

**优雅停机**

> Dubbo 是通过 JDK 的 ShutdownHook 来完成优雅停机的，所以如果用户使用 `kill -9 PID` 等强制关闭指令，是不会执行优雅停机的，只有通过 `kill PID` 时，才会执行。

- 因为大多数情况下，Dubbo 的声明周期是交给 Spring 进行管理，所以在最新的 Dubbo 版本中，增加了对 Spring 关闭事件的监听，从而关闭 Dubbo 服务。

**服务提供方的优雅停机过程**

1. 首先，从注册中心中取消注册自己，从而使消费者不要再拉取到它。
2. 然后，sleep 10 秒( 可配 )，等到服务消费，接收到注册中心通知到该服务提供者已经下线，加大了在不重试情况下优雅停机的成功率。😈 此处是个概率学，嘻嘻。
3. 之后，广播 READONLY 事件给所有 Consumer 们，告诉它们不要在调用我了！！！【很有趣的一个步骤】并且，如果此处注册中心挂掉的情况，依然能达到告诉 Consumer ，我要下线了的功能。
4. 再之后，sleep 10 毫秒，保证 Consumer 们，尽可能接收到该消息。
5. 再再之后，先标记为不接收新请求，新请求过来时直接报错，让客户端重试其它机器。
6. 再再再之后，关闭心跳线程。
7. 最后，检测线程池中的线程是否正在运行，如果有，等待所有线程执行完成，除非超时，则强制关闭。
8. 最最后，关闭服务器。

**服务消费方的优雅停机过程**

1. 停止时，不再发起新的调用请求，所有新的调用在客户端即报错。
2. 然后，检测有没有请求的响应还没有返回，等待响应返回，除非超时，则强制关闭。

### 38.Dubbo 接口如何实现幂等性？

所谓幂等，简单地说，就是对接口的多次调用所产生的结果和调用一次是一致的。扩展一下，这里的接口，可以理解为对外发布的 HTTP 接口或者 Thrift 接口，也可以是接收消息的内部接口，甚至是一个内部方法或操作。

那么我们为什么需要接口具有幂等性呢？设想一下以下情形：

- 在 App 中下订单的时候，点击确认之后，没反应，就又点击了几次。在这种情况下，如果无法保证该接口的幂等性，那么将会出现重复下单问题。
- 在接收消息的时候，消息推送重复。如果处理消息的接口无法保证幂等，那么重复消费消息产生的影响可能会非常大。

所以，从这段描述中，幂等性不仅仅是 Dubbo 接口的问题，包括 HTTP 接口、Thrift 接口都存在这样的问题，甚至说 MQ 消息、定时任务，都会碰到这样的场景。那么应该怎么办呢？

所谓**幂等性**，就是说一个接口，多次发起同一个请求，你这个接口得保证结果是准确的，比如不能多扣款、不能多插入一条数据、不能将统计值多加了 1。这就是幂等性。

其实保证幂等性主要是三点：

- 对于每个请求必须有一个唯一的标识，举个栗子：订单支付请求，肯定得包含订单 id，一个订单 id 最多支付一次，对吧。
- 每次处理完请求之后，必须有一个记录标识这个请求处理过了。常见的方案是在 mysql 中记录个状态啥的，比如支付之前记录一条这个订单的支付流水。
- 每次接收请求需要进行判断，判断之前是否处理过。比如说，如果有一个订单已经支付了，就已经有了一条支付流水，那么如果重复发送这个请求，则此时先插入支付流水，orderId 已经存在了，唯一键约束生效，报错插入不进去的。然后你就不用再扣款了。

实际运作过程中，你要结合自己的业务来，比如说利用 redis，用 orderId 作为唯一键。只有成功插入这个支付流水，才可以执行实际的支付扣款。

要求是支付一个订单，必须插入一条支付流水，order_id 建一个唯一键 `unique key`。你在支付一个订单之前，先插入一条支付流水，order_id 就已经进去了。你就可以写一个标识到 redis 里面去，`set order_id payed`，下一次重复请求过来了，先查 redis 的 order_id 对应的 value，如果是 `payed` 就说明已经支付过了，你就别重复支付了。

### 39.Dubbo 如何实现分布式事务？

说起分布式，理论的文章很多，落地的实践很少。笔者翻阅了各种分布式事务组件的选型，大体如下：

- TCC 模型：TCC-Transaction、Hmily
- XA 模型：Sharding Sphere、MyCAT
- 2PC 模型：raincat、lcn
- MQ 模型：RocketMQ
- BED 模型：Sharding Sphere
- Saga 模型：ServiceComb Saga

那怎么选择呢？目前社区对于分布式事务的选择，暂时没有定论，至少笔者没有看到。笔者的想法如下：

- 从覆盖场景来说，TCC 无疑是最优秀的，但是大家觉得相对复杂。实际上，复杂场景下，使用 TCC 来实现，反倒会容易很多。另外，TCC 模型，一直没有大厂开源，也是一大痛点。
- 从使用建议来说，MQ 可能是相对合适的( 不说 XA 的原因还是性能问题 )，并且基本轮询了一圈朋友，发现大多都是使用 MQ 实现最终一致性居多。
- 2PC 模型的实现，笔者觉得非常新奇，奈何隔离性是一个硬伤。
- Saga 模型，可以认为是 TCC 模型的简化版，所以在理解和编写的难度上，简单非常多。

所以结论是什么呢？

- TCC 模型：TCC-Transaction、Hmily 。
  - 已经提供了和 Dubbo 集成的方案，胖友可以自己去试试。
- XA 模型：Sharding Sphere、MyCAT 。
  - 无需和 Dubbo 进行集成。
- 2PC 模型：raincat、lcn 。
  - 已经提供了和 Dubbo 集成的方案，胖友可以自己去试试。
- MQ 模型：RocketMQ 。
  - 无需和 Dubbo 进行集成。
- BED 模型：Sharding Sphere 。
  - 无需和 Dubbo 进行集成。
- Saga 模型：ServiceComb Saga 。
  - **好像**已经提供了和 Dubbo 集成的方案

另外，胖友在理解分布式事务时，一定要记住，分布式事务需要由多个**本地**事务组成。无论是上述的那种事务组件模型，它们都是扮演一个**协调者**，使多个**本地**事务达到最终一致性。而协调的过程中，就非常依赖每个方法操作可以被重复执行不会产生副作用，那么就需要：

- 幂等性！因为可能会被重复调用。如果调用两次退款，结果退了两次钱，那就麻烦大了。
- 本地事务！因为执行过程中可能会出错，需要回滚。

### 40.Dubbo 如何集成网关服务？

Dubbo 如何集成到网关服务，需要思考两个问题：

- 网关如何**调用** Dubbo 服务。
- 网关如何**发现** Dubbo 服务。

我们先来看看，市面上有哪些网关服务：

- Zuul
- Spring Cloud Gateway
- Kong

如上三个解决方案，都是基于 HTTP 调用后端的服务。那么，这样的情况下，Dubbo 只能通过暴露 `rest://` 协议的服务，才能被它们调用。

那么 Dubbo 的 `rest://` 协议的服务，怎么能够被如上三个解决方案注册发现呢？

- 因为 Dubbo 可用的注册中心有 Zookeeper ，如果要被 Zuul 或 Spring Cloud Gateway 注册发现，可以使用 `spring-cloud-starter-zookeeper-discovery` 库。
- Dubbo 与 Kong 的集成，相对比较麻烦，需要通过 Kong 的 API 添加相应的路由规则。

可能会有胖友问，有没支持 `dubbo://` 协议的网关服务呢？目前有新的网关开源 [Soul](https://dromara.org/website/zh-cn/docs/soul/soul.html) ，基于 Dubbo 泛化调用的特性，实现对 `dubbo://` 协议的 Dubbo 服务的调用。

实际场景下，我们真的需要 Dubbo 集成到网关吗？具艿艿了解到，很多公司，并未使用网关，而是使用 Spring Boot 搭建一个 Web 项目，引入 `*-api.jar` 包，然后进行调用，从而对外暴露 HTTP API 。

### 41.拆分后不用 Dubbo 可以吗？

当然是可以，方式还有很多：

- 第一种，使用 Spring Cloud 技术体系，这个也是目前可能最主流的之一。
- 第二种，Dubbo 换成 gRPC 或者 Thrift 。当然，此时要自己实现注册发现、负载均衡、集群容错等等功能。
- 第三种，Dubbo 换成同等定位的服务化框架，例如微博的 Motan 、蚂蚁金服的 SofaRPC 。
- 第四种，Spring MVC + Nginx 。
- 第五种，每个服务拆成一个 Maven 项目，打成 Jar 包，给其它服务使用。😈 当然，这个不是一个比较特别的方案。

dubbo 说白了，是一种 rpc 框架，就是说本地就是进行接口调用，但是 dubbo 会代理这个调用请求，跟远程机器网络通信，给你处理掉负载均衡了、服务实例上下线自动感知了、超时重试了，等等乱七八糟的问题。那你就不用自己做了，用 dubbo 就可以了。

### 42.如何自己设计一个类似 Dubbo 的 RPC 框架？

最简单的回答思路：

- 上来你的服务就得去注册中心注册吧，你是不是得有个注册中心，保留各个服务的信心，可以用 zookeeper 来做，对吧。
- 然后你的消费者需要去注册中心拿对应的服务信息吧，对吧，而且每个服务可能会存在于多台机器上。
- 接着你就该发起一次请求了，咋发起？当然是基于动态代理了，你面向接口获取到一个动态代理，这个动态代理就是接口在本地的一个代理，然后这个代理会找到服务对应的机器地址。
- 然后找哪个机器发送请求？那肯定得有个负载均衡算法了，比如最简单的可以随机轮询是不是。
- 接着找到一台机器，就可以跟它发送请求了，第一个问题咋发送？你可以说用 netty 了，nio 方式；第二个问题发送啥格式数据？你可以说用 hessian 序列化协议了，或者是别的，对吧。然后请求过去了。
- 服务器那边一样的，需要针对你自己的服务生成一个动态代理，监听某个网络端口了，然后代理你本地的服务代码。接收到请求的时候，就调用对应的服务代码，对吧。

### 43.dubbo ⽀持的序列化协议？

dubbo ⽀持 hession、Java ⼆进制序列化、json、SOAP ⽂本序列化多种序列化协议。但是 hessian 是其默认的序列化协议。 说⼀下 Hessian 的数据结构？ Hessian 的对象序列化机制有 8 种原始类型： 原始⼆进制数据 boolean 64-bit date（64 位毫秒值的⽇期） 64-bit double 32-bit int 64-bit long null UTF-8 编码的 string 另外还包括 3 种递归类型： list for lists and arrays map for maps and dictionaries object for objects 还有⼀种特殊的类型： ref：⽤来表⽰对共享对象的引⽤。

### 44.⼀般使⽤什么注册中⼼？还有别的选择吗？

答：推荐使⽤ zookeeper 注册中⼼，还有 Multicast注册中⼼, Redis注册中⼼, Simple注册中⼼. ZooKeeper的节点是通过像树⼀样的结构来进⾏维护的，并且每⼀个节点通过路径来标⽰以及访问。除此之外，每⼀个节点还拥有⾃⾝的⼀ 些信息，包括：数据、数据⻓度、创建时间、修改时间等等。

### 45.默认使⽤什么序列化框架，你知道的还有哪些？

答：默认使⽤ Hessian 序列化，还有 Duddo、FastJson、Java ⾃带序列化。hessian是⼀个采⽤⼆进制格式传输的服务框架，相对传统 soap web service，更轻量，更快速。 Hessian原理与协议简析： http的协议约定了数据传输的⽅式，hessian也⽆法改变太多： 1) hessian中client与server的交互，基于http-post⽅式。 2) hessian将辅助信息，封装在http header中，⽐如“授权token”等，我们可以基于http-header来封装关于“安全校验”“meta数 据”等。hessian提供了简单的”校验”机制。 3) 对于hessian的交互核⼼数据，⽐如“调⽤的⽅法”和参数列表信息，将通过post请求的body体直接发送，格式为字节流。 4) 对于hessian的server端响应数据，将在response中通过字节流的⽅式直接输出。 hessian的协议本⾝并不复杂，在此不再赘⾔；所谓协议(protocol)就是约束数据的格式，client按照协议将请求信息序列化成字节序列发送 给server端，server端根据协议，将数据反序列化成“对象”，然后执⾏指定的⽅法，并将⽅法的返回值再次按照协议序列化成字节流，响 应给client，client按照协议将字节流反序列话成”对象”。

### 46.dubbo 推荐⽤什么协议？

答：默认使⽤ dubbo 协议。

### 47.dubbo 在安全机制⽅⾯如何解决的？

dubbo 通过 token 令牌防⽌⽤⼾绕过注册中⼼直连，然后在注册中⼼管理授权，dubbo 提供了⿊⽩名单，控制服务所允许的调⽤⽅。

### 48.集群容错怎么做？

答：读操作建议使⽤ Failover 失败⾃动切换，默认重试两次其他服务器。写操作建议使⽤ Failfast 快速失败，发⼀次调⽤失败就⽴即报错。

### 49.在使⽤过程中都遇到了些什么问题？如何解决的？

1) 同时配置了 XML 和 properties ⽂件，则 properties 中的配置⽆效 只有 XML 没有配置时，properties 才⽣效。 2) dubbo 缺省会在启动时检查依赖是否可⽤，不可⽤就抛出异常，阻⽌ spring 初始化完成，check 属性默认为 true。 测试时有些服务不关⼼或者出现了循环依赖，将 check 设置为 false 3) 为了⽅便开发测试，线下有⼀个所有服务可⽤的注册中⼼，这时，如果有⼀个正在开发中的服务提供者注册，可能会影响消费者不能正常 运⾏。 解决：让服务提供者开发⽅，只订阅服务，⽽不注册正在开发的服务，通过直连测试正在开发的服务。设置 dubbo:registry 标签的 register 属性为 false。 4) spring 2.x 初始化死锁问题。 在 spring 解析到 dubbo:service 时，就已经向外暴露了服务，⽽ spring 还在接着初始化其他 bean，如果这时有请求进来，并且服务的实现 类⾥有调⽤ applicationContext.getBean() 的⽤法。getBean 线程和 spring 初始化线程的锁的顺序不⼀样，导致了线程死锁，不能提供服 务，启动不了。

解决：不要在服务的实现类中使⽤ applicationContext.getBean(); 如果不想依赖配置顺序，可以将 dubbo:provider 的 deplay 属性设置为

-1，使 dubbo 在容器初始化完成后再暴露服务。 5) 服务注册不上 检查 dubbo 的 jar 包有没有在 classpath 中，以及有没有重复的 jar 包 检查暴露服务的 spring 配置有没有加载 在服务提供者机器上测试与注册中⼼的⽹络是否通 6) 出现 RpcException: No provider available for remote service 异常 表⽰没有可⽤的服务提供者， a. 检查连接的注册中⼼是否正确 b. 到注册中⼼查看相应的服务提供者是否存在 c. 检查服务提供者是否正常运⾏ 7) 出现” 消息发送失败” 异常 通常是接⼝⽅法的传⼊传出参数未实现 Serializable 接⼝。

### 50.Dubbo ⽀持哪些协议，每种协议的应⽤场景，优缺点？

dubbo：单⼀⻓连接和 NIO 异步通讯，适合⼤并发⼩数据量的服务调⽤，以及消费者远⼤于提供者。传输协议 TCP，异步，Hessian 序列 化； rmi：采⽤ JDK 标准的 rmi 协议实现，传输参数和返回参数对象需要实现 Serializable 接⼝，使⽤ java 标准序列化机制，使⽤阻塞式短连 接，传输数据包⼤⼩混合，消费者和提供者个数差不多，可传⽂件，传输协议 TCP。多个短连接，TCP 协议传输，同步传输，适⽤常规的远 程服务调⽤和 rmi 互操作。在依赖低版本的 Common-Collections 包，java 序列化存在安全漏洞； 关注微信公众号「web_resourc」,回复 Java 领取2019最新资源。 webservice:基于 WebService 的远程调⽤协议，集成 CXF 实现，提供和原⽣ WebService 的互操作。多个短连接，基于 HTTP 传输，同步 传输，适⽤系统集成和跨语⾔调⽤；http：基于 Http 表单提交的远程调⽤协议，使⽤ Spring 的 HttpInvoke 实现。多个短连接，传输协议 HTTP，传⼊参数⼤⼩混合，提供者个数多于消费者，需要给应⽤程序和浏览器 JS 调⽤； hessian：集成 Hessian 服务，基于 HTTP 通讯， 采⽤ Servlet 暴露服务，Dubbo 内嵌 Jetty 作为服务器时默认实现，提供与 Hession 服务互操作。多个短连接，同步 HTTP 传输，Hessian 序列化，传⼊参数较⼤，提供者⼤于消费者，提供者压⼒较⼤，可传⽂件； memcache：基于 memcached 实现的 RPC 协议 redis：基于 redis 实现的 RPC 协议