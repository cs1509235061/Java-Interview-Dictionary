# Maven

### 1.设置MAVEN_OPTS环境变量

maven也是用java写出来的项目，也是要启动jvm来运行maven代码，进而执行各种操作的。因此maven自身的jvm内存大小也是要关注的，如果在构建特别大的项目时，可能会出现maven自身jvm内存不够，导致构建失败，比如OOM的异常。

设置MAVEN_OPTS环境变量，就是设置maven的jvm参数，可以设置为-Xms128m -Xmx512m。

### 2.设置maven的配置文件位置

maven有一个重要的配置文件，就是settings.xml，这个文件默认是在%M2_HOME%/conf目录下面的，但是如果你升级maven的版本，那么可能导致新的安装包的settings文件又是一片空白。

所以一般maven的配置文件都会放在当前用户的目录下的~/.m2/settings.xml中，这样就是对当前用户有效。

### 3.使用阿里云的镜像仓库

为了加快速度，在settings.xml中加一段配置，用国内阿里云的镜像仓库去下载各种东西

```html
<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>*</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 4.pom.xml

pom.xml文件是一个项目最核心的maven配置文件，包含了大量的信息，maven正是基于这里的配置信息来对工程进行构建等管理工作。

<project>：pom.xml中的顶层元素。

<modelVersion>：POM本身的版本号，一般很少变化。

<groupId>：创建这个项目的公司或者组织，一般用公司网站后缀，比如com.company，或者cn.company，或者org.zhonghuashishan。

<artifactId>：这个项目的唯一标识，一般生成的jar包名称，会是<artifactId>-<version>.<extension>这个格式，比如说myapp-1.0.jar。

<packaging>：要用的打包类型，比如jar，war，等等。

<version>：这个项目的版本号。

<name>：这个项目用于展示的名称，一般在生成文档的时候使用。

<url>：这是这个项目的文档能下载的站点url，一般用于生成文档。

<description>：用于项目的描述。

### 5.mvn构建命令

1、maven一定会去考虑settings.xml配置文件里的一些配置。

2、maven会去解析你的maven工程的pom.xml。

3、maven会去看你的pom.xml里面声明了哪些依赖。

4、maven会去本地的仓库里去找有没有那些依赖，比如有没有junit。

5、如果本地仓库没有junit，那么就会去远程仓库去找，下载junit。所谓的远程仓库里包含了几乎所有的依赖包，~/.m2/repository，本地仓库。

6、远程仓库下载到了junit以后，就会放到本地仓库，缓存起来，供你以后去使用，到%M2_HOME%/lib下面找，maven-model-builder-3.5.2.jar，https://repo.maven.apache.org/maven2/，maven的远程中央仓库。

### 6.maven坐标的介绍

每个maven项目都有一个坐标。

groupId + artifactId + version + packaging + classifier，五个维度的坐标，唯一定位一个依赖包。

任何一个项目，都是用这五个维度唯一定位一个发布包。

实际上后面两个维度较为少用，99%的场景下，唯一定位一个依赖的就是三个维度，groupId + artifactId + version。

几乎所有你需要使用的依赖包的各个版本，都在maven的中央仓库里。

简单来说，就是在pom.xml里配置你需要的依赖的坐标，然后maven自动从中央仓库下载后缓存到你本地仓库里。

### 7.企业级的坐标设置

groupId：不是你的公司或者组织，但是以你的公司或者组织的官网的域名倒序来开头，然后加上项目名称。

artifactId：项目中的某个模块，或者某个服务。

version：这个工程的版本号。

packaging：这个工程的发布包打包方式，一般常用的就jar和war两种。war可以放到一个tomcat容器里去跑的web工程。

classifier：很少用，定义某个工程的附属项目。

### 8.设置了坐标的作用是啥？

一个groupId+artifactId+version，就定位了这个项目某个时间点的一个特定版本的代码，也就是一个特定的版本的代码的jar包。

然后，我们在自己的maven项目里，就可以通过<dependency>加上那个依赖的坐标，groupId+artifactId+version，去声明我要用那个依赖的哪个版本的代码的jar包。maven通过这个坐标唯一定位到那个项目的那个版本的代码的jar包。从中央仓库下载下来，给你放到本地仓库里去。

你自己的maven项目为什么也要一个坐标，因为可能你写好了一个模块之后，你就会需要将这个版本的代码打成一个jar包，放到仓库里去，给公司里其他人去依赖和使用。只有你也有一个坐标，才能让别人唯一定位你的项目某个版本的代码的jar包，让maven下载了之后给别人用。

### 9.version

version：1.0.0-SNAPSHOT版本。

第三位是最小版本，一般如果是修复一些bug，或者作出轻微的改动，会累加第三位小版本；

第二位是小版本，一般如果加入一些新的功能或者模块，或者做了一些重构，会累加第二位小版本；

第一位是大版本，一般就是如果整体架构有特别的升级或者变化，才会累加第一位大版本。

SNAPSHOT，就是当前这个版本下的快照版本，代表代码正在开发或者测试中，可以试用，但是没有经过完善测试，不会承诺非常稳定。

如果没有SNAPSHOT，那么就是说已经经过完善测试，是可以发布的稳定版本。

### 10.依赖范围

<scope></scope>

maven有三套classpath，classpath，就是项目中用到的各种依赖的类，jvm在运行的时候需要去classpath下面加载对应的类。

maven在编译源代码的时候有一套classpath；在编译测试代码以及执行测试代码的时候，有一套classpath；运行项目的时候，有一套classpath；依赖范围就是用来控制依赖包与这三种classpath的关系的。

简单来说，不同的依赖范围，会导致那个依赖包可能在编译、测试或者打包运行的时候，有时候可以使用，有时候不能够使用。

compile：默认，对编译、测试和运行的classpath都有效。一般都是用这种scope。

test：仅仅对于运行测试代码的classpath有效，编译或者运行主代码的时候无效，仅仅测试代码需要用的依赖一般都会设置为这个范围，比如junit。一些测试框架，或者只有在测试代码中才会使用的一些依赖，会设置为test，这个的好处在于说，打包的时候这种test scope的依是不会放到最终的发布包里去的。减少发布包的体积。

provided：编译和测试的时候有效，但是在运行的时候无效，因为可能环境已经提供了，比如servlet-api，一般就是这个范围，在运行的时候，servlet容器会提供依赖。servlet-api是用来开发java web项目的，可能你在开发代码和执行单元测试的时候，需要在pom.xml里面声明这个servlet-api的依赖，因为要写代码和测试代码。但是最终打完包之后，放到tomcat容器里面去跑的时候，是不需要将这个servlet-api的依赖包打入发布包中的，因为tomcat容器本身就会给你提供servlet-api的包。

runtime：测试和运行classpath有效，但是编译代码时无效，比如jdbc的驱动实现类，比如mysql驱动。因为写代码的时候是基于javax.sql包下的标准接口去写代码的。然后在测试的时候需要用这个包，在实际运行的时候才需要用这个包的，但是编译的时候只要javax.sql接口就可以了，不需要mysql驱动类。一般我们声明mysql驱动的时候，不会设置为runtime，因为也许你开发代码的时候会用到mysql驱动特定的api接口，不仅仅只是用javax.sql。

### 11.传递性依赖

maven的传递性依赖，就是说会自动递归解析所有的依赖，然后负责将依赖下载下来，接着所有层级的依赖，都会成为我们的项目的依赖，不需要我们手工干预。所有需要的依赖全部下载下来，不管有多少层级。这个就是maven的传递性依赖机制，自动给我递归依赖链条下载所有依赖的这么一个特性。

好比说我们依赖了junit，junit依赖了A。

我们对junit的依赖范围是test，junit对A的依赖范围是compile，那么我们对A的依赖范围是什么呢？ -> test。

### 12.依赖调解

既然说maven会自动解析所有层级的依赖，给我们自动下载所有的依赖，但是可能会出现依赖冲突的问题。

比如A->B->C->X(1.0)，A->D->X(2.0)，A有两个传递性依赖X，不同的版本。

就产生了依赖冲突的问题，maven如何解决呢？依赖调解的机制。

此时就会依赖调解，就近原则，离A最近的选用，就是X的2.0版本。

如果A->B->X(1.0)和A->D->X(2.0)，路径等长呢？那么会选择第一声明原则，哪个依赖在pom.xml里先声明，就用哪个。

### 13.可选依赖

<optional>true</optional>

此时依赖传递失效，不会向上传递。

如果A依赖于B，B依赖于C，B对C的依赖是optional，那么A就不会依赖于C。反之，如果没有optional，根据传递性依赖机制，A会依赖于C。

### 14.依赖冲突

比如你依赖了A和B，此时A依赖了C-1.0，B依赖了D，D依赖了C-2.0。

此时就会导致的事情是，由于A -> C1.0是最短路径，所以会用C1.0，导致D可能用不了想用的C-2.0的一个方法。

如何解决

一般来说，用最新的版本，因为新版本一般可以兼容旧版本，但是旧版本一定不会提供新版本的功能。

不要让maven自动去选择C-1.0来使用，而是手动强制使用C-2.0。

mvn dependency:tree，用dependency插件的tree goal，执行，进行依赖链条分析。

比如你发现C这个项目下报出了类似的问题，然后就用mvn dependency:tree这个命令看一下整个项目的依赖路径树，看看所有的依赖路径里，有哪几个版本的C项目，看一下，自己看看要用哪个版本的C，然后将其他的C版本的依赖全部手动排除掉。

```html
<dependency>
<groupId>A</groupId>
<artifactId>A</artifactId>
<version>1.0</version>
<exclusions>
    <exclusion>
        <groupId>C</groupId>
        <artifactId>C</artifactId>
    </exclusion>
