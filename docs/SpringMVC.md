## SpringMVC

## 1.Spring MVC核心架构

（1）tomcat的工作线程将请求转交给spring mvc框架的DispatcherServlet.

（2）DispatcherServlet查找@Controller注解的controller，我们一般会给controller加上你@RequestMapping的注解，标注说哪些controller用来处理哪些请求，此时根据请求的uri，去定位到哪个controller来进行处理.

（3）根据@RequestMapping去查找，使用这个controller内的哪个方法来进行请求的处理，对每个方法一般也会加@RequestMapping的注解.

（4）他会直接调用我们的controller里面的某个方法来进行请求的处理.

（5）我们的controller的方法会有一个返回值，以前的时候，一般来说还是走jsp、模板技术，我们会把前端页面放在后端的工程里面，返回一个页面模板的名字，spring mvc的框架使用模板技术，对html页面做一个渲染；返回一个json串，前后端分离，可能前端发送一个请求过来，我们只要返回json数据.

（6）再把渲染以后的html页面返回给浏览器去进行显示；前端负责把html页面渲染给浏览器就可以了.

