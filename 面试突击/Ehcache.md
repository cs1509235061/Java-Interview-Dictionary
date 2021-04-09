# Ehcache

### 1.Ehcache的原理及应用

Ehcache是基于Java实现的一套简单、高效、线程安全的缓存管理类库。Ehcache提供了内存、磁盘文件及分布式存储方式等多种灵活的Cache管理方案，特点是快速、轻量、可伸缩、操作灵活、支持持久化等。

Ehcache的原理

Ehcache是基于Java实现的高效缓存框架，其内部采用多线程实现，采用LinkedHashMap存储元素，同时支持将数据持久化到物理磁盘上。

Ehcache的特点

（1）快速：Ehcache内部采用多线程机制实现，数据存取性能高。 （2）轻量：Ehcache的安装包大小只有1.6MB，可以被快速、方便地继承到系统中。 （3）可伸缩：Ehcache缓存在内存和硬盘的存储可以伸缩到数几十GB，可轻松应对大数据场景。 （4）操作灵活：Ehcache提供了丰富的API接口，可实现基于主键、条件进行数据读取等。同时，Ehcache支持在运行时修改缓存配置 （存活时间、空闲时间、内存的最大数据、磁盘的最大数量），提高了系统维护的灵活性。 （5）支持多种淘汰算法：Ehcache支持最近最少被使用、最少被使用和先进先出缓存策略。 （6）支持持久化：Ehcache支持将缓存数据持久化到磁盘上，在机器重启后从磁盘上重新加载缓存数据。

Ehcache的架构

Ehcache在架构上由Cache Replication、In-Process API和Core组成。其中，Cache Replication存储缓存副本；In-Process API封装操作缓存数据的API ， 包括Hibernate API 、JMX API 、ServletCacheing Filter API等；Core是Ehcache的核心部分，包括用于管理缓存的CacheManger、用于存储缓存的Store和用于操作缓存的CacheAPI等；NetWork APIs提供RESTful API、SOAP API等Web API接口。

Ehcache的存储方式

Ehcache的存储方式包括堆存储、堆外存储和磁盘存储。 （1）堆存储：将缓存数据存储在Java堆内存中，其特点是存取速度快，但容量有限。 （2）堆外存储：基于NIO的DirectByteBuffers实现，将缓存数据存储在堆外内存上。其特点是比磁盘存取速度快，而且不受GC的影响，可以保证响应时间的稳定性，在内存分配上开销比堆内存大，而且要求必须以字节数组方式存储，因此对象必须在存储过程中进行序列化，对读取操作则进行反序列化，数据存取速度比堆内存慢一个数量级。 （3）磁盘存储：将数据存储在磁盘上，保障服务重启后内存数据能够重新从磁盘上加载，其读取效率最低，是内存数据持久化的一种方式。

### 2.Ehcache的应用

（ 1 ） 引入jar 包。

（ 2 ） 设置ehcache.xml 。

在项目resource 的目录下新建ehcache.xml配置文件。

```html
<?xml version="1.0" encoding="UTF-8">
<ehcache>
    <cache name="user" eternal="true" overflowToDisk="true" maxElementsInMemor="1000"/>
</ehcache>      
```

以上代码在ehcache.xml配置文件中声明了一个名称为user的缓存，其中eternal=true表示缓存对象永不过期，maxElementsInMemory表示内存中该Cache可存储最大的数据量，overflowToDisk=true表示在内存缓存的对象数量达到了maxElementsInMemory界限后，会把溢出的对象写到磁盘缓存中。注意：如果需要将缓存的对象写入磁盘中，则该对象必须实现了Serializable接口。

（3）使用Ehcache缓存

在方法上通过@Cacheable(value = "user", key ="#user.id")开启Ehcache缓存。

首先会去Ehcache缓存中查找数据，如果在缓存中存在该数据，则将该数据返回，如果在缓存中不存在该数据，则会去数据库中查询并返回结果。