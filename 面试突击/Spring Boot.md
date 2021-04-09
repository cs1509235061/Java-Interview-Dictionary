# Spring Boot

### 1.Spring Boot特点

Spring Boot的特点如下。

( l ）快速创建独立的Spring应用程序

( 2 ）嵌入Tomcat和Undertow等Web容器，实现快速部署。

( 3 ）自动配置JAR包依赖和版本控制，简化Maven配置

( 4 ）自动装配Spring实例，不需要XML配置

( 5 ）提供诸如性能指标、健康检查、外部配置等线上监控和配置功能

### 2.Spring Boot 的核心配置文件有哪几个？它们的区别是什么？

- Spring Boot的核心配置文件是 application 和 bootstrap 配置文件。
- application 配置文件这个容易了解，主要用于 Spring Boot 项目的自动化配置。
- bootstrap 配置文件有以下几个应用场景。
  - 使用 Spring Cloud Config 配置中心时，这时需要在 bootstrap 配置文件中增加连接到配置中心的配置属性来加载外部配置中心的配置信息；
  - 少量固定的不能被覆盖的属性；
  - 少量加密/解密的场景；

### 3.Spring Boot 的配置文件有哪几种格式？它们有什么区别？

.properties 和 .yml，它们的区别主要是书写格式不同。 1).properties

```html
app.user.name = javastack
```

2).yml

```html
app:
    user:
        name: javastack
```

另外，.yml 格式不支持@PropertySource注解导入配置。

### 4.Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？

启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下3 个注解： @SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。 @EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能：@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。 @ComponentScan：Spring组件扫描。

### 5.开启 Spring Boot 特性有哪几种方式？

1）继承spring-boot-starter-parent项目 2）导入spring-boot-dependencies项目依赖

### 6.Spring Boot 自动配置原理是什么？

注解 @EnableAutoConfiguration, @Configuration, @ConditionalOnClass 就是自动配置的核心，首先它得是一个配置文件，其次根据类路径下能否有这个类去自动配置。

### 7.如何在 Spring Boot 启动的时候运行少量特定的代码？

可以实现接口 ApplicationRunner 或者者 CommandLineRunner，这两个接口实现方式一样，它们都只提供了一个 run 方法

### 8.Spring Boot 有哪几种读取配置的方式？

Spring Boot 可以通过 @PropertySource,@Value,@Environment, @ConfigurationProperties 来绑定变量。

### 9.SpringBoot 实现热部署有哪几种方式？

主要有两种方式： Spring Loaded Spring-boot-devtools

### 10.你如何了解 Spring Boot 配置加载顺序？

在 Spring Boot 里面，可以使用以下几种方式来加载配置。 1）properties文件； 2）YAML文件； 3）系统环境变量； 4）命令行参数； 等等……

### 11.Spring Boot 可以兼容老 Spring 项目吗，如何做？

可以兼容，使用@ImportResource注解导入老 Spring 项目配置文件。

### 12.保护 Spring Boot 应用有哪些方法？

在生产中使用HTTPS 使用Snyk检查你的依赖关系 更新到最新版本 启用CSRF保护 使用内容安全策略防止XSS攻击...

### 13.Spring Boot 2.X 有什么新特性？与 1.X 有什么区别？

配置变更 JDK 版本更新 第三方类库更新 响应式 Spring 编程支持 HTTP/2 支持 配置属性绑定 更多改进与增强…

### 14.启动类注解

@SpringBootConfiguration:Spring Boot的配置类; 标注在某个类上，表示这是一个Spring Boot的配置类; @Configuration:配置类上来标注这个注解; 配置类 ----- 配置文件;配置类也是容器中的一个组件;@Component@EnableAutoConfiguration:开启自动配置功能; 以前我们需要配置的东西，Spring Boot帮我们自动配置;@EnableAutoConfiguration告诉SpringBoot开启自动配置功能;这样自动配置才能生效; Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就失效，帮我们进行自动配置工作

### 15.配置文件的加载顺序?

由jar包外向jar包内进行寻找; 优先加载带profile jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件 jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件 再来加载不带profile jar包外部的application.properties或application.yml(不带spring.profile)配置文件? jar包内部的application.properties或application.yml(不带spring.profile)配置文件

### 16.自动配置原理?

1)、SpringBoot启动的时候加载主配置类，开启了自动配置功能 @EnableAutoConfiguration 2)、@EnableAutoConfiguration 作用:将 类路径下 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中;每一个这样的 xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中;用他们来做自动配置; 3)、每一个自动配置类进行自动配置功能;根据当前不同的条件判断，决定这个配置类是否生效?一但这个配置类生效;这个配置类就会给容器中添加各种组件;这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的; 5)、所有在配置文件中能配置的属性都是在xxxxProperties类中封装者‘;配置文件能配置什么就可以参照某个功 能对应的这个属性类怎么用好自动配置，