</exclusions>
</dependency>
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 15.仓库的基本介绍

maven的仓库，就是用来统一存放各种依赖的地方。哪怕是有几十个工程，但是每个工程如果有相同的依赖，那么那个依赖在仓库里只会存在一次，不会放在各个工程自己的lib包下。消除了重复，如果要升级某个依赖，直接在各个工程的pom.xml里升级依赖的版本即可，大家都会自动引用最新版本的依赖了。在仓库里所有依赖就放一次，然后每个工程在pom.xml里面声明对依赖的引用即可，打包的时候，可以将所有依赖打入发布包即可。

### 16.maven的多层仓库架构

maven仓库的大类分成本地仓库和远程仓库两种，如果你声明了一个依赖，那么在构建打包的时候，先会去本地仓库里找，这个本地仓库的地址默认就在用户主目录下面的~/.m2/repository目录下面，里面各个依赖的存放路径就是跟上面说的那样；如果本地仓库找不到，那么就会去远程仓库找，默认是去maven自己的中央仓库里找，maven的中央仓库里几乎涵盖所有的依赖；然后会将中央仓库里的依赖下载下来放到本地仓库，缓存起来，供下次使用。

中央仓库是属于一种远程仓库，但是不只是有中央仓库一种远程仓库而已。

远程仓库还有私服和其他远程仓库，私服指的是在公司内部的局域网内，架设一个服务器，上面放一个公司内私有的仓库，就是私服；其他远程仓库，就是其他公司开放出来的仓库，比如java.net maven仓库，jboss maven仓库，等等。

