# Shiro

### 1.简单介绍一下 Shiro框架

Apache Shiro是 Java 的一个安全框架 。使用 shiro 可以非常容易的开发出足够好的应用，其不仅可以用在 JavaSE环境，也可以用在 JavaEE 环境。 Shiro 可以帮助我们完成：认证、授权、加密、会话管理、与 Web 集成、缓存等。

三个核心组件：Subject, SecurityManager 和 Realms。

Subject：即当前操作用户 。但是，在 Shiro 中， Subject 这一概念并不仅仅指人，也可以是第三方进程、后台帐户（ Daemon Account ）或其他类似事物。它仅仅意味着 当前跟软件交互的东西 。但考虑到大多数目的和用途，你可以把它认为是 Shiro 的 用户概念。 Subject 代表了当前用户的安全操作， SecurityManager 则管理所有用户的安全操作。 SecurityManager ：它是 Shiro 框架的核心，典型的 Facade 模式， Shiro 通过 SecurityManager 来管理内部组件实例，并通过它来提供安全管理的各种服务。 Realm Realm 充当了 Shiro 与应用安全数据间的 桥梁 或者 连接器 。也就是说，当对用户执行认证（登录）和授权（访问控制）验证时， Shiro 会从应用配置的 Realm 中查找用户及其权限信息。

### 2.Shiro 主要的四个组件

1）SecurityManager 典型的Facade Shiro 通过它对外提供安全管理的各种服务。 2）Authenticator 对“Who are you ？”进行核实。通常涉及用户名和密码。 这个组件负责收集 principals 和 credentials并将它们提交给应用系统。如果提交的 credentials 跟应用系统中提供的 credentials 吻合，就能够继续访问，否则需要重新提交 principals 和 credentials 或者直接终止访问。 3）Authorizer 身份份验证通过后，由这个组件对登录人员进行访问控制的筛查，比如“who can do what 或者“ whocan do which actions ”。 Shiro 采用“基于 Realm ”的方法，即用户（又称 Subject ）、 用户组、角 色和permission 的聚合体。

4）Session Manager 这个组件保证了异构客户端的访问，配置简单。它是基于POJO/J2SE 的，不跟任何的客户 端或者协议绑定。

### 3.Shiro 运行原理

1、 Application Code: 应用程序代码，就是我们自己的编码，如果在程序中需要进行权限控制，需要调用Subject 的 API 。 2、 Subject: 主体，代表的了当前用户。所有的 Subject 都绑定到 SecurityManager 与 Subject 的所有交互都会委托给 SecurityManager, 可以将 Subject 当成一个 门面，而真正执行者是 SecurityManager 。 3、 SecurityManage: 安全管理器，所有与安全有关的操作都会与 SecurityManager 交互，并且它管理所有的 Subject 。 4、 Realm: 域 shiro 是从 Realm 来获取安全数据（用户，角色 ，权限）。就是说 SecurityManager要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户 身份是否合法；也需要从Realm 得到用户相应的角色 权限进行验证用户是否 能进行操作； 可以把 Realm 看成 DataSource ，即安全数据源 。

### 4.Shiro 的四种权限控制方式

1）url 级别权限控制 2）方法注解权限控制 3）代码级别权限控制 4）页面标签权限控制

### 5.授权实现的流程

（1 ）、什么是粗颗粒和细颗粒权限 对资源类型的管理称为粗颗粒度权限控制，即只控制到菜单、按钮、方法，粗粒度的例子比如：用户具有用户管理的权限，具有导出订单明细的权限。对资源实例的控制称为细颗粒度权限管理，即控制到数据级别的权限，比如：用户只允许修改本部门的员工信息，用户只允许导出自己创建的订单明细。 总结： 粗颗粒权限：针对url 链接的控制。 细颗粒权限：针对数据级别的控制。