精髓: 1)、SpringBoot启动会加载大量的自动配置类 2)、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类; 3)、我们再来看这个自动配置类中到底配置了哪些组件;(只要我们要用的组件有，我们就不需要再来配置了) 4)、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这 些属性的值;

### 17.运行Spring Boot有哪几种方式

1）打包用命令或者放到容器中运行 2）用 Maven/Gradle 插件运行 3）直接执行 main 方法运行

### 18.如何理解 Spring Boot 中的 Starters？

Starters是什么： Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成Spring及其他技术，而不需要到处找示例代 码和依赖包。如你想使用Spring JPA访问数据库，只要加入springboot-starter-data-jpa启动器依赖就能使用了。Starters包含了许多项目中 需要用到的依赖，它们能快速持续的运行，都是一系列得到支持的管理传递性依赖。 Starters命名： Spring Boot官方的启动器都是以spring-boot-starter-命名的，代表了一个特定的应用类型。第三方的 启动器不能以spring-boot开头命名，它们都被Spring Boot官方保留。一般一个第三方的应该这样命 名，像mybatis的mybatis-spring-boot-starter。

Starters分类：

1. Spring Boot应用类启动器
2. Spring Boot生产启动器
3. Spring Boot技术类启动器

### 19.Spring Boot中的监视器是什么？

Spring boot actuator是spring启动框架中的重要功能之一。Spring boot监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。 有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开 了一组可直接作为HTTP URL访问的REST端点来检查状态

### 20.SpringBoot 实现热部署有哪几种方式

主要有两种方式： Spring Loaded Spring-boot-devtools

### 21.什么是 YAML？

YAML 是一种人类可读的数据序列化语言。它通常用于配置文件。 与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML 文件就更加结构化，而且更少混淆。可以看出 YAML 具有分层配置数据。

### 22.什么是 Swagger？你用 Spring Boot 实现了它吗？

Swagger 广泛用于可视化 API，使用 Swagger UI 为前端开发人员提供在线沙箱。Swagger 是用于生成 RESTful Web 服务的可视化表示的 工具，规范和完整框架实现。它使文档能够以与服务器相同的速度更新。当通过 Swagger 正确定义时，消费者可以使用最少量的实现逻辑 来理解远程服务并与其进行交互。因此，Swagger 消除了调用服务时的猜测。

### 23.什么是 Spring Profiles？

Spring Profiles 允许用户根据配置文件（dev，test，prod 等）来注册 bean。因此，当应用程序在开发中运行时，只有某些 bean 可以加 载，而在 PRODUCTION中，某些其他 bean 可以加载。假设我们的要求是 Swagger 文档仅适用于 QA 环境，并且禁用所有其他文档。这可 以使用配置文件来完成。Spring Boot 使得使用配置文件非常简单。

### 24.什么是 Spring Batch？

Spring Boot Batch 提供可重用的函数，这些函数在处理大量记录时非常重要，包括日志/跟踪，事务管理，作业处理统计信息，作业重新启 动，跳过和资源管理。它还提供了更先进的技术服务和功能，通过优化和分区技术，可以实现极高批量和高性能批处理作业。简单以及复杂 的大批量批处理作业可以高度可扩展的方式利用框架处理重要大量的信息。

### 25.Spring Boot 如何定义多套不同环境配置？

提供多套配置文件，如：

