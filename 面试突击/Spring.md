



# Spring

### 1.Spring的特性

Spring特性包括轻量、控制反转（Inversion of Control, IoC ）、面向容器、面向切面（Aspect Oriented Programming, AOP）和框架灵活。

轻量

从大小与开销两方面而言Spring都是轻量的，完整的Spring框架可以在一个大小只有1M多的JAR文件里发布，并且Spring所需要的处理开销也是微不足道的。

此外，Spring是非侵入式的，典型的，Spring应用中的对象不依赖于Spring的特定类

控制反转IOC

Spring的控制反转指一个对象依赖的其他对象将会在容器的初始化完成后主动将其依赖的对象传递给它，而不需要这个对象自己创建或者查找其依赖的对象。Spring基于控制反转技术实现系统对象间依赖的解耦。

面向容器

Spring实现了对象的配置化生成和对象的生命周期管理，因此，可以理解为其是面向容器的，通过Spring的XML文件或者注解方式，应用程序可以配置每个Bean对象被创建和销毁的时间，以及Bean对象创建的先后顺序和依赖关系。Spring中的实例对象可以是全局唯一的单例模式，也可以在每次需要时都重新生成一个新的实例，具体以哪种方式创建Bean对象由Bean的生命周期决定，通过prototype属性来定义。

面向切面

Spring提供了面向切面的编程支持，面向切面技术通过分离系统逻辑和业务逻辑来提高系统的内聚性。在具体的使用过程中，业务层只需要关注并实现和业务相关的代码逻辑，而不需要关注系统功能（例如系统日志、事务支持）和业务功能的复杂关系，Spring通过面向切面技术将系统功能自动织入业务逻辑的关键点。

框架灵活

基于容器化的对象管理技术， Spring 中的对象可以被声明式地创建，例如通过XML文件或注解的方式定义对象和对象之间的依赖关系。Spring 作为一个轻量级的J2EE Web框架，具有事务管理、持久化框架集成和Java Web 服务等功能，应用程序可以根据需求引入相应的模块，以实现不同的功能。

### 2.Spring的模块

Spring提供的常用模块有核心容器层（Core Container）、 数据访问层（DataAccess ）、Web应用层（Web Access）。除此 之外，Spring还包括AOP、Aspects、Instrumentation、Messaging和Test。

核心容器层

核心容器层由Spring-Beans、Spring-Core、Spring-Context和SpEL( Spring Expression Language, Spring表达式语言)等模块组成。

Spring-Beans

Spring-Beans 模块基于工厂模式实现对象的创建。Spring-Beans通过XML配置文件实现了声明式的对象管理，将对象之间复杂的依赖关系从实际编码逻辑中解耦出来。

Spring-Core

Spring-Core模块是Spring的核心功能实现，具体包括控制反转和依赖注入，所谓依赖注入，是指在一个Bean实例中引用另外一个Bean实例时，Spring容器会自动创建其所依赖的Bean实例，并将该Bean实例注入（传递）至对应的Bean中。

Spring-Context

Spring-Context模块是在Spring-Beans和Spring-Core模块的基础上构建起来的。Spring-Context模块继承自Spring-Beans模块，并且添加了国际化、事件传播、资源加载和透明地创建上下文等功能。

同时，Spring-Context模块提供了一些J2EE的功能，比如EJB、JMX和远程调用等。ApplicationContext接口是Spring-Context模块操作Bean的入口。

