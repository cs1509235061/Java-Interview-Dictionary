# Mybatis

### 1.Mybatis 中 #和 $的区别？

\#相当于对数据 加上 双引号， $相当于直接显示数据

1. \#将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号。如： order by #user_id# id#，如果传入的值是111, 那么解析成 sql 时的值为 order by "111", 如果传入的值是 id ，则解析成的 sql 为 order by "id".
2. $ 将传入的数据直接显示生成在 sql 中。如： order by $user_id$ id$，如果传入的值是 111, 那么解析成 sql 时的值为order by user_id, 如果传入的值是 id ，则解析成的 sql 为 order by id.
3. \#方式能够很大程度防止 sql 注入。
4. $ 方式无法防止 Sql 注入。
5. $ 方式一般用于传入数据库对象，例如传入表名
6. 一般能用 #的就别用$

### 2.Mybatis 的编程步骤是什么样的？

1、创建 SqlSessionFactory 2、通过 SqlSessionFactory 创建 SqlSession 3、通过 sqlsession 执行数据库操作 4、调用 session.commit() 提交事务 5、调用 session.close() 关闭会话

### 3.使用 MyBatis 的 mapper 接口调用时有哪些要求？

1. Mapper接口方法名和 mapper.xml 中定义的每个 sql 的 id 相同
2. Mapper接口方法的输入参数类型和 mapper.xml 中定义的每个 sql 的 parameterType 的类型相同
3. Mapper接口方法的输出参数类 型和 mapper.xml 中定义的每个 sql 的 resultType 的类型相同
4. Mapper.xml文件中的 namespace 即是 mapper 接口的类路径。

### 4.JDBC 编程有哪些不足之处， MyBatis 是如何解决这些问题的？

1.数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。 解决：在SqlMapConfig.xml 中配置数据链接池，使用连接池管理数据库链接。

2.Sql语句写在代码中造成代码不易维护，实际应用 sql 变化的可能较大， sql 变动需要改变 java 代码。 解决：将Sql 语句配置在 XXXXmapper.xml 文件中与 java 代码分离。 3.向 sql 语句传参数麻烦，因为 sql 语句的 where 条件不一定，可能多也可能少，占位符需要和参数一一对应。 解决：Mybatis 自动将 java 对象映射至 sql 语句。 4.对结果集解析麻烦， sql 变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成 pojo 对象解析比较方便。

解决：Mybatis 自动将 sql 执行结果映射至 java 对象。

### 5.MyBatis 在 insert 插入操作时返回主键 ID

数据库为MySql 时：