```html
applcation.properties
application-dev.properties
application-test.properties
application-prod.properties
```

运行时指定具体的配置文件

### 26.如何重新加载Spring Boot上的更改，而无需重新启动服务器？

这可以使用DEV工具来实现。通过这种依赖关系，您可以节省任何更改，嵌入式tomcat将重新启动。 Spring Boot有一个开发工具（DevTools）模块，它有助于提高开发人员的生产力。Java开发人员面临的一个主要挑战是将文件更改自动部 署到服务器并自动重启服务器。 开发人员可以重新加载Spring Boot上的更改，而无需重新启动服务器。这将消除每次手动部署更改的需要。Spring Boot在发布它的第一个 版本时没有这个功能。 这是开发人员最需要的功能。DevTools模块完全满足开发人员的需求。该模块将在生产环境中被禁用。它还提供H2数据库控制台以更好地 测试应用程序。 org.springframework.boot spring-boot-devtools true

### 27.如何使用 SpringBoot 自动重装我的应用程序？

使用 Spring Boot 开发工具。 把 Spring Boot 开发工具添加进入你的项目是简单的。 把下面的依赖项添加至你的 Spring Boot Project pom.xml 中

```html
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-devtools</artifactId>
<scope>runtime</scope>
</dependency>
```

重启应用程序，然后就可以了。

### 28.发布 Spring Boot 用户应用程序自定义配置的最好方法是什么？

@Value 的问题在于，您可以通过应用程序分配你配置值。更好的操作是采取集中的方法。 你可以使用 @ConfigurationProperties 定义一个配置组件。

```html
@Component
@ConfigurationProperties("basic")
public class BasicConfiguration {
private boolean value;
private String message;
private int number;
```

你可以在 application.properties 中配置参数。

```html
basic.value: true
basic.message: Dynamic Message
basic.number: 100
```

### 29.如何禁用特定的自动配置？

如果我们要禁用特定的自动配置，我们可以使用@EnableAutoConfiguration 注解的exclude 属性来指示它。如下禁用了数据源自动配置 DataSourceAutoConfiguration

如果我们使用@SpringBootApplication 注解。 它具有@EnableAutoConfiguration 作为元注解 - 我们同样可以配置exclude属性来禁用 自动配置

我们还可以使用spring.autoconfigure.exclude 环境属性禁用自动配置。在application.properties (也可以是application.yml )配 置文件设置如下也可以达到同样的目的

### 30.Spring boot支持哪些外部配置？

Spring Boot支持外部配置，允许我们在各种环境中运行相同的应用程序。我们可以使用properties 文件， YAML文件，环境变量，系统属 性和命令行选项参数来指定配置属性。 然后，我们可以访问使用这些属性@Value注释，经由绑定对象 的@ConfigurationProperties 注 释或Environment 环境抽象类注入。 以下是最常见的外部配置来源： 命令行属性：命令行选项参数是以双连字符开头的程序参数，例如-server.port = 8080 。Spring Boot将所有参数转换为属性，并 将它们添加到环境属性集中。 应用程序属性：应用程序属性是从application.properties 文件或其YAML 对应文件加载的属性。默认情况下，Spring Boot会在当 前目录，类路径根或其config 子目录中搜索此文件。 特定于配置文件的属性：特定于配置文件的属性从 application- {profile} .properties 文件或其YAML 对应文件加载。{profile} 占位符是指活性轮廓。这些文件与非特定属性文件位于相同位置，并且优先 于非特定属性文件。

### 31.如何对Spring Boot应用进行测试？

在为Spring应用程序运行集成测试时，我们必须有一个ApplicationContext 。 为了简化测试，Spring Boot为测试提供了一个特殊的注释 @SpringBootTest 。此批注从其classes 属性指示的配置类创建ApplicationContext 。 如果未设置classes属性，Spring Boot将搜索主 配置类。搜索从包含测试的包开始，直到找到使用@SpringBootApplication或@SpringBootConfiguration注释的类。 请注意，如果我们使 用JUnit 4 ，我们必须用@RunWith（SpringRunner.class） 装饰测试类。

