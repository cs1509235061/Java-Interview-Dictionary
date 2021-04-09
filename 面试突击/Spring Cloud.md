# Spring Cloud

### 1.Spring Cloud

Spring Cloud为企业级分布式Web系统构建提供了一站式的解决方案。为了简化分布式系统的开发流程和降低开发难度，Spring Cloud以组件化的形式提供了配置管理、服务发现、断路器、智能路由、负载均衡和消息总线等模块，应用程序只需要根据需求引入模块，便可方便地实现对应的功能.。

- Spring Cloud Config: Spring Cloud的配置中心，用于将配置存储到服务器中进行中集中化管理，支持本地存储、Git和Subversion 3种存储方式。配置中心除了Spring Cloud Config，还有Apollo配置中心和基于ZooKeeper等方式实现的配置中心。
- Spring Cloud Bus: Spring Cloud的事件消息总线，用于监听和传播集群中事件的状态的变化，例如集群中配置的变化检测和广播等。
- Eureka:Netflix提供的服务注册和发现组件，集群中各个服务以REST的方式将服务注册到注册中心，并与注册中心保持心跳连接，主要用于服务发现和自动故障转移。
- Hystrix:Netflix提供的服务熔断器，主要提供了服务负载过高或服务故障时的容错处理机制，以便集群在出现故障时依然能够对外提供服务，防止服务雪崩。
- Zuul:Netflix Zuul为集群提供通用网关的功能，前端服务访问后端服务均需要通过Zuul的动态路由来实现。同时可以在Zuul上实现服务的弹性扩展、安全监测、统一权限认证等功能。
- Consul: Consul是基于Golang开发的一个服务注册、发现和配置工具，其功能与Eureka类似。
- SpringCloud Sleuth: Spring的日志收集工具包，封装了Dapper和Log-based追踪及Zipkin和HTrace操作，为Spring Cloud的应用实现了一种分布式链路追踪解决方案。
- Spring Cloud Security：基于Spring Security工具包实现的安全管理组件，主要用于应用程序的安全访问和控制。
- Spring Cloud ZooKeeper: Spring Cloud ZooKeeper封装了操作ZooKeeper的API,用于方便地操作ZooKeeper并实现服务发现和配置管理功能。
- Ribbon: Netflix Ribbon用于分布式系统API调用的负载均衡，提供随机负载、轮询负载等多种负载均衡策略，常配合服务发现和断路器使用。
- Feign:Feign是一种声明式、模板化的HTTP访问客户端。

### 2.SpringCloud和Dubbo

- 服务的调用方式Dubbo使用的是RPC远程调用,而SpringCloud使用的是 Rest API,
- 服务的注册中心来看,Dubbo使用了第三方的ZooKeeper作为其底层的注册中心,实现服务的注册和发现,SpringCloud使用Spring Cloud Netflix Eureka实现注册中心,当然SpringCloud也可以使用ZooKeeper实现,但一般我们不会这样做
- 服务网关,Dubbo并没有本身的实现,只能通过其他第三方技术的整合,而SpringCloud有Zuul路由网关,作为路由服务器,进行消费者的请求分发,SpringCloud还支持断路器,与git完美集成分布式配置文件支持版本控制,事务总线实现配置文件的更新与服务自动装配等等一系列的微服务架构要素

### 3.SpringBoot和SpringCloud的区别？

SpringBoot专注于快速方便的开发单个个体微服务。 SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并 管理起来，为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞 选、分布式会话等等集成服务SpringBoot可以离开SpringCloud独立使用开发项目， 但是SpringCloud离不开SpringBoot ，属于依赖的关系. SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架

### 4.说说 RPC 的实现原理

首先需要有处理网络连接通讯的模块，负责连接建立、管理和消息的传输。其次需要有编解码的模块，因为网络通讯都是传输的字节码，需 要将我们使用的对象序列化和反序列化。剩下的就是客户端和服务器端的部分，服务器端暴露要开放的服务接口，客户调用服务接口的一个 代理实现，这个代理实现负责收集数据、编码并传输给服务器然后等待结果返回。

### 5.springcloud如何实现服务的注册?

1.服务发布时，指定对应的服务名,将服务注册到 注册中心(eureka zookeeper) 2.注册中心加@EnableEurekaServer,服务用@EnableDiscoveryClient，然后用ribbon或feign进行服务直接的调用发现。

### 6.springcloud断路器作用?

当一个服务调用另一个服务由于网络原因或自身原因出现问题，调用者就会等待被调用者的响应 当更多的服务请求到这些资源导致更多的 请求等待，发生连锁效应（雪崩效应） 断路器有完全打开状态:一段时间内 达到一定的次数无法调用 并且多次监测没有恢复的迹象 断路器完全打开 那么下次请求就不会请求到该 服务 半开:短时间内 有恢复迹象 断路器会将部分请求发给该服务，正常调用时 断路器关闭 关闭：当服务一直处于正常状态 能正常调用

### 7.如果使用 Spring Cloud 来实现，需要哪些组件？