（1）本地仓库

本地仓库，windows默认路径是~.m2\repository，linux默认路径是/home/root/.m2/repository

可以修改本地仓库的路径，在~/.m2/settings.xml配置文件里，可以设置：

<localRepository>某路径</localRepository>

（2）远程仓库

如果本地仓库没有某个依赖，那么maven就会从远程仓库去下载，默认就是从中央仓库去下载。

（3）中央仓库

maven有一个自带的超级pom.xml文件，里面配置了一个默认的中央仓库。

如果你不做任何配置，那么当你声明了某个依赖之后，就是如果本地仓库没有，就会去maven自带配置的这个中央仓库去拉取，就是http://repo1.maven.org/maven2这个地址。

（4）私服

一般正经公司都会自己假设一个maven私服，也就是在公司局域网内部，假设一个服务器，上面放一个maven私有仓库。

因为很多公司的服务器是不允许访问外网的，只能访问局域网，为了更好的安全性起见。而且依赖下载的速度也很快。

在本地配置远程仓库为私服，如果本地仓库没有，先去私服找，如果私服没有，再去中央仓库找。在中央仓库找到后，先缓存在私服中，然后再缓存在本地仓库中。

另外一个意义，就在于说，自己公司内部的一些发布包，可以放到私服上，供公司内的项目组之间使用。

（5）其他远程仓库

有些依赖可能在中央仓库没有，或者中央仓库的速度太慢，此时可能会用其他的一些远程仓库，比如jboss的仓库。java.net，google，codehaus，jboss，还有一些其他公司自己搞的Maven仓库，有少数的依赖包可能在中央仓库里找不到，只在其他仓库里。

那么私服除了连接中央仓库，还可以连接其他远程仓库。

（6）镜像仓库

比如说，像中央仓库在国外，很慢的，直接从中央仓库下载的话，是很慢的。

所以一般国内的一些大型的互联网公司，阿里云，会搞一个镜像仓库，完全跟中央仓库一模一样的，代理了中央仓库所有的请求。

你可以直接从阿里云镜像仓库去请求，如果有就直接返回了，国内网络的速度很快的，上百倍；阿里云如果自己没有，就会去从国外的中央仓库去下载。

### 17.nexus中的仓库

nexus安装好之后本身就内置了一些仓库。

包括四种仓库类型。