### 32.SpringBoot

不用Spring Boot的痛苦是什么？

（1）各种技术整合在一起，版本混乱，大量依赖自己去找，依赖冲突。

（2）基于xml格式的配置文件，对各种技术框架进行大量的繁琐配置，mvc-servlet.xml，applicationContext.xml，mybatis-config.xml，web.xml。

（3）web系统跑起来测一下，需要与tomcat等web容器整合起来才能测试。

（4）单元测试的时候需要自己去选择和导入需要的各种测试组件的依赖，junit，hamcrest，mockito，很多组件。

（5）部署打包的时候需要自己去配置打包插件。

（6）部署应用上线之后，没法去对线上的应用，包括jvm堆栈等方方面面进行监控，没有方便的办法去看到这些东西。

传统的以spring为核心的web系统开发，从启动项目、开发、测试、部署以及监控，都很麻烦，有大量需要手工做的事情 。

用了Spring Boot以后的好处是什么？

类似于一个封装在各种技术之上的一个基础框架，基础模板。脚手架帮助我们快速整合需要使用的技术框架，快速开发、测试以及部署和监控，节约我们的成本。

（1）spring boot负责统一各个依赖的版本，保证各种技术的版本之间兼容，自动引入需要的各种依赖。spring boot 1.5.9，在这个版本基础之上，你引入的spring、mybatis、spring mvc、redis、zookeeper、kafka、mongodb，等等各种技术，在spring boot1.5.9这个大版本的基础之上，其实所有技术的版本都是互相兼容的，省去了我们自己去寻找版本整合，解决不兼容问题的一个过程。

（2）所有技术整合进来之后，不需要xml配置，spring boot全部是大量基于按照约定的自动配置，自动生成那些技术相关的一些bean，注入spring容器供使用，基于注解进行少量注释，基于application.properties，少量的配置即可。

（3）spring boot支持内嵌的web容器，上来直接启动一个main方法就可以启动一个内嵌的tomcat web容器+web程序，快速上手测试，http://localhost:8080/。

（4）一键引入需要的所有单元测试组件依赖，所有测试组件的版本兼容，支持controller、service、dao各种测试。

（5）默认声明一个插件，自己给你把插件配置好了，支持打包成可以执行的jar包或者是war包。

（6）系统上线之后，默认支持大量的线上应用的监控metrics，可以看到线上应用的jvm堆栈，等等信息。

### 33.启动失败的处理

如果启动失败了，那么失败的异常会交给spring boot预先注册好的某个FailureAnalyzer来处理，这个FailureAnalyzer就会打印出完整的失败原因以及解决的办法。

spring boot内置了多个FailureAnalyzer，分别用于处理不同的启动失败问题。

### 34.ApplicationRunner / CommandLineRunner

如果要在SpringApplication.run()开始运行，但是完成应用启动之前，同时并行运行某些代码，比如一些系统初始化的工作，那么可以用ApplicationRunner / CommandLineRunner，两者唯一的区别，就是ApplicationRunner会传递进来ApplicationArguments，CommandLineRunner会传递进来数组。

```html
import org.springframework.boot.*
import org.springframework.stereotype.*
@Component
public class MyBean implements CommandLineRunner {
    public void run(String... args) {
        // Do something...
    }
}
```

### 35.spring boot为spring mvc做的auto configuration

（1）自动注册了ContentNegotiatingViewResolver和BeanNameViewResolver两个bean

（2）支持处理静态资源

（3）自动注册了Converter、GenericConverter、Formatter几个bean

（4）支持HttpMessageConverters

（5）自动注册了MessageCodesResolver

（6）静态index.html支持

（7）自定义Favicon图标支持

（8）自动使用了ConfigurableWebBindingInitializer bean

### 36.Spring Boot对请求校验的支持

Spring Boot支持JSR-303验证框架，默认实现是Hibernate Validator，只要在Java Bean上放一些校验注解，就可以实现校验支持 。

常用的校验注解包括下面这些：

（1）空检查：@Null，@NotNull，@NotBlank，@NotEmpty。

（2）长度检查：@Size(min=20, max=50)，@Length。