比如：查询用户权限。 卫生局可以查询所有用户。 卫生室可以查询本单位的用户。 通常在 service 中编程实现。 （2 ）、粗颗粒和细颗粒如何授权 对于粗颗粒度的授权可以很容易做系统架构级别的功能，即系统功能操作使用统一的粗颗粒度的权限管理。 对于细颗粒度的授权不建议做成系统架构级别的功能，因为对数据级别的控制是系统的业务需求，随着业务需求的变更业务功能变化的可能性很大，建议对数据级别的权限控制在业务层个性化开发，比如：用户只允许修改自己创建的商品信息可以在 service 接口添加校验实现， service 接口需 要传入当前操作人的标识，与商品信息创建人标识对比，不一致则不允许修改商品信息。 粗颗粒权限：可以使用过虑器统一拦截url 。 细颗粒权限：在service 中控制，在程序级别来控制，个性化 编程。

### 6.Shiro 能做什么呢？

验证用户身份 用户访问权限控制，比如：1、判断用户是否分配了一定的安全角色。2、判断用户是否被授予完成某个操作的权限 在非 Web 或 EJB 容器的环境下可以任意使用 Session API 可以响应认证、访问控制，或者 Session 生命周期中发生的事件 可将一个或以上用户安全数据源数据组合成一个复合的用户 "view"(视图) 支持单点登录(SSO)功能 支持提供“Remember Me”服务，获取用户关联信息而无需登录

### 7.Shiro 功能架构

Authentication（认证）, Authorization（授权）, Session Management（会话管理）, Cryptography（加密）被 Shiro 框架的开发团队 称之为应用安全的四大基石。那么就让我们来看看它们吧： Authentication（认证）：用户身份识别，通常被称为用户“登录” Authorization（授权）：访问控制。比如某个用户是否具有某个操作的使用权限。 Session Management（会话管理）：特定于用户的会话管理,甚至在非web 或 EJB 应用程序。 Cryptography（加密）：在对数据源使用加密算法加密的同时，保证易于使用。

注意： Shiro 不会去维护用户、维护权限，这些需要我们自己去设计/提供，然后通过相应的接口注入给 Shiro

High-Level Overview 高级概述

在概念层，Shiro 架构包含三个主要的理念：Subject，SecurityManager和 Realm。

Subject：当前用户，Subject 可以是一个人，但也可以是第三方服务、守护进程帐户、时钟守护任务或者其它--当前和软件交互的任何事件。 SecurityManager：管理所有Subject，SecurityManager 是 Shiro 架构的核心，配合内部安全组件共同组成安全伞。 Realms：用于进行权限信息的验证，我们自己实现。Realm 本质上是一个特定的安全 DAO：它封装与数据源连接的细节，得到 Shiro 所需的相关的数据。在配置 Shiro 的时候，你必须指定至少一个Realm 来实现认证（authentication）和/或授权（authorization）。

我们需要实现Realms的Authentication 和 Authorization。其中 Authentication 是用来验证用户身份，Authorization 是授权访问控 制，用于对用户进行的操作授权，证明该用户是否允许进行当前操作，如访问某个链接，某个资源文件等。

登录认证实现

在认证、授权内部实现机制中都有提到，最终处理都将交给Real进行处理。因为在 Shiro 中，最终是通过 Realm 来获取应用程序中的 用户、角色及权限信息的。通常情况下，在 Realm 中会直接从我们的数据源中获取 Shiro 需要的验证信息。可以说，Realm 是专用于 安全框架的 DAO. Shiro 的认证过程最终会交由 Realm 执行，这时会调用 Realm 的 getAuthenticationInfo(token) 方法。 该方法主要执行以下操作: 1、检查提交的进行认证的令牌信息 2、根据令牌信息从数据源(通常为数据库)中获取用户信息 3、对用户信息进行匹配验证。 4、验证通过将返回一个封装了用户信息的 AuthenticationInfo 实例。 5、验证失败则抛出 AuthenticationException 异常信息。

而在我们的应用程序中要做的就是自定义一个 Realm 类，继承AuthorizingRealm 抽象类，重载 doGetAuthenticationInfo()，重写获取 用户信息的方法。