hosted：宿主仓库，这个仓库，是用来让你把你公司内部的发布包部署到这个仓库里来，然后公司内的其他人就可以从这个宿主仓库里下载依赖去使用。

proxy：代理仓库，这个仓库不是用来给你公司内部的发布包部署的，是代理了公司外部的各种仓库的，比如说java.net，codehaus，jboss仓库，最最重要的，就是代理了公司外部的中央仓库，但是这里其实可以修改为nexus连接的应该是国内的阿里云镜像仓库，阿里云去连接中央仓库。

group：仓库组，其实就是将，各种宿主仓库、代理仓库全部组成一个虚拟的仓库组，然后我们的项目只要配置依赖于一个仓库组，相当于就是可以自动连接仓库组对应的各种仓库。

最后还有仓库的状态和路径。

maven-central：这是maven中央仓库的代理仓库。

maven-releases：该仓库是个宿主仓库，用于部署公司内部的release版本的发布包（类似于1.0.0,，release的意思就是你的工程已经经过了完善的测试，单元测试，集成测试，QA测试，上生产环境使用了）到这个仓库里面，供其他同事在生产环境依赖和使用。

maven-snapshots：该仓库是个宿主仓库，用于部署公司内部的snapshot版本的发布包到这个仓库里（如果你的某个工程还在开发过程中，测试还没结束，但是，此时公司里其他同事也在开发一些工程，需要依赖你的包进行开发和测试，联调，此时你的工程的版本就是类似1.0.0-SNAPSHOT这样的版本），供其他同事在开发和测试的时候使用。

3rd party：该仓库是个宿主仓库，主要用来部署没法从公共仓库获取的第三方依赖包，比如说，你的公司依赖于第三方支付厂商的一个依赖包，那个依赖包不是开源的，是商业的包。那么你是没有办法从maven中央仓库获取的。此时，我们可能会自己手动从支付厂商那里获取到一个jar包，下载之后上传到私服里来，就放这个仓库里，3rd-party仓库。

maven-public：仓库组，上面所有release仓库都在这个仓库组内。

### 18.mvn命令的说明

mvn clean package：清理、编译、测试、打包。

mvn clean install：清理、编译、测试、打包、安装到本地仓库，比如你自己负责了3个工程的开发，互相之间有依赖，那么如果你开发好其中一个工程，需要在另外一个工程中引用它，此时就需要将开发好的工程jar包安装到本地仓库，然后才可以在另外一个工程声明对它的依赖，此时会直接取用本地仓库中的jar包。

mvn clean deploy：清理、编译、测试、打包、安装到本地仓库、部署到远程私服仓库，这个其实就是你负责的工程写好了部分代码，别人需要依赖你的jar包中提供的接口来写代码和测试。此时你需要将snapshot jar包发布到私服的maven-snapshots仓库中。供别人在本地声明对你的依赖和使用。

### 19.maven生命周期

maven的生命周期，就是对传统软件项目构建工作的抽象。

清理、初始化、编译、测试、打包、集成测试、验证、部署、站点生成。

maven有三套完全独立的生命周期，clean，default和site。每套生命周期都可以独立运行，每个生命周期的运行都会包含多个phase，每个phase又是由各种插件的goal来完成的，一个插件的goal可以认为是一个功能。

这就是maven的生命周期 -> phase（可以理解为阶段） -> 插件的关系，也是maven构建执行的核心原理。

你每次执行一个生命周期，都会依次执行这个生命周期内部的多个phase，每个phase执行时都会执行某个插件的goal完成具体的功能。

clean生命周期包含的phase如下：

pre-clean

clean

post-clean

default生命周期包含的phase如下：

validate：校验这个项目的一些配置信息是否正确

initialize：初始化构建状态，比如设置一些属性，或者创建一些目录

generate-sources：自动生成一些源代码，然后包含在项目代码中一起编译

process-sources：处理源代码，比如做一些占位符的替换

generate-resources：生成资源文件，才是干的时我说的那些事情，主要是去处理各种xml、properties那种配置文件，去做一些配置文件里面占位符的替换

process-resources：将资源文件拷贝到目标目录中，方便后面打包

compile：编译项目的源代码

process-classes：处理编译后的代码文件，比如对java class进行字节码增强

generate-test-sources：自动化生成测试代码

process-test-sources：处理测试代码，比如过滤一些占位符

generate-test-resources：生成测试用的资源文件

process-test-resources：拷贝测试用的资源文件到目标目录中

test-compile：编译测试代码

process-test-classes：对编译后的测试代码进行处理，比如进行字节码增强

test：使用单元测试框架运行测试

prepare-package：在打包之前进行准备工作，比如处理package的版本号