（3）数值检查：@Min，@Max，@Range(min=1, max=99)。

（4）其他检查：@Email，@Pattern。

而且这个校验是支持group的概念的，对于不同的group生效的校验不一样。这个很有用，因为对于增删改查等不同的操作，需要执行的校验本来就是不一样的。

### 37.Spring Boot的基础配置

在application.properties中即可完成spring boot的配置。

（1）监听端口配置

默认的监听端口是8080，但是可以用如下三种方式来修改监听的端口。

在application.properties中：server.port=9090

启动系统的时候：java -jar target\springboot-demo-1.0.0.jar --server.port=9090

启动系统的时候：java -Dserver.port=9090 -jar target\springboot-demo-1.0.0.jar

1.2 web上下文配置

默认的web上下文是：/，可以通过属性来修改web上下文。

server.context-path=/springboot-demo

1.3 使用其他web服务器

默认的web服务器是用的内嵌的tomcat，可以使用jetty或者是undertow。

比如使用jetty作为web服务器

```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

但是一般来说，都是用tomcat作为web容器即可，较为重要的配置参数如下

```html
# 打开tomcat访问日志server.tomcat.accesslog.enabled=true# 访问日志所在的目录server.tomcat.accesslog.directory=logs# 允许HTTP请求缓存到请求队列的最大个数，默认是不限制的server.tomcat.accept-count=# 最大连接数，默认是不限制的，如果连接数达到了上限，那么剩下的连接就会保存到请求缓存队列里，也就是上面参数指定的个数server.tomcat.max-connections=# 最大工作线程数server.tomcat.max-threads=# HTTP POST内容最大长度，默认不限制server.tomcat.max-http-post-size=
```

### 38.spring boot多环境支持

spring boot支持使用@Profile注解来标志，在哪个环境profile下，可以激活使用某个@Configuration类。

```html
@Configuration
@Profile("production")
public class ProductionConfiguration {
    // ...
} 
```

接着可以在启动的时候，命令行中，使用如下参数来指定某个环境profile：--spring.profiles.active=dev,hsqldb。

但是实际上如果基于spring boot的profile支持来做，一般是没法完全满足我们的期望的，所以多环境profile支持，通常还是基于彻底的maven profile来使用，不同的profile直接对应不同的文件夹，然后mvn pakcage打包的时候，指定对应的profile来打包，将对应环境的配置文件，全部放到src/main/resources下面去。

这种方式是最彻底的，而且足够灵活。

### 39.**bootstrap.yml和application.yml有什么区别?**

1、Spring Cloud 构建于 Spring Boot 之上，在 Spring Boot 中有两种上下文，一种是 bootstrap，另外一种是 application。 2、application 配置文件这个容易理解，主要用于 Spring Boot 项目的`自动化配置`。 3、bootstrap 是应用程序的父上下文，也就是说 `bootstrap 加载优先于 applicaton`。 4、bootstrap 主要用于从`额外的资源来加载配置信息`，还可以在本地外部配置文件中解密属性。 5、这两个上下文`共用一个环境`，它是任何Spring应用程序的外部属性的来源。 6、bootstrap 里面的属性会`优先加载`，它们默认也不能被本地相同配置覆盖。 7、boostrap 由父 ApplicationContext 加载，`比 applicaton 优先加载` 8、boostrap 里面的属性`不能被覆盖`

### 40.**Spring Boot 配置加载顺序?**

1、properties文件

2、YAML文件

3、系统环境变量

4、命令行参数

### 41.**Spring Boot 有哪几种读取配置的方式？**

- `@PropertySource`
- `@Value`
- `@Environment`
- `@ConfigurationPropertie`

### 42.**Spring Boot监听器流程?**

1、通过`app.addListeners`注册进入 2、初始化一个`SpringApplicationRunListeners`进行处理 3、从`spring.factories`中读取监听器处理类`EventPublishingRunListener` 4、通过`createSpringFactoriesInstances`创建监听器处理类实例 5、调用监听器`listeners.starting()`的方法来启动。 6、底层把事件处理交给`线程池`去处理

### 43.**Spring Boot初始化环境变量流程?**

1、调用`prepareEnvironment`方法去设置环境变量 2、接下来有三个方法`getOrCreateEnvironment`，`configureEnvironment`，`environmentPrepared` 3、`getOrCreateEnvironment`去初始化系统环境变量 4、`configureEnvironment`去初始化命令行参数 5、`environmentPrepared`当广播到来的时候调用`onApplicationEnvironmentPreparedEvent`方法去使用`postProcessEnvironment`方法`load yml`和`properties变量`

### 44.**Spring Boot扫描流程?**

1、调用run方法中的`refreshContext`方法 2、用AbstractApplicationContext中的`refresh`方法 3、委托给`invokeBeanFactoryPostProcessors`去处理调用链 4、其中一个方法`postProcessBeanDefinitionRegistry会`去调用`processConfigBeanDefinitions`解析`beandefinitions` 5、在`processConfigBeanDefinitions`中有一个`parse`方法，其中有`componentScanParser.parse`的方法，这个方法会扫描当前路径下所有`Component`组件

### 45.如何统一引入 Spring Boot 版本？

**目前有两种方式**。

① 方式一：继承 `spring-boot-starter-parent` 项目。

```html
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.1.RELEASE</version>
</parent>
```

② 方式二：导入 spring-boot-dependencies 项目依赖。

```html
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**如何选择？**

