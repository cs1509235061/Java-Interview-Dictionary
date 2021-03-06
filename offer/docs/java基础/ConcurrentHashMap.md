## 1.ConcurrentHashMap：分段锁实现，线程安全

ConcurrentHashMap和HashMap的实现方式类似，不同的是它采用分段锁的思想支持并发操作，所以是线程安全的。

### 减小锁粒度

减小锁粒度指通过缩小锁定对象的范围来减少锁冲突的可能性，最终提高系统的并发能力。减小锁粒度是一种削弱多线程锁竞争的有效方法，ConcurrentHashMap并发下的安全机制就是基于该方法实现的。

ConcurrentHashMap是线程安全的Map，对于HashMap而言，最重要的方法是get和set方法，如果为了线程安全对整个HashMap加锁，则可以得到线程安全的对象，但是加锁粒度太大，意味着同时只能有一个线程操作HashMap，在效率上就会大打折扣；而ConcurrentHashMap在内部使用多个Segment，在操作数据时会给每个Segment都加锁，这样就通过减小锁粒度提高了并发度。

### ConcurrentHashMap的实现

ConcurrentHashMap在内部细分为若干个小的HashMap，叫作数据段（Segment）。在默认情况下，一个ConcurrentHashMap被细分为 16个数据段，对每个数据段的数据都单独进行加锁操作。Segment的个数为锁的并发度。

ConcurrentHashMap是由Segment数组和HashEntry数组组成的。Segment继承了可重入锁（ReentrantLock），它在ConcurrentHashMap里扮演锁的角色。HashEntry则用于存储键值对数据。

在每一个ConcurrentHashMap里都包含一个Segment数组，Segment的结构和HashMap类似，是数组和链表结构。在每个Segment里都包含一个HashEntry数组，每个HashEntry都是一个链表结构的数据，每个Segment都守护一个HashEntry数组里的元素，在对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。

在操作ConcurrentHashMap时，如果需要在其中添加一个新的数据，则并不是将整个HashMap加锁，而是先根据HashCode查询该数据应该被存放在哪个段，然后对该段加锁并完成put操作。在多线程环境下，如果多个线程同时进行put操作，则只要加入的数据被存放在不同的段中，在线程间就可以做到并行的线程安全。

![](D:\workspace\Java-Interview-Offer\images\集合006.png)

Java 8在ConcurrentHashMap中引入了红黑树。

![](D:\workspace\Java-Interview-Offer\images\集合007.png)