```html
<insert id="insert" parameterType="com.test.User" keyProperty="userId" useGeneratedKeys="true">
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

“keyProperty ”表示返回的 id 要保存到对象的那个属性中，“ useGeneratedKeys ”表示主键 id 为自增长模式。MySQL 中做以上配置就 OK 了。

### 6.Mybatis 是如何将sql 执行结果封装为目标对象并返回的？都有哪些映射形式？

第一种是使用<resultMap>标签，逐一定义列名和对象属性名之间的映射关系。 第二种是使用sql 列的别名功能，将列别名书写为对象属性名，比如T_NAME ASNAME，对象属性名一般是name，小写，但是列名不区分大小写，Mybatis 会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成T_NAME AS NaMe，Mybatis一样可以正常工作。 有了列名与属性名的映射关系后，Mybatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

### 7.MyBatis 接口绑定有几种实现方式,分别是怎么实现的?

接口绑定有两种实现方式,一种是通过注解绑定,就是在接口的方法上面加上@Select@Update 等注解里面包含Sql 语句来绑定,另外一种就是通过xml 里面写SQL 来绑定,在这种情况下,要指定xml 映射文件里面的namespace 必须为接口的全路径名.

### 8.MyBatis 实现一对一有几种方式?具体怎么操作的？

有联合查询和嵌套查询,联合查询是几个表联合查询,只查询一次,通过在resultMap 里面配置association 节点配置一对一的类就可以完成;嵌套查询是先查一个 表,根据这个表里面的结果的外键id,去再另外一个表里面查询数据,也是通过association配置,但另外一个表的查询通过select 属性配置。

### 9.MyBatis 有几种分页方式

分页方式：逻辑分页和物理分页。 逻辑分页：使用 MyBatis 自带的 RowBounds 进行分页，它是一次性查询很多数据，然后在数据中再进行检索。 物理分页：自己手写 SQL 分页或使用分页插件 PageHelper ，去数据库查询指定条数的分页数据的形式。

### 10.RowBounds 是一次性查询全部结果吗？为什么？

RowBounds 表面是在“所有”数据中检索数据，其实并非是一次性查询出所有数据，因为 MyBatis 是对jdbc 的封装，在 jdbc 驱动中有一个 Fetch Size 的配置，它规定了每次最多从数据库查询多少条数据，假如你要查询更多数据，它会在你执行 next()的时候，去查询更多的数据。 就好比你去自动取款机取 10000 元，但取款机每次最多能取 2500 元，所以你要取 4 次才能把钱取完。只是对于 jdbc 来说，当你调用 next()的时候会自动帮你完成查询工作。这样做的好处可以有效的防止内存溢出。

### 11.MyBatis 逻辑分页和物理分页的区别是什么？

逻辑分页是一次性查询很多数据，然后再在结果中检索分页的数据。这样做弊端是需要消耗大量的内存、有内存溢出的风险、对数据库压力较大。 物理分页是从数据库查询指定条数的数据，弥补了一次性全部查出的所有数据的种种缺点，比如需要大量的内存，对数据库查询压力较大等问题。

### 12.什么是MyBatis 的接口绑定？有哪些实现方式？

接口绑定，就是在MyBatis 中任意定义接口,然后把接口里面的方法和SQL 语句绑定, 我们直接调用接口方法就可以,这样比起原来了SqlSession 提供的方法我们可 以有更加灵活的选择和设置。 接口绑定有两种实现方式,一种是通过注解绑定， 就是在接口的方法上面加上@Select、@Update 等注解，里面包含Sql 语句来绑定； 另外一种就是通过xml里面写SQL 来绑定, 在这种情况下,要指定xml 映射文件里面的namespace 必须为接口的全路径名。当Sql 语句比较简单时候,用注解绑定, 当SQL 语句比较复杂时候,用xml 绑定,一般用xml 绑定的比较多。

### 13.MyBatis 有哪些执行器（Executor）？

MyBatis 有三种基本的Executor执行器： SimpleExecutor：每执行一次 update 或 select 就开启一个 Statement 对象，用完立刻关闭Statement 对象； ReuseExecutor：执行 update 或 select，以 SQL 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后不关闭 Statement 对象，而是放置于 Map 内供下一次使用。简言之，就是重复使用Statement 对象；

BatchExecutor：执行 update（没有 select，jdbc 批处理不支持 select），将所有 SQL 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理，与 jdbc 批处理相同。

### 14.当实体类中的属性名和表中的字段名不一样，怎么办？

第1 种： 通过在查询的sql 语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。

第2 种： 通过<resultMap>来映射字段名和实体类属性名的一一对应的关系。

### 15.通常一个Xml 映射文件，都会写一个Dao 接口与之对应，请问，这个Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？

Dao 接口即Mapper 接口。接口的全限名，就是映射文件中的namespace 的值；接口的方法名，就是映射文件中Mapper 的Statement 的id 值； 接口方法内的 参数，就是传递给sql 的参数。 Mapper 接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key 值，可唯一定位一个MapperStatement。在Mybatis 中， 每一个<select>、<insert>、<update>、<delete>标签，都会被解析为一个MapperStatement 对象。

Mapper 接口里的方法，是不能重载的，因为是使用全限名+方法名的保存和寻找策略。Mapper 接口的工作原理是JDK 动态代理，Mybatis 运行时会使用JDK动态代理为Mapper 接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement 所代表的sql，然后将sql 执行结果返回。

### 16.在mapper 中如何传递多个参数?

1、DAO 层的函数

```html
public UserselectUser(String name,String area);
对应的xml,#{0}代表接收的是dao 层中的第一个参数，#{1}代表dao 层中第二参数，更多参数一致往后加即可。
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```html
<select id="selectUser"resultMap="BaseResultMap">
    select * fromuser_user_t whereuser_name = #{0}anduser_area=#{1}
</select>
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2、 使用@param 注解

3、多个参数封装成map

### 17.动态sql

Mybatis 动态sql 可以在Xml 映射文件内，以标签的形式编写动态sql，执行原理是根据表达式的值完成逻辑判断并动态拼接sql 的功能。 Mybatis 提供了9 种动态sql 标签：trim | where | set | foreach | if | choose| when | otherwise | bind。

### 18.Mybatis 的Xml 映射文件中， 不同的Xml 映射文件， id 是否可以重复？

不同的Xml 映射文件，如果配置了namespace，那么id 可以重复；如果没有配置namespace，那么id 不能重复； 原因就是namespace+id 是作为Map<String, MapperStatement>的key使用的， 如果没有namespace，就剩下id，那么， id 重复会导致数据互相覆盖。有了namespace，自然id 就可以重复，namespace 不同，namespace+id 自然也就不同。

### 19.MyBatis 实现一对一有几种方式?具体怎么操作的？

有联合查询和嵌套查询,联合查询是几个表联合查询,只查询一次, 通过在resultMap 里面配置association 节点配置一对一的类就可以完成； 嵌套查询是先查一个表，根据这个表里面的结果的外键id，去再另外一个表里面查询数据,也是通过association 配置，但另外一个表的查询通过select 属性配置。

### 20.MyBatis 实现一对多有几种方式,怎么操作的？

有联合查询和嵌套查询。联合查询是几个表联合查询,只查询一次,通过在resultMap 里面的collection 节点配置一对多的类就可以完成；嵌套查询是先查一个表,根据这个表里面的结果的外键id,去再另外一个表里面查询数据,也是通过配置collection,但另外一个表的查询通过select 节点配置。

### 21.Mybatis 是否支持延迟加载？如果支持，它的实现原理是什么？

Mybatis 仅支持association 关联对象和collection 关联集合对象的延迟加载，association 指的就是一对一，collection 指的就是一对多查询。在Mybatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。 它的原理是，使用CGLIB 创建目标对象的代理对象， 当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null 值， 那么就会单独发送事先保存好的查询关联B 对象的sql，把B 查询上来，然后调用a.setB(b)，于是a 的对象b 属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。 当然了， 不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。

### 22.简述Mybatis 的插件运行原理，以及如何编写一个插件。

Mybatis 仅可以编写针对ParameterHandler、ResultSetHandler、StatementHandler、Executor 这4 种接口的插件，Mybatis 使用JDK 的动态代理， 为需要拦截的接口生成代理对象以实现接口方法拦截功能， 每当执行这4 种接口对象的方法时，就会进入拦截方法，具体就是InvocationHandler 的invoke()方法，当然，只会拦截那些你指定需要拦截的方法。 编写插件：实现Mybatis 的Interceptor 接口并复写intercept()方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住， 别忘了在配置文件中配置你编写的插件。

### 23.MyBatis的初始化

MyBatis的初始化可以有两种方式：

- 基于XML配置文件：基于XML配置文件的方式是将MyBatis的所有配置信息放在XML文件中，MyBatis通过加载并XML配置文件，将配置文信息组装成内部的Configuration对象
- 基于Java API：这种方式不使用XML配置文件，需要MyBatis使用者在Java代码中，手动创建Configuration对象，然后将配置参数set 进入Configuration对象中

**MyBatis初始化基本过程：**

SqlSessionFactoryBuilder根据传入的数据流生成Configuration对象，然后根据Configuration对象创建默认的SqlSessionFactory实例。

mybatis初始化要经过简单的以下几步：

1. 调用SqlSessionFactoryBuilder对象的build(inputStream)方法；
2. SqlSessionFactoryBuilder会根据输入流inputStream等信息创建XMLConfigBuilder对象;
3. SqlSessionFactoryBuilder调用XMLConfigBuilder对象的parse()方法；
4. XMLConfigBuilder对象返回Configuration对象；
5. SqlSessionFactoryBuilder根据Configuration对象创建一个DefaultSessionFactory对象；
6. SqlSessionFactoryBuilder返回 DefaultSessionFactory对象给Client，供Client使用。

### 24.使用的设计模式

1. Builder模式，例如SqlSessionFactoryBuilder、XMLConfigBuilder、XMLMapperBuilder、XMLStatementBuilder、CacheBuilder；
2. ⼯⼚模式，例如SqlSessionFactory、ObjectFactory、MapperProxyFactory；
3. 单例模式，例如ErrorContext和LogFactory；
4. 代理模式，Mybatis实现的核⼼，⽐如MapperProxy、ConnectionLogger，⽤的jdk的动态代理；还有executor.loader包使 ⽤了cglib或者javassist达到延迟加载的效果；
5. 组合模式，例如SqlNode和各个⼦类ChooseSqlNode等；
6. 模板⽅法模式，例如BaseExecutor和SimpleExecutor，还有BaseTypeHandler和所有的⼦类例如IntegerTypeHandler；
7. 适配器模式，例如Log的Mybatis接⼝和它对jdbc、log4j等各种⽇志框架的适配实现；
8. 装饰者模式，例如Cache包中的cache.decorators⼦包中等各个装饰者的实现；
9. 迭代器模式，例如迭代器模式PropertyTokenizer；

## 缓存

### 1.MyBatis的缓存

MyBatis缓存分为一级缓存和二级缓存，在默认情况下，一级缓存是开启的，而且不能被关闭。

- 一级缓存：SqlSession级别的缓存，当在同一个SqISession中执行相同的SQL语句查询时将查询结果集缓存，第二次以后的查询不会从数据库中查询，而是直接从缓存中获取，一级缓存最多能缓存1024条SQL语句。基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session ，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空。

- 二级缓存：指跨SqlSession的缓存，即Mapper级别的缓存。在Mapper级别的缓存内，不同的SqlSession缓存可以共享。Namespace)，并且可自定义存储源，如 Ehcache 。作用域为 namespance 是指对该 namespance 对应的配置文件中所有的 select 操作结果都缓存，这样不同线程之间就可以共用二级缓存。启动二级缓存：在 mapper 配置文件中： <cache/> 。

  二级缓存可以设置返回的缓存对象策略：<cache readOnly="true"> 。当 readOnly="true" 时，表示二级缓存返回给所有调用者同一个缓存对象实例，调用者可以 updat e 获取的缓存实例，但是这样可能会造成其他调用者出现数据不一致的情况（因为所有调用者调用的是同一个实例）。当 readOnly="false" 时，返回给调用者的是二级缓存总缓存对象的拷贝，即不同调用者获取的是缓存对象不同的实例，这样调用者对各自的缓存对象的修改不会影响到其他的调用者，即是安全的，所以默认是 readOnly="false";

- 对于缓存数据更新机制，当某一个作用域(一级缓存 Session/ 二级缓存 Namespaces) 的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被

  clear 。

### 2.MyBatis的一级缓存原理

当客户端第一次发出一个SQL查询语句时，MyBatis执行SQL查询并将查询结果写入SqlSession的一级缓存，当第二次有相同的SQL查询语句时，则直接从缓存中获取数据。在缓存中使用的数据结构是Map，其中，Key为Mapperld+Offset+Limit+SQL＋所有的入参。value：用户信息。

当同一个SqlSession多次发出相同的SQL查询语句时，MyBatis直接从缓存中获取数据。如果两次查询中间出现Commit操作（修改、添加、删除），则认为数据发生了变化，MyBatis会把该SqlSession中的一级缓存区域全部清空，当下次再到缓存中查询时将找不到对应的缓存数据，因此要再次从数据库中查询数据并将查询的结果写入缓存。

### 3.MyBatis的二级缓存原理

MyBatis二级缓存的范围是Mapper级别的，Mapper以命名空间为单位创建缓存数据结构，数据结构是Map类型，Map中Key为Mapperld+Offset+Limit+SQL＋所有的入参。My Batis的二级缓存是通过CacheExecuto实现的。CaceExecutor是Executor的代理对象。当MyBatis接收到用户的查询请求时，首先会根据Map的Key在CacheExecutor缓存中查询数据是否存在，如果不存在则在数据库中查询。

开启二级缓存需要做以下配置

( 1）在MyBatis全局配置中启用二级缓存配置。

( 2）在对应的Mapper.xml中配置Cache节点。

( 3 ）在对应的Select查询节点中添加useCache=true。