Spring-Context-Support提供了将第三方库集成到Spring-Context的支持，比如缓存( EhCache、Guava、JCache）、邮件(JavaMail）、调度（CommonJ、Quartz）和模板引擎( FreeMarker、JasperReports、Velocity）等。

SpEL

SpEL模块提供了丰富的表达式语言支持，用于在运行过程中查询和操作对象实例。SpEL在JSP2.1表达式语言规范的基础上进行了扩展，支持set方法、get方法、属性赋值、方法调用、访问数组集合、索引内容、逻辑算术运算、命名变量、基于名称在Spring IoC容器检索对象，同时支持列表的投影、选择和聚合等功能。

数据访问层

数据访问层包括JDBC、ORM、OXM、JMS和事务处理模块。

它们的术语描述。

- JDBC：Java Date Base Connectivity
- ORM：Object Relational Mapping
- OXM：Object XML Mapping
- JMS：Java Message Service

JDBC

JDBC模块提供了JDBC抽象层。Spring持久化层基于JDBC抽象层实现了在不同数据库之间灵活切换，而不用担心不同数据库之间SQL语法的不兼容。

ORM

ORM模块提供了对象关系映射API的集成，包括JPA( Java Persistence API）、JDO( Java Data Object）和 Hibernate等。基于该模块，ORM框架能很容易地和Spring的其它功能（例如事务管理）整合。

OXM

OXM模块提供了对OXM实现的支持，比如JAXB、Castor、XMLBeans、JiBX、XStream等

JMS

JMS模块包含消息的生产（Produc）和消费（onsume）功能。从Spring4.1开始，Spring集成了Spring-Messaging模块，用于实对消息队列的支持。

事务处理

事务处理（Transactions ）模块基于接口方式实现了声明式事务管式。编程式事务的实现需要应用程序调用相应的beanTransaction（）、commit（）、rollback（）等方法来实现事务的管理，Spring声明式事务只需要通过注解或配置即可实现事务的管理，具体的事务管理工作由Spring自动处理，应用程序不需要关心事务的提交（Commit）和回滚（Rollback）。

Web应用层

Web应用层主要包含Web交互和数据传输等相关功能，由WebWeb-MVWeb�SocketWeb-Portlet组成。

Web

Web模块不但提供了面向Web应用的基本功能，还提供了HTTP( Hyper Text Transfer Protocol，超文本传输协议）客户端及Spring远程调用中与Web相关的部分。Web模块基于Servlet监听器初始化IoC容器

Web-MVC

Web-MVC模块为Web应用提供了模型视图控制（ModeView Controller, MVC）和REST API服务的实现。Spring的MVC框架使数据模型和视图分离，数据模型负责数据的业务逻辑，视图负责数据的展示。同时，Web-MVC可与Spring框架的其他模块方便地集成

Web-Socket

Web-Socket模块提供了对WebSocket-Base的支持，用于实现在Web应用程序中服务端和客户端实时双向通信，尤其在实时消息推送中应用广泛

Web-Portlet

Web-Portlet模块提供了基于Portlet环境的MVC实现，井提供了与Spring web-MVC模块相关的功能。

其他重要模块

除了上述介绍的模块，Spring还有其他一些重要的模块，例如，AOP、Aspects、Instrumentation、Messaging和Test等。

AOP

AOP模块提供了面向切面的编程实现，允许应用程序通过定义方法拦截器和切人点来实现系统功能和业务功能之间的解耦。

Aspects

Aspects模块提供了Spring与AspectJ的集成，是一个面向切面编程的模块

Instrumentation

Instrumentation模块在应用中提供了对Instrumentation的支持和类加载器的实现。

Messaging

Messaging模块为STOMP( Simple Text Orientated Messaging Protocol ，简单文本定向消息协议）提供了支持，主要用于应用程序中WebSocket子协议的实现。同时，Messaging模块可通过注解的方式来选择和处理来自WebSocket客户端的STOMP消息。

Test

Test （测试）模块用于对JUnit或TestNG等测试框架提供支持，以实现Spring代码的自动化测试。

### 3.Spring 注解

bean注入与装配的的方式有很多种，可以通过xml，getset方式，构造函数或者注解等。简单易用的方式就是使用Spring的注解了，Spring提供了大量的注解方式。

Spring在 2.5 版本以后开始支持注解的方式来配置依赖注入。可以用注解的方式来代替 xml 中 bean 的描述。注解注入将会被容器在 XML 注入之前被处理，所以后者会覆盖掉前者对于同 一个属性的处理结果。

注解装配在spring 中默认是关闭的。所以需要在 spring 的核心配置文件中配置一下才能使用基于注解的装配模式。配置方式如下：

```html
<context:annotationconfig />
```

- Bean 声明
  - @Component：定义基础层的通用组件，没有明确的角色
  - @ Service：定义业务逻辑层的服务组件
  - @Repository：在数据访问层定义数据资源服务
  - @Controller：在展现层使用， 用于定义控制器
- Bean 注入
  - @Autowired：服务依赖注入， 一般用于注入＠Component 、@Se rvice 定义的组件
  - @Resource：服务依赖注入，一般用于注入＠ Repository 定义的组件
- 配置类注解
  - @Configuration：声明该类为配置类，其中＠Value 属性可以直接和配置文件属性映射
  - @Bean：注解在方法上，声明该方法的返回值为一个Bean实例
  - @ComponentScan：用于对Component 进行扫描配置
- AOP 注解
  - @EnableAspectJAutoProxy：开启Spring 对AspectJ代理的支持
  - @Aspect：声明一个切面，使用＠After 、＠Before 、＠Around定义通知（ Advice ），可直接将拦截规则（切点）作为参数
  - @After：在方法执行之后执行
  - @ Before：在方法执行之前执行
  - @ Around：在方法执行之前和之后都执行
  - @ PointCut：声明一个切点
- @ Bean 属性支持注解
  - @Scope：设置Spring 容器Bean 实例的生命周期，取值有singleton 、prototype 、request 、session 和global session
  - @PostConstruct：声明方法在构造函数执行完之后开始执行
  - @ PreDestroy：声明方法在Bean 销毁之前执行
  - @ Value： 为属性注入值
  - @ PropertySource： 声明和加载配置文件
- 异步操作注解
  - @ EnableAsync： 声明在类上，开启对异步任务的支持
  - @ Async： 声明方法是一个异步任务， Spring 后台基于线程池异步执行该方法
- 定时任务相关
  - @EnableScheduling： 声明在调度类上，开启对任务调度的支持
  - @Scheduled：声明一个定时任务，包括cron 、fixDelay 、fixRate等参数
- 开启功能支持
  - @ EnableAspectJAutoProxy： 开启对AspectJ 自动代理的支持
  - @EnableAsync： 开启对异步方法的支持
  - @ EnableScheduling： 开启对计划任务的支持
  - @ EnableWebMVC： 开启对Web MVC 的配置支持
  - @EnableConfigurationProperties：开启对＠Configuration Properties 注解配置Bean 的支持
  - @ EnableJpaRepositories ：开启对SpringData JPA Repository 的支持
  - @EnableTransactionManagement： 开启对事务的支持
  - @EnableCaching： 开启对缓存的支持
- 测试相关注解
  - @ RunWith： 运行器， Spring 中通常用于对JUnit 的支持
  - @ContextConfiguration： 用来加载配置Application Context ，其中classes 属性用来加载配置类

### 4.谈谈你对 Spring 的理解

Spring是一个开源框架，为简化企业级应用开发而生。 Spring 可以是使简单的 JavaBean 实现以前只有 EJB 才能实现的功能。 Spring 是一个 IOC 和 AOP 容器框架。 Spring容器的主要核心是： 控制反转（IOC ），传统的 java 开发模式中，当需要一个对象时，我们会自己使用 new 或者 getInstance 等直接或者间接调用构造方法创建一个对象。而在 spring 开发模式中， spring 容器使用了工厂模式为我们创建了所需要的对象，不需要我们自己创建了，直接调用 spring 提供的对象就可以了，这是控制反转的思想。 依赖注入（DI）， spring 使用 javaBean 对象的 set 方法或者带参数的构造方法为我们在创建所需对象时将其属性自动设置所需要的值的过程，就是依赖注入的思想。 面向切面编程（AOP ），在面向对象编程（ oop ）思想中，我们将事物纵向抽成一个个的对象。而在面向切面编程中，我们将一个个的对象某些类似的方面横向抽成一个切面，对这个切面进行一些如权限控制、事物管理，记录日志等公用操作处理的过程就是面向切面编程的思想。 AOP 底层是动态代理，如果是接口采用 JDK 动态代理，如果是类采用CGLIB 方式实现动态代理。

### 5.Spring Bean的装配流程

Sprin在启动时会从XML配置文件或注解中读取应用程序提供的Bean配置信息，并在Spring容器中生成一份相应的Bean配置注册表；然后根据这张注册表实例化Bean，装配好Bean之间的依赖关系，为上层业务提供基础的运行环境，其中Bean缓存池为HashMap实现。

### 6.Spring Bean的作用域

Spring为Bean定义了5种作用域，分别为Singleton（单例）、Prototype（原型）、Request （请求级别）、Session（会话级别）和Global Session （全局会话）。

Singleton ：单例模式（多线程下不安全）

Singleton是单例模式，当实例类型为单例模式时，Spring IoC容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，都始终指向同一个Bean对象。该模式在多线程下是不安全的。Singleton作用域是Spring中的默认作用域，也可以通过配置将Bean定义为Singleton模式，具体配置如下

```html
<bean id="userDao" class="com.alex.UserDaoImpl" scope="singleton">
```

Prototype：原型模式每次使用时创建

Prototype是原型模式，每次通过Spring容器获取Prototype定义的Bean时，容器都将创建一个新的Bean实例，每个Bean实例都有自己的属性和状态，而Singleton全局只有一个对象。因此，对有状态的Bean经常使用Prototype作用域，而对无状态的Bean则使用Singleton作用域。具体配置如下

```html
<bean id="userSerice" class="com.alex.UserSerice" scope="prototype">
```

Request：一次request一个实例

Request指在一次HTTP请求中容器会返回该Bean的同一个实例，而对不同的HTTP请求则会创建新的Bean实例，并且该Bean实例仅在当前HTTP请求内有效，当前HTTP请求结束后，该Bean实例也将会随之被销毁。具体配置如下

```html
<bean id="loginAction" class="com.alex.Login" scope="request">
```

Session

Session指在一次HTTP Session中容器会返回该Bean的同一个实例，而对不同的Session请求则会创建新的Bean实例，该Bean实例仅在当前Session内有效。和HTTP请求相同，每一次Session都会创建新的Bean实例，而不同的Bean实例之间不共享数据，且Bean实例仅在自己的Session内有效，请求结束，则Bean实例将随之被销毁。具体配置如下

```html
<bean id="userSession" class="com.alex.UserSession" scope="session">
```

Global Session

Global Session指在一个全局的HTTP Session中容器会返回该Bean的同一个实例，且仅在使用Portlet Cotext时有效。

### 7.Spring Bean的生命周期

实例化

1.实例化一个Bean，也就是我们常说的new。

IOC依赖注入

2.按照Spring上下文对实例化的Bean进行配置，也就是IOC注入。

setBeanName实现

3.如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String)方法，此处传递的就是Spring配置文件中Bean的id值

BeanFactoryAware实现

4.如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory，setBeanFactory(BeanFactory)传递的是Spring工厂自身（可以用这个方式来获取其它Bean，只需在Spring配置文件中配置一个普通的Bean就可以）。

ApplicationContextAware实现

5.如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文（同样这个方式也可以实现步骤4的内容，但比4更好，因为ApplicationContext是BeanFactory的子接口，有更多的实现方法）

postProcessBeforeInitialization接口实现-初始化预处理

6.如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization(Object obj, String s)方法，该方法在Bean初始化前调用，常用于定义初始化Bean的前置工作，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用那个的方法，也可以被应用于内存或缓存技术。

init-method

7.如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。

postProcessAfterInitialization

8.如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法。注：以上工作完成以后就可以应用这个Bean了，那这个Bean是一个Singleton的，所以一般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在Spring配置文件中也可以配置非Singleton。

Destroy过期自动清理阶段

9.当Bean不再被需要时，会在清理阶段被清理掉。如果Bean实现了DisposableBean接口，则Spring会在退出前调用实现类的destroy（）方法

destroy-method自配置清理

10.如果某个Bean的Spring配置文件中配置了destroy-method属性，在Bean被销毁前会自动调用其配置的销毁方法。

11.bean 标签有两个重要的属性（init-method和destroy-method）。用它们你可以自己定制初始化和注销方法。它们也有相应的注解（@PostConstruct和@PreDestroy）。

总结

bean定义：在配置文件里面用 <bean></ 来进行定义。 bean初始化：有两种方式初始化 1.在配置文件中通过指定 init method 属性来完成 2.实现 org.springframwork.beans.factory.InitializingBean 接口 bean调用：有三种方式可以得到 bean 实例，并进行调用 bean 销毁：销毁有两种方式 1.使用配置文件指定的 destroy method 属性 2.实现 org.springframw ork.bean.factory.DisposeableBean 接口

### 8.Spring的4种依赖注入

构造器注入

构造器注入指通过在类的构造函数中注入属性或对象来实现依赖注入。如下代码通过＜bean></bean＞标签配置了一个id为persionDaolmpl的Bean，并通过标签在其构造函数中注入了一个message属性，注入完成后在类中可以直接通过this.message获取注入的属性值。

```html
／／在构造函数中注入message属性
public PersonDaoImpl(String message) { 
    this.message= message; 
}   
<!--定义Bean实例并在构造函数(constructor-arg)中注入message属性-->
<bean id=”persionDaoImpl" class=”com.PersionDaoImpl”>       <constructor-arg value=”message”></constructor-arg> </bean> 
```

set 方法注入

set方法注入是通过在类中实现get、set方法来实现属性或对象的依赖注入的。如下代码通过标签配置了一个id为persionDaolmpl的Bean，并通过标签在Bean中注入了一个id为123的属性值，注入完成后在类中可以直接使用getld（）获取注入的属性。

```html
public class PersionDaoImpl { 
    private int id; ／／定义属性和其对应的get、set方法
    public int getId () { 
        return id; 
    } 
    public void setId（int id) { 
        this.id= id; 
    }
    <！--定义Bean实例并通过property注入id为123的属性值-->
<bean id="persionDaoImpl" class="com.PersionDaoImpl">       <property name=”id" value=”123”></property> 
</bean> 
```

静态工厂注入

静态工厂注入是通过调用工厂类中定义的静态方法来获取需要的对象的，为了让Spring管理所有对象，应用程序不能直接通过“工厂类．静态方法（）”的方式获取对象，而需要通过Spring注入的方式获取。

```html
public class DaoFactory { //1 ：定义静态工厂
    public static final FactoryDao getStaticFactoryDaoImpl() { 
        return new StaticFactoryDaoImpl(); 
    }
}
public class SpringAction { 
    private FactoryDao staticFactoryDao; //2.定义工厂对象
    //3：注入工厂对象
    public void setStaticFactoryDao(FactoryDao staticFactoryDao){ 
        this.staticFactoryDao = staticFactoryDao; 
    }
}
```

上述代码定义了一个DaoFactory工厂类和getStaticFactoryDaolmpl（）静态工厂方法，该方法实例化并返回一个StaticFactoryDaolmpl实例；同时定义了一个SpringAction类，并通过setStaticFactory Dao获取注入的FactoryDao。具体的XML注入语法如下

```html
<!--：定义获取工厂对象的静态方法一〉<bean name="staticFactoryDao" class="DaoFactory" factory-method="getStaticFactoryDaoImpl"></bean>
<!-- factory-method＝”getStaticFactoryDaoImpl用于指定调用哪个工厂方法-->
<bean name="springActon" class="SpringAction">
	<!--2：注入静态工厂实例-->
	<property name="staticFactoryDao ref＝"staticFactoryDao"></property>
</bean> 
```

上述代码定义了一个name为staticFactoryDao的工厂类，并通过factory-method定义了实例化对象的方法，这里实例化对象的方法是一个名为getStaticFactoryDaoImpl的静态方法。该静态方法返回一个工厂类实例，在springAction中通过＜property></property＞标签注入静态工厂实例。

实例工厂注入

实例工厂注入指的是获取象实例的方法是非静态的，因此首先需要实例化，然后调用对例化方法来实例化对象

```html
public class DaoFactory { //1： 实例工厂
	public FactoryDao getFactoryDaoImpl() { 
		return new FactoryDaoImpl(); 
	}
}
public class SpringAction { 
	private FactoryDao factoryDao; //2:注入对象				public void setFactoryDao(FactoryDao factoryDao) { 			this.factoryDao = factoryDao; 
	}
}
<bean name=”springAction”class=”SpringAction”>〈
	<！--1：使用实例工厂的方式注入对象-->
	<property name=”factoryDao" ref="factoryDao"></property>
</bean> 
<!--2:获取对象的方式是从工厂类中获取实例-->
<bean name=”daoFactory”class=”com.DaoFactory”></bean> <bean name="factoryDao" factory-bean=”daoFactory”factory-method=”getFactoryDaoImpl”>
</bean>
```

上述代码定义了一个name为factoryDao的工厂类，并通过factory-method定义了实例化对象的方法，这里实例化对象的方法是一个名为getFactoyDaolmpl的方法。该方法返回一个工厂类，在springAction中通过</property＞标签注入工厂实例。

### 9.自动装配的5种方式

Spring的装配方式包括手动装配和自动装配。手动装配包括基于XML装配（构造方法、set方法等）和基于注解装配2种方式。自动装配包括5种装配方式，这5种方式均可以用来引导Spring容器自动完成依赖注入，具体如下。

( I ) no：关闭自动装配，通过显式设置ref属性来进行对象装配。

( 2) byName：通过参数名自动装配，Bean的autowire被设置为byName后，Spring容器试图匹配并装配与该属性具有相同名字的Bean。

( 3) byType：通过参数类型自动装配，Bean的autowire被设置为byTyp后，Spring容器试图匹配并装配与该Bean的属性具有相同类型的Bean。

( 4 ) constructor：通过设置构造器参数的方式来装配对象，如果没有匹配到带参数的构造器参数类型，则Spring会抛出异常。

( 5 ) autodetect：首先尝试使用constructor来自动装配，如果无法完成自动装配，则使用byType方式进行装配。

### 10.impon 标签的解析

1. 获取resource 属性所表示的路径。
2. 解析路径中的系统属性，格式如“$｛user.dir ｝ ”。
3. 判定location 是绝对路径还是相对路径。
4. 如果是绝对路径则递归调用bean 的解析过程，进行另一次的解析。
5. 如果是相对路径则计算出绝对路径并进行解析。
6. 通知监听器，解析完成。

### 11.Spring 加载bean 的过程

1.转换对应beanName

传入的参数可能是别名，也可能是FactoryBean ，所以需要进行一系列的解析。

2.尝试从缓存中加载单例

单例在S pring 的同一个容器内只会被创建一次，后续再获取bean ，就直接从单例缓存中获取了。当然这里也只是尝试加载，首先尝试从缓存中力11 载，如果加载不成功则再次尝试从s ingletonFactories 中加载。因为在创建单例bean 的时候会存在依赖注入的情况，而在创建依赖的时候为了避免循环依赖，在Spring 中创建bean 的原则是不等bean 创建完成就会将创建bean的Obj ectFactory 提早曝光加入到缓存中， 一旦下一个bean 创建时候需要依赖上一个bean 则直接使用0均ectFactory。

3.bean 的实例化

如果从缓存中得到了bean 的原始状态，则需要对bean 进行实例化。缓存中记录的只是最原始的bean 状态， 井不一定是我们最终想要的bean 。

4.原型模式的依赖检查

只有在单例情况下才会尝试解决循环依赖，如果存在A 巾有B 的属性， B 巾有A 的属性，那么当依赖注入的时候，就会产生当A 还未创建完的时候因为对于B 的创建再次返回创建A,造成循环依赖，也就是情况： isPrototyp巳CurrentlyinCreation(beanName）判断true 。

5.检测parentBeanFactory

在检测如果当前加载的XML 配置文件中不包含beanName 所对应的配置，就只能到parentBeanFactory 去尝试下了，然后再去递归的调用getBean 方法。

6.将存储XML 配置文件的GernericBeanDefinition 转换为RootBean Definition

7.寻找依赖

因为bean 的初始化过程中很可能会用到某些属性，而某些属性很可能是动态配置的，并且配置成依赖于其他的bean ，那么这个时候就有必要先力［J载依赖的bean ，所以，在Spring 的加载顺序中，在初始化某一个bean 的时候首先会初始化这个bean 所对应的依赖。

8.针对不同的scope 进行bean 的创建

在Spring 中存在着不同的scope ，其巾默认的是singleton ，但是还有些其他的配置诸如prototype 、request 之类的。在这个步骤中， Spring 会根据不同的配置进行不同的初始化策略。

9.类型转换

将返回的bean 转换为requiredType 所指定的类型。

### 12.循环依赖

什么是循环依赖

循环依赖就是循环引用，就是两个或多个bean 相互之间的持有对方，比如CircleA 引用CircleB , CircleB 引用CircleC, Circl eC 引用CircleA ，则它们最终反映为一个环。

Spring 如何解决循环依赖

1 .构造器循环依赖

表示通过构造器注入构成的循环依赖， 此依赖是无法解决的，只能抛出BeanCurrentl ylnCreationException异常表示循环依赖。

2.setter循环依赖

对于Setter 注入造成的依赖是通过Spring 容器提前暴露刚完成构造器注入但未完成其他步骤（如setter 注入）的bean 来完成的，而且只能解决单例作用域的bean 循环依赖。通过提前暴露一个单例工厂方法，从而使其他bean 能引用到该bean。

3.prototype 范围的依赖处理

对于“prototype”作用域bean, Spring 容器无法完成依赖注入，因为Spring 容器不进行缓存“prototype”作用域的bean ，因此无法提前暴露一个创建中的bean。

### 13.Spring Bean 的生命周期

- 单例（ Singleton ）的生命周期和Spring 上下文的生命周期一致，自始至终只被Sping 创建一次。
- 多例（ Prototype ）的生命周期在事件级别， 每触发一次getBean ，调用方获取的实例就是新的。
- Request 的生命周期在HTTP 请求级别， 在每次Web HTTP 请求中都被创建一次。
- Session 的生命周期在HTTP Session 级别， 在每个HTTP Session 中都被创建一次。
- Global Session 的生命周期类似于Session 的生命周期，但仅在基于Portlet 的Web应用中有意义。PorletSession 继承自Http Session ，使用目的和Http Session 一致。
- Application 的生命周期与整个Web 应用的生命周期一致。

### 14.Spring 中的设计模式

a.单例模式 ——spring 中两种代理方式，若目标对象实现了若干接口， spring 使用 jdk 的 java.lang.reflect.Pro xy类代理。若目标兑现没有实现任何接口，spring 使用 CGLIB 库生成目标类的子类。 单例模式——在 spring 的配置文件中设置 bean 默认为单例模式。 b.模板方式模式——用来解决代码重复的问题。 比如：RestTemplate 、 JmsTemplate 、 JpaTemplate c.前端控制器模式 ——spring 提供了前端控制器 DispatherServlet 来对请求进行分发。 d.视图帮助（ view helper）—— spring 提供了一系列的 JSP 标签，高效宏来帮助将分散的代码整合在试图中。 e.依赖注入——贯穿于 BeanFactory/ApplacationContext 接口的核心理念。 f.工厂模式—— 在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用同一个接口来指向新创建的对象。 Spring 中使用 beanFactory 来创建对象的实例。

### 15.BeanFactory 常用的实现类有哪些？

Bean工厂是工厂模式的一个实现，提供了控制反转功能，用来把应用的配置和依赖从正真的应用代码中分离。常用的 BeanFactory 实现 有 DefaultListableBeanFactory 、 XmlBeanFactory 、 ApplicationContext 等。XMLBeanFactory最常用的就是 org.springframework.beans.factory.xml.

XmlBeanFactory ，它根据 XML 文件中的定义加载beans 。该容器从 XML 文件读取配置元数据并用它去创建一个完全配置的系统或应用。

### 16.BeanFact ory 与 AppliacationContext 有什么区别

1.BeanFactory 基础类型的IOC 容器，提供完成的 IOC 服务支持。如果没有特殊指定，默认采用延迟初始化策略。相对来说，容器启动初期速度较快，所需资源有限。 2.ApplicationContext ApplicationContext是在 BeanFactory 的基础上构建，是相对比较高级的容器实现，除了 BeanFactory 的所有支持外， ApplicationContext 还提供了事件发布、国际化支持等功能 。 ApplicationContext 管理的对象，在容器启动后默认全部初始化并且绑定完成。

### 17.ApplicationContext 的实现类有哪些

FileSystemXmlApplicationContext：此容器从一个 XML 文件中加载 beans 的定义， XML Bean 配置文件的全路径名必须提供给它的构造函数。 ClassPathXmlApplicationContext ：此容器也从一个 XML 文件中加载 beans 的定义，这里，你需要正确设置classpath 因为这个容器将在 classpath 里找 bean 配置。 WebXmlApplicationContext：此容器加载一个 XML 文件，此文件定义了一个 WEB 应用的所有 bean 。

### 18.什么是 Spring beans?

Spring beans 是那些形成 Spring 应用的主干的 java 对象。它们被 Spring IOC 容器初始化，装配和管理。这些 beans 通过容器中配置的元数据创建。比如 ，以 XML 文件中 <bean/> 的形式定义。Spring框架定义的 beans 都是 单例 beans 。

### 19.spring 中的 bean 是线程安全的吗

spring 中的 bean 默认是单例模式，spring 框架并没有对单例 bean 进行多线程的封装处理。 实际上大部分时候 spring bean 无状态的（比如 dao 类），所有某种程度上来说 bean 也是安全的，但如果 bean 有状态的话（比如 view model 对象），那就要开发者自己去保证线程安全了，最简单的就是改变 bean 的作用域，把“singleton”变更为“prototype”，这样请求 bean 相当于 new Bean()了，所以就可以保证线程安全了。 有状态就是有数据存储功能。 无状态就是不会保存数据。

### 20.什么是 Spring 的内部 bean

当一个bean 仅被用作另一个 bean 的属性时，它能被声明为一个内部 bean ，为了定义 inner bean ，在Spring 的 基于 XML 的 配置元数据中，可以在

<property/>或 <constructor arg/> 元素内使用 <bean/> 元素，内部 bean 通常是匿名的，它们的 Scope 一般是 prototype 。

### 21.在 Spring 中如何注入一个 java 集合？

Spring提供以下几种集合的配置元素： <list>类型用于注入一列值，允许有相同的值。 <set>类型用于注入一组值，不允许有相同的值。

< map>类型用于注入一组键值对，键和值都可以为任意类型。 <props>类型用于注入一组键值对，键和值都只能为 String 类型。

### 22.@Autowired 和@Resource 的区别？

1、@Autowired默认按照byType方式进行bean匹配，@Resource默认按照byName方式进行bean匹配 2、@Autowired是Spring的注解，@Resource是J2EE的注解，这个看一下导入注解的时候这两个注解的包名就一清二楚了 Spring属于第三方的，J2EE是Java自己的东西，因此，建议使用@Resource注解，以减少代码和Spring之间的耦合。

### 23.import 过程

1. 获取 source 属性的值，该值表示资源的路径
2. 解析路径中的系统属性，如"${user.dir}"
3. 判断资源路径 location 是绝对路径还是相对路径
4. 如果是绝对路径，则调递归调用 Bean 的解析过程，进行另一次的解析
5. 如果是相对路径，则先计算出绝对路径得到 Resource，然后进行解析
6. 通知监听器，完成解析

### 24.spring 提供了哪些配置方式？

基于 xml 配置

bean 所需的依赖项和服务在 XML 格式的配置文件中指定。这些配置文件通常包含许多 bean 定义和特定于应用程序的配置选项。它们通常以 bean 标签开头。

基于注解配置

您可以通过在相关的类，方法或字段声明上使用注解，将 bean 配置为组件类本身，而不是使用 XML 来描述 bean 装配。默认情况下，Spring 容器中未打开注解装配。因此，您需要在使用它之前在 Spring 配置文件中启用它。例如：

基于 Java API 配置

Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。 1、 @Bean 注解扮演与 元素相同的角色。 2、 @Configuration 类允许通过简单地调用同一个类中的其他 @Bean 方法来定义 bean 间依赖关系。

### 25.@Component, @Controller, @Repository，@Service 有何区别？

@Component ：这将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring 的组件扫描机制现在可以将其拾取并将其拉 入应用程序环境中。 @Controller ：这将一个类标记为 Spring Web MVC 控制器。标有它的Bean 会自动导入到 IoC 容器中。 @Service ：此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用@Service 而不是 @Component，因为它以更好的方式指定了意图。 @Repository ：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器， 并使未经检查的异常有资格转换为 Spring DataAccessException。

### 26.@Required 注解有什么用？

@Required 应用于 bean 属性 setter 方法。此注解仅指示必须在配置时使用bean 定义中的显式属性值或使用自动装配填充受影响的 bean 属性。如果尚未填充受影响的 bean 属性，则容器将抛出 eanInitializationException。

### 27.@Autowired 注解有什么用？

@Autowired 可以更准确地控制应该在何处以及如何进行自动装配。此注解用于在 setter 方法，构造函数，具有任意名称或多个参数的属性 或方法上自动装配bean。默认情况下，它是类型驱动的注入。

### 28.@Qualifier 注解有什么用？

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪 个确切的 bean来消除歧义。

### 29.@RequestMapping 注解有什么用？

@RequestMapping 注解用于将特定 HTTP 请求方法映射到将处理相应请求的 控制器中的特定类/方法。此注释可应用于两个级别： 类级别：映射请求的 URL 方法级别：映射 URL 以及 HTTP 请求方法

### 30.Spring中Autowired和Resource关键字的区别

@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是Spring的注解，它的包是javax.annotation.Resource，需要 导入，但是Spring支持该注解的注入。 1、共同点 两者都可以写在字段和setter方法上。两者如果都写在字段上，那么就不需要再写setter方法。 2、不同点 （1）@Autowired @Autowired为Spring提供的注解，需要导入包 org.springframework.beans.factory.annotation.Autowired;只按照byType注入

@Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属 性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。

（2）@Resource @Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和 type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使 用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制 使用byName自动注入策略

@Resource装配顺序： ①如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异 常。 ②如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。 ③如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常 ④如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹 配则自动装配。@Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入

### 31.IoC 和 DI 有什么区别？

IoC 是个更宽泛的概念，DI 是更具体的。

**IoC控制反转**（IoC，Inversion of Control）

是一个概念，是一种思想。控制反转就是对对象控制权的转移，从程序代码本身反转到了外部容器。把对象的创建、初始化、销毁等工作交给spring容器来做。由spring容器控制对象的生命周期。即是**将new 的过程交给spring容器去处理**.

**DI依赖注入**

依赖注入DI是指程序运行过程中，若需要调用另一个对象协助时，无须在代码中创建被调用者，而是依赖于外部容器，由外部容器创建后传递给程序。依赖注入是目前最优秀的解耦方式。依赖注入让Spring的Bean之间以配置文件的方式组织在一起，而不是以硬编码的方式耦合在一起的。

### 32.Spring 中有多少种 IoC 容器？

Spring 提供了两种( 不是“个” ) IoC 容器，分别是 BeanFactory、ApplicationContext 。

**BeanFactory**

> BeanFactory 在 `spring-beans` 项目提供。

BeanFactory ，就像一个包含 Bean 集合的工厂类。它会在客户端要求时实例化 Bean 对象。

**ApplicationContext**

> ApplicationContext 在 `spring-context` 项目提供。

ApplicationContext 接口扩展了 BeanFactory 接口，它在 BeanFactory 基础上提供了一些额外的功能。内置如下功能：

- MessageSource ：管理 message ，实现国际化等功能。
- ApplicationEventPublisher ：事件发布。
- ResourcePatternResolver ：多资源加载。
- EnvironmentCapable ：系统 Environment（profile + Properties）相关。
- Lifecycle ：管理生命周期。
- Closable ：关闭，释放资源
- InitializingBean：自定义初始化。
- BeanNameAware：设置 beanName 的 Aware 接口。

另外，ApplicationContext 会自动初始化非懒加载的 Bean 对象们。

| BeanFactory                | ApplicationContext       |
| -------------------------- | ------------------------ |
| 它使用懒加载               | 它使用即时加载           |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象 |
| 不支持国际化               | 支持国际化               |
| 不支持基于依赖的注解       | 支持基于依赖的注解       |

另外，BeanFactory 也被称为**低级**容器，而 ApplicationContext 被称为**高级**容器。

### 33.请介绍下常用的 ApplicationContext 容器？

以下是三种较常见的 ApplicationContext 实现方式：

- 1、ClassPathXmlApplicationContext ：从 ClassPath 的 XML 配置文件中读取上下文，并生成上下文定义。应用程序上下文从程序环境变量中取得。示例代码如下：

  ```html
  ApplicationContext context = new ClassPathXmlApplicationContext(“bean.xml”);
  ```

- 2、FileSystemXmlApplicationContext ：由文件系统中的XML配置文件读取上下文。示例代码如下：

  ```html
  ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”);
  ```

- 3、XmlWebApplicationContext ：由 Web 应用的XML文件读取上下文。例如我们在 Spring MVC 使用的情况。

当然，目前我们更多的是使用 Spring Boot 为主，所以使用的是第四种 ApplicationContext 容器，ConfigServletWebServerApplicationContext 。

### 34.Spring 框架中有哪些不同类型的事件？

Spring 提供了以下五种标准的事件：

1. 上下文更新事件（ContextRefreshedEvent）：该事件会在ApplicationContext 被初始化或者更新时发布。也可以在调用ConfigurableApplicationContext 接口中的 `#refresh()` 方法时被触发。
2. 上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext 的 `#start()` 方法开始/重新开始容器时触发该事件。
3. 上下文停止事件（ContextStoppedEvent）：当容器调用 ConfigurableApplicationContext 的 `#stop()` 方法停止容器时触发该事件。
4. 上下文关闭事件（ContextClosedEvent）：当ApplicationContext 被关闭时触发该事件。容器被关闭时，其管理的所有单例 Bean 都被销毁。
5. 请求处理事件（RequestHandledEvent）：在 We b应用中，当一个HTTP 请求（request）结束触发该事件。

------

除了上面介绍的事件以外，还可以通过扩展 ApplicationEvent 类来开发**自定义**的事件。

① 示例自定义的事件的类，代码如下：

```html
public class CustomApplicationEvent extends ApplicationEvent{  

    public CustomApplicationEvent(Object source, final String msg) {  
        super(source);
    }  

}
```

② 为了监听这个事件，还需要创建一个监听器。示例代码如下：

```html
public class CustomEventListener implements ApplicationListener<CustomApplicationEvent> {

    @Override  
    public void onApplicationEvent(CustomApplicationEvent applicationEvent) {  
        // handle event  
    }
    
}
```

③ 之后通过 ApplicationContext 接口的 `#publishEvent(Object event)` 方法，来发布自定义事件。示例代码如下：

```html
// 创建 CustomApplicationEvent 事件
CustomApplicationEvent customEvent = new CustomApplicationEvent(applicationContext, "Test message");
// 发布事件
applicationContext.publishEvent(customEvent);
```

### 35.Spring中Autowired和Resource关键字的区别？

@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是Spring的注解，它的包 是javax.annotation.Resource，需要导入，但是Spring支持该注解的注入。

1、共同点 两者都可以写在字段和setter方法上。两者如果都写在字段上，那么就不需要再写setter方法。 2、不同点 （1）@Autowired @Autowired为Spring提供的注解，需要导入包 org.springframework.beans.factory.annotation.Autowired;只按照byType注入。

@Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允 许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结 合@Qualifier注解一起使用

（2）@Resource @Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。 @Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的 名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策 略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通 过反射机制使用byName自动注入策略。

注：最好是将@Resource放在setter方法上，因为这样更符合面向对象的思想，通过set、get去操作属 性，而不是直接去操作属性。

@Resource装配顺序： ①如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异 常。 ②如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。 ③如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛 出异常。 ④如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回 退为一个原始类型进行匹配，如果匹配则自动装配。 @Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。

### 36.注解

@Controller 标识⼀个该类是Spring MVC controller处理器，⽤来创建处理http请求的对象.

@RestController Spring4之后加⼊的注解，原来在@Controller中返回json需要@ResponseBody来配合，如果直接⽤@RestController替代 @Controller就不需要再配置@ResponseBody，默认返回json格式。

@Service ⽤于标注业务层组件，说⽩了就是加⼊你有⼀个⽤注解的⽅式把这个类注⼊到spring配置中

@Autowired ⽤来装配bean，都可以写在字段上，或者⽅法上。 默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false

@RequestMapping 类定义处: 提供初步的请求映射信息，相对于 WEB 应⽤的根⽬录。 ⽅法处: 提供进⼀步的细分映射信息，相对于类定义处的 URL。

@RequestParam ⽤于将请求参数区数据映射到功能处理⽅法的参数上

@ModelAttribute 使⽤地⽅有三种： 1、标记在⽅法上。 标记在⽅法上，会在每⼀个@RequestMapping标注的⽅法前执⾏，如果有返回值，则⾃动将该返回值加⼊到ModelMap中。 (1) 在有返回的⽅法上: 当ModelAttribute设置了value，⽅法返回的值会以这个value为key，以参数接受到的值作为value，存⼊到Model中

(2) 在没返回的⽅法上： 需要⼿动model.add⽅法

2、标记在⽅法的参数上。 标记在⽅法的参数上，会将客⼾端传递过来的参数按名称注⼊到指定对象中，并且会将这个对象⾃动加⼊ModelMap中，便于View层使⽤.

@Cacheable ⽤来标记缓存查询。可⽤⽤于⽅法或者类中，当标记在⼀个⽅法上时表⽰该⽅法是⽀持缓存的，当标记在⼀个类上时则表⽰该类 所有的⽅法都是⽀持缓存的。

@CacheEvict ⽤来标记要清空缓存的⽅法，当这个⽅法被调⽤后，即会清空缓存。@CacheEvict(value=”UserCache”

@Resource @Resource的作⽤相当于@Autowired 只不过@Autowired按byType⾃动注⼊， ⽽@Resource默认按 byName⾃动注⼊罢了。 @Resource有两个属性是⽐较重要的，分是name和type，Spring将@Resource注解的name属性解析为bean的名字，⽽ type属性则解析为bean的类型。所以如果使⽤name属性，则使⽤byName的⾃动注⼊策略，⽽使⽤type属性时则使⽤byType ⾃动注⼊策略。如果既不指定name也不指定type属性，这时将通过反射机制使⽤byName⾃动注⼊策略。

@Resource装配顺序: 1、如果同时指定了name和type，则从Spring上下⽂中找到唯⼀匹配的bean进⾏装配，找不到则抛出异常 2、如果指定了name，则从上下⽂中查找名称（id）匹配的bean进⾏装配，找不到则抛出异常 3、如果指定了type，则从上下⽂中找到类型匹配的唯⼀bean进⾏装配，找不到或者找到多个，都会抛出异常 4、如果既没有指定name，⼜没有指定type，则⾃动按照byName⽅式进⾏装配；如果没有匹配，则回退为⼀个原始类型进⾏匹 配，如果匹配则⾃动装配；

@PostConstruct ⽤来标记是在项⽬启动的时候执⾏这个⽅法。⽤来修饰⼀个⾮静态的void()⽅法 也就是spring容器启动时就执⾏，多⽤于⼀些全局配置、数据字典之类的加载 被@PostConstruct修饰的⽅法会在服务器加载Servlet的时候运⾏，并且只会被服务器执⾏⼀次。PostConstruct在构造函数 之后执⾏,init()⽅法之前执⾏。PreDestroy（）⽅法在destroy()⽅法执⾏执⾏之后执

@PreDestroy 被@PreDestroy修饰的⽅法会在服务器卸载Servlet的时候运⾏，并且只会被服务器调⽤⼀次，类似于Servlet的destroy()⽅ 法。被@PreDestroy修饰的⽅法会在destroy()⽅法之后运⾏，在Servlet被彻底卸载之前

@Repository ⽤于标注数据访问组件，即DAO组件

@Component 泛指组件，当组件不好归类的时候，我们可以使⽤这个注解进⾏标注

@Scope ⽤来配置 spring bean 的作⽤域，它标识 bean 的作⽤域。 默认值是单例 1、singleton:单例模式,全局有且仅有⼀个实例 2、prototype:原型模式,每次获取Bean的时候会有⼀个新的实例 3、request:request表⽰该针对每⼀次HTTP请求都会产⽣⼀个新的bean，同时该bean仅在当前HTTP request内有效 4、session:session作⽤域表⽰该针对每⼀次HTTP请求都会产⽣⼀个新的bean，同时该bean仅在当前HTTP session内有效 5、global session:只在portal应⽤中有⽤，给每⼀个 global http session 新建⼀个Bean实例。

@SessionAttributes 默认情况下Spring MVC将模型中的数据存储到request域中。当⼀个请求结束后，数据就失效了。如果要跨⻚⾯使⽤。那么需要 使⽤到session。⽽@SessionAttributes注解就可以使得模型中的数据存储⼀份到session域中 参数： 1、names：这是⼀个字符串数组。⾥⾯应写需要存储到session中数据的名称。 2、types：根据指定参数的类型，将模型中对应类型的参数存储到session中 3、value：和names是⼀样的。

@Required 适⽤于bean属性setter⽅法，并表⽰受影响的bean属性必须在XML配置⽂件在配置时进⾏填充。否则，容器会抛出⼀个 BeanInitializationException异常。

@Qualifier 当你创建多个具有相同类型的 bean 时，并且想要⽤⼀个属性只为它们其中的⼀个进⾏装配，在这种情况下，你可以使⽤ @Qualifier 注释和 @Autowired 注释通过指定哪⼀个真正的 bean 将会被装配来消除混乱。

## IOC

### 1.依赖注入

当某一个Java 类，需要另一个Java 类的协助时，在传统的程序设计过程中，通常由当前类（调用者）来创建被调用者的实例，然后使用被调用者的方法。但在Spring 里，创建被调用者的工作不再由调用者来完成，而是由其它类（往往是工厂类）或容器（Spring IOC容器）完成，当前调用者从其它类或容器中来获取被调用者的实例，这种方式称为控制反转;创建被调用者实例的工作通常由Spring 容器来完成，然后注入调用者，因此也称为依赖注入，这是Spring 的一种编程思想的体现。

### 2.DI 是什么

控制反转DI(Dependency Injection)模式，就是Inversion of Control，控制反转。在Java开发中，IoC 意味着将你设计好的类交给系统去控制，而不是在你的类内部控制。这称为控制反转。 di （ dependency injection ） 依赖注入： 实际上DI 和IOC 是同一个概念， 因为在ApplicationContext.xml 配置文件中bean 和bean 之间通过ref 来维护的时候是相互依赖的，所以又叫做依赖注入。也就是控制反转。

### 3.Spring loC简介

概念

Spring通过一个配置文件描述Bean和Bean之间的依赖关系，利用Java的反射功能实例化Bean并建立Bean之间的依赖关系。Spring的IoC容器在完成这些底层工作的基础上，还提供了Bean实例缓存管理、Bean生命周期管理、Bean实例代理、事件发布和资源装载等高级服务。

Spring 容器

Spring在启动时会从XML配置文件或注解中读取应用程序提供的Bean配置信息，并在Spring容器中生成一份相应的Bean配置注册表；然后根据这张注册表实例化Bean，装配好Bean之间的依赖关系，为上层业务提供基础的运行环境。其中Bean缓存池为HashMap实现。

IOC容器实现

BeanFactory-框架基础设施

BeanFactory 是Spring 框架的基础设施，面向Spring 本身；ApplicationContext 面向使用Spring 框架的开发者，几乎所有的应用场合我们都直接使用ApplicationContext 而非底层的BeanFactory。

BeanDefinitionRegistry注册表

Spring 配置文件中每一个节点元素在Spring 容器里都通过一个BeanDefinition 对象表示，它描述了Bean 的配置信息。而BeanDefinitionRegistry 接口提供了向容器手工注册BeanDefinition 对象的方法。

BeanFactory 顶层接口

位于类结构树的顶端，它最主要的方法就是getBean(String beanName)，该方法从容器中返回特定名称的Bean，BeanFactory 的功能通过其他的接口得到不断扩展：

ListableBeanFactory

该接口定义了访问容器中Bean 基本信息的若干方法，如查看Bean 的个数、获取某一类型Bean 的配置名、查看容器中是否包括某一Bean 等方法；

HierarchicalBeanFactory父子级联

父子级联IoC 容器的接口，子容器可以通过接口方法访问父容器；通过HierarchicalBeanFactory 接口，Spring 的IoC 容器可以建立父子层级关联的容器体系，子容器可以访问父容器中的Bean，但父容器不能访问子容器的Bean。Spring 使用父子容器实现了很多功能，比如在Spring MVC 中，展现层Bean 位于一个子容器中，而业务层和持久层的Bean 位于父容器中。这样，展现层Bean 就可以引用业务层和持久层的Bean，而业务层和持久层的Bean 则看不到展现层的Bean。

ConfigurableBeanFactory

是一个重要的接口，增强了IoC 容器的可定制性，它定义了设置类装载器、属性编辑器、容器初始化后置处理器等方法；

AutowireCapableBeanFactory自动装配

定义了将容器中的Bean 按某种规则（如按名字匹配、按类型匹配等）进行自动装配的方法；

SingletonBeanRegistry运行期间注册单例Bean

定义了允许在运行期间向容器注册单实例Bean 的方法；对于单实例（singleton）的Bean 来说，BeanFactory会缓存Bean 实例，所以第二次使用getBean() 获取Bean 时将直接从IoC 容器的缓存中获取Bean 实例。Spring 在DefaultSingletonBeanRegistry 类中提供了一个用于缓存单实例Bean 的缓存器，它是一个用HashMap 实现的缓存器，单实例的Bean 以beanName 为键保存在这个HashMap 中。

依赖日志框框

在初始化BeanFactory 时，必须为其提供一种日志框架，比如使用Log4J，即在类路径下提供Log4J 配置文件，这样启动Spring 容器才不会报错。

ApplicationContext面向开发应用

ApplicationContext 由BeanFactory 派生而来，提供了更多面向实际应用的功能。ApplicationContext 继承了HierarchicalBeanFactory 和ListableBeanFactory 接口，在此基础上，还通过多个其他的接口扩展了BeanFactory 的功能：

1.ClassPathXmlApplicationContext：默认从类路径加载配置文件。

2.FileSystemXmlApplicationContext：默认从文件系统中装载配置文件。

3.ApplicationEventPublisher：让容器拥有发布应用上下文事件的功能，包括容器启动事件、关闭事件等。

4.MessageSource：为应用提供i18n 国际化消息访问的功能；

5.ResourcePatternResolver ：所有ApplicationContext 实现类都实现了类似于PathMatchingResourcePatternResolver 的功能，可以通过带前缀的Ant 风格的资源文件路径装载Spring 的配置文件。

6.LifeCycle：该接口是Spring 2.0 加入的，该接口提供了start()和stop()两个方法，主要用于控制异步处理过程。在具体使用时，该接口同时被ApplicationContext 实现及具体Bean 实现，ApplicationContext 会将start/stop 的信息传递给容器中所有实现了该接口的Bean，以达到管理和控制JMX、任务调度等目的。

7.ConfigurableApplicationContext 扩展于ApplicationContext，它新增加了两个主要的方法：refresh()和close()，让ApplicationContext 具有启动、刷新和关闭应用上下文的能力。在应用上下文关闭的情况下调用refresh()即可启动应用上下文，在已经启动的状态下，调用refresh()则清除缓存并重新装载配置信息，而调用close()则可关闭应用上下文。

WebApplication体系架构

WebApplicationContext 是专门为Web 应用准备的，它允许从相对于Web 根目录的路径中装载配置文件完成初始化工作。从WebApplicationContext 中可以获得ServletContext 的引用，整个Web 应用上下文对象将作为属性放置到ServletContext 中，以便Web 应用环境可以访问Spring 应用上下文。

### 4.Spring IOC 部分特性介绍

alias

alias 的中文意思是“别名”，在 Spring 中，我们可以使用 alias 标签给 bean 起个别名。

```html
<bean id="hello" class="xyz.coolblog.service.Hello">
	<property name="content" value="hello"/>
</bean>
<alias name="hello" alias="alias-hello"/>
```

autowire

autowire即自动注入的意思，通过使用 autowire特性，我们就不用再显示的配置 bean 之间的依赖了。把依赖的发现和注入都交给 Spring 去处理。

autowire 几个可选项，比如 byName 、byType 和 constructor 等。

当 bean 配置中的 autowire = byName 时， Spring 会首先通过反射获取该 bean 所依赖 bean 的名字（ beanName ），然后再通过调用 BeanFactory.getName(beanName) 方法即可获取对应的依赖实例。

FactoryBean

FactoryBean 是一种工厂bean， 与普通的bean 不一样，FactoryBean 是一种可以产生bean 的bean ，FactoryBean 是一个接口，我们可以实现这个接口。

factory-method

factorymethod可用于标识静态工厂的工厂方法（工厂方法是静态的）。

```html
<bean id="staticHelloFactory" class="xyz.coolblog.service.StaticHelloFactory" factory-method="getHello"/>
```

lookup-method

我们通过 BeanFactory getBean 方法获取 bean 实例时，对于 singleton类型的 bean ， BeanFactory 每次返回的都是同一个 bean 。BeanFactory 在实例化 singleton 类型的 bean 时，会向其注入一个 prototype 类型的实例。但是 singleton 类型的 bean 只会实例化一次，那么它内部的 prototype 类型的成员变量也就不会再被改变。

一个 singleton 类型的 bean 中有一个 prototype 类型的成员变量。如果我们每次从 singleton bean 中获取这个 prototype 成员变量时，都想获取一个新的对象。

Spring 会在运行时对 NewsProvider 进行增强，使其getNews 可以每次都返回一个新的实例。

depends-on

当一个 bean 直接依赖另一个 bean ，可以使用 <ref/> 标签进行配置。不过如某个 bean 并不直接依赖于其他 bean ，但又需要其他 bean 先实例化好，这个时候就需要使用 depends-on 特性了。

BeanPostProcessor

BeanPostProcessor 是 bean 实例化时的后置处理器，包含两个方法。

BeanPostProcessor 是 Spring 框架的一个扩展点，通过实现 BeanPostProcessor 接口，我们就可插手 bean 实例化的过程。比如大家熟悉的 AOP 就是在 bean 实例后期间将切面逻辑织入 bean 实例中的， AOP 也正是通过 BeanPostProcessor 和 IOC 容器建立起了联系。

### 5.从缓存中获取 bean 实例

对于单例 bean ， Spring 容器只会实例化一次。后续再次获取时，只需直接从缓存里获取即可，无需且不能再次实例化（否则单例就没意义了）。从缓存中取 bean 实例的方法是getSingleton(String)。

### 6.从FactoryBean中获取 bean 实例

- 检测参数 beanInstance 的类型，如果是非 FactoryBean 类型的 bean ，直接返回
- 检测 FactoryBean 实现类是否单例类型，针对单例和非单例类型进行不同处理
- 对于单例 FactoryBean ，先从缓存里获取 FactoryBean 生成的实例
- 若缓存未命中，则调用 FactoryBean.getObject() 方法生成实例，并放入缓存中
- 对于非单例的 FactoryBean，每次直接创建新的实例即可，无需缓存
- 如果 shouldPostProcess = true ，不管是单例还是非单例 FactoryBean 生成的实例，都要进行后置处理

### 7.IOC 容器的使用过程

- 获取资源
- 获取 BeanFactory
- 根据新建的 BeanFactory 创建一个BeanDefinitionReader对象，该Reader 对象为资源的解析器
- 装载资源

### 8.BeanDefinition

BeanDefinition 是一个接口，它描述了一个 Bean 实例，包括属性值、构造方法值和继承自它的类的更多信息。它继承 AttributeAccessor 和 BeanMetadataElement 接口。两个接口定义如下：

- AttributeAccessor ：定义了与其它对象的（元数据）进行连接和访问的约定，即对属性的修改，包括获取、设置、删除。
- BeanMetadataElement：Bean 元对象持有的配置元素可以通过getSource() 方法来获取。

### 9.spring 中有多少种IOC 容器？

BeanFactory - BeanFactory 就像一个包含bean 集合的工厂类。它会在客户端要求时实例化bean。 ApplicationContext - ApplicationContext 接口扩展了BeanFactory 接口。它在BeanFactory 基础上提供了一些额外的功能。

## AOP

### 1.Spring AOP简介

"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

使用"横切"技术，AOP把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。

AOP主要应用场景有：

- Authentication ：权限统一管理和授权
- Caching ：缓存统一维护
- Context Passing ：内容传递
- Error Handling ：系统统一错误处理
- Lazy Loading ：数据懒加载
- Debugging ：系统调试
- logging, tracing, profiling and monitoring：记录跟踪优化校准
- Performance Optimization ：性能优化
- Persistence：持久化
- Resource Pooling：资源池统一管理和申请
- Synchronization ：操作同步
- Transactions：统一事务管理

### 2.AOP的核心概念

- 横切关注点：定义对哪些方法进行拦截，拦截后执行哪些操作，这些关注点称之为横切关注点。
- 切面（Aspect）：横切关注点的抽象。
- 连接点（Joinpoint）：在Spring中，连接点指被拦截到的方法，但是从广义上来说，连接点还可以是字段或者构造器。
- 切入点（Pointcut）：对连接点进行拦截的定义。
- 通知（Advice)：拦截到连接点之后要执行的具体操作，通知分为前置通知、后置通知、成功通知、异常通知和环绕通知5类。
- 目标对象：代理的目标对象。
- 织入（Weave）：将切面应用到目标对象并执行代理对象创建的过程。
- 引入（Introduction）： 在运行期为类动态地添加一些方法或字段而不用修改类的代码。

### 3.AOP的2种代理方式

Spring提供了JDK和CGLib2种方式来生成代理对象，具体生成代理对象的方式由AopProxyFactory根据AdvisedSupport对象的配置来决定。Spring默认的代理对象生成策略为：如果是目标类接口，则使用JDK动态代理技术，否则使用CGLib动态代理技术。

- JDK动态代理：JDK动态代理主要通过java.lang.reflect包中Proxy类和InvocationHandler接口来实现。InvocationHandler是一个接口，不同的实现定义不同的横切逻辑，并通过反射机制调用目标类的代码，动态地将横切逻辑和业务逻辑编制在一起。Proxy类利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象。JDK1.8中Proxy类的定义如下。

```html
public class Proxy implement java.io.Serializable{ 			
	private static final long serialVersionUID= -2222568056686623797L; 
	//1 ：在构造方法参数中定义不同的InvocationHandler实现类
	private static final Class<?>[] constructorParams = { InvocationHandler.class }; 
	//2: Proxy类缓存列表
	private static final WeakCache<ClassLoader, Class<?>[], Class<?>>proxyClassCache =new WeakCache<>(new KeyFactory(), new ProxyClassFactory()); 
	//3 ：当前代理需要调用的Handler实例对象（该对象需要经过序列化〉
	protected InvocationHandler h; 
	／／．．．．此处忽略部分实现
	//4: Proxy类构造函数，参数InvocationHandler为当前代理的对象
	protected Proxy(InvocatonHandler h) { 				
		Objects.requireNonNull(h); 
		this.h = h; 
	}
	.......
}
```

- CGLib动态代理：CGLib即 Code Generation Library ，它是一个高性能的代码生成类库，可以在运行期间扩展Java类和实现Java接口。CGLib包的底层通过字节码处理框架ASM来实现，通过转换字节码生成新的类。
- CGLib动态代理和JDK动态代理的区别：JDK只能为接口创建代理实例，而对于没有通过接口定义业务方法的类，则只能通过CGLib创建动态代理来实现。

### 4.AOP的5种通知类型

- 前置通知：在一个方法执行之前执行通知
- 后置通知：在一个方法执行之后执行通知（无论方法执行成功还是失败通知都会执行）
- 成功通知：在一个方法执行成功之后执行通知（只有在方法执行成功时才执行通知）
- 异常通知：当一个方法执行抛出异常退出时，才执行该通知
- 环绕通知：在拦截方法调用之前和之后，分别执行通知

### 5.实现AOP

实现AOP 的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。

### 6.AOP 有哪些实现方式？

实 现 AOP 的 技 术 ， 主 要 分 为 两 大 类 ： 静态代理 指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强； 􀀀 编译时编织（特殊编译器实现） 􀀀 类加载时编织（特殊的类加载器实现）。 动态代理 在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。 􀀀 JDK 动态代理 􀀀 CGLIB

### 7.Spring AOP and AspectJ AOP 有什么区别？

Spring AOP 基于动态代理方式实现；AspectJ 基于静态代理方式实现。SpringAOP 仅支持方法级别的 PointCut；提供了完全的 AOP 支持， 它还支持属性级别的 PointCut。

## 事务

### 1.请描述一下 Spring 的事务

声明式事务管理的定义：用在Spring 配置文件中声明式的处理事务来代替代码式的处理事务。这样的好处是，事务管理不侵入开发的组件，具体来说，业务逻辑对象就不会意识到正在事务管理之中，事实上也应该如此，因为事务管理是属于系统层面的服务，而不是业务逻辑的一部分，如果想要改变事务管理策划的话，也只需要在定义文件中重新配置即可，这样维护起来极其方便。

声明式事务

基于TransactionInterceptor 的声明式事务管理：两个次要的属性： transactionManager ，用来指定一个事务治理器，并 将具体事务相关的操作请托给它；其他一个是 Properties 类型的transactionAttributes 属性，该属性的每一个键值对中，键指定的是方法名，方法名可以行使通配符，而值就是表现呼应方法的所运用的事务属性。

基于TransactionProxyFactoryBean 的声明式事务管理：设置配置文件与先前比照简化了许多。我们把这类设置配置文件格式称为 Spring 经典的声明式事务治理。

基于<tx> 命名空间的声明式事务治理：在前两种方法的基础上， Spring 2.x 引入了 <tx> 命名空间，连络行使 <aop> 命名空间，带给开发人员设置配备声明式事务的全新体验。

基于@Transactional 的声明式事务管理： Spring 2.x 还引入了基于 Annotation 的体式格式，具体次要触及@Transactional 标注。 @Transactional 可以浸染于接口、接口方法、类和类方法上。算作用于类上时，该类的一切public 方法将都具有该类型的事务属性。

编程式事务

编程式事物管理的定义：在代码中显式挪用beginTransaction() 、 commit() 、 rollback() 等事务治理相关的方法，这就是编程式事务管理。 Spring 对事物的编程式管理有基于底层 API 的编程式管理和基于 TransactionTemplate 的编程式事务管理两种方式。 基于底层API 的编程式管理：凭证 PlatformTransactio nManager 、 TransactionDefinition 和TransactionStatus 三个焦点接口，来实现编程式事务管理。

基于TransactionTemplate 的编程式事务管理 为了不损坏代码原有的条理性，避免出现每一个方法中都包括相同的启动事物、提交、回滚事物样板代码的现象， spring 提供了 transactionTemplate 模板来实现编程式事务管理。

编程式事务与声明式事务的区别

1）编程式事务是自己写事务处理的类，然后调用。 2）声明式事务是在配置文件中配置，一般搭配在框架里面使用。

### 1.spring 的事务传播行为

Spring 在TransactionDefinition 接口中规定了7 种类型的事务传播行为，它们规定了事务方法和事务方法发生嵌套调用时事务如何进行传播： PROPAGATION_REQUIRED：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 PROPAGATION_SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。 PROPAGATION_MANDATORY：使用当前的事务，如果当前没有事务，就抛出异常。 PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。 PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。 PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED 类似的操作。

### 2.Spring 的隔离级别

1、Serializable：最严格的级别，事务串行执行，资源消耗最大； 2、REPEATABLE READ：保证了一个事务不会修改已经由另一个事务读取但未提交（回滚）的数据。避免了“脏读取”和“不可重复读取”的情况，但是带来了更多的性能损失。 3、READ COMMITTED:大多数主流数据库的默认事务等级，保证了一个事务不会读到另一个并行事务已修改但未提交的数据，避免了“脏读取”。该级别适用于大多数系统。 4、Read Uncommitted：保证了读取过程中不会读取到非法数据。

### 3.spring 事务管理的配置方式

1 用TransactionFactoryBean 代理dao 事务处理； 2 用aop:config 声明要进行事务增强的切面，用tx:advice 声明具体方法的事务属性（传播行为，隔离级别，是否可读，抛出异常是否回滚）及应用的事务管理器；

3 用@Transactional 注解配置事务管理；

### 4.什么是事务的超时属性？

所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。

在 TransactionDefinition 中以 `int` 的值来表示超时时间，其单位是秒。

当然，这个属性，貌似我们基本也没用过。

### 5.什么是事务的只读属性？

事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。

- 所谓事务性资源就是指那些被事务管理的资源，比如数据源、JMS 资源，以及自定义的事务性资源等等。
- 如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。

在 TransactionDefinition 中以 `boolean` 类型来表示该事务是否只读。

### 6.什么是事务的回滚规则？

回滚规则，定义了哪些异常会导致事务回滚而哪些不会。

- 默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的）。
- 但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

注意，事务的回滚规则，并不是数据库事务规范中的名词，**而是 Spring 自身所定义的**。

## 面试题

### 1.Spring框架中Bean的生命周期？

1. 通过构造方法实例化 Bean 对象。
2. 通过 setter 方法设置对象的属性。
3. 通过Aware，也就是他的子类BeanNameAware，调用Bean的setBeanName()方法传递Bean的ID(XML里面注册的ID)，setBeanName方法是在bean初始化时调用的，通过这个方法可以得到BeanFactory和 Bean 在 XML 里面注册的ID。
4. 如果说 Bean 实现了 BeanFactoryAware,那么工厂调用setBeanFactory(BeanFactory var1) 传入的参数也是自身。
5. 把 Bean 实例传递给 BeanPostProcessor 中的 postProcessBeforeInitialization 前置方法。
6. 完成 Bean 的初始化
7. 把 Bean 实例传递给 BeanPostProcessor 中的 postProcessAfterInitialization 后置方法。
8. 此时 Bean 已经能够正常时候，在最后的时候调用 DisposableBean 中的 destroy 方法进行销毁处理。

可以从四到五个方面来记忆， 构造实例化 属性赋值 完成初始化 (前后处理) 使用后销毁

### 2.Spring是怎么去解决循环依赖的

Spring循环依赖处理一 (构造器循环依赖)

构造器循环依赖的意思就是说，通过构造及注入构成的循环依赖，而这种依赖的话，是没有办法解决的，如果你敢强行依赖，不要意 思，出现了你久违的异常 BeanCurrentlyInCreationException 出现这个异常的时候，就是表示循环依赖的问题。

在 Spring 的配置文件中，如果这么配置 A ，B ，C 的循环依赖的时候，在创建 A 的时候，发现，构造器需要 B 类，然后去创建 B , 而创建 B 的时候，发现又需要 C ，然后去创建 C ，创建的时候发现，竟然需要 A ，于是又掉头回去了，于是就形成了一个闭环，没 有办法创建。

在这种情况下，Spring实例化bean是通过ApplicationContext.getBean()方法来进行的。如果要获取的对象依赖了另一个对象，那么 其首先会创建当前对象，然后通过递归的调用ApplicationContext.getBean()方法来获取所依赖的对象，最后将获取到的对象注入到当 前对象中。而和刚才阿粉说的一样，创建了闭环，所以就没有办法创建了。

Spring循环依赖处理二（setter循环依赖）

setter循环注入是指通过setter注入方式构成的循环依赖。而这种方式，是Spring可以进行解决的。 而对于这种使用setter注入造成的依赖是通过Spring容器来提前暴露刚完成的构造注入器的bean来完成的，但是这时候还没有完成其 他的步骤的时候。 这个时候我们就需要提前暴露出来一个单例的工厂方法，让其他的bean来引用这个bean

```html
addSingletonFactory(beanName，new ObjectFactory(){
    public Object getObject() throws BeanException{
        return getEarlyBeanReference(beanName,mbd,bean)
    }
})
```

Spring在创建 A 的时候，根据无参构造来创建 A，并且暴露出 ObjectFactory 用来返回一个提前暴露好的 bean 然后再进行setter来注入，

同理的B和C都是这个样子的，这个时候就能完成setter注入了。

Spring循环依赖处理三（作用域循环依赖）

对于 “singleton”作用域的话，他是可以通过“setAllowCircularReference（false）”这种方式来进制循环依赖的。 而且也是有缺陷的，这种方式只能解决单例作用域的bean循环依赖。

三级缓存

第一级缓存singletonObjects里面放置的是实例化好的单例对象。 第二级earlySingletonObjects里面存放的是提前曝光的单例对象（没有完全装配好）。 第三级singletonFactories里面存放的是要被实例化的对象的对象工厂。

### 3.Spring 定时任务

使用Scheduled 注解

在使用 Scheduled 注解的时候，必须设置 cron ()， fixedDelay() ， fixedRate() 三个中的一个属性。该注解使用 的方法不期望接收任何参数，并且要使用 void 返回类型，即使设置了返回值，也会在回调的时候被忽略。我们在使 用 Scheduled 注解的时候需要注册 ScheduledAnnotationBeanPostProcessor ，可以通过手动，或者配置 或者在 Springboot 项 目中，直接在启动类上添加 @EnableScheduling 注解都可以。总结一下就是

1. 设置定时策略；
2. 不设置返回类型；
3. 注册 processor 。

cron() 配置

cron 是一个六位或者七位的字符串，每一位都对应的相应的含义，如上0 0 0 * * ? 表示的是每天 0 点执行。

fixedDelay() 方式

执行完毕后调用，意思是说在上一次执行完毕和下一次开始调用的中间，延迟一段时间，以毫秒为单位。

fixedRate() 方式

开始执行后延迟调用，固定一个时间间隔进行前后方法的调用。

单线程问题

Spring 的任务调用默认使用的是单线程模式，

通过实现 SchedulingConfigurer 接口中的 configureTasks 方法，来创建一个 ScheduledTask 并注入到 ScheduledTaskRegistrar 中。

多实例问题

另外还有一个问题就是如果一个模块定时任务很多，最好可以封装为一个独立的任务调度模块，对于负责场景的任务调度可以采用一 些开源的调度框架，比如 XXL-JOB ， Azkaban 。 如果只是一些简单的定时任务，不需要引入开源框架的时候，那么自己开发后在部署的时候需要注意多实例会重复运行的情况，意思 是说一个定时任务模块，如果部署了两个实例就会导致任务重复执行。这种时候只能采用单实例部署，单实例部署的时候需要加守护 进程和程序监控，避免程序出问题。 如果是在需要高可用，可以自己开发调度模块或者采用开源的框架实现。大部分场景 XXL-JOB 应该都可以满足。

### 4.Bean 标签解析

xml 中配置的属性对应到 document 对象中，然后进行注册，下面来完整描述这个方 法的处理流程： •创建实例 bdHolder ：首先委托 BeanDefinitionParserDelegate 类的 parseBeanDefinitionElement 方法进行元素解析，经过解 析后， bdHolder 实例已经包含刚才我们在配置文件中设定的各种属性，例如 class 、 id 、 name 、 alias 等属性。 •对实例 bdHolder 进行装饰：在这个步骤中，其实是扫描默认标签下的自定义标签，对这些自定义标签进行元素解析，设定自定义 属性。 •注册 bdHolder 信息：解析完成了，需要往容器的 beanDefinitionMap 注册表注册 bean 信息，注册操作委托给了 BeanDefinitionReaderUtils.registerBeanDefinition ，通过工具类完成信息注册。 •发送通知事件：通知相关监听器，表示这个 bean 已经加载完成

### 5.bean加载过程

1. 如果加载的 bean 是单例，要清除缓存
2. 实例化 bean，将 BeanDifinition 转化成 BeanWrapper
3. 后处理器修改合并后的 bean 定义：bean 合并后的处理，Autowired 注解正式通过此方法实现诸如类型的预解析
4. 依赖处理
5. 属性填充：将所有属性填充到 bean 的实例中
6. 循环依赖检查
7. 注册 DisposableBean：这一步是用来处理 destroy-method 属性，在这一步注册，以便在销毁对象时调用。
8. 完成创建并返回。

## SpringBean

### 1.什么是 Spring Bean ？

- Bean 由 Spring IoC 容器实例化，配置，装配和管理。
- Bean 是基于用户提供给 IoC 容器的配置元数据 Bean Definition 创建。

### 2.Spring 有哪些配置方式

单纯从 Spring Framework 提供的方式，一共有三种：

- 1、XML 配置文件。

  Bean 所需的依赖项和服务在 XML 格式的配置文件中指定。这些配置文件通常包含许多 bean 定义和特定于应用程序的配置选项。它们通常以 bean 标签开头。例如：

  ```html
  <bean id="studentBean" class="org.edureka.firstSpring.StudentBean">
      <property name="name" value="Edureka"></property>
  </bean>
  ```

- 2、注解配置。

  您可以通过在相关的类，方法或字段声明上使用注解，将 Bean 配置为组件类本身，而不是使用 XML 来描述 Bean 装配。默认情况下，Spring 容器中未打开注解装配。因此，您需要在使用它之前在 Spring 配置文件中启用它。例如：

  ```html
  <beans>
  <context:annotation-config/>
  <!-- bean definitions go here -->
  </beans>
  ```

- 3、Java Config 配置。

  Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。

  - `@Bean` 注解扮演与 `<bean />` 元素相同的角色。

  - `@Configuration` 类允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。

  - 例如：

    ```html
    @Configuration
    public class StudentConfig {
        
        @Bean
        public StudentBean myStudent() {
            return new StudentBean();
        }
        
    }
    ```

目前主要使用 **Java Config** 配置为主。当然，三种配置方式是可以混合使用的。例如说：

- Dubbo 服务的配置，艿艿喜欢使用 XML 。
- Spring MVC 请求的配置，艿艿喜欢使用 `@RequestMapping` 注解。
- Spring MVC 拦截器的配置，艿艿喜欢 Java Config 配置。

### 3.Spring Bean 在容器的生命周期是什么样的？

Spring Bean 的**初始化**流程如下：

- 实例化 Bean 对象

  - Spring 容器根据配置中的 Bean Definition(定义)中**实例化** Bean 对象。

    > Bean Definition 可以通过 XML，Java 注解或 Java Config 代码提供。

  - Spring 使用依赖注入**填充**所有属性，如 Bean 中所定义的配置。

- Aware 相关的属性，注入到 Bean 对象

  - 如果 Bean 实现 **BeanNameAware** 接口，则工厂通过传递 Bean 的 beanName 来调用 `#setBeanName(String name)` 方法。
  - 如果 Bean 实现 **BeanFactoryAware** 接口，工厂通过传递自身的实例来调用 `#setBeanFactory(BeanFactory beanFactory)` 方法。

- 调用相应的方法，进一步初始化 Bean 对象

  - 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则调用 `#preProcessBeforeInitialization(Object bean, String beanName)` 方法。
  - 如果 Bean 实现 **InitializingBean** 接口，则会调用 `#afterPropertiesSet()` 方法。
  - 如果为 Bean 指定了 **init** 方法（例如 `<bean />` 的 `init-method` 属性），那么将调用该方法。
  - 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则将调用 `#postProcessAfterInitialization(Object bean, String beanName)` 方法。

Spring Bean 的**销毁**流程如下：

- 如果 Bean 实现 **DisposableBean** 接口，当 spring 容器关闭时，会调用 `#destroy()` 方法。
- 如果为 bean 指定了 **destroy** 方法（例如 `<bean />` 的 `destroy-method` 属性），那么将调用该方法。

### 4.什么是 Spring 的内部 bean？

只有将 Bean **仅**用作另一个 Bean 的属性时，才能将 Bean 声明为内部 Bean。

- 为了定义 Bean，Spring 提供基于 XML 的配置元数据在 `<property>`或 `<constructor-arg>` 中提供了 `<bean>`元素的使用。
- 内部 Bean 总是**匿名**的，并且它们总是作为**原型 Prototype** 。

### 5.什么是 Spring 装配？

当 Bean 在 Spring 容器中组合在一起时，它被称为**装配**或 **Bean 装配**。Spring 容器需要知道需要什么 Bean 以及容器应该如何使用依赖注入来将 Bean 绑定在一起，同时装配 Bean 。

### 6.**自动装配有什么局限？**

- 覆盖的可能性 - 您始终可以使用 `<constructor-arg>` 和 `<property>` 设置指定依赖项，这将覆盖自动装配。

- 基本元数据类型 - 简单属性（如原数据类型，字符串和类）无法自动装配。

  > 这种，严格来说，也不能称为局限。因为可以通过配置文件来解决。

- 令人困惑的性质 - 总是喜欢使用明确的装配，因为自动装配不太精确。

## Spring注解

### 1.什么是基于注解的容器配置？

不使用 XML 来描述 Bean 装配，开发人员通过在相关的类，方法或字段声明上使用**注解**将配置移动到组件类本身。它可以作为 XML 设置的替代方案。例如：

Spring 的 Java 配置是通过使用 `@Bean` 和 `@Configuration` 来实现。

- `@Bean` 注解，扮演与 `<bean />` 元素相同的角色。
- `@Configuration` 注解的类，允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。

### 2.如何在 Spring 中启动注解装配？

默认情况下，Spring 容器中未打开注解装配。因此，要使用基于注解装配，我们必须通过配置 `<context：annotation-config />` 元素在 Spring 配置文件中启用它。

当然，如果胖友是使用 Spring Boot ，默认情况下已经开启。

### 3.@Component, @Controller, @Repository, @Service 有何区别？

- `@Component` ：它将 Java 类标记为 Bean 。它是任何 Spring 管理组件的**通用**构造型。
- `@Controller` ：它将一个类标记为 Spring Web MVC **控制器**。
- `@Service` ：此注解是组件注解的特化。它不会对 `@Component` 注解提供任何其他行为。您可以在**服务层**类中使用 @Service 而不是 `@Component` ，因为它以更好的方式指定了意图。
- `@Repository` ：这个注解是具有类似用途和功能的 `@Component` 注解的特化。它为 **DAO** 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException 。

### 4.@Required 注解有什么用？

`@Required` 注解，应用于 Bean 属性 setter 方法。

- 此注解仅指示必须在配置时使用 Bean 定义中的显式属性值或使用自动装配填充受影响的 Bean 属性。
- 如果尚未填充受影响的 Bean 属性，则容器将抛出 BeanInitializationException 异常。

### 5.@Autowired 注解有什么用？

`@Autowired` 注解，可以更准确地控制应该在何处以及如何进行自动装配。

- 此注解用于在 setter 方法，构造函数，具有任意名称或多个参数的属性或方法上自动装配 Bean。
- 默认情况下，它是类型驱动的注入。

### 6.@Qualifier 注解有什么用？

当你创建多个**相同类型**的 Bean ，并希望仅使用属性装配**其中一个** Bean 时，您可以使用 `@Qualifier` 注解和 `@Autowired` 通过指定 ID 应该装配哪个**确切的** Bean 来消除歧义。

例如，应用中有两个类型为 Employee 的 Bean ID 为 `"emp1"` 和 `"emp2"` ，此处，我们希望 EmployeeAccount Bean 注入 `"emp1"` 对应的 Bean 对象。代码如下：

```html
public class EmployeeAccount {

    @Autowired
    @Qualifier(emp1)
    private Employee emp;

}
```











