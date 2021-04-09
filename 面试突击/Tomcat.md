# Tomcat

### 1.Tomcat 文件组成

bin:启动、关闭Tomcat 的命令。 common/lib:网络编程的jar 文件。 conf:配置文件。 logs:日志文件. server:自带的web 应用(三个). shared:所有web 应用都可以访问的内容. temp:临时. webapps:默认站点文件夹.

work:jsp 生成的类.

### 2.浏览器页面与Tomcat 的交互过程？

当一个JSP 页面第一次被访问的时候，JSP 引擎将执行以下步骤： （1）将JSP 页面翻译成一个Servlet，这个Servlet 是一个java 文件，同时也是一个完整的java 程序 （2）再由java 编译器对这个Servlet 进行编译，得到可执行class 文件 （3）再由JVM 解释器来解释执行class 文件，生成向客户端发送的应答，然后发送给客户端 以上三个步骤仅仅在JSP 页面第一次被访问时才会执行，以后的访问速度会因为class文件已经生成而大大提高。

### 3.Servlet 的生命周期

实例化：Servlet 容器创建Servlet 类的实例。 初始化：该容器调用init()方法，通常会申请资源。 服务：由容器调用service()方法，（也就是doGet()和doPost()）。 破坏：在释放Servlet 实例之前调用destroy()方法，通常会释放资源。 不可用：释放内存的实例。

### 4.**Tomcat缺省端口是多少，如何修改？**

Tomcat缺省端口是`8080`；

修改tomcat 端口： 1、找到tomcat目录下的`conf`文件夹； 2、进入conf文件夹找到`server.xml`文件 3、在`server.xml`文件里面找到`Connector` 标签，把`port="8080"`，改成需求端口即可。

### 5.**Servlet请求过程？**

- Tomcat容器中通过web.xml加载所有的Servlet。
- 用户在浏览器输入不同的地址，向Tomcat容器请求资源。
- Tomcat容器根据地址首先在容器内找到应用ServletTest。
- Tomcat容器再根据地址去web.xml找到相应的servlet地址。
- Tomcat容器根据找到的servlet地址去web.xml找到相应的Servlet类，并实例化。
- Tomcat容器实例化相应的Servlet，首先调用init方法。
- Tomcat容器实例化相应的Servlet，首先调用service方法处理用户请求，比如post或者是get。
- Servlet处理完成之后，先将数据给Tomcat容器，Tomcat容器再把处理结果给浏览器客户端。
- Tomcat容器调用servlet实例的destory方法销毁这个实例。

### 6.**Tomcat执行流程？**

1.请求被发送到本机端口8080，被在那里侦听的`Coyote HTTP/1.1 Connector`获得 2.Connector把该请求交给它所在的Service的Engine来处理，并等待来自Engine的回应 3.Engine获得请求`localhost/项目/页面.jsp`，匹配它所拥有的所有虚拟主机Host 4.Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机） 5.localhost Host获得请求`/项目/页面.jsp`，匹配它所拥有的所有Context 6.Host匹配到路径为`/项目的Context`（如果匹配不到就把该请求交给路径名为””的Context去处理） 7.`path="/项目"`的Context获得请求`/页面.jsp`，在它的mapping table中寻找对应的servlet 8.Context匹配到`URL PATTERN`为`*.jsp`的servlet，对应于JspServlet类 9.构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的`doGet`或`doPost`方法 10.Context把执行完了之后的HttpServletResponse对象返回给`Host` 11.Host把HttpServletResponse对象返回给`Engine` 12.Engine把HttpServletResponse对象返回给`Connector` 13.Connector把HttpServletResponse对象返回给客户`browser`

### 7.**Tomcat部署方式?**

1.直接把Web项目放在`webapps`下，Tomcat会自动将其部署 2.在`server.xml`文件上配置`<Context>\`节点，设置相关的属性即可 3.通过`Catalina`来进行配置:进入到conf\Catalina\localhost文件下，创建一个xml文件，该文件的名字就是站点的名字。