package：将代码进行打包，比如jar包

pre-integration-test：在集成测试之前进行准备工作，比如建立好需要的环境

integration-test：将package部署到一个环境中以运行集成测试

post-integration-test：在集成测试之后执行一些操作，比如清理测试环境

verify：对package进行一些检查来确保质量过关

install：将package安装到本地仓库中，这样开发人员自己在本地就可以使用了

deploy：将package上传到远程仓库中，这样公司内其他开发人员也可以使用了

site生命周期的phase：

pre-site

site

post-site

site-deploy

### 20.默认的phase和plugin绑定

我们直接运行mvn clean package的时候，每个phase都是由插件的goal来完成的，phase和plugin绑定关系是？

实际上，默认maven就绑定了一些plugin goal到phase上去。

process-resources resources:resources

compile compiler:compile

process-test-resources resources:testResources

test-compile compiler:testCompile

test surefire:test

package jar:jar或者war:war

install install:install

deploy deploy:deploy

site生命周期的默认绑定是：

site site:site

site-deploy site:deploy

clean生命周期的默认

clean clean:clean

### 21.maven的命令行和生命周期

比如mvn clean package。

clean是指的clean生命周期中的clean phase。

package是指的default生命周期中的package phase。

此时就会执行clean生命周期中，在clean phase之前的所有phase和clean phase，pre clean，clean。

同时会执行default生命周期中，在package phase之前的所有phase和package phase。

clean生命周期的pre clean，clean两个phase。

但是，pre clean phase默认是没有绑定任何一个plugin goal的，所以默认什么也不会干；clean phase默认是绑定了clean:clean，clean plugin的clean goal，所以就会去执行clean插件的clean goal，就会实现一个功能，就是清理target目录下的文件。

### 22.插件和goal

每个插件都有多个goal，每个goal都是一个具体的功能。

举个例子，dependency插件有十几个goal，可以进行分析项目依赖，列出依赖树，等等。

插件的goal，写法就是plugin:goal，比如dependency:tree。

用mvn dependency:tree，就可以手动执行某个插件的goal，执行某种功能。

### 23.父工程