因为一般我们的项目中，都有项目自己的 Maven parent 项目，所以【方式一】显然会存在冲突。所以实际场景下，推荐使用【方式二】。

### 46.Spring Boot 默认配置文件是什么？

对于 Spring Boot 应用，默认的配置文件根目录下的 **application** 配置文件，当然可以是 Properties 格式，也可以是 YAML 格式。

还有一个根目录下的 **bootstrap** 配置文件。这个是 Spring Cloud 新增的启动配置文件，[需要引入 `spring-cloud-context` 依赖后，才会进行加载](https://my.oschina.net/freeskyjs/blog/1843048)。它的特点和用途主要是：

- 【特点】因为 bootstrap 由父 ApplicationContext 加载，比 application 优先加载。
- 【特点】因为 bootstrap 优先于 application 加载，所以不会被它覆盖。
- 【用途】使用配置中心 Spring Cloud Config 时，需要在 bootstrap 中配置配置中心的地址，从而实现父 ApplicationContext 加载时，从配置中心拉取相应的配置到应用中。

### 47.Spring Boot 配置加载顺序？

1. `spring-boot-devtools` 依赖的 `spring-boot-devtools.properties` 配置文件。
2. 单元测试上的 `@TestPropertySource` 和 `@SpringBootTest` 注解指定的参数。前者的优先级高于后者。
3. 命令行指定的参数。例如 `java -jar springboot.jar --server.port=9090` 。
4. 命令行中的 `spring.application.json` 指定参数。例如 `java -Dspring.application.json='{"name":"Java"}' -jar springboot.jar` 。
5. ServletConfig 初始化参数。
6. ServletContext 初始化参数。
7. JNDI 参数。例如 `java:comp/env` 。
8. Java 系统变量，即 `System#getProperties()` 方法对应的。
9. 操作系统环境变量。
10. RandomValuePropertySource 配置的 `random.*` 属性对应的值。
11. Jar **外部**的带指定 profile 的 application 配置文件。例如 `application-{profile}.yaml` 。
12. Jar **内部**的带指定 profile 的 application 配置文件。例如 `application-{profile}.yaml` 。
13. Jar **外部** application 配置文件。例如 `application.yaml` 。
14. Jar **内部** application 配置文件。例如 `application.yaml` 。
15. 在自定义的 `@Configuration` 类中定于的 `@PropertySource` 。
16. 启动的 main 方法中，定义的默认配置。即通过 `SpringApplication#setDefaultProperties(Map<String, Object> defaultProperties)` 方法进行设置。

### 48.Spring Boot 有哪些配置方式？

和 Spring 一样，一共提供了三种方式。

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

    - 是不是很熟悉 😈

目前主要使用 **Java Config** 配置为主。当然，三种配置方式是可以混合使用的。例如说：

- Dubbo 服务的配置，艿艿喜欢使用 XML 。
- Spring MVC 请求的配置，艿艿喜欢使用 `@RequestMapping` 注解。
- Spring MVC 拦截器的配置，艿艿喜欢 Java Config 配置。

### 49.Spring Boot 有哪几种读取配置的方式？

Spring Boot 目前支持 **2** 种读取配置：

1. `@Value` 注解，读取配置到属性。最最最常用。另外，支持和 `@PropertySource` 注解一起使用，指定使用的配置文件。
2. `@ConfigurationProperties` 注解，读取配置到类上。另外，支持和 `@PropertySource` 注解一起使用，指定使用的配置文件。

### 50.如下⽅式会使@Async失效

异步⽅法使⽤static修饰 异步类没有使⽤@Component注解（或其他注解）导致spring⽆法扫描到异步类 异步⽅法不能与被调⽤的异步⽅法在同⼀个类中 类中需要使⽤@Autowired或@Resource等注解⾃动注⼊，不能⾃⼰⼿动new对象 如果使⽤SpringBoot框架必须在启动类中增加@EnableAsync注解

### 51.SpringApplication执⾏流程

1） 如果我们使⽤的是SpringApplication的静态run⽅法，那么，这个⽅法⾥⾯⾸先要创建⼀个SpringApplication对象实例， 然后调⽤这个创建好的SpringApplication的实例⽅法。在SpringApplication实例初始化的时候，它会提前做⼏件事情： 根据classpath⾥⾯是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext） 来决定是否应该创建⼀个为Web应⽤使⽤的ApplicationContext类型。 使⽤SpringFactoriesLoader在应⽤的classpath中查找并加载所有可⽤的ApplicationContextInitializer。 使⽤SpringFactoriesLoader在应⽤的classpath中查找并加载所有可⽤的ApplicationListener。 推断并设置main⽅法的定义类。

2） SpringApplication实例初始化完成并且完成设置后，就开始执⾏run⽅法的逻辑了，⽅法执⾏伊始，⾸先遍历执⾏所有通过 SpringFactoriesLoader可以查找到并加载的SpringApplicationRunListener。调⽤它们的started()⽅法，告诉这些 SpringApplicationRunListener，“嘿，SpringBoot应⽤要开始执⾏咯！”。 3） 创建并配置当前Spring Boot应⽤将要使⽤的Environment（包括配置要使⽤的PropertySource以及Profile）。 4） 遍历调⽤所有SpringApplicationRunListener的environmentPrepared()的⽅法，告诉他们：“当前SpringBoot应⽤使⽤ 的Environment准备好了咯！”。 5） 如果SpringApplication的showBanner属性被设置为true，则打印banner。 6） 根据⽤⼾是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应⽤创建什 么类型的ApplicationContext并创建完成，然后根据条件决定是否添加ShutdownHook，决定是否使⽤⾃定义的 BeanNameGenerator，决定是否使⽤⾃定义的ResourceLoader，当然，最重要的，将之前准备好的Environment设置给创建 好的ApplicationContext使⽤。 7） ApplicationContext创建好之后，SpringApplication会再次借助Spring-FactoriesLoader，查找并加载classpath中所有可 ⽤的ApplicationContext-Initializer，然后遍历调⽤这些ApplicationContextInitializer的initialize（applicationContext）⽅ 法来对已经创建好的ApplicationContext进⾏进⼀步的处理。

8） 遍历调⽤所有SpringApplicationRunListener的contextPrepared()⽅法。 9） 最核⼼的⼀步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的 ApplicationContext。 10） 遍历调⽤所有SpringApplicationRunListener的contextLoaded()⽅法。 11） 调⽤ApplicationContext的refresh()⽅法，完成IoC容器可⽤的最后⼀道⼯序。 12） 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执⾏它们。 13） 正常情况下，遍历执⾏SpringApplicationRunListener的finished()⽅法、（如果整个过程出现异常，则依然调⽤所有 SpringApplicationRunListener的finished()⽅法，只不过这种情况下会将异常信息⼀并传⼊处理）