Eureka 首先，我们需要一个注册中心 Eureka ，主要负责每个服务的注册和发现。 每个微服务中都有一个Euraka client组件，专门负责将这个服务的服务id（serviceId）、ip、端口等信息注册到Eureka server中。 Euraka Server是一个注册中心，该组件内部维护了一个注册表，保存了各个服务所在的机器ip和端口号等信息。 Feign 其次每个服务还需要一个远程服务调用的组件 Feign ，他主要负责与其他服务建立连接，构造请求，然后发起请求来调用其他服务来 获取数据。 Ribbon 然后我们一个服务可能会部署很多台机器，那么我们使用Feign 去调用这个服务的时候，到底把请求发送到哪台机器上去呢？此时我 们就需要一个组件来根据一定的策略来选择一台机器。不管怎么选的，总之得选一台机器给 Feign 去调用就好了。 这个组件就是 Ribbon，Ribbon 主要负责就是负载均衡。Ribbon 会定期去从Eureka 注册中心拉取注册中心，缓存到本地，每次发起 远程调用的时候，Ribbon 就会从 Eureka 注册表拉去下来的数据中挑选一个机器让 Feign 来发起远程调用。 Zuul 我们这么多的微服务，如果一个服务一个IP，使用方都需要进行调用的话，是不是得知道每一个服务的IP地址才行呢？那得记住多少 才行呀，多不好管理。 如果有一个统一的地址，然后根据不同的请求路径来跟我进行转发多少是不，比如 /user/* 是转发到用户服务 ，/product/* 是转向到 商品服务等等。我使用的时候，只需要访问同一个IP ，只是路径不一样，就行了。 Spring Cloud 也给我们提供了一个组件，那就是 Zuul ，他是一个网关，就是负责网络的路由的。每个请求都经过这个网关，我们还 可以做统一鉴权等等很多事情。 Hystrix 还有一个东西也得说一下，就是 Hystrix，它是一个隔离、熔断以及降级的一个框架 。 在微服务的相互调用过程中，可能会出现被调用服务错误或者超时的情况，从而导致整个系统崩溃不可用，也就是我们常说的服务雪 崩问题，Hystrix 的存在就是为了结局这种问题的。

整个调用流程：

1. 首先每个服务启动的时候都需要往注册中心进行注册。
2. 用户先对网关发起下单请求，网关收到请求后发现呃，是下单操作，要到订单系统，然后把请求路由到订单系统。
3. 订单系统啪啦啪啦一顿操作，然后通过 Feign 去调用 库存系统减库存，通知仓储服务发货，调用积分系统加积分。
4. 在发起调用之前，订单系统还得通过Ribbon 去注册中心去拉取各系统的注册表信息，并且挑一台机器给 Feign 来发起网络调用。



## Spring Cloud **Eureka**

### 1.服务发现（Service Discovery）

它抽象出来了一个注册中心，当一个新的服务上线时，它会将自己的 IP 和端口注册到注册中心去，会对注册的服务进行定期的心跳检测，当发现服务状态异常时将其从注册中心剔除下线。服务 A 只要从注册中心中获取服务 B 的信息即可，即使当服务 B 的 IP 或者端口变更了，服务 A 也无需修改，从一定程度上解耦了服务。服务发现目前业界有很多开源的实现，比如 `apache` 的 [zookeeper](https://github.com/apache/zookeeper)、 `Netflix` 的 [eureka](https://github.com/Netflix/eureka)、 `hashicorp` 的 [consul](https://github.com/hashicorp/consul)、 `CoreOS` 的 [etcd](https://github.com/etcd-io/etcd)。

### 2.Eureka 是什么

采用的是 Client / Server 模式进行设计，基于 http 协议和使用 Restful Api 开发的服务注册与发现组件，提供了完整的服务注册和服务发现，可以和 `Spring Cloud` 无缝集成。其中 Server 端扮演着服务注册中心的角色，主要是为 Client 端提供服务注册和发现等功能，维护着 Client 端的服务注册信息，同时定期心跳检测已注册的服务当不可用时将服务剔除下线，Client 端可以通过 Server 端获取自身所依赖服务的注册信息，从而完成服务间的调用。

### 3.服务注册中心（Eureka Server）

我们在项目中引入 `Eureka Server` 的相关依赖，然后在启动类加上注解 `@EnableEurekaServer` ，就可以将其作为注册中心。

我们继续添加两个模块 `service-provider` ， `service-consumer` ，然后在启动类加上注解 `@EnableEurekaClient` 并指定注册中心地址为我们刚刚启动的 `Eureka Server` ，再次访问可以看到两个服务都已经注册进来了。

可以看到 `Eureka` 的使用非常简单，只需要添加几个注解和配置就实现了服务注册和服务发现，接下来我们看看它是如何实现这些功能的。

服务注册（Register）

注册中心提供了服务注册接口，用于当有新的服务启动后进行调用来实现服务注册，或者心跳检测到服务状态异常时，变更对应服务的状态。服务注册就是发送一个 `POST` 请求带上当前实例信息到类 `ApplicationResource` 的 `addInstance` 方法进行服务注册。

可以看到方法调用了类 `PeerAwareInstanceRegistryImpl` 的 `register` 方法，该方法主要分为两步：

1. 调用父类 `AbstractInstanceRegistry` 的 `register` 方法把当前服务注册到注册中心
2. 调用 `replicateToPeers` 方法使用异步的方式向其它的 `Eureka Server` 节点同步服务注册信息

服务注册信息保存在一个嵌套的 `map` 中，

第一层 `map` 的 `key` 是应用名称（对应 `Demo` 里的 `SERVICE-PROVIDER` ），第二层 `map` 的 `key` 是应用对应的实例名称（对应 `Demo` 里的 `mghio-mbp:service-provider:9999` ），一个应用可以有多个实例。

服务续约（Renew）

服务续约会由服务提供者（比如 `Demo` 中的 `service-provider` ）定期调用，类似于心跳，用来告知注册中心 `Eureka Server` 自己的状态，避免被 `Eureka Server` 认为服务时效将其剔除下线。服务续约就是发送一个 `PUT` 请求带上当前实例信息到类 `InstanceResource` 的 `renewLease` 方法进行服务续约操作。

进入到 `PeerAwareInstanceRegistryImpl` 的 `renew` 方法可以看到，服务续约步骤大体上和服务注册一致，先更新当前 `Eureka Server` 节点的状态，服务续约成功后再用异步的方式同步状态到其它 `Eureka Server` 节上。

服务下线（Cancel）

当服务提供者（比如 `Demo` 中的 `service-provider` ）停止服务时，会发送请求告知注册中心 `Eureka Server` 进行服务剔除下线操作，防止服务消费者从注册中心调用到不存在的服务。服务下线就是发送一个 `DELETE` 请求带上当前实例信息到类 `InstanceResource` 的 `cancelLease` 方法进行服务剔除下线操作。

进入到 `PeerAwareInstanceRegistryImpl` 的 `cancel` 方法可以看到，服务续约步骤大体上和服务注册一致，先在当前 `Eureka Server` 节点剔除下线该服务，服务下线成功后再用异步的方式同步状态到其它 `Eureka Server` 节上。

服务剔除（Eviction）

服务剔除是注册中心 `Eureka Server` 在启动时就启动一个守护线程 `evictionTimer` 来定期（默认为 `60` 秒）执行检测服务的，判断标准就是超过一定时间没有进行 `Renew` 的服务，默认的失效时间是 `90` 秒，也就是说当一个已注册的服务在 `90` 秒内没有向注册中心 `Eureka Server` 进行服务续约（Renew），就会被从注册中心剔除下线。失效时间可以通过配置 `eureka.instance.leaseExpirationDurationInSeconds` 进行修改，定期执行检测服务可以通过配置 `eureka.server.evictionIntervalTimerInMs` 进行修改。

### 4.服务提供者（Service Provider）

对于服务提供方（比如 `Demo` 中的 `service-provider` 服务）来说，主要有三大类操作，分别为 `服务注册（Register）` 、 `服务续约（Renew）` 、 `服务下线（Cancel）` ，接下来看看这三个操作是如何实现的。

服务注册（Register）

一个服务要对外提供服务，首先要在注册中心 `Eureka Server` 进行服务相关信息注册，能进行这一步的前提是你要配置 `eureka.client.register-with-eureka=true` ，这个默认值为 `true` ，注册中心不需要把自己注册到注册中心去，把这个配置设为 `false`。

服务续约（Renew）

服务续约是由服务提供者方定期（默认为 `30` 秒）发起心跳的，主要是用来告知注册中心 `Eureka Server` 自己状态是正常的还活着，可以通过配置 `eureka.instance.lease-renewal-interval-in-seconds` 来修改，当然服务续约的前提是要配置 `eureka.client.register-with-eureka=true` ，将该服务注册到注册中心中去。

服务下线（Cancel）

当服务提供者方服务停止时，要发送 `DELETE` 请求告知注册中心 `Eureka Server` 自己已经下线，好让注册中心将自己剔除下线，防止服务消费方从注册中心获取到不可用的服务。这个过程实现比较简单，在类 `DiscoveryClient` 的 `shutdown` 方法加上注解 `@PreDestroy` ，当服务停止时会自动触发服务剔除下线，执行服务下线逻辑。

### 5.服务消费者（Service Consumer）

这里的服务消费者如果不需要被其它服务调用的话，其实只会涉及到两个操作，分别是从注册中心 `获取服务列表（Fetch）` 和 `更新服务列表（Update）` 。如果同时也需要注册到注册中心对外提供服务的话，那么剩下的过程和上文提到的服务提供者是一致的，这里不再阐述，接下来看看这两个操作是如何实现的。

获取服务列表（Fetch）

服务消费者方启动之后首先肯定是要先从注册中心 `Eureka Server` 获取到可用的服务列表同时本地也会缓存一份。这个获取服务列表的操作是在服务启动后 `DiscoverClient` 类实例化的时候执行的。

能发生这个获取服务列表的操作前提是要保证配置了 `eureka.client.fetch-registry=true` ，该配置的默认值为 `true`。

更新服务列表（Update）

由上面的 `获取服务列表（Fetch）` 操作过程可知，本地也会缓存一份，所以这里需要定期的去到注册中心 `Eureka Server` 获取服务的最新配置，然后比较更新本地缓存，这个更新的间隔时间可以通过配置 `eureka.client.registry-fetch-interval-seconds` 修改，默认为 `30` 秒，能进行这一步更新服务列表的前提是你要配置 `eureka.client.register-with-eureka=true` ，这个默认值为 `true` 。

### 6.**注册表存储结构**

- **registry**的**CocurrentHashMap**，就是注册表的核心结构。
- Eureka Server的注册表直接基于**纯内存**，即在内存里维护了一个数据结构。各个服务的注册、服务下线、服务故障，全部会在内存里维护和更新这个注册表。
- 各个服务每隔30秒拉取注册表的时候，Eureka Server就是直接提供内存里存储的有变化的注册表数据给他们就可以了。同样，每隔30秒发起心跳时，也是在这个纯内存的Map数据结构里更新心跳时间。
- 这个ConcurrentHashMap的key就是服务名称,value则代表了一个服务的多个服务实例。
- **Map<String, Lease<InstanceInfo**>>,这个Map的key就是**服务实例的id**,value是一个叫做**Lease**的类，它的泛型是一个叫做**InstanceInfo**的东西,这个InstanceInfo就代表了**服务实例的具体信息**，比如机器的ip地址、hostname以及端口号。而这个Lease，里面则会维护每个服务**最近一次发送心跳的时间**.

### 7.**Eureka Server端优秀的多级缓存机制**

- 在拉取注册表的时候：
  - 首先从**ReadOnlyCacheMap**里查缓存的注册表。
  - 若没有，就找**ReadWriteCacheMap**里缓存的注册表。
  - 如果还没有，就从**内存中获取实际的注册表数据。**
- 在注册表发生变更的时候：
  - 会在内存中更新变更的注册表数据，同时**过期掉ReadWriteCacheMap**。
  - 此过程不会影响ReadOnlyCacheMap提供人家查询注册表。
  - 一段时间内（默认30秒），各服务拉取注册表会直接读ReadOnlyCacheMap
  - 30秒过后，Eureka Server的后台线程发现ReadWriteCacheMap已经清空了，也会清空ReadOnlyCacheMap中的缓存
  - 下次有服务拉取注册表，又会从内存中获取最新的数据了，同时填充各个缓存。

**多级缓存机制的优点是什么？**

- 尽可能保证了内存注册表数据不会出现频繁的读写冲突问题。
- 并且进一步保证对Eureka Server的大量请求，都是快速从纯内存走，性能极高。

### 8.简单谈一下Eureka 中的三种角色分别是什么？

1、Eureka Server 通过Register、Get、Renew 等接口提供服务的注册和发现。 2、Application Service (Service Provider) 服务提供方，把自身的服务实例注册到Eureka Server 中 3、Application Client (Service Consumer) 服务调用方，通过Eureka Server 获取服务列表，消费服务。

### 9.Eureka和zookeeper都可以提供服务注册与发现的功能，请说说两个的区别？

Zookeeper保证了CP（C：一致性，P：分区容错性），Eureka保证了AP（A：高可用）1.当向注册中心查询服务列表时，我们可以容忍注 册中心返回的是几分钟以前的信息，但不能容忍直接down掉不可用。也就是说，服务注册功能对高可用性要求比较高，但zk会出现这样一 种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新选leader。问题在于，选取leader时间过长，30 ~ 120s，且 选取期间zk集群都不可用，这样就会导致选取期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率 会发生的事，虽然服务能够恢复，但是漫长的选取时间导致的注册长期不可用是不能容忍的。 2.Eureka保证了可用性，Eureka各个节点是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点仍然可以提供注册和查询服务。 而Eureka的客户端向某个Eureka注册或发现时发生连接失败，则会自动切换到其他节点，只要有一台Eureka还在，就能保证注册服务可 用，只是查到的信息可能不是最新的。除此之外，Eureka还有自我保护机制，如果在15分钟内超过85%的节点没有正常的心跳，那么 Eureka就认为客户端与注册中心发生了网络故障，此时会出现以下几种情况： ①、Eureka不在从注册列表中移除因为长时间没有收到心跳而应该过期的服务。 ②、Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上（即保证当前节点仍然可用） ③、当网络稳定时，当前实例新的注册信息会被同步到其他节点。因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况， 而不会像Zookeeper那样使整个微服务瘫痪

### 10.eureka自我保护机制是什么?

当Eureka Server 节点在短时间内丢失了过多实例的连接时（比如网络故障或频繁启动关闭客户端）节点会进入自我保护模式，保护注册 信息，不再删除注册数据，故障恢复时，自动退出自我保护模式。

### 11.作为服务注册中心，Eureka比Zookeeper好在哪里?

1.Eureka保证的是可用性和分区容错性，Zookeeper 保证的是一致性和分区容错性 。 2.Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故 障。而不会像zookeeper那样使整个注册服务瘫痪。

### 12.服务的时效性

Zookeeper 的时效性更好一些，注册或者是服务挂了，一般秒级别就能感知到。 而 Eureka，默认的配置可能会有从几十秒到分钟级别。上线一个新的服务，到其他人发现他可能要一分钟，还可能不止,因为SpringCloud是通过 ribbon 去获取每个服务缓存的 Eureka 的注册表进行负载均衡的。ribbon 本身还有自己的缓存机制，还有自己的时间间隔。 当服务发生故障了，默认是隔 60秒采取检查，发现这个服务上一次是在 60s 之前，Eureka 默认是超过 90秒才会任务它已经死掉了。这时候差不多已经过去 2分钟了。 再则，Eureka 里面默认是 30 秒才会把 ReadWrite 缓存的数据同步到 ReadOnly 缓存中，其他服务默认也是 30秒 才会去重新拉取一次ReadOnly 缓存到本地服务表中。 这一趟下来算算时间还是很长的。

Eureka 服务发现慢的问题及调优

zk是因为一上线服务跟下线服务就会立马发生数据同步，同时有服务进行节点的监听，很快的时间内就可以感知到服务上线下线。所以基本上不需要怎么去优化。

Eureka 可以针对下面的几个点来优化：

1. 优化服务发现时间；
2. 优化定时从ReadWrite缓存到 ReadOnly 缓存的时间；
3. 优化定时心跳检查的时间；

```html
1 而 eureka，必须优化参数
2 ## Eureka 两个缓存之间的同更新时间 配置成 3s
3 eureka.server.responseCacheUpdateIntervalMs = 3000
4 ## Eureka client 去 Eureka 中拉取服务注册表的时间 配置 3 秒
eureka.client.registryFetchIntervalSeconds 5 = 30000
6 ## 心跳上报时间 默认 30s ，调整为 3s
7 eureka.client.leaseRenewalIntervalInSeconds = 30
8 ## 线程多少时间检查一次心跳 默认 60s ,可以配置为 6000,6秒中
9 eureka.server.evictionIntervalTimerInMs = 60000
10 ## 服务发现的时效性，默认90 秒钟才下线。可以调整为 9s
11 eureka.instance.leaseExpirationDurationInSeconds = 90
```

### 13.Eureka底层架构

服务注册与发现怎么实现的

实际上当服务在拉取服务注册表的时候，其实客户端不是直接从 Eureka 中的 服务注册表中获取数据的。 Eureka 做了二级缓存，第一级叫做 ReadOnly 缓存，二级叫做 ReadWrite 缓存。 客户端会直接从ReadOnly 缓存中读取注册表信息。 当服务在进行注册的时候，先往服务注册表中写入注册信息，服务注册表更新了，立马会同步一份数据到 ReadWrite 缓存中去。 那什么时候 ReadWrite 缓存中的数据会到 ReadOnly 缓存中去? 此时有一个定时任务会定时去检查 ReadWrite 是否跟 ReadOnly 不一致,不一致就把数据同步到 ReadOnly 中去。 这个定时任务也默认是 30S。也可以自己配置。

这么做的好处在于，优化并发读写的冲突。 如果服务进行注册的时候，同时有服务来读去注册表信息，就会存在频繁的读写加锁的操作，写的时候就不能读，导致性能下降，所以我们需要避免大量的读写都去操作一个表。 那么有了这两层，其实大部分的读操作都会走 ReadOnly 缓存。只需要定时把 ReadWrite 缓存中的数据写入到 ReadOnly 就好了。

心跳与故障检测

服务注册中心还有一个很重要的功能就是 心跳与故障检查。心跳跟故障检测其实就是为了知道注册上来的这些服务是不是还活着的。 Eureka 还会开启一个定时任务定时去检查心跳，默认也是30秒，也可以自己设置。 当出现机器故障没有在约定的时间间隔内上报自己的状态，那么Eureka 就会把这台机器剔除注册表，同时更新到 ReadWrite 缓存中去。

但是把数据从ReadWrite 缓存同步到 ReadOnly 缓存是有时间间隔的。当服务消费者A 也只有等待下一次请求更新的时候才会把自己列表里面的服务给更新掉。 所以有时候会出现你注册上去的服务经过及时秒才被服务消费者发现，或者服务的某个节点出现故障，没有及时剔除掉。这里就是同步机制的时间差问题。

### 3.Spring Cloud Eureka 的使用

1 . 注册中心的定义

Eureka Server 是服务的注册中心，服务提供者在启动时会将服务信息注册到注册中心。它维护了集群中的服务列表和状态， 要实现一个注册中心分为4 步： 首先在pom .xml 中引人spring - cloud- starter- eureka - server 依赖， 然后通过＠ EnableEurekaServer 注解开启服务注册、发现功能， 接下来配置application . properties 配置文件，最后一步是服务的访问和使用。 ( 1 ) pom . xml 添加依赖。 ( 2 ) 通过＠EnableEurekaServer 注解开启对注册中心的支持。

```html
@SpringBootApplication
@EnableEurekaServer
public class EurekaserverApplication {
    public static void main（String[] args ) {
        SpringApplication.run(EurekaserverApplication.class,args) ;
    }
}
```

( 3 ) application .properties 配置。 配置Eureka Server 的服务名称、端口和服务地址。在默认情况下，服务注册中心会将自己也作为客户端来尝试注册，因此，应用程序需要禁用其注册行为。具体参数为 eureka.client. register-with- eureka =false 和eureka.client. fetch- registry = false。

( 4 ）访问服务地址。 启动应用程序，在浏览器地址栏中输入http: // localhost:9001访问注册中心。

2.服务提供者的定义

服务提供者（ Service Provider ） 即Eureka Client ， 是服务的定义者。服务提供者在启动时，将自身的服务注册到注册中心，服务消费者从注册中心请求到服务列表后，便会调用具体的服务提供者的服务接口，实现远程过程调用。要实现一个服务提供者分为5 步：首先在pom.xml 中引人spring-cloud - starter- eureka 依赖，然后通过＠ Enable EurekaClient 注解开启服务发现的功能，接着配置application. properties 配置文件，再定义服务接口， 最后一步是服务的访问和使用。 ( 1 ) p om.x ml 添加依赖。 ( 2 ) 通过＠EnableEurekaClient 注解开启服务发现的功能。

```html
@SpringBootApplication
@EnableEurekaClient
public class EurekaclientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaclientApplication.class,args) ;
    }
}
```

( 3 ) application.properties 配置。 配置Eureka Server 的服务名称、端口和注册中心的地址。

( 4 ) 定义服务。

3.服务消费者的定义

服务消费者（ Service Consumer ） 即Eureka Consumer ， 是服务的具体使用者， 一个服务实例常常既是服务消费者也是服务提供者。要实现一个服务消费者分为5 步： 首先在pom.xm l 中引人spring-cloud-starter-eureka 依赖，然后通过＠ EnableEurekaClient 注解开启服务发现的功能，如果通过Feign 调用， 则需要通过＠ EnableFeignClients 开启对Feign 的支持， 接下来配置application. properties 配置文件， 最后调用服务提供者暴露出来的服务接口。 ( 1) pom.xml 添加依赖。

( 2 ）通过＠ EnableEurekaClient 注解开启服务发现的功能。如果通过F e ign 远程调用，则需要通过＠ EnableFeignClients 开启对Feign 的支持； 如果通过

RestTemplate 调用， 则需要定义RestTemplate 实例。

( 3 ) application.properties 配置。 配置服务名称、端口和注册中心的地址。

( 4 ) 调用RestTemplate 服务。

( 5 ) 调用服务。

( 6 ）定义基于Feign 接口的服务。 基于Feign 接口的服务调用简单方便，上述代码在EurekaconsumerApplication 中已经通过加入＠ EnableFeignClients 开启了对Feign 的支持，这里只需要应用程序定义Feign接口便可。 定义Feign 接口分为2 步：首先通过＠ FeignClient（“ serviceProducerName”）定义需要代理的服务实例名（“ EUREKA-CLIENT ”即服务提供者的实例名称），然后定义一个调用远程服务的方法。方法中＠ RequestMapping 的value （这里为“／serviceProducer" ）需要与服务提供者的地址相对应，这样， Feign 才能知道代理需要具体调用服务提供者的哪个方法。 ( 7 ）调用基于Feign 接口的服务。

## Spring Cloud Config

### 1.Spring Cloud Config

随着项目复杂度的增加和微服务开发组件的细化，散落在服务器各个角落的微服务组件需要一套在线的配置服务；一方面为整个服务提供统一的配置，避免在每个微服务中修改配置带来的不便和易出错的问题；另一方面保证了微服务配置能自动化更新到各个组件中，避免在修改配置后重启时出现服务不稳定的情况。

Spring Cloud Config为分布式系统提供统一的配置管理工具，应用程序在使用过程中可以像使用本地配置一样方便地添加、访问、修改配心的配置。SpringCloud Config将Environment的PropertySource抽象和配置中心的配置进行映射，以便应用程序可以在任何场景下获取和修改配置。

创建一个Config Server分为4步，首先在pom.xml中引人spring-cloud-config-server和spring-boot-starter-actuator依赖，然后通过＠EnableConfigServer注解开启配置服务，接着配置appIication. properties配置文件，最后一步是访问和使用。

### 2.Spring Cloud Config的原理

Spring Cloud Config支持将配置存储在配置中心的本地服务器、Git仓库或Subversion。在SpringCloud Config的线上环境中，通常将配置文件集中放置在一个Git仓库里，然后通过配置中心服务端（Config Server）来管理所有的配置文件；当某个服务实例需要添加或更新配置时，只要在该服务实例的本地将配置文件进行修改，然后推送到Git仓库，其他服务实例通过配置中心从Git服务端获取最新的配置信息。对于配置中心来说，每个服务实例都相当于客户端（Config Client）。

为了保证系统的稳定，配置中心服务端可以进行多副本集群部署，前端使用负载均衡实现服务之间的请求转发。

## Spring Cloud Consul

### 1.Consul

Consul是HashiCorp公司推的用于实现分布式系统服务注册、发现和配置的开源工具。和SpringCloud Eureka一样，Consul也是一个一站式的服务注册与发现框架，内置了服务注册与发现、分布式一致性协议实现、健康检查、Key-Vlaue存储和多数据中心方案，因此不需要依赖第三方工具（例如Zookeeper）便可完成服务注册与发现，简单易用。

Consul采用Go编写，支持Linux、Windows和macOS系统，可移植性强。安装包仅是一个可执行文件，方便部署，且与Docker配合方便。

### 2.Spring Cloud Consul的原理

Consul是一个支持多数据中心的高可用的分布式服务注册、发现和配置服务，采用Raft一致性协议算法来保证服务的高可用；使用Gossip协议管理成员状态和广播消息，并且支持ACL访问控制。相对于SpringCloud Eureka或其他服务注册与发现框架，Consul有以下特点。

Consul的特性

- 高效的Raft一致性算法：Consul使用Raft一致性算法来保证集群状态的一致性，在实现上比Paxos一致性算法更简单（ZooKeeper采用Paxos一致性算法实现，Etcd采用Raft一致性算法实现）。
- 支持多数据中心：Consul支持多数据中心。多数据中心可以使集群避免单数据中心的单点故障问题，但在部署的过程中需要考虑网络延迟、数据分片等情况。ZooKeeper 和Etcd均不支持多数据中心。
- 健康检查：Consul支持健康检查，默认每10s做一次健康检查，保证注册中心的服务均可用，Etcd不提供此功能。
- HTTP和DNS支持：Consul支持HTTP和DNS协议接口。ZooKeeper集成了DNS协议，实现复杂，Etcd只支持HTTP。

Consul的角色

Consul按照功能可分为服务端和客户端。

- 服务端：Server，用于保存服务配置信息的高可用集群，在局域网内与本地客户端通信，在广域网内与其他数据中心通信。每个数据中心的Server数量都推荐为3个以上以保证服务高可用。在集群中，Server又分为Server Leader和Server。Server Leader负责同步注册信息和进行各个节点的健康检查，同时负责整个集群的写请求。Server负责把配置信息持久化并接收读请求。Server在一个数据中心( Data Center, DC ）内使用LAN Gossip协议的一致性算法，在跨数据中心内使用WAN Gossip协议的一致性算法。
- 客户端：Client，是无状态的服务，将HTTP和 DNS协议的接口请求转发给局域网内的服务端集群。

Consul的服务端和客户端还支持跨数据中心的访问，提供了跨区域的高可用功能。

Consul的服务注册与发现流程

(1 ）服务注册：Producer在启动时会向Consul服务端发送一个POST注册请求，注册请求中包含服务地址和端口等信息。

(2 ）健康检查：Consul服务端在接收Producer的注册信息后，每10s （默认）会向Producer发送一个健康检查的请求，检验Producer是否健康。

( 3 ）服务发现：当Consumer发起请求（请求格式为“/api/address”）时，首先会从Consul服务端获取一个包含Producer的服务地址和端口的临时表。该表只包含通过了健康检查的Producer的可用服务列表。

( 4 ）服务请求：客户端从临时表中获取一个可用的服务地址，向Producer发送请求。Producer在收到请求后返回请求响应。

### 3.Consul的使用

Consul是应用服务的注册中心，服务注册与发现均在Consul上被完成。Consu的使用分3步：下载安装包、启动服务端、启动客户端。

Consul服务提供者

服务提供者在启动的时候会将自身的服务地址状态上报Consul Server，服务消费者在每次发起请求时都要先从Consul Server获取可用的临时服务列表（该列表维护了可用的服务提供者的地址），再选择一个可用的服务提供者的地址进行服务调用。实现一个服务提供者分为5步：首先在pom.xml中加入spring-cloud-starter-consul-discovery依赖，然后通过@EnableDiscoveryClient注解开启服务发现的功能，接着配置application. properties配置文件、定义服务接口，最后一步是访问和使用。

Consul服务消费者

服务消费者是具体的服务调用方，它每次发起请求都先从Consul Server获取可用的临时服务列表（该列表维护了可用的服务提供者的地址），然后选择一个可用的服务提供者的地址实现服务调用。要实现一个服务消费者分为4步：首先需要在pom.xml中加spring-cloud-spring-cloud-starter-consul-discovery依赖，然后配置apIication. properties配置文件，接着获取临时服务列表，最后一步是访问和使用。

## **Spring Cloud Feign**

### 1.Spring Cloud Feign

Feign是一种声明式、模板化的HTTP Client。它的目标是编写Java HTTP Client变得更简单。Feign通过使用Jersey和CXF等工具实现一个HTTP Client，用于构建REST或SOAP的服务。Feign还支持用户基于常用的HTTP工具包（OkHTTP、HTTPComponents ）实现自定义的HTTP Client。

Feign基于注解的方式将HTTP请求模板化。Feign将HTTP请求参数写入Template,极大地简化了HTTP请求。尽管Feign目前只支持基于文本的HTTP请求，不适合文件的上传和下载，但它为HTTP请求提供了更多可能性，比如请求回放等功能。同时，Feign使HTTP单元测试变得更加方便。

### 2.Feign的应用

Feign提供了声明式接口编程的方式，其应用一般依赖服务发现组件来实现远程接口调用，在并发要求不高的情况下可以作为RPC方案使用，实现服务之间的解耦。Feign应用分为服务提供者和服务消费者。

要实现一个服务提供者分为5步：首先在pomxm中加入spring-cloud-starter-feign 依赖，然后通过＠EnableFeignClients注解开启对Feign的支持，接下来配置application. properties配置文件，再创建接口并通过@FeignClient（"EUREKA-CLIENT）来实现注解的配置，定义需要远程调用的方法，最后一步是注入FeignClientInterface依赖，服务的访问和使用。

### 3.Feign的常用注解

Feign通过SpringMVC的注解来实现对HTTP参数的封调用，常用注解。

- @RequestLine：Request定义一个HTTP Method和UriTemplate，使用｛｝进行包装，并对＠Param注释参数进行解析
- @Param：定义一个HTTP请求模饭，模板中的参数将根据名称来解析
- @Headers：定义一个HeaderTemplate，在UriTemplate中使用
- @QueryMap：定义一个基于Name-Value的参数列表，也可以是Java实体类，最终以查询字符串的方式传输
- @HeaderMap：定义一个基于Name-Value的参数列表，用于扩展HTTP Headers
- @Body：定义一个类似UriTemplate或者HeaderTemplate的模板，该模板使用＠Param注释参数解析相应的表达式

### 4.Feign底层

Feign，它其实就是对一个接口打了一个注解，它会针对这个注解标注的接口生成动态代理对象，然后针对你的 feign 的动态代理代理 对象去调用他方法的时候，此时会在底层生成，http 协议格式的请求如：/order/create?productId=1 Feign底层的使用的HTTP 通信框架 HttpClient ,先会使用 Ribbon 从本地的 Eureka 注册表的缓存里面取出要调用服务的机器列表出 来，然后根据负载均衡算法，选择一台机器出来，然后针对选择出来的机器发送 Http 请求过去。

### 5.**feign的工作流程？**

1、通过动态代理生成实现类 2、根据接口类的注解生命规则，解析出底层的`methodhandler` 3、拦截器负责对请求和返回进行包装和处理 4、通过均衡负载http去访问远程服务

## **Spring Cloud Hystrix**

### 1.Hystrix的特性

服务熔断

Hystrix熔断器就像家中的安全阀一样，一旦某个服务不可用，熔断器会直接切断该链路上的请求，避免大量的元效请求影响系统稳定，并且熔断器有自我检测和恢复的功能，在服务状态恢复正常后会自动关闭。

服务降级

Hystrix通过fallback实现服务降级。在需要进行服务降级的类中定义一个fallback 方法，当请求的远程服务出现异常时，可以直接使用fallback方法返回异常信息，而不调用远程服务。fallback方法的返回值一般是系统默认的错误消息或者来向缓存中的数据，用以告知服务消费者当前服务处于不可用状态。Hystrix通过HystrixCommand实现服务降级，熔断器有闭路、开路和半开路3种状态。

( 1）当调用远程服务请求的失败数量超过一定比例（默认为50%)时，熔断器会切换到开路状态，这时所有请求都会直接返回失败信息而不调用远程服务。

( 2 ）熔断器保持开路状态一段时间后（默认为5s），会自动切换到半开路状态。

( 3 ）熔断器判断下一次请求的返回情况，如果请求成功，则熔断器切换回闭路状态，服务进入正常链路调用流程；否则重新切换到开路状态，并保持开路状态。

依赖隔离

Hystrix通过线程池和信号量两种方式实现服务之间的依赖隔离，这样即使其中一个服务出现异常，资源迟迟不能释放，也不会影响其他业务线程的正常运行。

( 1 ）线程池的隔离策略

Hystrix线程池的资源隔离为每个依赖的服务都分配一个线程池，每个线程池都处理特定的服务，多个服务之间的线程资源互不影向，以达到资源隔离的目标。当突然发生流量洪峰、请求增多时，来不及处理的任务将在线程队列中排队等候，这样做的好处是不会丢弃客户端请求，保障所有数据最终都会得到处理。

( 2）信号量的隔离策略

Hystrix信号量的隔离策略是为每个依赖的服务都分配一个信号量（原子数数器），当接收到用户请求时，先判断该请求依赖的服务所在的信号量值是否超过最大线程设置。若超过最大线程设置，则丢弃该类型的请求；若不超过，则在处理请求前执行"信号量+1"的操作，在请求返回后执行“信号量-1”的操作。当流量洪峰来临，收到的请求数量超过设置的最大值时，这种方式会直接将错误状态返回给客户端，不继续去请求依赖的服务。

请求缓存

Hytrix按照请求参数把请求结果缓存起来，当后面有相同的请求时不会再走完整的调用链流程，而是把上次缓存的结果直接返回，以达到服务快速响应和性能优化的目的；同时，缓存可作为服务降级的数据源，当远程服务不可用时，直接返回缓存数据。对于消费者来说，只是可能获取了过期的数据，这样就优雅地处理了系统异常。

请求合并

当微服务需要调用多个远程服务做结果的汇总时，需要使用请求合并。Hystrix采用异步消息订阅的方式进行请求合并。当应用程序需要请求多个接口时，采用异步调用的方式提交请求，然后订阅返回值，这时应用程序的业务可以接着执行其他任务而不用阻塞等待，当所有请求都返回时，应用程序会得到一个通知，取出返回值合并即可。

### 2.Hystrix的服务降级流程

Hystrix的服务熔断是依赖HystrixCommand指令来实现的，具体流程如下：

( 1 ）当有服务请求时，首先会根据注解创建一个HystrixCommand指令对象，该对象设置了服务调用失败的场景（如服务请求超时等）和调用失败后服务降级的业务逻辑方法。

( 2 ）熔断器判断状态，当熔断器处于开路状态时，直接调用服务降级的业务逻辑方法返回调用失败的反馈信息。

( 3 ）当熔断器处于半开路或者闭路状态时，服务会进行线程池和信号量等资源检查，如果有可用资源，则调用正常业务逻辑。如果调用正常业务逻辑成功，则返回成功后的消息；如果失败，则调用服务降级的业务逻辑，进行服务降级。

( 4 ）当熔断器处于半开路或者闭路状态时，如果当前服务线程池和信号量中没有可用资源，则执行服务降级的业务逻辑，返回失败信息。

( 5 ）当熔断器处于半开路状态并且本次服务执行失败时，熔断器会进入开路状态。

( 6）当正常业务逻辑处理超时或者出现错误时，HystrixCommand会执行服务降级的业务逻辑，返回失败信息.

( 7 ）线程池和信号量的资源检查及正常业务逻辑会将自己的状态和调用结果反馈给监控，监控将服务状态反馈给熔断器，以便熔断器判断熔断状态

### 3.Hystrix的使用

Hystrix的使用主要分为服务熔断、服务降级和服务监控3个方面

通过@EnableHystrix注解开启对服务熔断的支持，通过@EnableHysrixDashboard注解开启对服务监控的支持。注意，Hystrix一般和服务发现配合使用，这里使用@EnableEurekaClient开启了对服务发现客户端的支持。

通过@HystrixCommand(fallbackMethod=”exceptionHandler＂）在方法上定义了一个服务降级的命令。当远程方法调用失败时，Hystrix会向动调用fallbackMethod来完成服务熔断和降级，这里会调用exceptionHandler（）方法。

当关闭EUREKA-CLIENT远程服务时，远程服务将不可用，Hystrix的熔断器打开，程序直接调用fallbackMethod方法实现服务降级。

### 4.设计原则

- 阻止任何一个依赖服务耗尽所有的资源，比如 tomcat 中的所有线程资源。
- 避免请求排队和积压，采用限流和 `fail fast` 来控制故障。
- 提供 fallback 降级机制来应对故障。
- 使用资源隔离技术，比如 `bulkhead`（舱壁隔离技术）、`swimlane`（泳道技术）、`circuit breaker`（断路技术）来限制任何一个依赖服务的故障的影响。
- 通过近实时的统计/监控/报警功能，来提高故障发现的速度。
- 通过近实时的属性和配置**热修改**功能，来提高故障处理和恢复的速度。
- 保护依赖服务调用的所有故障情况，而不仅仅只是网络故障情况。

### 5.基于 Hystrix 线程池技术实现资源隔离

Hystrix 实现资源隔离，主要有两种技术：

- 线程池
- 信号量

默认情况下，Hystrix 使用线程池模式。

线程池

资源隔离，就是说，你如果要把对某一个依赖服务的所有调用请求，全部隔离在同一份资源池内，不会去用其它资源了，这就叫资源隔离。哪怕对这个依赖服务，比如说商品服务，现在同时发起的调用量已经到了 1000，但是分配给商品服务线程池内就 10 个线程，最多就只会用这 10 个线程去执行。不会因为对商品服务调用的延迟，将 Tomcat 内部所有的线程资源全部耗尽。

Hystrix 进行资源隔离，其实是提供了一个抽象，叫做 Command。这也是 Hystrix 最最基本的资源隔离技术。

利用 HystrixCommand 获取单条数据

我们通过将调用商品服务的操作封装在 HystrixCommand 中，限定一个 key，在这里我们可以简单认为这是一个线程池，每次调用商品服务，就只会用该线程池中的资源，不会再去用其它线程资源了。

利用 HystrixObservableCommand 批量获取数据

只要是获取商品数据，全部都绑定到同一个线程池里面去，我们通过 HystrixObservableCommand 的一个线程去执行，而在这个线程里面，批量把多个 productId 的 productInfo 拉回来。

信号量

信号量的资源隔离只是起到一个开关的作用，比如，服务 A 的信号量大小为 10，那么就是说它同时只允许有 10 个 tomcat 线程来访问服务 A，其它的请求都会被拒绝，从而达到资源隔离和限流保护的作用。

线程池与信号量区别

线程池隔离技术，并不是说去控制类似 tomcat 这种 web 容器的线程。更加严格的意义上来说，Hystrix 的线程池隔离技术，控制的是 tomcat 线程的执行。Hystrix 线程池满后，会确保说，tomcat 的线程不会因为依赖服务的接口调用延迟或故障而被 hang 住，tomcat 其它的线程不会卡死，可以快速返回，然后支撑其它的事情。

线程池隔离技术，是用 Hystrix 自己的线程去执行调用；而信号量隔离技术，是直接让 tomcat 线程去调用依赖服务。信号量隔离，只是一道关卡，信号量有多少，就允许多少个 tomcat 线程通过它，然后去执行。

**适用场景**：

- **线程池技术**，适合绝大多数场景，比如说我们对依赖服务的网络请求的调用和访问、需要对调用的 timeout 进行控制（捕捉 timeout 超时异常）。
- **信号量技术**，适合说你的访问不是对外部依赖的访问，而是对内部的一些比较复杂的业务逻辑的访问，并且系统内部的代码，其实不涉及任何的网络请求，那么只要做信号量的普通限流就可以了，因为不需要去捕获 timeout 类似的问题。

### 6.基于本地缓存的 fallback 降级机制

Hystrix 出现以下四种情况，都会去调用 fallback 降级机制：

- 断路器处于打开的状态。
- 资源池已满（线程池+队列 / 信号量）。
- Hystrix 调用各种接口，或者访问外部依赖，比如 MySQL、Redis、Zookeeper、Kafka 等等，出现了任何异常的情况。
- 访问外部依赖的时候，访问时间过长，报了 TimeoutException 异常。

两种最经典的降级机制

- 纯内存数据 在降级逻辑中，你可以在内存中维护一个 ehcache，作为一个纯内存的基于 LRU 自动清理的缓存，让数据放在缓存内。如果说外部依赖有异常，fallback 这里直接尝试从 ehcache 中获取数据。
- 默认值 fallback 降级逻辑中，也可以直接返回一个默认值。

在 `HystrixCommand`，降级逻辑的书写，是通过实现 getFallback() 接口；而在 `HystrixObservableCommand` 中，则是实现 resumeWithFallback() 方法。

### 7.Hystrix 线程池隔离与接口限流

Hystrix 通过判断线程池或者信号量是否已满，超出容量的请求，直接 Reject 走降级，从而达到限流的作用。

限流是限制对后端的服务的访问量，比如说你对 MySQL、Redis、Zookeeper 以及其它各种后端中间件的资源的访问的限制，其实是为了避免过大的流量直接打死后端的服务。

线程池隔离技术的设计

Hystrix 采用了 Bulkhead Partition 舱壁隔离技术，来将外部依赖进行资源隔离，进而避免任何外部依赖的故障导致本服务崩溃。

**舱壁隔离**，是说将船体内部空间区隔划分成若干个隔舱，一旦某几个隔舱发生破损进水，水流不会在其间相互流动，如此一来船舶在受损时，依然能具有足够的浮力和稳定性，进而减低立即沉船的危险。

Hystrix 对每个外部依赖用一个单独的线程池，这样的话，如果对那个外部依赖调用延迟很严重，最多就是耗尽那个依赖自己的线程池而已，不会影响其他的依赖调用。

线程池机制的优点

- 任何一个依赖服务都可以被隔离在自己的线程池内，即使自己的线程池资源填满了，也不会影响任何其他的服务调用。
- 服务可以随时引入一个新的依赖服务，因为即使这个新的依赖服务有问题，也不会影响其他任何服务的调用。
- 当一个故障的依赖服务重新变好的时候，可以通过清理掉线程池，瞬间恢复该服务的调用，而如果是 tomcat 线程池被占满，再恢复就很麻烦。
- 如果一个 client 调用库配置有问题，线程池的健康状况随时会报告，比如成功/失败/拒绝/超时的次数统计，然后可以近实时热修改依赖服务的调用配置，而不用停机。
- 基于线程池的异步本质，可以在同步的调用之上，构建一层异步调用层。

简单来说，最大的好处，就是资源隔离，确保说任何一个依赖服务故障，不会拖垮当前的这个服务。

线程池机制的缺点

- 线程池机制最大的缺点就是增加了 CPU 的开销。 除了 tomcat 本身的调用线程之外，还有 Hystrix 自己管理的线程池。
- 每个 command 的执行都依托一个独立的线程，会进行排队，调度，还有上下文切换。

我们可以用 Hystrix semaphore 技术来实现对某个依赖服务的并发访问量的限制，而不是通过线程池/队列的大小来限制流量。

semaphore 技术可以用来限流和削峰，但是不能用来对调用延迟的服务进行 timeout 和隔离。

`execution.isolation.strategy` 设置为 `SEMAPHORE`，那么 Hystrix 就会用 semaphore 机制来替代线程池机制，来对依赖服务的访问进行限流。如果通过 semaphore 调用的时候，底层的网络调用延迟很严重，那么是无法 timeout 的，只能一直 block 住。一旦请求数量超过了 semaphore 限定的数量之后，就会立即开启限流。

### 8.Hystrix断路器机制

断路器很好理解, 当Hystrix Command请求后端服务失败数量超过一定比例(默认50%), 断路器会切换到开路状态(Open). 这时所有请求会直接失败而不会发送到后端服务. 断路器保持在开路状态一段时间后(默认5秒), 自动切换到半开路状态(HALF-OPEN). 这时会判断下一次请求的返回情况, 如果请求成功, 断路器切回闭路状态(CLOSED), 否则重新切换到开路状态(OPEN). Hystrix的断路器就像我们家庭电路中的保险丝, 一旦后端服务不可用, 断路器会直接切断请求链, 避免发送大量无效请求影响系统吞吐量, 并且断路器有自我检测并恢复的能力。

### 9.Hystrix 隔离策略细粒度控制

execution.isolation.strategy

指定了 HystrixCommand.run() 的资源隔离策略：`THREAD` or `SEMAPHORE`，一种基于线程池，一种基于信号量。

```html
// to use thread isolation
HystrixCommandProperties.Setter().withExecutionIsolationStrategy(ExecutionIsolationStrategy.THREAD)

// to use semaphore isolation
HystrixCommandProperties.Setter().withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)
```

线程池机制，每个 command 运行在一个线程中，限流是通过线程池的大小来控制的；信号量机制，command 是运行在调用线程中（也就是 Tomcat 的线程池），通过信号量的容量来进行限流。

如何在线程池和信号量之间做选择？

**默认的策略**就是线程池。

**线程池**其实最大的好处就是对于网络访问请求，如果有超时的话，可以避免调用线程阻塞住。

而使用信号量的场景，通常是针对超大并发量的场景下，每个服务实例每秒都几百的 `QPS`，那么此时你用线程池的话，线程一般不会太多，可能撑不住那么高的并发，如果要撑住，可能要耗费大量的线程资源，那么就是用信号量，来进行限流保护。一般用信号量常见于那种基于纯内存的一些业务逻辑服务，而不涉及到任何网络访问请求。

command key & command group

每一个 command，都可以设置一个自己的名称 command key，同时可以设置一个自己的组 command group。

command group 是一个非常重要的概念，默认情况下，就是通过 command group 来定义一个线程池的，而且还会通过 command group 来聚合一些监控和报警信息。同一个 command group 中的请求，都会进入同一个线程池中。

command thread pool

ThreadPoolKey 代表了一个 HystrixThreadPool，用来进行统一监控、统计、缓存。默认的 ThreadPoolKey 就是 command group 的名称。每个 command 都会跟它的 ThreadPoolKey 对应的 ThreadPool 绑定在一起。

如果不想直接用 command group，也可以手动设置 ThreadPool 的名称。

command key & command group & command thread pool

**command key** ，代表了一类 command，一般来说，代表了下游依赖服务的某个接口。

**command group** ，代表了某一个下游依赖服务，这是很合理的，一个依赖服务可能会暴露出来多个接口，每个接口就是一个 command key。command group 在逻辑上对一堆 command key 的调用次数、成功次数、timeout 次数、失败次数等进行统计，可以看到某一个服务整体的一些访问情况。**一般来说，推荐根据一个服务区划分出一个线程池，command key 默认都是属于同一个线程池的。**

比如说有一个服务 A，你估算出来服务 A 每秒所有接口加起来的整体 `QPS` 在 100 左右，你有一个服务 B 去调用服务 A。你的服务 B 部署了 10 个实例，每个实例上，用 command group 去对应下游服务 A。给一个线程池，量大概是 10 就可以了，这样服务 B 对服务 A 整体的访问 QPS 就大概是每秒 100 了。

但是，如果说 command group 对应了一个服务，而这个服务暴露出来的几个接口，访问量很不一样，差异非常之大。你可能就希望在这个服务对应 command group 的内部，包含对应多个接口的 command key，做一些细粒度的资源隔离。**就是说，希望对同一个服务的不同接口，使用不同的线程池。**

```html
command key -> command group

command key -> 自己的 thread pool key
```

逻辑上来说，多个 command key 属于一个 command group，在做统计的时候，会放在一起统计。每个 command key 有自己的线程池，每个接口有自己的线程池，去做资源隔离和限流。

说白点，就是说如果你的 command key 要用自己的线程池，可以定义自己的 thread pool key，就 ok 了。

coreSize

设置线程池的大小，默认是 10。一般来说，用这个默认的 10 个线程大小就够了。

```html
HystrixThreadPoolProperties.Setter().withCoreSize(int value);
```

queueSizeRejectionThreshold

如果说线程池中的 10 个线程都在工作中，没有空闲的线程来做其它的事情，此时再有请求过来，会先进入队列积压。如果说队列积压满了，再有请求过来，就直接 reject，拒绝请求，执行 fallback 降级的逻辑，快速返回。

控制 queue 满了之后 reject 的 threshold，因为 maxQueueSize 不允许热修改，因此提供这个参数可以热修改，控制队列的最大大小。

```html
HystrixThreadPoolProperties.Setter().withQueueSizeRejectionThreshold(int value);
```

execution.isolation.semaphore.maxConcurrentRequests

设置使用 SEMAPHORE 隔离策略的时候允许访问的最大并发量，超过这个最大并发量，请求直接被 reject。

这个并发量的设置，跟线程池大小的设置，应该是类似的，但是基于信号量的话，性能会好很多，而且 Hystrix 框架本身的开销会小很多。

默认值是 10，尽量设置的小一些，因为一旦设置的太大，而且有延时发生，可能瞬间导致 tomcat 本身的线程资源被占满。

```html
HystrixCommandProperties.Setter().withExecutionIsolationSemaphoreMaxConcurrentRequests(int value);
```

### 10.Hystrix 执行时内部原理

Hystrix 最基本的支持高可用的技术：**资源隔离** + **限流**。

- 创建 command；
- 执行这个 command；
- 配置这个 command 对应的 group 和线程池。

步骤一：创建 command

一个 HystrixCommand 或 HystrixObservableCommand 对象，代表了对某个依赖服务发起的一次请求或者调用。创建的时候，可以在构造函数中传入任何需要的参数。

- HystrixCommand 主要用于仅仅会返回一个结果的调用。
- HystrixObservableCommand 主要用于可能会返回多条结果的调用。

```html
// 创建 HystrixCommand
HystrixCommand hystrixCommand = new HystrixCommand(arg1, arg2);

// 创建 HystrixObservableCommand
HystrixObservableCommand hystrixObservableCommand = new HystrixObservableCommand(arg1, arg2);
```

步骤二：调用 command 执行方法

执行 command，就可以发起一次对依赖服务的调用。

要执行 command，可以在 4 个方法中选择其中的一个：execute()、queue()、observe()、toObservable()。

其中 execute() 和 queue() 方法仅仅对 HystrixCommand 适用。

- execute()：调用后直接 block 住，属于同步调用，直到依赖服务返回单条结果，或者抛出异常。
- queue()：返回一个 Future，属于异步调用，后面可以通过 Future 获取单条结果。
- observe()：订阅一个 Observable 对象，Observable 代表的是依赖服务返回的结果，获取到一个那个代表结果的 Observable 对象的拷贝对象。
- toObservable()：返回一个 Observable 对象，如果我们订阅这个对象，就会执行 command 并且获取返回结果。

```html
K             value    = hystrixCommand.execute();
Future<K>     fValue   = hystrixCommand.queue();
Observable<K> oValue   = hystrixObservableCommand.observe();
Observable<K> toOValue = hystrixObservableCommand.toObservable();
```

execute() 实际上会调用 queue().get() 方法，而在 queue() 方法中，会调用 toObservable().toBlocking().toFuture()。

也就是说，先通过 toObservable() 获得 Future 对象，然后调用 Future 的 get() 方法。那么，其实无论是哪种方式执行 command，最终都是依赖于 toObservable() 去执行的。

步骤三：检查是否开启缓存（不太常用）

从这一步开始，就进入到 Hystrix 底层运行原理啦，看一下 Hystrix 一些更高级的功能和特性。

如果这个 command 开启了请求缓存 Request Cache，而且这个调用的结果在缓存中存在，那么直接从缓存中返回结果。否则，继续往后的步骤。

步骤四：检查是否开启了断路器

检查这个 command 对应的依赖服务是否开启了断路器。如果断路器被打开了，那么 Hystrix 就不会执行这个 command，而是直接去执行 fallback 降级机制，返回降级结果。

步骤五：检查线程池/队列/信号量是否已满

如果这个 command 线程池和队列已满，或者 semaphore 信号量已满，那么也不会执行 command，而是直接去调用 fallback 降级机制，同时发送 reject 信息给断路器统计。

步骤六：执行 command

调用 HystrixObservableCommand 对象的 construct() 方法，或者 HystrixCommand 的 run() 方法来实际执行这个 command。

- HystrixCommand.run() 返回单条结果，或者抛出异常。
- HystrixObservableCommand.construct() 返回一个 Observable 对象，可以获取多条结果。

如果是采用线程池方式，并且 HystrixCommand.run() 或者 HystrixObservableCommand.construct() 的执行时间超过了 timeout 时长的话，那么 command 所在的线程会抛出一个 TimeoutException，这时会执行 fallback 降级机制，不会去管 run() 或 construct() 返回的值了。另一种情况，如果 command 执行出错抛出了其它异常，那么也会走 fallback 降级。这两种情况下，Hystrix 都会发送异常事件给断路器统计。

**注意**，我们是不可能终止掉一个调用严重延迟的依赖服务的线程的，只能说给你抛出来一个 TimeoutException。

如果没有 timeout，也正常执行的话，那么调用线程就会拿到一些调用依赖服务获取到的结果，然后 Hystrix 也会做一些 logging 记录和 metric 度量统计。

步骤七：断路健康检查

Hystrix 会把每一个依赖服务的调用成功、失败、Reject、Timeout 等事件发送给 circuit breaker 断路器。断路器就会对这些事件的次数进行统计，根据异常事件发生的比例来决定是否要进行断路（熔断）。如果打开了断路器，那么在接下来一段时间内，会直接断路，返回降级结果。

如果在之后，断路器尝试执行 command，调用没有出错，返回了正常结果，那么 Hystrix 就会把断路器关闭。

步骤八：调用 fallback 降级机制

在以下几种情况中，Hystrix 会调用 fallback 降级机制。

- 断路器处于打开状态；
- 线程池/队列/semaphore 满了；
- command 执行超时；
- run() 或者 construct() 抛出异常。

一般在降级机制中，都建议给出一些默认的返回值，比如静态的一些代码逻辑，或者从内存中的缓存中提取一些数据，在这里尽量不要再进行网络请求了。

在降级中，如果一定要进行网络调用的话，也应该将那个调用放在一个 HystrixCommand 中进行隔离。

- HystrixCommand 中，实现 getFallback() 方法，可以提供降级机制。
- HystrixObservableCommand 中，实现 resumeWithFallback() 方法，返回一个 Observable 对象，可以提供降级结果。

如果没有实现 fallback，或者 fallback 抛出了异常，Hystrix 会返回一个 Observable，但是不会返回任何数据。

不同的 command 执行方式，其 fallback 为空或者异常时的返回结果不同。

- 对于 execute()，直接抛出异常。
- 对于 queue()，返回一个 Future，调用 get() 时抛出异常。
- 对于 observe()，返回一个 Observable 对象，但是调用 subscribe() 方法订阅它时，立即抛出调用者的 onError() 方法。
- 对于 toObservable()，返回一个 Observable 对象，但是调用 subscribe() 方法订阅它时，立即抛出调用者的 onError() 方法。

不同的执行方式

- execute()，获取一个 Future.get()，然后拿到单个结果。
- queue()，返回一个 Future。
- observe()，立即订阅 Observable，然后启动 8 大执行步骤，返回一个拷贝的 Observable，订阅时立即回调给你结果。
- toObservable()，返回一个原始的 Observable，必须手动订阅才会去执行 8 大步骤。

### 11.Hystrix 断路器执行原理

状态机

Hystrix 断路器有三种状态，分别是关闭（Closed）、打开（Open）与半开（Half-Open）。

1. `Closed` 断路器关闭：调用下游的请求正常通过
2. `Open` 断路器打开：阻断对下游服务的调用，直接走 Fallback 逻辑
3. `Half-Open` 断路器处于半开状态：SleepWindowInMilliseconds

Enabled

```html
HystrixCommandProperties.Setter()
    .withCircuitBreakerEnabled(boolean)
```

控制断路器是否工作，包括跟踪依赖服务调用的健康状况，以及对异常情况过多时是否允许触发断路。默认值 `true`。

circuitBreaker.requestVolumeThreshold

```html
HystrixCommandProperties.Setter()
    .withCircuitBreakerRequestVolumeThreshold(int)
```

表示在一次统计的**时间滑动窗口中（这个参数也很重要，下面有说到）**，至少经过多少个请求，才可能触发断路，默认值 20。**经过 Hystrix 断路器的流量只有在超过了一定阈值后，才有可能触发断路。**比如说，要求在 10s 内经过断路器的流量必须达到 20 个，而实际经过断路器的请求有 19 个，即使这 19 个请求全都失败，也不会去判断要不要断路。

circuitBreaker.errorThresholdPercentage

```html
HystrixCommandProperties.Setter()
    .withCircuitBreakerErrorThresholdPercentage(int)
```

表示异常比例达到多少，才会触发断路，默认值是 50(%)。

circuitBreaker.sleepWindowInMilliseconds

```html
HystrixCommandProperties.Setter()
    .withCircuitBreakerSleepWindowInMilliseconds(int)
```

断路器状态由 Close 转换到 Open，在之后 `SleepWindowInMilliseconds` 时间内，所有经过该断路器的请求会被断路，不调用后端服务，直接走 Fallback 降级机制，默认值 5000(ms)。

而在该参数时间过后，断路器会变为 `Half-Open` 半开闭状态，尝试让一条请求经过断路器，看能不能正常调用。如果调用成功了，那么就自动恢复，断路器转为 Close 状态。

ForceOpen

```html
HystrixCommandProperties.Setter()
    .withCircuitBreakerForceOpen(boolean)
```

如果设置为 true 的话，直接强迫打开断路器，相当于是手动断路了，手动降级，默认值是 `false`。

ForceClosed

```html
HystrixCommandProperties.Setter()
    .withCircuitBreakerForceClosed(boolean)
```

如果设置为 true，直接强迫关闭断路器，相当于手动停止断路了，手动升级，默认值是 `false`。

### 12.基于 timeout 机制为服务接口调用超时提供安全保护

TimeoutMilliseconds

在 Hystrix 中，我们可以手动设置 timeout 时长，如果一个 command 运行时间超过了设定的时长，那么就被认为是 timeout，然后 Hystrix command 标识为 timeout，同时执行 fallback 降级逻辑。

`TimeoutMilliseconds` 默认值是 1000，也就是 1000ms。

```html
HystrixCommandProperties.Setter()
    ..withExecutionTimeoutInMilliseconds(int)
```

TimeoutEnabled

这个参数用于控制是否要打开 timeout 机制，默认值是 true。

```html
HystrixCommandProperties.Setter()
    .withExecutionTimeoutEnabled(boolean)
```

### 13.**服务容错的最佳实践**

1. 电路熔断器模式(Circuit Breaker Patten)：该模式的原理类似于家里的电路熔断器，如果家里的电路发生短路，熔断器能够主动熔断电路，以避免灾难性损失。在分布式系统中应用电路熔断器模式后，当目标服务慢或者大量超时，调用方能够主动熔断，以防止服务被进一步拖垮；如果情况又好转了，电路又能自动恢复，这就是所谓的弹性容错，系统有自恢复能力。
2. 舱壁隔离模式(Bulkhead Isolation Pattern)：顾名思义，该模式像舱壁一样对资源或失败单元进行隔离，如果一个船舱破了进水，只损失一个船舱，其它船舱可以不受影响 。线程隔离(Thread Isolation)就是舱壁隔离模式的一个例子，假定一个应用程序A调用了Svc1/Svc2/Svc3三个服务，且部署A的容器一共有120个工作线程，采用线程隔离机制，可以给对Svc1/Svc2/Svc3的调用各分配40个线程，当Svc2慢了，给Svc2分配的40个线程因慢而阻塞并最终耗尽，线程隔离可以保证给Svc1/Svc3分配的80个线程可以不受影响，如果没有这种隔离机制，当Svc2慢的时候，120个工作线程会很快全部被对Svc2的调用吃光，整个应用程序会全部慢下来。
3. 限流(Rate Limiting/Load Shedder)：服务总有容量限制，没有限流机制的服务很容易在突发流量(秒杀，双十一)时被冲垮。限流通常指对服务限定并发访问量，比如单位时间只允许100个并发调用，对超过这个限制的请求要拒绝并回退。
4. 回退(fallback)：在熔断或者限流发生的时候，应用程序的后续处理逻辑是什么？回退是系统的弹性恢复能力，常见的处理策略有，直接抛出异常，也称快速失败(Fail Fast)，也可以返回空值或缺省值，还可以返回备份数据，如果主服务熔断了，可以从备份服务获取数据。

### 14.**Hystrix**的工作流程

1.创建一个 HystrixCommand 或 HystrixObservableCommand 实例来代表向其它组件发出的操作请求（指令）；

2.通过相关的方法执行操作指令（同步执行、异步执行、或响应式执行）；

3.执行操作指令时，Hystrix首先会检查缓存内是否有对应指令的结果，如果有的话，将缓存的结果直接以Observable对象的形式返回；注：从底层实现来讲，HystrixCommand也是利用Observable实现

4.如果没有对应的缓存，Hystrix会检查Circuit Breaker的状态。如果Circuit Breaker的状态为开启状态，Hystrix将不会执行对应指令，而是直接进入失败处理状态（图中8 Fallback）。如果Circuit Breaker的状态为关闭状态，Hystrix会继续进行线程池、任务队列、信号量的检查（图中5）；

5.确认是否有足够的资源执行操作指令。如果资源满，Hystrix同样将不会执行对应指令并且直接进入失败处理状态（图中8）；

6.如果资源充足，Hystrix将会执行操作指令。操作指令的调用最终都会到这两个方法：HystrixCommand.run()、HystrixObservableCommand.construct()。如果执行指令的时间超时，执行线程会抛出TimeoutException异常。Hystrix会抛弃结果并直接进入失败处理状态。如果执行指令成功，Hystrix会进行一系列的数据记录，然后返回执行的结果。

7.同时，Hystrix会根据记录的数据来计算失败比率，一旦失败比率达到某一阈值将自动开启Circuit Breaker。

### 15.**Hystrix**常用配置

**超时时间（默认1000ms，单位：ms）**

hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds

在调用方配置，被该调用方的所有方法的超时时间都是该值，优先级低于下边的指定配置。

hystrix.command.*HystrixCommandKey*.execution.isolation.thread.timeoutInMilliseconds

在调用方配置，被该调用方的指定方法（*HystrixCommandKey**方法名*）的超时时间是该值。

**线程池核心线程数**

hystrix.threadpool.default.coreSize（默认为10）

**Queue**

hystrix.threadpool.default.maxQueueSize（最大排队长度。默认-1，使用SynchronousQueue。其他正值则使用 LinkedBlockingQueue。如果要从-1换成其他值则需重启，即该值不能动态调整，若要动态调整，需要使用到下边这个配置）

hystrix.threadpool.default.queueSizeRejectionThreshold（排队线程数量阈值，默认为5，达到时拒绝，如果配置了该选项，队列的大小是该队列）

注意：如果maxQueueSize=-1的话，则该选项不起作用

**断路器**

hystrix.command.default.circuitBreaker.requestVolumeThreshold（当在配置时间窗口内达到此数量的失败后，进行短路。默认20个）

For example, if the value is 20, then if only 19 requests are received in the rolling window (say a window of **10 seconds**) the circuit will not trip open even if all 19 failed.

hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds（短路多久以后开始尝试是否恢复，默认5s）

hystrix.command.default.circuitBreaker.errorThresholdPercentage（出错百分比阈值，当达到此阈值后，开始短路。默认50%）

**Fallback**

hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests（调用线程允许请求HystrixCommand.getFallback()的最大数量，默认10。超出时将会有异常抛出，注意：该项配置对于THREAD隔离模式也起作用）

### 16.Hystrix相关注解

@EnableHystrix：开启熔断 @HystrixCommand(fallbackMethod=”XXX”)：声明一个失败回滚处理函数XXX，当被注解的方法执行 超时（默认是1000毫秒），就会执行fallback函数，返回错误提示

### 17.Hystrix底层

Hystrix使用舱壁模式实现线程池的隔离，它会为每一个依赖服务创建一个独立的线程池，这样就算某个依赖服务出现延迟过高的情 况，也只是对该依赖服务的调用产生影响，而不会拖慢其他的依赖服务。

### 18.**什么是Hystrix？**

在分布式系统，我们一定会依赖各种服务，那么这些个服务一定会出现失败的情况，就会导致雪崩，Hystrix就是这样的一个工具，防雪崩利器，它具有服务降级，服务熔断，服务隔离，监控等一些防止雪崩的技术。 Hystrix有四种防雪崩方式：

- `服务降级`：接口调用失败就调用本地的方法返回一个空
- `服务熔断`：接口调用失败就会进入调用接口提前定义好的一个熔断的方法，返回错误信息
- `服务隔离`：隔离服务之间相互影响
- `服务监控`：在服务发生调用时,会将每秒请求数、成功请求数等运行指标记录下来。

### 19.**Hystrix流程?**

1、构造一个 `HystrixCommand`或`HystrixObservableCommand`对象，用于封装请求，并在构造方法配置请求被执行需要的参数； 2、执行命令，Hystrix提供了4种执行命令的方法； 3、判断是否使用缓存响应请求，若启用了缓存，且缓存可用，直接使用缓存响应请求。Hystrix支持请求缓存，但需要用户自定义启动； 4、判断熔断器是否打开，如果打开，跳到第8步； 5、判断线程池/队列/信号量是否已满，已满则跳到第8步； 6、执行`HystrixObservableCommand.construct()`或`HystrixCommand.run()`，如果执行失败或者超时，跳到第8步；否则，跳到第9步； 7、统计熔断器监控指标； 8、走`Fallback`备用逻辑 9、返回请求响应

### 20.Hystrix的设计原则是什么？

hystrix为了实现高可用性的架构，设计hystrix的时候，一些设计原则是什么？？？

（1）对依赖服务调用时出现的调用延迟和调用失败进行控制和容错保护。

（2）在复杂的分布式系统中，阻止某一个依赖服务的故障在整个系统中蔓延，服务A->服务B->服务C，服务C故障了，服务B也故障了，服务A故障了，整套分布式系统全部故障，整体宕机。

（3）提供fail-fast（快速失败）和快速恢复的支持。

（4）提供fallback优雅降级的支持。

（5）支持近实时的监控、报警以及运维操作。

调用延迟+失败，提供容错。

阻止故障蔓延。

快速失败+快速恢复。

降级。

监控+报警+运维。

完全描述了hystrix的功能，提供整个分布式系统的高可用的架构。

Hystrix的更加细节的设计原则

（1）阻止任何一个依赖服务耗尽所有的资源，比如tomcat中的所有线程资源。

（2）避免请求排队和积压，采用限流和fail fast来控制故障。

（3）提供fallback降级机制来应对故障。

（4）使用资源隔离技术，比如bulkhead（舱壁隔离技术），swimlane（泳道技术），circuit breaker（短路技术），来限制任何一个依赖服务的故障的影响。

（5）通过近实时的统计/监控/报警功能，来提高故障发现的速度。

（6）通过近实时的属性和配置热修改功能，来提高故障处理和恢复的速度。

（7）保护依赖服务调用的所有故障情况，而不仅仅只是网络故障情况。

调用这个依赖服务的时候，client调用包有bug，阻塞，等等，依赖服务的各种各样的调用的故障，都可以处理。

Hystrix是如何实现它的目标的？

（1）通过HystrixCommand或者HystrixObservableCommand来封装对外部依赖的访问请求，这个访问请求一般会运行在独立的线程中，资源隔离。

（2）对于超出我们设定阈值的服务调用，直接进行超时，不允许其耗费过长时间阻塞住。这个超时时间默认是99.5%的访问时间，但是一般我们可以自己设置一下。

（3）为每一个依赖服务维护一个独立的线程池，或者是semaphore，当线程池已满时，直接拒绝对这个服务的调用。

（4）对依赖服务的调用的成功次数，失败次数，拒绝次数，超时次数，进行统计。

（5）如果对一个依赖服务的调用失败次数超过了一定的阈值，自动进行熔断，在一定时间内对该服务的调用直接降级，一段时间后再自动尝试恢复。

（6）当一个服务调用出现失败，被拒绝，超时，短路等异常情况时，自动调用fallback降级机制。

（7）对属性和配置的修改提供近实时的支持。

### 21.资源隔离

hystrix进行资源隔离，其实是提供了一个抽象，叫做command，就是说，你如果要把对某一个依赖服务的所有调用请求，全部隔离在同一份资源池内。

对这个依赖服务的所有调用请求，全部走这个资源池内的资源，不会去用其他的资源了，这个就叫做资源隔离。

hystrix最最基本的资源隔离的技术，线程池隔离技术。

对某一个依赖服务，商品服务，所有的调用请求，全部隔离到一个线程池内，对商品服务的每次调用请求都封装在一个command里面。

每个command（每次服务调用请求）都是使用线程池内的一个线程去执行的。

所以哪怕是对这个依赖服务，商品服务，现在同时发起的调用量已经到了1000了，但是线程池内就10个线程，最多就只会用这10个线程去执行。

不会说，对商品服务的请求，因为接口调用延迟，将tomcat内部所有的线程资源全部耗尽，不会出现了。

### 22.隔离技术

hystrix里面，核心的一项功能，其实就是所谓的资源隔离，要解决的最最核心的问题，就是将多个依赖服务的调用分别隔离到各自自己的资源池内。

避免说对某一个依赖服务的调用，因为依赖服务的接口调用的延迟或者失败，导致服务所有的线程资源全部耗费在这个服务的接口调用上。

一旦说某个服务的线程资源全部耗尽的话，可能就导致服务就会崩溃，甚至说这种故障会不断蔓延。

hystrix，资源隔离，两种技术，线程池的资源隔离，信号量的资源隔离。

信号量，semaphore。

线程池和信号量做资源隔离，限流，容量的限制，默认的容量都是10。核心的区别，线程池隔离技术，是用自己的线程去执行调用的，信号量的隔离技术，是直接让tomcat的线程去调用依赖服务的。

### 23.线程池隔离技术和信号量隔离技术，分别在什么样的场景下去使用呢？？

线程池：适合绝大多数的场景，99%的，线程池，对依赖服务的网络请求的调用和访问，timeout这种问题。

信号量：适合，你的访问不是对外部依赖的访问，而是对内部的一些比较复杂的业务逻辑的访问，但是像这种访问，系统内部的代码，其实不涉及任何的网络请求，那么只要做信号量的普通限流就可以了，因为不需要去捕获timeout类似的问题，算法+数据结构的效率不是太高，并发量突然太高，因为这里稍微耗时一些，导致很多线程卡在这里的话，不太好，所以进行一个基本的资源隔离和访问，避免内部复杂的低效率的代码，导致大量的线程被hang住。

### 24.execution.isolation.strategy

指定了HystrixCommand.run()的资源隔离策略，THREAD或者SEMAPHORE，一种是基于线程池，一种是信号量。

线程池机制，每个command运行在一个线程中，限流是通过线程池的大小来控制的。

信号量机制，command是运行在调用线程中，但是通过信号量的容量来进行限流。

如何在线程池和信号量之间做选择？

默认的策略就是线程池。

线程池其实最大的好处就是对于网络访问请求，如果有超时的话，可以避免调用线程阻塞住。

而使用信号量的场景，通常是针对超大并发量的场景下，每个服务实例每秒都几百的QPS，那么此时你用线程池的话，线程一般不会太多，可能撑不住那么高的并发，如果要撑住，可能要耗费大量的线程资源，那么就是用信号量，来进行限流保护。

一般用信号量常见于那种基于纯内存的一些业务逻辑服务，而不涉及到任何网络访问请求。

netflix有100+的command运行在40+的线程池中，只有少数command是不运行在线程池中的，就是从纯内存中获取一些元数据，或者是对多个command包装起来的facacde command，是用信号量限流的。

```html
// to use thread isolation
HystrixCommandProperties.Setter().withExecutionIsolationStrategy(ExecutionIsolationStrategy.THREAD)
// to use semaphore isolation
HystrixCommandProperties.Setter().withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)
```

### 25.command名称和command组

线程池隔离，依赖服务->接口->线程池，如何来划分。

你的每个command，都可以设置一个自己的名称，同时可以设置一个自己的组。

```html
private static final Setter cachedSetter = 
    Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
        .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld"));    
public CommandHelloWorld(String name) {
    super(cachedSetter);
    this.name = name;
}
```

command group，是一个非常重要的概念，默认情况下，因为就是通过command group来定义一个线程池的，而且还会通过command group来聚合一些监控和报警信息。

同一个command group中的请求，都会进入同一个线程池中。

### 26.command线程池

threadpool key代表了一个HystrixThreadPool，用来进行统一监控，统计，缓存。

默认的threadpool key就是command group名称。

每个command都会跟它的threadpool key对应的thread pool绑定在一起。

如果不想直接用command group，也可以手动设置thread pool name。

```html
public CommandHelloWorld(String name) {
    super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
            .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld"))
            .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("HelloWorldPool")));
    this.name = name;
}
```

command threadpool -> command group -> command key。

command key，代表了一类command，一般来说，代表了底层的依赖服务的一个接口。

command group，代表了某一个底层的依赖服务，合理，一个依赖服务可能会暴露出来多个接口，每个接口就是一个command key。

command group，在逻辑上去组织起来一堆command key的调用，统计信息，成功次数，timeout超时次数，失败次数，可以看到某一个服务整体的一些访问情况。

command group，一般来说，推荐是根据一个服务去划分出一个线程池，command key默认都是属于同一个线程池的。

比如说你以一个服务为粒度，估算出来这个服务每秒的所有接口加起来的整体QPS在100左右。

你调用那个服务的当前服务，部署了10个服务实例，每个服务实例上，其实用这个command group对应这个服务，给一个线程池，量大概在10个左右，就可以了，你对整个服务的整体的访问QPS大概在每秒100左右。

一般来说，command group是用来在逻辑上组合一堆command的。

举个例子，对于一个服务中的某个功能模块来说，希望将这个功能模块内的所有command放在一个group中，那么在监控和报警的时候可以放一起看。

command group，对应了一个服务，但是这个服务暴露出来的几个接口，访问量很不一样，差异非常之大。

你可能就希望在这个服务command group内部，包含的对应多个接口的command key，做一些细粒度的资源隔离。

对同一个服务的不同接口，都使用不同的线程池。

command key -> command group。

command key -> 自己的threadpool key。

逻辑上来说，多个command key属于一个command group，在做统计的时候，会放在一起统计。

每个command key有自己的线程池，每个接口有自己的线程池，去做资源隔离和限流。

但是对于thread pool资源隔离来说，可能是希望能够拆分的更加一致一些，比如在一个功能模块内，对不同的请求可以使用不同的thread pool。

command group一般来说，可以是对应一个服务，多个command key对应这个服务的多个接口，多个接口的调用共享同一个线程池。

如果说你的command key，要用自己的线程池，可以定义自己的threadpool key，就ok了。

### 27.coreSize

设置线程池的大小，默认是10。

HystrixThreadPoolProperties.Setter().withCoreSize(int value)

一般来说，用这个默认的10个线程大小就够了。

### 28.queueSizeRejectionThreshold

控制queue满后reject的threshold，因为maxQueueSize不允许热修改，因此提供这个参数可以热修改，控制队列的最大大小。

HystrixCommand在提交到线程池之前，其实会先进入一个队列中，这个队列满了之后，才会reject。

默认值是5。

HystrixThreadPoolProperties.Setter().withQueueSizeRejectionThreshold(int value)

### 29.execution.isolation.semaphore.maxConcurrentRequests

设置使用SEMAPHORE隔离策略的时候，允许访问的最大并发量，超过这个最大并发量，请求直接被reject。

这个并发量的设置，跟线程池大小的设置，应该是类似的，但是基于信号量的话，性能会好很多，而且hystrix框架本身的开销会小很多。

默认值是10，设置的小一些，否则因为信号量是基于调用线程去执行command的，而且不能从timeout中抽离，因此一旦设置的太大，而且有延时发生，可能瞬间导致tomcat本身的线程资源本占满。

HystrixCommandProperties.Setter().withExecutionIsolationSemaphoreMaxConcurrentRequests(int value)

### 30.hystrix执行时的8大流程以及内部原理

1、构建一个HystrixCommand或者HystrixObservableCommand

一个HystrixCommand或一个HystrixObservableCommand对象，代表了对某个依赖服务发起的一次请求或者调用。

构造的时候，可以在构造函数中传入任何需要的参数。

HystrixCommand主要用于仅仅会返回一个结果的调用。 HystrixObservableCommand主要用于可能会返回多条结果的调用。

HystrixCommand command = new HystrixCommand(arg1, arg2); HystrixObservableCommand command = new HystrixObservableCommand(arg1, arg2);

2、调用command的执行方法

执行Command就可以发起一次对依赖服务的调用。

要执行Command，需要在4个方法中选择其中的一个：execute()，queue()，observe()，toObservable()。

其中execute()和queue()仅仅对HystrixCommand适用。

execute()：调用后直接block住，属于同步调用，直到依赖服务返回单条结果，或者抛出异常。 queue()：返回一个Future，属于异步调用，后面可以通过Future获取单条结果。 observe()：订阅一个Observable对象，Observable代表的是依赖服务返回的结果，获取到一个那个代表结果的Observable对象的拷贝对象。 toObservable()：返回一个Observable对象，如果我们订阅这个对象，就会执行command并且获取返回结果。

K value = command.execute(); Future<K> fValue = command.queue(); Observable<K> ohValue = command.observe(); Observable<K> ocValue = command.toObservable();

execute()实际上会调用queue().get().queue()，接着会调用toObservable().toBlocking().toFuture()。

也就是说，无论是哪种执行command的方式，最终都是依赖toObservable()去执行的。

3、检查是否开启缓存

从这一步开始，进入我们的底层的运行原理啦，了解hysrix的一些更加高级的功能和特性。

如果这个command开启了请求缓存，request cache，而且这个调用的结果在缓存中存在，那么直接从缓存中返回结果。

4、检查是否开启了短路器

检查这个command对应的依赖服务是否开启了短路器。

如果断路器被打开了，那么hystrix就不会执行这个command，而是直接去执行fallback降级机制。

5、检查线程池/队列/semaphore是否已经满了

如果command对应的线程池/队列/semaphore已经满了，那么也不会执行command，而是直接去调用fallback降级机制。

6、执行command

调用HystrixObservableCommand.construct()或HystrixCommand.run()来实际执行这个command。

HystrixCommand.run()是返回一个单条结果，或者抛出一个异常。 HystrixObservableCommand.construct()是返回一个Observable对象，可以获取多条结果。

如果HystrixCommand.run()或HystrixObservableCommand.construct()的执行，超过了timeout时长的话，那么command所在的线程就会抛出一个TimeoutException。

如果timeout了，也会去执行fallback降级机制，而且就不会管run()或construct()返回的值了。

这里要注意的一点是，我们是不可能终止掉一个调用严重延迟的依赖服务的线程的，只能说给你抛出来一个TimeoutException，但是还是可能会因为严重延迟的调用线程占满整个线程池的。

即使这个时候新来的流量都被限流了。。。

如果没有timeout的话，那么就会拿到一些调用依赖服务获取到的结果，然后hystrix会做一些logging记录和metric统计。

7、短路健康检查

Hystrix会将每一个依赖服务的调用成功，失败，拒绝，超时，等事件，都会发送给circuit breaker断路器。

短路器就会对调用成功/失败/拒绝/超时等事件的次数进行统计。

短路器会根据这些统计次数来决定，是否要进行短路，如果打开了短路器，那么在一段时间内就会直接短路，然后如果在之后第一次检查发现调用成功了，就关闭断路器。

8、调用fallback降级机制

在以下几种情况中，hystrix会调用fallback降级机制：run()或construct()抛出一个异常，短路器打开，线程池/队列/semaphore满了，command执行超时了。

一般在降级机制中，都建议给出一些默认的返回值，比如静态的一些代码逻辑，或者从内存中的缓存中提取一些数据，尽量在这里不要再进行网络请求了。

即使在降级中，一定要进行网络调用，也应该将那个调用放在一个HystrixCommand中，进行隔离。

在HystrixCommand中，上线getFallback()方法，可以提供降级机制。

在HystirxObservableCommand中，实现一个resumeWithFallback()方法，返回一个Observable对象，可以提供降级结果。

如果fallback返回了结果，那么hystrix就会返回这个结果。

对于HystrixCommand，会返回一个Observable对象，其中会发返回对应的结果。 对于HystrixObservableCommand，会返回一个原始的Observable对象。

如果没有实现fallback，或者是fallback抛出了异常，Hystrix会返回一个Observable，但是不会返回任何数据。

不同的command执行方式，其fallback为空或者异常时的返回结果不同。

对于execute()，直接抛出异常。 对于queue()，返回一个Future，调用get()时抛出异常。 对于observe()，返回一个Observable对象，但是调用subscribe()方法订阅它时，理解抛出调用者的onError方法。 对于toObservable()，返回一个Observable对象，但是调用subscribe()方法订阅它时，理解抛出调用者的onError方法。

9、不同的执行方式

execute()，获取一个Future.get()，然后拿到单个结果。 queue()，返回一个Future。 observer()，立即订阅Observable，然后启动8大执行步骤，返回一个拷贝的Observable，订阅时理解回调给你结果。 toObservable()，返回一个原始的Observable，必须手动订阅才会去执行8大步骤。

### 31.request cache的原理

1、创建command，2种command类型。 2、执行command，4种执行方式。 3、查找是否开启了request cache，是否有请求缓存，如果有缓存，直接取用缓存，返回结果。

首先，有一个概念，叫做reqeust context，请求上下文，一般来说，在一个web应用中，hystrix。

我们会在一个filter里面，对每一个请求都施加一个请求上下文，就是说，tomcat容器内，每一次请求，就是一次请求上下文。

然后在这次请求上下文中，我们会去执行N多代码，调用N多依赖服务，有的依赖服务可能还会调用好几次。

在一次请求上下文中，如果有多个command，参数都是一样的，调用的接口也是一样的，其实结果可以认为也是一样的。

那么这个时候，我们就可以让第一次command执行，返回的结果，被缓存在内存中，然后这个请求上下文中，后续的其他对这个依赖的调用全部从内存中取用缓存结果就可以了。

不用在一次请求上下文中反复多次的执行一样的command，提升整个请求的性能。

HystrixCommand和HystrixObservableCommand都可以指定一个缓存key，然后hystrix会自动进行缓存，接着在同一个request context内，再次访问的时候，就会直接取用缓存。

用请求缓存，可以避免重复执行网络请求。

多次调用一个command，那么只会执行一次，后面都是直接取缓存。

对于请求缓存（request caching），请求合并（request collapsing），请求日志（request log），等等技术，都必须自己管理HystrixReuqestContext的声明周期。

在一个请求执行之前，都必须先初始化一个request context。

HystrixRequestContext context = HystrixRequestContext.initializeContext();

然后在请求结束之后，需要关闭request context。

context.shutdown();

一般来说，在java web来的应用中，都是通过filter过滤器来实现的。

### 32.fallback降级机制

hystrix调用各种接口，或者访问外部依赖，mysql，redis，zookeeper，kafka，等等，如果出现了任何异常的情况。

比如说报错了，访问mysql报错，redis报错，zookeeper报错，kafka报错，error。

对每个外部依赖，无论是服务接口，中间件，资源隔离，对外部依赖只能用一定量的资源去访问，线程池/信号量，如果资源池已满，reject。

访问外部依赖的时候，访问时间过长，可能就会导致超时，报一个TimeoutException异常，timeout。

上述三种情况，都是我们说的异常情况，对外部依赖的东西访问的时候出现了异常，发送异常事件到短路器中去进行统计。

如果短路器发现异常事件的占比达到了一定的比例，直接开启短路，circuit breaker。

上述四种情况，都会去调用fallback降级机制。

fallback，降级机制，你之前都是必须去调用外部的依赖接口，或者从mysql中去查询数据的，但是为了避免说可能外部依赖会有故障。

比如，你可以在内存中维护一个ehcache，作为一个纯内存的基于LRU自动清理的缓存，数据也可以放入缓存内。

如果说外部依赖有异常，fallback这里，直接尝试从ehcache中获取数据。

比如说，本来你是从mysql，redis，或者其他任何地方去获取数据的，获取调用其他服务的接口的，结果人家故障了，人家挂了，fallback，可以返回一个默认值。

两种最经典的降级机制：纯内存数据，默认值。

run()抛出异常，超时，线程池或信号量满了，或短路了，都会调用fallback机制。

给大家举个例子，比如说我们现在有个商品数据，brandId，品牌，一般来说，假设，正常的逻辑，拿到了一个商品数据以后，用brandId再调用一次请求，到其他的服务去获取品牌的最新名称。

假如说，那个品牌服务挂掉了，那么我们可以尝试本地内存中，会保留一份时间比较过期的一份品牌数据，有些品牌没有，有些品牌的名称过期了，Nike++，Nike。

调用品牌服务失败了，fallback降级就从本地内存中获取一份过期的数据，先凑合着用着。

### 33.fallback.isolation.semaphore.maxConcurrentRequests

这个参数设置了HystrixCommand.getFallback()最大允许的并发请求数量，默认值是10，也是通过semaphore信号量的机制去限流。

如果超出了这个最大值，那么直接被reject。

HystrixCommandProperties.Setter().withFallbackIsolationSemaphoreMaxConcurrentRequests(int value)

### 34.短路器深入的工作原理

1、如果经过短路器的流量超过了一定的阈值，HystrixCommandProperties.circuitBreakerRequestVolumeThreshold()

举个例子，可能看起来是这样子的，要求在10s内，经过短路器的流量必须达到20个；在10s内，经过短路器的流量才10个，那么根本不会去判断要不要短路。

2、如果断路器统计到的异常调用的占比超过了一定的阈值，HystrixCommandProperties.circuitBreakerErrorThresholdPercentage()

如果达到了上面的要求，比如说在10s内，经过短路器的流量（你，只要执行一个command，这个请求就一定会经过短路器），达到了30个；同时其中异常的访问数量，占到了一定的比例，比如说60%的请求都是异常（报错，timeout，reject），会开启短路。

3、然后断路器从close状态转换到open状态。

4、断路器打开的时候，所有经过该断路器的请求全部被短路，不调用后端服务，直接走fallback降级。

5、经过了一段时间之后，HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds()，会half-open，让一条请求经过短路器，看能不能正常调用。如果调用成功了，那么就自动恢复，转到close状态。

短路器，会自动恢复的，half-open，半开状态。

6、circuit breaker短路器的配置

（1）circuitBreaker.enabled

控制短路器是否允许工作，包括跟踪依赖服务调用的健康状况，以及对异常情况过多时是否允许触发短路，默认是true

HystrixCommandProperties.Setter().withCircuitBreakerEnabled(boolean value)

（2）circuitBreaker.requestVolumeThreshold

设置一个rolling window，滑动窗口中，最少要有多少个请求时，才触发开启短路。

举例来说，如果设置为20（默认值），那么在一个10秒的滑动窗口内，如果只有19个请求，即使这19个请求都是异常的，也是不会触发开启短路器的。

HystrixCommandProperties.Setter().withCircuitBreakerRequestVolumeThreshold(int value)

（3）circuitBreaker.sleepWindowInMilliseconds

设置在短路之后，需要在多长时间内直接reject请求，然后在这段时间之后，再重新导holf-open状态，尝试允许请求通过以及自动恢复，默认值是5000毫秒。

HystrixCommandProperties.Setter().withCircuitBreakerSleepWindowInMilliseconds(int value)

（4）circuitBreaker.errorThresholdPercentage

设置异常请求量的百分比，当异常请求达到这个百分比时，就触发打开短路器，默认是50，也就是50%。

HystrixCommandProperties.Setter().withCircuitBreakerErrorThresholdPercentage(int value)

（5）circuitBreaker.forceOpen

如果设置为true的话，直接强迫打开短路器，相当于是手动短路了，手动降级，默认false。

HystrixCommandProperties.Setter().withCircuitBreakerForceOpen(boolean value)

（6）circuitBreaker.forceClosed

如果设置为ture的话，直接强迫关闭短路器，相当于是手动停止短路了，手动升级，默认false。

HystrixCommandProperties.Setter().withCircuitBreakerForceClosed(boolean value)

### 35.线程池隔离技术的设计原则

1、command的创建和执行：资源隔离。 2、request cache：请求缓存。 3、fallback：优雅降级。 4、circuit breaker：短路器，快速熔断（一旦后端服务故障，立刻熔断，阻止对其的访问）。

把一个分布式系统中的某一个服务，打造成一个高可用的服务。

资源隔离，优雅降级，熔断。

5、判断，线程池或者信号量的容量是否已满，reject，限流。

限流，限制对后端的服务的访问量，比如说你对mysql，redis，zookeeper，各种后端的中间件的资源，访问，其实为了避免过大的流浪打死后端的服务，线程池，信号量，限流。

限制服务对后端的资源的访问。

### 36、线程池隔离技术的设计原则

Hystrix采取了bulkhead舱壁隔离技术，来将外部依赖进行资源隔离，进而避免任何外部依赖的故障导致本服务崩溃。

线程池隔离，学术名称：bulkhead，舱壁隔离。

外部依赖的调用在单独的线程中执行，这样就能跟调用线程隔离开来，避免外部依赖调用timeout耗时过长，导致调用线程被卡死。

Hystrix对每个外部依赖用一个单独的线程池，这样的话，如果对那个外部依赖调用延迟很严重，最多就是耗尽那个依赖自己的线程池而已，不会影响其他的依赖调用。

Hystrix选择用线程池机制来进行资源隔离，要面对的场景如下：

（1）每个服务都会调用几十个后端依赖服务，那些后端依赖服务通常是由很多不同的团队开发的。 （2）每个后端依赖服务都会提供它自己的client调用库，比如说用thrift的话，就会提供对应的thrift依赖。 （3）client调用库随时会变更。 （4）client调用库随时可能会增加新的网络请求的逻辑。 （5）client调用库可能会包含诸如自动重试，数据解析，内存中缓存等逻辑。 （6）client调用库一般都对调用者来说是个黑盒，包括实现细节，网络访问，默认配置，等等。 （7）在真实的生产环境中，经常会出现调用者，突然间惊讶的发现，client调用库发生了某些变化。 （8）即使client调用库没有改变，依赖服务本身可能有会发生逻辑上的变化。 （9）有些依赖的client调用库可能还会拉取其他的依赖库，而且可能那些依赖库配置的不正确。 （10）大多数网络请求都是同步调用的。 （11）调用失败和延迟，也有可能会发生在client调用库本身的代码中，不一定就是发生在网络请求中。

简单来说，就是你必须默认client调用库就很不靠谱，而且随时可能各种变化，所以就要用强制隔离的方式来确保任何服务的故障不能影响当前服务。

线程池机制的优点如下：

（1）任何一个依赖服务都可以被隔离在自己的线程池内，即使自己的线程池资源填满了，也不会影响任何其他的服务调用。 （2）服务可以随时引入一个新的依赖服务，因为即使这个新的依赖服务有问题，也不会影响其他任何服务的调用。 （3）当一个故障的依赖服务重新变好的时候，可以通过清理掉线程池，瞬间恢复该服务的调用，而如果是tomcat线程池被占满，再恢复就很麻烦。 （4）如果一个client调用库配置有问题，线程池的健康状况随时会报告，比如成功/失败/拒绝/超时的次数统计，然后可以近实时热修改依赖服务的调用配置，而不用停机。 （5）如果一个服务本身发生了修改，需要重新调整配置，此时线程池的健康状况也可以随时发现，比如成功/失败/拒绝/超时的次数统计，然后可以近实时热修改依赖服务的调用配置，而不用停机。 （6）基于线程池的异步本质，可以在同步的调用之上，构建一层异步调用层。

简单来说，最大的好处，就是资源隔离，确保说，任何一个依赖服务故障，不会拖垮当前的这个服务。

线程池机制的缺点：

（1）线程池机制最大的缺点就是增加了cpu的开销。

除了tomcat本身的调用线程之外，还有hystrix自己管理的线程池。

（2）每个command的执行都依托一个独立的线程，会进行排队，调度，还有上下文切换。 （3）Hystrix官方自己做了一个多线程异步带来的额外开销，通过对比多线程异步调用+同步调用得出，Netflix API每天通过hystrix执行10亿次调用，每个服务实例有40个以上的线程池，每个线程池有10个左右的线程。 （4）最后发现说，用hystrix的额外开销，就是给请求带来了3ms左右的延时，最多延时在10ms以内，相比于可用性和稳定性的提升，这是可以接受的。

我们可以用hystrix semaphore技术来实现对某个依赖服务的并发访问量的限制，而不是通过线程池/队列的大小来限制流量。

sempahore技术可以用来限流和削峰，但是不能用来对调研延迟的服务进行timeout和隔离。

execution.isolation.strategy，设置为SEMAPHORE，那么hystrix就会用semaphore机制来替代线程池机制，来对依赖服务的访问进行限流。

如果通过semaphore调用的时候，底层的网络调用延迟很严重，那么是无法timeout的，只能一直block住。

一旦请求数量超过了semephore限定的数量之后，就会立即开启限流。

### 37.timeout机制

一般来说，在调用依赖服务的接口的时候，比较常见的一个问题，就是超时。

超时是在一个复杂的分布式系统中，导致不稳定，或者系统抖动，或者出现说大量超时，线程资源hang死，吞吐量大幅度下降，甚至服务崩溃。

如果你不对各种依赖接口的调用，做超时的控制，来给你的服务提供安全保护措施，那么很可能你的服务就被各种垃圾的依赖服务的性能给拖死了。

大量的接口调用很慢，大量线程就卡死了，资源隔离，线程池的线程卡死了，超时的控制。

（1）execution.isolation.thread.timeoutInMilliseconds

手动设置timeout时长，一个command运行超出这个时间，就被认为是timeout，然后将hystrix command标识为timeout，同时执行fallback降级逻辑。

默认是1000，也就是1000毫秒。

HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(int value)

（2）execution.timeout.enabled

控制是否要打开timeout机制，默认是true。

HystrixCommandProperties.Setter().withExecutionTimeoutEnabled(boolean value)

让一个command执行timeout，然后看是否会调用fallback降级。

### 38.hystrix的核心知识

1、hystrix内部工作原理：8大执行步骤和流程。 2、资源隔离：你如果有很多个依赖服务，高可用性，先做资源隔离，任何一个依赖服务的故障不会导致你的服务的资源耗尽，不会崩溃。 3、请求缓存：对于一个request context内的多个相同command，使用request cache，提升性能。 4、熔断：基于短路器，采集各种异常事件，报错，超时，reject，短路，熔断，一定时间范围内就不允许访问了，直接降级，自动恢复的机制。 5、降级：报错，超时，reject，熔断，降级，服务提供容错的机制。 6、限流：在你的服务里面，通过线程池，或者信号量，限制对某个后端的服务或资源的访问量，避免从你的服务这里过去太多的流量，打死某个资源。 7、超时：避免某个依赖服务性能过差，导致大量的线程hang住去调用那个服务，会导致你的服务本身性能也比较差。

### 39.hystrix的高阶知识

1、request collapser，请求合并技术。 2、fail-fast和fail-slient，高阶容错模式。 3、static fallback和stubbed fallback，高阶降级模式。 4、嵌套command实现的发送网络请求的降级模式。 5、基于facade command的多级降级模式。 6、request cache的手动清理。 7、生产环境中的线程池大小以及timeout配置优化经验。 8、线程池的自动化动态扩容与缩容技术。 9、hystrix的metric高阶配置。 10、基于hystrix dashboard的可视化分布式系统监控。 11、生产环境中的hystrix工程运维经验。

### 40.request collapser

hystrix，高级的技术，request collapser，请求合并技术，collapser折叠。

优化过一个批量查询的接口了，request cache来做优化，可能有相同的商品就可以直接取用缓存了。

多个商品，需要发送多次网络请求，调用多次接口，才能拿到结果。

可以使用HystrixCollapser将多个HystrixCommand合并到一起，多个command放在一个command里面去执行，发送一次网络请求，就拉取到多条数据。

用请求合并技术，将多个请求合并起来，可以减少高并发访问下需要使用的线程数量以及网络连接数量，这都是hystrix自动进行的。

其实对于高并发的访问来说，是可以提升性能的。

请求合并有很多种级别。

（1）global context，tomcat所有调用线程，对一个依赖服务的任何一个command调用都可以被合并在一起，hystrix就传递一个HystrixRequestContext。

（2）user request context，tomcat内某一个调用线程，将某一个tomcat线程对某个依赖服务的多个command调用合并在一起。

（3）object modeling，基于对象的请求合并，如果有几百个对象，遍历后依次调用每个对象的某个方法，可能导致发起几百次网络请求，基于hystrix可以自动将对多个对象模型的调用合并到一起。

请求合并技术的开销有多大。

使用请求合并技术的开销就是导致延迟大幅度增加，因为需要一定的时间将多个请求合并起来。

发送过来10个请求，每个请求本来大概是2ms可以返回，要把10个请求合并在一个command内，统一一起执行，先后等待一下，5ms。

所以说，要考量一下，使用请求合并技术是否合适，如果一个请求本来耗费的时间就比较长，那么进行请求合并，增加一些延迟影响并不大。

请求合并技术，不是针对那种访问延时特别低的请求的，比如说你的访问延时本身就比较高，20ms，10个请求合并在一起，25ms，这种情况下就还好。

好处在哪里，大幅度削减你的线程池的资源耗费，线程池，10个线程，一秒钟可以执行10个请求，合并在一起，1个线程执行10个请求，10个线程就可以执行100个请求。

增加你的吞吐量。

减少你对后端服务访问时的网络资源的开销，10个请求，10个command，10次网络请求的开销，1次网络请求的开销了。

每个请求就2ms，batch，8 ~ 10ms，延迟增加了4~5倍。

每个请求本来就30ms ~ 50ms，batch，35ms~55ms，延迟增加不太明显。

将多个command请求合并到一个command中执行。

请求合并时，可以设置一个batch size，以及elapsed time（控制什么时候触发合并后的command执行）。

有两种合并模式，一种是request scope，另一种是global scope，默认是rquest scope，在collapser构造的时候指定scope模式。

request scope的batch收集是建立在一个request context内的，而global scope的batch收集是横跨多个request context的。

所以对于global context来说，必须确保能在一个command内处理多个requeset context的请求。

在netflix，是只用request scope请求合并的，因为默认是用唯一一个request context包含所有的command，所以要做合并，肯定就是request scope。

一般请求合并技术，对于那种访问同一个资源的command，但是参数不同，是很有效的。

批量查询，HystrixObservableCommand，HystrixCommand+request cache，都是每个商品发起一次网络请求。

一个批量的商品过来以后，我们还是多个command的方式去执行，request collapser+request cache，相同的商品还是就查询一次，不同的商品合并到一起通过一个网络请求得到结果。

timeout问题解释：开发机上，特别慢，第一次请求的时候，几百毫秒，默认的timeout时长比较短。

第二次的时候，访问的速度会快很多，就不会超时了。

反应在系统上，第一次启动的时候，会有个别的超时，但是后面就好了，手动将timeout时长设置的大一些。

（1）maxRequestsInBatch

控制一个Batch中最多允许多少个request被合并，然后才会触发一个batch的执行。

默认值是无限大，就是不依靠这个数量来触发执行，而是依靠时间。

HystrixCollapserProperties.Setter().withMaxRequestsInBatch(int value)

（2）timerDelayInMilliseconds

控制一个batch创建之后，多长时间以后就自动触发batch的执行，默认是10毫秒。

HystrixCollapserProperties.Setter().withTimerDelayInMilliseconds(int value)

super(Setter.withCollapserKey(HystrixCollapserKey.Factory.asKey("GetProductInfosCollapser")).andCollapserPropertiesDefaults(HystrixCollapserProperties.Setter().withMaxRequestsInBatch(100).withTimerDelayInMilliseconds(20)));

### 41.fail-fast

fail-fast，就是不给fallback降级逻辑，HystrixCommand.run()，直接报错，直接会把这个报错抛出来，给你的tomcat调用线程。

fail-silent，给一个fallback降级逻辑，如果HystrixCommand.run()，报错了，会走fallback降级，直接返回一个空值，HystrixCommand，就给一个null。

HystrixObservableCommand，Observable.empty()

很少会用fail-fast模式，比较常用的可能还是fail-silent，特别常用，既然都到了fallback里面，肯定要做点降级的事情。

### 42.stubbed fallback，残缺的降级

用请求中的部分数据拼装成结果，然后再填充一些默认值，返回。

比如说你发起了一个请求，然后请求中可能本身就附带了一些信息，如果主请求失败了，走到降级逻辑。

在降级逻辑里面，可以将这个请求中的数据，以及部分本地缓存有的数据拼装在一起，再给数据填充一些简单的默认值。

然后尽可能将自己有的数据返回到请求方。

stubbed，残缺了，比如说应该查询到一个商品信息，里面包含20个字段。

请求参数搂出来一两个字段，从本地的少量缓存中比如说，可以搂出来那么两三个字段，最终的话返回的字段可能就五六个，其他的字段都是填充的默认值。

### 43.双层嵌套command

多级降级。

先降一级，尝试用一个备用方案去执行，如果备用方案失败了，再用最后下一个备用方案去执行。

command嵌套command。

尝试从备用服务器接口去拉取结果。

给大家科普一下，常见的多级降级的做法，有一个操作，要访问MySQL数据库。

mysql数据库访问报错，降级，去redis中获取数据。

如果说redis又挂了，然后就去从本地ehcache缓存中获取数据。

hystrix command fallback语义，很容易就可以实现多级降级的策略。

商品服务接口，多级降级的策略。

command，fallback，又套了一个command，第二个command其实是第一级降级策略。

第二个command的fallback是第二级降级策略。

第一级降级策略，可以是。

storm，我们之前做storm这块，第一级降级，一般是搞一个storm的备用机房，部署了一套一模一样的拓扑，如果主机房中的storm拓扑挂掉了，备用机房的storm拓扑定顶上。

如果备用机房的storm拓扑也挂了。

第二级降级，可能就降级成用mysql/hbase/redis/es，手工封装的一套，按分钟粒度去统计数据的系统。

第三季降级，离线批处理去做，hdfs+spark，每个小时执行一次数据统计，去降级。

特别复杂，重要的系统，肯定是要搞好几套备用方案的，一个方案死了，立即上第二个方案，而且要尽量做到是自动化的。

商品接口拉取。

主流程，访问的商品服务，是从主机房去访问的，服务，如果主机房的服务出现了故障，机房断电，机房的网络负载过高，机器硬件出了故障。

第一级降级策略，去访问备用机房的服务。

第二级降级策略，用stubbed fallback降级策略，比较常用的，返回一些残缺的数据回去。

### 44.facade command

手动降级

你写一个command，在这个command它的主流程中，根据一个标识位，判断要执行哪个流程。

可以执行主流程，command，也可以执行一个备用降级的command。

一般来说，都是去执行一个主流程的command，如果说你现在知道有问题了，希望能够手动降级的话，动态给服务发送个请求。

在请求中修改标识位，自动就让command以后都直接过来执行备用command。

3个command，套在最外面的command，是用semaphore信号量做限流和资源隔离的，因为这个command不用去care timeout的问题，嵌套调用的command会自己去管理timeout超时的。

商品服务接口的手动降级的方案。

主流程还是去走GetProductInfoCommand，手动降级的方案，比如说是从某一个数据源，自己去简单的获取一些数据，尝试封装一下返回。

手动降级的策略，就比较low了，调用别人的接口去获取数据的，业务逻辑的封装。

主流程有问题，那么可能你就需要立即自己写一些逻辑发布上去，从mysql数据库的表中获取一些数据去返回，手动调整一下降级标识，做一下手动降级。

### 45.线程池的大小怎么设置，timeout时长怎么设置

在生产环境中部署一个短路器，一开始需要将一些关键配置设置的大一些，比如timeout超时时长，线程池大小，或信号量容量。

然后逐渐优化这些配置，直到在一个生产系统中运作良好。

（1）一开始先不要设置timeout超时时长，默认就是1000ms，也就是1s。 （2）一开始也不要设置线程池大小，默认就是10。 （3）直接部署hystrix到生产环境，如果运行的很良好，那么就让它这样运行好了。 （4）让hystrix应用，24小时运行在生产环境中。 （5）依赖标准的监控和报警机制来捕获到系统的异常运行情况。 （6）在24小时之后，看一下调用延迟的占比，以及流量，来计算出让短路器生效的最小的配置数字。 （7）直接对hystrix配置进行热修改，然后继续在hystrix dashboard上监控。 （8）看看修改配置后的系统表现有没有改善。

假设对一个依赖服务的高峰调用QPS是每秒30次。

一开始如果默认的线程池大小是10。

我们想的是，理想情况下，每秒的高峰访问次数 * 99%的访问延时 + buffer = 30 * 0.2 + 4 = 10线程，10个线程每秒处理30次访问应该足够了，每个线程处理3次访问。

此时，我们合理的timeout设置应该为300ms，也就是99.5%的访问延时，计算方法是，因为判断每次访问延时最多在250ms（TP99如果是200ms的话），再加一次重试时间50ms，就是300ms，感觉也应该足够了。

因为如果timeout设置的太多了，比如400ms，比如如果实际上，在高峰期，还有网络情况较差的时候，可能每次调用要耗费350ms，也就是达到了最长的访问时长。

那么每个线程处理2个请求，就会执行700ms，然后处理第三个请求的时候，就超过1秒钟了，此时会导致线程池全部被占满，都在处理请求。

这个时候下一秒的30个请求再进来了，那么就会导致线程池已满，拒绝请求的情况，就会调用fallback降级机制。

因此对于短路器来说，timeout超时一般应该设置成TP99.5，比如设置成300ms，那么可以确保说，10个线程，每个线程处理3个访问，每个访问最多就允许执行300ms，过时就timeout了。

这样才能保证说每个线程都在1s内执行完，才不会导致线程池被占满，然后后续的请求过来大量的reject。

对于线程池大小来说，一般应该控制在10个左右，20个以内，最少5个，不要太多，也不要太少。

大家可能会想，每秒的高峰访问次数是30次，如果是300次，甚至是3000次，30000次呢？？？

30000 * 0.2 = 6000 + buffer = 6100，一个服务器内一个线程池给6000个线程把。

如果你一个依赖服务占据的线程数量太多的话，会导致其他的依赖服务对应的线程池里没有资源可以用了。

6000 / 20 = 300台虚拟机也是ok的。

虚拟机，4个cpu core，4G内存，虚拟机，300台。

物理机，十几个cpu core，几十个G的内存，5~8个虚拟机，300个虚拟机 = 50台物理机。

### 46.线程池的自动化动态扩容与缩容技术

可能会出现一种情况，比如说我们的某个依赖，在高峰期，需要耗费100个线程，但是在那个时间段，刚好其他的依赖的线程池其实就维持一两个就可以了。

但是，如果我们都是设置死的，每个服务就给10个线程，那就很坑，可能就导致有的服务在高峰期需要更多的资源，但是没资源了，导致很多的reject。

但是其他的服务，每秒钟就一两个请求，结果也占用了10个线程，占着茅坑不拉屎。

做成弹性的线程资源调度的模式。

刚开始的时候，每个依赖服务都是给1个线程，3个线程，但是我们允许说，如果你的某个线程池突然需要大量的线程，最多可以到100个线程。

如果你使用了100个线程，高峰期过去了，自动将空闲的线程给释放掉。

（1）coreSize

设置线程池的大小，默认是10

HystrixThreadPoolProperties.Setter().withCoreSize(int value)

（2）maximumSize

设置线程池的最大大小，只有在设置allowMaximumSizeToDivergeFromCoreSize的时候才能生效。

默认是10

HystrixThreadPoolProperties.Setter().withMaximumSize(int value)

（5）keepAliveTimeMinutes

设置保持存活的时间，单位是分钟，默认是1。

如果设置allowMaximumSizeToDivergeFromCoreSize为true，那么coreSize就不等于maxSize，此时线程池大小是可以动态调整的，可以获取新的线程，也可以释放一些线程。

如果coreSize < maxSize，那么这个参数就设置了一个线程多长时间空闲之后，就会被释放掉

HystrixThreadPoolProperties.Setter().withKeepAliveTimeMinutes(int value)

（6）allowMaximumSizeToDivergeFromCoreSize

允许线程池大小自动动态调整，设置为true之后，maxSize就生效了，此时如果一开始是coreSize个线程，随着并发量上来，那么就会自动获取新的线程，但是如果线程在keepAliveTimeMinutes内空闲，就会被自动释放掉。

默认是fales。

HystrixThreadPoolProperties.Setter().withAllowMaximumSizeToDivergeFromCoreSize(boolean value)

生产环境中，这块怎么玩儿的。

也是根据你的服务的实际的运行的情况切看的，比如说你发现某个服务，平时3个并发QPS就够了，高峰期可能要到30个。

那么你就可以给设置弹性的资源调度。

因为你可能一个服务会有多个线程池，你要计算好，每个线程池的最大的大小加起来不能过大，30个依赖，30个线程池，每个线程池最大给到30,900个线程，很坑的。

还有一种模式，就是说让多个依赖服务共享一个线程池，我们不推荐，多个依赖服务就做不到资源隔离，互相之间会影响的。

### 47.为什么需要监控与报警？

HystrixCommand执行的时候，会生成一些执行耗时等方面的统计信息。这些信息对于系统的运维来说，是很有帮助的，因为我们通过这些统计信息可以看到整个系统是怎么运行的。hystrix对每个command key都会提供一份metric，而且是秒级统计粒度的。

这些统计信息，无论是单独看，还是聚合起来看，都是很有用的。如果将一个请求中的多个command的统计信息拿出来单独查看，包括耗时的统计，对debug系统是很有帮助的。聚合起来的metric对于系统层面的行为来说，是很有帮助的，很适合做报警或者报表。hystrix dashboard就很适合。

### 48.hystrix的事件类型

对于hystrix command来说，只会返回一个值，execute只有一个event type，fallback也只有一个event type，那么返回一个SUCCESS就代表着命令执行的结束。

对于hystrix observable command来说，多个值可能被返回，所以emit event代表一个value被返回，success代表成功，failure代表异常。

（1）execute event type

EMIT observable command返回一个value SUCCESS 完成执行，并且没有报错 FAILURE 执行时抛出了一个异常，会触发fallback TIMEOUT 开始执行了，但是在指定时间内没有完成执行，会触发fallback BAD_REQUEST 执行的时候抛出了一个HystrixBadRequestException SHORT_CIRCUITED 短路器打开了，触发fallback THREAD_POOL_REJECTED 线程成的容量满了，被reject，触发fallback SEMAPHORE_REJECTED 信号量的容量满了，被reject，触发fallback

（2）fallback event type

FALLBACK_EMIT observable command，fallback value被返回了 FALLBACK_SUCCESS fallback逻辑执行没有报错 FALLBACK_FAILURE fallback逻辑抛出了异常，会报错 FALLBACK_REJECTION fallback的信号量容量满了，fallback不执行，报错 FALLBACK_MISSING fallback没有实现，会报错

（3）其他的event type

EXCEPTION_THROWN command生命自周期是否抛出了异常 RESPONSE_FROM_CACHE command是否在cache中查找到了结果 COLLAPSED command是否是一个合并batch中的一个

（4）thread pool event type

EXECUTED 线程池有空间，允许command去执行了 REJECTED 线程池没有空间，不允许command执行，reject掉了

（5）collapser event type

BATCH_EXECUTED collapser合并了一个batch，并且执行了其中的command ADDED_TO_BATCH command加入了一个collapser batch RESPONSE_FROM_CACHE 没有加入batch，而是直接取了request cache中的数据

### 49.metric storage

metric被生成之后，就会按照一段时间来存储，存储了一段时间的数据才会推送到其他系统中，比如hystrix dashboard。

另外一种方式，就是每次生成metric就实时推送metric流到其他地方，但是这样的话，会给系统带来很大的压力。

hystrix的方式是将metric写入一个内存中的数据结构中，在一段时间之后就可以查询到。

hystrix 1.5x之后，采取的是为每个command key都生成一个start event和completion event流，而且可以订阅这个流。每个thread pool key也是一样的，包括每个collapser key也是一样的。

每个command的event是发送给一个线程安全的RxJava中的rx.Subject，因为是线程安全的，所以不需要进行线程同步。

因此每个command级别的，threadpool级别的，每个collapser级别的，event都会发送到对应的RxJava的rx.Subject对象中。这些rx.Subject对象接着就会被暴露出Observable接口，可以被订阅。

### 50.metric统计相关的配置

（1）metrics.rollingStats.timeInMilliseconds

设置统计的rolling window，单位是毫秒，hystrix只会维持这段时间内的metric供短路器统计使用。

这个属性是不允许热修改的，默认值是10000，就是10秒钟。

HystrixCommandProperties.Setter().withMetricsRollingStatisticalWindowInMilliseconds(int value)

（2）metrics.rollingStats.numBuckets

该属性设置每个滑动窗口被拆分成多少个bucket，而且滑动窗口对这个参数必须可以整除，同样不允许热修改。

默认值是10，也就是说，每秒钟是一个bucket。

随着时间的滚动，比如又过了一秒钟，那么最久的一秒钟的bucket就会被丢弃，然后新的一秒的bucket会被创建。

HystrixCommandProperties.Setter().withMetricsRollingStatisticalWindowBuckets(int value)

（3）metrics.rollingPercentile.enabled

控制是否追踪请求耗时，以及通过百分比方式来统计，默认是true。

HystrixCommandProperties.Setter().withMetricsRollingPercentileEnabled(boolean value)

（4）metrics.rollingPercentile.timeInMilliseconds

设置rolling window被持久化保存的时间，这样才能计算一些请求耗时的百分比，默认是60000，60s，不允许热修改。

相当于是一个大的rolling window，专门用于计算请求执行耗时的百分比。

HystrixCommandProperties.Setter().withMetricsRollingPercentileWindowInMilliseconds(int value)

（5）metrics.rollingPercentile.numBuckets

设置rolling percentile window被拆分成的bucket数量，上面那个参数除以这个参数必须能够整除，不允许热修改。

默认值是6，也就是每10s被拆分成一个bucket。

HystrixCommandProperties.Setter().withMetricsRollingPercentileWindowBuckets(int value)

（6）metrics.rollingPercentile.bucketSize

设置每个bucket的请求执行次数被保存的最大数量，如果再一个bucket内，执行次数超过了这个值，那么就会重新覆盖从bucket的开始再写。

举例来说，如果bucket size设置为100，而且每个bucket代表一个10秒钟的窗口，但是在这个bucket内发生了500次请求执行，那么这个bucket内仅仅会保留100次执行。

如果调大这个参数，就会提升需要耗费的内存，来存储相关的统计值，不允许热修改。

默认值是100。

HystrixCommandProperties.Setter().withMetricsRollingPercentileBucketSize(int value)

（7）metrics.healthSnapshot.intervalInMilliseconds

控制成功和失败的百分比计算，与影响短路器之间的等待时间，默认值是500毫秒。

HystrixCommandProperties.Setter().withMetricsHealthSnapshotIntervalInMilliseconds(int value)

### 51.生产环境中的hystrix分布式系统的工程运维经验总结

如果发现了严重的依赖调用延时，先不用急着去修改配置，如果一个command被限流了，可能本来就应该限流。

在netflix早期的时候，经常会有人在发现短路器因为访问延时发生的时候，去热修改一些皮遏制，比如线程池大小，队列大小，超时时长，等等，给更多的资源，但是这其实是不对的。

如果我们之前对系统进行了良好的配置，然后现在在高峰期，系统在进行线程池reject，超时，短路，那么此时我们应该集中精力去看底层根本的原因，而不是调整配置。

为什么在高峰期，一个10个线程的线程池，搞不定这些流量呢？？？代码写的太烂了，异步，更好的算法。

千万不要急于给你的依赖调用过多的资源，比如线程池大小，队列大小，超时时长，信号量容量，等等，因为这可能导致我们自己对自己的系统进行DDOS攻击。

疯狂的大量的访问你的机器，最后给打垮。

举例来说，想象一下，我们现在有100台服务器组成的集群，每台机器有10个线程大小的线程池去访问一个服务，那么我们对那个服务就有1000个线程资源去访问了。

在正常情况下，可能只会用到其中200~300个线程去访问那个后端服务。

但是如果再高峰期出现了访问延时，可能导致1000个线程全部被调用去访问那个后端服务，如果我们调整到每台服务器20个线程呢？

如果因为你的代码等问题导致访问延时，即使有20个线程可能还是会导致线程池资源被占满，此时就有2000个线程去访问后端服务，可能对后端服务就是一场灾难。

这就是断路器的作用了，如果我们把后端服务打死了，或者产生了大量的压力，有大量的timeout和reject，那么就自动短路，一段时间后，等流量洪峰过去了，再重启访问。

简单来说，让系统自己去限流，短路，超时，以及reject，直到系统重新变得正常了。

就是不要随便乱改资源配置，不要随便乱增加线程池大小，等待队列大小，异常情况是正常的。

## **Spring Cloud Zuul**

### 1.Zuul的概念和特点

Zuul是Netflix开源的微服务网关，和Eureka、Ribbon、Hystrix等组件配合使用完成服务网关的动态路由、负载均衡等功能。其核心特点如下。

( 1 ）资源审查：对每个请求都进行资源验证审查，拒绝非法请求。

( 2 ）身份认证：到每个请求的用户都进行身份认证，拒绝非法用户，身份认证一般基于HTTP消息头完成。

( 3 ）资源监控：通过对有意义的数据进行追踪和请求统计，为分析生产环境中接口的调用状态和用户的行为提供依据。

( 4 ）动态路由：对外提供统一的网关服务，动态地将不同类型的请求路由到不同的后端集群，实现对外提供统一的网关服务和对内进行有效的服务拆分。

( 5 ）压力测试：通过配置设置不同集群的负载流量，预估集群的性能。

( 6 ）负载均衡：为每一种负载类型都分配对应的容量，针对不同的请求做更细粒度的负载均衡，并弃用超出限定值的请求，进行服务保护。

( 7 ）多区域弹性：跨越区域进行请求路由，旨在实现ELB( Elastic Load Balance，弹性负载均衡）使用的多样化，并保证边缘位置与使用者尽可能接近。

### 2.Zuul的原理

Zuul的核心是通过系列Filter将整个HTTP请求过程连成一系列操作来实现对HTTP请求的控制。Zuul提供了一个对Filter进行动态地加载、编译和运行的框架。Zuul的各个Filter之间不进行直接通信，而是通过一个RequestContext静态类来进行数据的传递，每个Web请求所需要传递的参数都通过ThreadLocal变量来记录。

Zuul Filter有filterType（）、filterOrder（）、shouldFilter（）、run()4种核心方法。

- filterType（）：用以表示路由过程中的阶段（内置包含PRE、ROUTING、POST和ERROR
- filterOrder（）：表示相同Type的Filter的执行顺序
- shouldFilter（）：表示Filter的执行条件
- run() ：表示Filter具体要执行的业务逻辑

Zuul定义了4种Filter Type，这些Filter Type分别对应请求的不同生命周期。

( 1 ) PRE Filter: PRE Filter在请求被路由之前调用。一般用于实现身份验证、资源审查、记录调试信息等。

( 2) ROUTING Filter: ROUTING Filter将请求路由到微服务实例，该Filter用于构建发送给微服务实例的请求，并使用Apache HTTPClient或Netflix Ribbon请求微服务实例。

( 3) POST Filter: POST Filter一般用来为响应添加标准的HTTP Header、收集统计信息和指标，以及将响应从微服务发送给到客户端等，该Filter在将请求路由到微服务实例以后被执行。

( 4) ERROR Filter：在其他阶段发生错误时执行ERROR Filter。

### 3.网关服务中，路由器的4 种路由规则方法是什么？

采用URL 指定路由方式 采用服务名称指定路由方式 路由的排除方法 路由的添加前缀方法

### 4.**Zuul**实现原理-架构

1. Zuul提供了一个框架，可以对过滤器进行动态的加载、编译、运行。过滤器之间没有直接的相互通信。他们是通过一个RequestContext的静态类来进行数据传递的。
2. RequestContext类中有ThreadLocal变量来记录每个Request所需要传递的数据。
3. 过滤器是由Groovy写成。这些过滤器文件被放在Zuul Server上的特定目录下面。Zuul会定期轮询这些目录。修改过的过滤器会动态的加载到Zuul Server中以便于request使用。

### 5.**Zuul实现原理-过滤器**

**有五种标准的过滤器类型：**

- **pre：**这种过滤器在请求到达Origin Server之前调用。比如身份验证，在集群中选择请求的Origin Server，记log等。
- **routing：**在这种过滤器中把用户请求发送给Origin Server。发送给Origin Server的用户请求在这类过滤器中build。并使用Apache HttpClient或者Netfilx Ribbon发送给Origin Server。
- **post：**这种过滤器在用户请求从Origin Server返回以后执行。比如在返回的response上面加response header，做各种统计等。并在该过滤器中把response返回给客户。
- **error：**在其他阶段发生错误时执行该过滤器。
- **客户定制：**比如我们可以定制一种STATIC类型的过滤器，用来模拟生成返回给客户的response。

### 6.Zuul底层

Zuul 配置请求路径与服务的对应关系,你的请求到网关,他就直接查找到匹配的服务,然后就直接把请求转发给那个服务的某台机器, Ribbon 从 Eureka 本地缓存列表里面获取一台机器,然后通过负载均衡算法选择一台,把请求直接用 http 通信框架发送到指定的机器上面去。

### 7.**zuul的工作流程?**

在Spring Cloud Netflix中，Zuul巧妙的整合了Eureka来实现面向服务的路由。 实际上，API网关将自己注册到Eureka服务注册中心上，也会从注册中心获取所有服务以及它们的实例清单。在Eureka的帮助下，API网关已经维护了系统中所有`serviceId与实例地址的映射关系`。当有外部请求到达API网关的时候，根据请求的URL找到最匹配的path，API网关就可以知道要将该请求"路由"到哪个具体的serviceId上去。最终通过`Ribbon的负载均衡策略`实现请求的路由。

## **Spring Cloud链路监控**

### 1.Sleuth+Zipkin

Spring Cloud Sleuth是Spring Cloud的组成部分之一，为Spring Cloud应用提供一种分布式追踪解决方案，同时结合Zipkin 、HTrace和Log采集服务实现分布式链路追踪和展示。

Sleuth的介绍

Sleuth主要依赖Span、Trace和Annotation实现分布式链路追踪。

( 1 ) Span: Span是基本的工作单元，包括一个64位的唯一id、一个64位的Trace码、服务调用的描述信息、时间戳事件、Key-Value注解（Tags）和Span处理者的id（通常为IP）等信息。最初的Span被称为根Span，该Span种的Spanld与TraceId值相同。

( 2 ) Trance: Trance由一系列Span组成，这些Span组成一个树型结构。

( 3 ) Annotation ：用于及时记录存在的事件。常用的Annotation如下。

- cs-Client Sent：客户端发送一个请求到服务端，作为请求的开始，也即Span的开始。
- sr-Server Receive ：服务端接收到客户端的请求并开始处理该请求。sr和cs之间的时间间隔为网络的延迟时间。
- ss-Server Sent：服务端处理完客户端的请求，开始向客户端返回处理结果。ss和sr之间的时间间隔表示服务端处理请求的执行时间。
- er-Client Received ：客户端接收到服务端返回的结果，至此请求结束。er和sr之间的时间间隔表示客户端接收服务端的数据所使用的时间。

Sleuth+Zipkin实现分布式链路追踪

Sleuth为分布式链路追踪提供了数据结构基础，但是在使用过程中往往需要一个可视化的集中式的链路数据的存储和展示系统，方便快速定位问题，Zipkin为应用程序提供了该功能。

## **Spring Cloud Gateway**

### 1.API 网关

API Gateway是一个服务器，也可以说是进入系统的唯一节点。这跟面向对象设计模式中的Facade模式很像。API Gateway封装内部系统的架构，并且提供API给各个客户端。它还可能有其他功能，如授权、监控、负载均衡、缓存、请求分片和管理、静态响应处理等。

API Gateway负责请求转发、合成和协议转换。所有来自客户端的请求都要先经过API Gateway，然后路由这些请求到对应的微服务。API Gateway将经常通过调用多个微服务来处理一个请求以及聚合多个服务的结果。它可以在web协议与内部使用的非Web友好型协议间进行转换，如HTTP协议、WebSocket协议。

请求转发

服务转发主要是对客户端的请求安装微服务的负载转发到不同的服务上。

响应合并

把业务上需要调用多个服务接口才能完成的工作合并成一次调用对外统一提供服务。

协议转换

重点是支持SOAP，JMS，Rest间的协议转换。

数据转换

重点是支持XML和Json之间的报文格式转换能力（可选）

安全认证

1.基于Token的客户端访问控制和安全策略。

2.传输数据和报文加密，到服务端解密，需要在客户端有独立的SDK代理包。

3.基于Https的传输加密，客户端和服务端数字证书支持。

4.基于OAuth2.0的服务安全认证(授权码，客户端，密码模式等）

### 2.如何限流？

什么是限流

> 限流可以认为服务降级的一种，限流就是限制系统的输入和输出流量已达到保护系统的目的。一般来说系统的吞吐量是可以被测算的，为了保证系统的稳定运行，一旦达到的需要限制的阈值，就需要限制流量并采取一些措施以完成限制流量的目的。比如：延迟处理，拒绝处理，或者部分拒绝处理等等。

限流方法

计数器

实现方式

- 控制单位时间内的请求数量

劣势

- 假设在 00:01 时发生一个请求,在 00:01-00:58 之间不在发送请求,在 00:59 时发送剩下的所有请求

n-1(n 为限流请求数量),在下一分钟的 00:01 发送 n 个请求,这样在 2 秒钟内请求到达了2n - 1 个.

- 设每分钟请求数量为 60 个,每秒可以处理 1 个请求,用户在 00:59 发送 60 个请求,在 01:00 发送 60 个请求 此时 2 秒钟有 120 个请求(每秒 60 个请求),远远大于了每秒钟处理数量的阈值

滑动窗口

实现方式

- 滑动窗口是对计数器方式的改进, 增加一个时间粒度的度量单位
  - 把一分钟分成若干等分(6 份,每份 10 秒), 在每一份上设置独立计数器,在 00:00-00:09 之间发生请求计数器累加 1.当等分数量越大限流统计就越详细

Leaky Bucket 漏桶

实现方式

- 规定固定容量的桶, 有水进入, 有水流出. 对于流进的水我们无法估计进来的数量、速度, 对于流出的水我们可以控制速度.

Token Bucket 令牌桶

实现方式

- 规定固定容量的桶, token 以固定速度往桶内填充, 当桶满时 token 不会被继续放入, 每过来一个请求把 token 从桶中移除, 如果桶中没有 token 不能请求

spring cloud gateway

- spring cloud gateway 默认使用 redis 进行限流,

## Spring Cloud Sleuth

### 1.服务跟踪（starter-sleuth）

随着微服务数量不断增长，需要跟踪一个请求从一个微服务到下一个微服务的传播过程，Spring Cloud Sleuth 正是解决这个问题，它在日志中引入唯一ID，以保证微服务调用之间的一致性，这样你就能跟踪某个请求是如何从一个微服务传递到下一个。

1.为了实现请求跟踪，当请求发送到分布式系统的入口端点时，只需要服务跟踪框架为该请求创建一个唯一的跟踪标识，同时在分布式系统内部流转的时候，框架始终保持传递该唯一标识，直到返回给请求方为止，这个唯一标识就是前文中提到的Trace ID。通过Trace ID的记录，我们就能将所有请求过程日志关联起来。

2.为了统计各处理单元的时间延迟，当请求达到各个服务组件时，或是处理逻辑到达某个状态时，也通过一个唯一标识来标记它的开始、具体过程以及结束，该标识就是我们前文中提到的Span ID，对于每个Span来说，它必须有开始和结束两个节点，通过记录开始Span和结束Span的时间戳，就能统计出该Span的时间延迟，除了时间戳记录之外，它还可以包含一些其他元数据，比如：事件名称、请求信息等。'‘'

## Spring Cloud Ribbon

### 1.什么是Ribbon

1.Ribbon 是一个基于Http 和TCP 的客服端负载均衡工具，它是基于Netflix Ribbon实现的。 2.它不像spring cloud 服务注册中心、配置中心、API 网关那样独立部署，但是它几乎存在于每个spring cloud 微服务中。包括feign 提供的声明式服务调用也是基于该Ribbon实现的。 3.ribbon 默认提供很多种负载均衡算法，例如 轮询、随机 等等。甚至包含自定义的负载均衡算法。

### 2.Ribbon 的常见负载均衡策略有哪些？

轮询策略（默认）

RoundRobinRule

轮询策略表示每次都顺序取下一个provider，比如一共有5 个provider，第1 次取第1 个，第2 次取第2 个，第3 次取第3 个，以此类推。

权重轮询策略

WeightedResponseTimeRule

1.根据每个provider 的响应时间分配一个权重，响应时间越长，权重越小，被选中的可能性越低。 2.原理：一开始为轮询策略，并开启一个计时器，每30 秒收集一次每个provider 的平均响应时间，当信息足够时，给每个provider 附上一个权重，并按权重随机选择provider，高权越重的provider 会被高概率选中。

随机策略

RandomRule

从provider 列表中随机选择一个provider

最少并发数策略

BestAvailableRule

选择正在请求中的并发数最小的provider，除非这个provider 在熔断中。

在“选定的负载均衡策略”基础上进行重试机制

RetryRule

1.“选定的负载均衡策略”这个策略是轮询策略RoundRobinRule 2.该重试策略先设定一个阈值时间段，如果在这个阈值时间段内当选择provider 不成功，则一直尝试采用“选定的负载均衡策略：轮询策略”最后选择一个可用的provider。

可用性敏感策略

AvailabilityFilteringRule

过滤性能差的provider,有2 种： 第一种：过滤掉在eureka 中处于一直连接失败provider 第二种：过滤掉高并发的provider

区域敏感性策略

ZoneAvoidanceRule

1.以一个区域为单位考察可用性，对于不可用的区域整个丢弃，从剩下区域中选可用的provider 2.如果这个ip 区域内有一个或多个实例不可达或响应变慢，都会降低该ip 区域内其他ip 被选中的权重。

### 3.Ribbon和Feign的区别？

1.Ribbon都是调用其他服务的，但方式不同。 2.启动类注解不同，Ribbon是@RibbonClient feign的是@EnableFeignClients 3.服务指定的位置不同，Ribbon是在@RibbonClient注解上声明，Feign则是在定义抽象方法的接口中使用@FeignClient声明。 4.调用方式不同，Ribbon需要自己构建http请求，模拟http请求然后使用RestTemplate发送给其他服务，步骤相当繁琐。Feign需要将调用 的方法定义成抽象方法即可。

### 4.Ribbon负载均衡能干什么？

1.将用户的请求平摊的分配到多个服务上 2.集中式LB即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过 某种策略转发至服务的提供方； 3.进程内LB将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。 注意： Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。