```html
<groupId>com.zhss.oa</groupId>
<artifactId>oa-parent</artifactId>
<version>1.0.0-SNAPSHOT</version>
<packaging>pom</packaging>
<name>oa parent project</name>
<modules>
    <module>oa-organ</module>
    <module>oa-auth</module>
    <module>oa-flow</module>
</modules>
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果要一次性构建多个模块的工程，那么就需要创建一个父工程。

对oa-parent运行mvn clean install，此时就会对oa-parent中所有的工程都进行统一的构。

### 24.继承

maven里面提供了一个功能，叫做继承，也就是说，我们可以将一个项目中所有模块通用的一些配置，比如依赖和插件，全部放在一个父工程中，oa-parent，然后所有的子工程，声明一下从父工程中去继承。

如果直接将所有的依赖放入父工程，然后子工程用<parent>元素声明继承，然后此时会强制性继承父工程中所有的依赖和插件。

但是这样会导致每个子工程都强制性从父工程继承所有的依赖。

推荐的一个做法，是在父工程中，使用<depdendencyManagement>和<pluginManagement>两个元素，来声明要被子工程继承的依赖和插件。

有一个好处，就是，如果子工程用<parent>元素声明继承父工程，此时不会强制性继承所有的依赖和插件，子工程需要同时声明，自己要继承哪些依赖和插件，但是只要声明groupId和artifactId即可，不需要声明version版本号，因为version全部放在父工程中统一声明，强制约束所有子工程的相同依赖的版本要一样。

如果在父工程中，直接用dependencies和plugins来声明依赖和插件，子工程会强制全部继承；如果用dependencyManagemnent和pluginManagement来声明依赖和插件，默认情况下，子工程什么都不继承，只有当子工程声明了某个依赖或者插件的groupId+artifactId，但是不指定版本时，才会从父工程继承那个依赖。在规范的工程管理中，肯定都是用dependencyManagement和pluginManagemnent的。

### 25.< scope>import< /scope>

（1）如果你是公司基础组件的开发人，你的组件依赖了很多重要的第三方的开源框架，然后你为了保证依赖你的组件的人，不会自行导入一些开源框架的过旧的版本，导致跟你的组件出现依赖冲突。

（2）你需要为你的组件开发一个类型为pom的工程，后缀名为bom，这个工程，要声明一套dependencyManagement，里面声明好对你的组件的依赖的版本号，还有你的组件使用的重要的第三方开源框架的版本号。

（3）然后依赖方在引用你的组件的时候，需要在自己的dependencyManagement中，加一个scope范围为import，类型为pom的这么依赖导入，此时就可以将你的bom包声明的那一套依赖的版本号导入他那里，作为版本的约束。

（4）然后依赖方接着在dependencies里面可以声明对你的组件的依赖，此时版本号都不用写，因为已经被你约束了。

（5）同时，假设依赖方要自己依赖一个过旧的开源框架的版本，会有提示报警，不让他自行定义过旧版本的框架。

### 26.surefire

maven的自动化运行单元测试的插件，surefire。

maven中默认内置了surefire插件来运行单元测试，与最新流行的junit单元测试框架整合非常好。一般是在default生命周期的test阶段，会运行surefire插件的test goal，然后执行src/test/java下面的所有单元测试的。

而surefire插件会根据一定的规则在sre/test/java下面找单元测试类，具体规则如下：

**/Test*.java

**/*Test.java

**/*TestCase.java

通常比较流行的是用*Test.java格式类命名单元测试的类。

跳过单元测试

如果某次你的确不想要执行单元测试就打包，那么可以跳过单元测试，但是在实际开发过程中，这是绝对不被允许的。

用mvn package -DskipTests即可。

指定想要运行的测试类

mvn test -Dtest=**Test，直接指定你要运行的类即可。

通常见于开发人员自己刚写了某个单元测试类，然后就想自己运行一下看看效果。可以用逗号隔开，也可以用*匹配多个类。

测试报告

单元测试覆盖率是什么意思。

比如说，你总共写了100行代码，然后你写了3个单元测试方法，结果这3个单元测试的方法会执行你的100行代码中的80行，另外有20行代码是无论运行哪个单元测试都不会执行的，此时单元测试覆盖率就是80 / 100，80%。

一般单元测试覆盖率，要求的是，两种，方法覆盖率，行覆盖率。

surefire插件默认就会在target/surefire-reports目录下生成错误报告，就是如果有单元测试运行失败的话，就会有。

同时可以引入cobertura插件，可以看到测试覆盖率的报告。

### 27.版本到底是什么

版本分为两种，一种是快照（snapshot）版本，一种是发布（release）版本。

快照版本就是版本号后面加上SNAPSHOT，比如1.0.0-SNAPSHOT。

发布版本就是版本号上不加SNAPSHOT的，比如1.0.0

而且版本通常而言是三位的，特别是我们自己公司里的项目，就是比如1.0.0。

1.0.0中的第一个1指的是一个重大版本，比如已经基本成型的一个系统架构。

也有不少项目一开始是0.0.1这样的版本，这个指的就是这个系统刚开始起步，甚至都没能形成一个完整的东西出来，只是少数核心功能也出来了。

1.0.0中的第二个0指的是一个次要版本，比如1.1.0，那么就是加了一些新的功能或者开发了一些新的模块，或者做了一些较大的代码重构，技术升级改造。

1.0.0中的第三个0指的是日常迭代的一个增量版本，比如1.1.1，一般对应着修复了一个bug，或者对某些代码做了轻微的优化或者调整。

### 28.常见的六种依赖范围

1.compile:编译依赖范围 默认 对其三种都有效 2.test:测试依赖范围 只对测试 classpath 有效 3.runtime:运行依赖范围 只对测试和运行有效 编译主代码无效 例如 JDBC 4.provided:已提供依赖范围 只对编译和测试有效 运行时无效 例如 selvet api

5.system:系统依赖范围 谨慎使用 例如本地的 ,maven 仓库之外的类库文件 6.import(maven2.0.9 以上):导入依赖范围 不会对其他三种有影响

### 29.Maven 仓库

Maven 仓库是基于简单文件系统存储的，集中化管理Java API 资源（构件）的一个服务。仓库中的任何一个构件都有其唯一的坐标，根据这个坐标可以定义其在仓库中的唯一存储路径。得益于 Maven 的坐标机制，任何 Maven 项目使用任何一个构件的方式都是完全相同的，Maven 可以在某个位置统一存储所有的 Maven 项目共享的构件，这个统一的位置就是仓库，项目构建完毕后生成的构件也可以安装或者部署到仓库中，供其它项目使用。

对于Maven 来说，仓库分为两类：本地仓库和远程仓库。

### 30.Maven 常用命令

install

本地安装， 包含编译，打包，安装到本地仓库 编译 - javac 打包 - jar， 将java 代码打包为jar 文件 安装到本地仓库 - 将打包的jar 文件，保存到本地仓库目录中。

clean

清除已编译信息。 删除工程中的target 目录。

compile

只编译。 javac 命令

deploy

部署。 常见于结合私服使用的命令。 相当于是install+上传jar 到私服。

包含编译，打包，安装到本地仓库，上传到私服仓库。

package

打包。 包含编译，打包两个功能。

### 31.构建环节

- 清理：删除以前的编译结果，为重新编译做好准备
- 编译：将java源程序编译为字节码文件
- 测试：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性
- 报告：在每一次测试后以标准的格式记录和展示测试结果
- 打包：将一个包含诸多文件的工程封装为一个压缩文件用于安装和部署，java工程对应jar包，web工程对应war包
- 安装：在maven环境中特指将打包的结果——jar包或war包安装到本地仓库中
- 部署：将打包的结果部署到远程仓库或将war包部署到服务器上运行

### 32.maven核心概念

- POM
- 约定的目录结构
- 坐标
- 依赖管理
- 仓库管理
- 生命周期
- 插件和目标
- 继承
- 聚合

POM

Project Object Model，项目对象模型，将java工程的相关信息封装为对象作为便于操作和管理的模型

坐标

使用如下三个向量在maven的仓库中唯一的确定一个maven工程

- groupId：公司或组织的域名倒序+当前项目名称
- artifactId：当前项目的模块名称
- version：当前模块的版本

依赖管理

直接依赖和间接依赖

如果A依赖B，B依赖C，那么A和B，B和C都是直接依赖，而A和C是间接依赖

依赖范围

compile

- main目录下的java代码可以访问这个范围的依赖
- test目录下的java代码可以访问这个范围的依赖
- 部署到tomcat服务器上运行时要放在WEB-INF的lib目录下

test

- main目录下的java代码不能访问这个范围的依赖
- test目录下的java代码可以访问这个范围的依赖
- 部署到tomcat服务器上运行时不会放在WEB-INF的lib目录下

provided

- main目录下的java代码可以访问这个范围的依赖
- test目录下的java代码可以访问这个范围的依赖
- 部署到tomcat服务器上运行时不会放在WEB-INF的lib目录下

runtime

- main目录下的java代码不能访问这个范围的依赖
- test目录下的java代码可以访问这个范围的依赖
- 部署到tomcat服务器上运行时会放在WEB-INF的lib目录下

import

system

依赖的原则

- 路径最短者优先
- 路径相同时先申明者优先

依赖的排除

加入exclusions配置

仓库

- 本地仓库
- 远程仓库
  - 私服：架设在当前局域网环境下，为当前局域网范围内的所有maven工程服务
  - 中央仓库：架设在internet上，为全世界所有maven工程服务
    - 中央仓库的镜像：架设在各大洲，为中央仓库分担流量，减轻中央仓库压力，同时更快的响应用户请求

生命周期

maven有三套相互独立的生命周期

- Clean Lifecycle：在进行真正的构建之前进行一些清理工作
  - pre-clean：执行一些需要在clean之前完成的工作
  - clean：移除所有上一次构建生成的文件
  - post-clean：执行一些需要在clean之后立即完成的工作
- Default Lifecycle：构建的核心部分，编译、测试、打包、安装、部署等等
- Site Lifecycle：生成项目报告，站点，发布站点
  - pre-site：执行一些需要在生成站点文档之前完成的工作
  - site：生成项目的站点文档
  - post-site：执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
  - site-deploy：将生成的站点文档部署到特定服务器上

插件

maven的核心仅仅定义了抽象的生命周期，具体的任务都是交由插件完成的

### 33..**maven常见的依赖范围有哪些?**

- `compile`：编译依赖，默认的依赖方式，在编译（编译项目和编译测试用例），运行测试用例，运行（项目实际运行）三个阶段都有效，典型地有spring-core等jar。
- `test`：测试依赖，只在编译测试用例和运行测试用例有效，典型地有JUnit。
- `provided`：对于编译和测试有效，不会打包进发布包中，典型的例子为servlet-api,一般的web工程运行时都使用容器的servlet-api。
- `runtime`：只在运行测试用例和实际运行时有效，典型地是jdbc驱动jar包。
- `system`：不从maven仓库获取该jar,而是通过systemPath指定该jar的路径。
- `import`：用于一个dependencyManagement对另一个dependencyManagement的继承。

### 34.**maven 坐标的含义?**

- `groupId` ：定义当前 Maven 项目隶属的实际项目。
- `artifactId` ：该元素定义当前实际项目中的一个 Maven 项目(模块)。推荐的做法是使用实际项目名称作为 artifactId 的前缀。比如上例中的 junit ，junit 就是实际的项目名称，方便而且直观。在默认情况下，Maven 生成的构件，会以 artifactId 作为文件头。例如 junit-3.8.1.jar ，使用实际项目名称作为前缀，就能方便的从本地仓库找到某个项目的构件。
- `version` ：该元素定义了使用构件的版本。
- `packaging` ：定义 Maven 项目打包的方式，使用构件的什么包。打包方式通常与所生成构件的文件扩展名对应。
- `classifier` ：该元素用来帮助定义构建输出的一些附件。附属构件与主构件对应。

### 35.**maven 常用命令?**

- `mvn archetype`：create ：创建 Maven 项目。
- `mvn compile` ：编译源代码。
- `mvn deploy` ：发布项目。
- `mvn test-compile` ：编译测试源代码。
- `mvn test` ：运行应用程序中的单元测试。
- `mvn site` ：生成项目相关信息的网站。
- `mvn clean` ：清除项目目录中的生成结果。
- `mvn package` ：根据项目生成的 jar/war 等。
- `mvn install` ：在本地 Repository 中安装 jar 。
- `mvn clean package -Dmaven.test.skip=true` ：清除以前的包后重新打包，跳过测试类。

### 36.**maven的生命周期？**

Maven有三套相互独立的生命周期，分别是 `Clean`、`Default` 和 `Site`。每个生命周期包含一些阶段，阶段是有顺序的，后面的阶段依赖于前面的阶段。

**Clean 生命周期：** 清理项目： `pre-clean`：执行清理前需要完成的工作。 `clean`：清理上一次构建生成的文件。 `post-clean`：执行清理后需要完成的工作

**Default 生命周期：** 构建项目： `validate`：验证工程是否正确，所有需要的资源是否可用。 `compile`：编译项目的源代码。 `test`：使用合适的单元测试框架来测试已编译的源代码。这些测试不需要已打包和布署。 `package`：把已编译的代码打包成可发布的格式，比如 jar、war 等。 `integration-test`：如有需要，将包处理和发布到一个能够进行集成测试的环境。 `verify`：运行所有检查，验证包是否有效且达到质量标准。 `install`：把包安装到maven本地仓库，可以被其他工程作为依赖来使用。 `deploy`：在集成或者发布环境下执行，将最终版本的包拷贝到远程的repository，使得其他的开发者或者工程可以共享。

**Site 生命周期：** 建立和发布项目站点： `pre-site`：生成项目站点之前需要完成的工作 `site`：生成项目站点文档 `post-site`：生成项目站点之后需要完成的工作 `site-deploy`：将项目站点发布到服务器

各个生命周期相互独立，一个生命周期的阶段前后依赖。 mvn clean ：调用 Clean 生命周期的 clean 阶段，实际执行 pre-clean 和 clean 阶段 mvn test ：调用 Default 生命周期的 test 阶段，实际执行 test 以及之前所有阶段 mvn clean install ：调用 Clean 生命周期的 clean 阶段和 Default 生命周期 的 install 阶段，实际执行 pre-clean 和 clean ，install 以及之前所有阶段。

### 37.**使用“mvn clean package”命令进行项目打包，该命令具体做了什么？**

- 使用清理插件：`maven-clean-plugin`执行清理删除已有target目录；
- 使用资源插件：`maven-resources-plugin`执行资源文件的处理；
- 使用编译插件：`maven-compiler-plugin`编译所有源文件生成class文件至target\classes目录下；
- 使用资源插件：`maven-resources-plugin`执行测试资源文件的处理；
- 使用编译插件：`maven-compiler-plugin`编译测试目录下的所有源代码；
- 使用插件：`maven-surefire-plugin`运行测试用例；

### 38.**maven依赖原则？**

**依赖路径最短优先原则** 项目依赖了两个jar包，其中A-B-C-D ， A-D。由于第二条路径最短，所以项目使用的是第二个D。

**pom文件中申明顺序优先** 项目依赖了两个jar包，A-B-D ，A-C-D。maven会根据加载顺序。如果先申明了B，在申明了C，那么最后依赖就用A-C-D。

**覆写优先** 子pom内声明的优先于父pom中的依赖。

### 39.**说一下maven仓库？**

Maven仓库有2种

- 本地仓库
- 远程仓库

Maven 会先搜索本地仓库（repository），发现本地没有然后从远程仓库（中央仓库）获取。 私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，私服代理广域网上的远程仓库，供局域网内的 Maven 用户使用。当 Maven 需要下载构件的时候，它从私服请求，如果私服上不存在该构件，则从外部的远程仓库下载，缓存在私服上之后，再为 Maven 的下载请求提供服务。我们还可以把一些无法从外部仓库下载到的构件上传到私服上。







