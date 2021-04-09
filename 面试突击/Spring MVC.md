# Spring MVC

## 1.Spring MVC的原理

1）用户向服务器发送请求，请求被Spring 前端控制Servelt DispatcherServlet捕获； 2）DispatcherServlet 对请求URL 进行解析，得到请求资源标识符（URI）。然后根据该URI，调用HandlerMapping 获得该Handler 配置的所有相关的对象 （包括Handler 对象以及Handler 对象对应的拦截器），最后以HandlerExecutionChain 对象的形式返回； 3）DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter 后，此时将开始执行拦截器的preHandler(...)方法） 4） 提取Request 中的模型数据，填充Handler 入参，开始执行Handler（Controller)。 在填充Handler 的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作： HttpMessageConveter： 将请求消息（如Json、xml 等数据）转换成一个对象，将对象转换为指定的响应信息 数据转换：对请求消息进行数据转换。如String 转换成Integer、Double 等 数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error 中 5）Handler 执行完成后，向DispatcherServlet 返回一个ModelAndView 对象；

6）根据返回的ModelAndView，选择一个适合的ViewResolver（必须是已经注册到Spring 容器中的ViewResolver)返回给DispatcherServlet ； 7）ViewResolver 结合Model 和View，来渲染视图。 8）将渲染结果返回给客户端。

## 2.组件

1.DispatcherServlet

DispatcherServlet ，也被称为“分发器Servlet ”， 是Spring Web MVC 中的核心组件，它不是一个接口，而是一个实现类。

在初始化时， DispatcherServlet 会通过内部的Spring Web 应用程序环境，找到相应的Spring Web MVC 的各个组件。

在服务时， DispatcherServlet 会通过一组已注册的处理器映射找到一个处理器( Handler ），然后从一组己注册的处理器适配器中找到一个支持该处理器的处理器适配器，通过它把控制流转发给这个处理器。这个处理器在结束业务逻辑的调用后，会把模型数据和逻辅视图回传给DispatcherServlet 。最后，

DispatcherServlet 会通过视图解析器（ View Resolver ）得到真正的视图，把控制权交给视图，同时传入模型数据。视图会按照一定的视图层定义，将这些数据展现到用户的响应里。

2.处理器映射

处理器映射（ HandlerMapping ）用于将一个请求（ Request ）映射到一个处理器。在一个Spring Web MVC 实例中可能包含多个处理器映射，按照处理器映射所在的顺序， 第一个返回非空处理器执行链（ HandlerExecutionChain ）的处理器映射，会被当作有效的处理器映射。处理器执行链包含一个处理器和一组能够应用在处理器上的拦截器（ Interceptor ) 。

在处理器映射接口中输入一个HTTP 请求（ HttpServletRequest ）给方法（ getHandler ),这个方法就会输出一个处理器执行链，我们在处理器执行链中就能获得一个处理器。

3.处理器适配器

处理器适配器（ HandlerAdaptor ）用于转接一个控制流到一个指定类型的处理器。某种类型的处理器通常会对应某处理器适配器的一个实现。处理器适配器能够判断自己是否支持某个处理器，如果支持，就可以使用这个处理器处理当前的H TTP 请求。

处理器在处理HTTP 请求后会返回数据模型和逻辅视图名称的组合，处理器适配器会把这个组合返回给DispatcherServlet 。getLastModified （）方法用于处理一个带有LastModified 头信息的HTTP 请求（并不是所有的处理器适配器都需要支持这个方法）。

4.处理器与控制器

处理器是处理业务逻辑的一个基本单元，通过传人的HTTP 请求来决定如何处理业务逻辅和执行哪些服务，在执行服务后返回相应的模型数据和逻辑视图名。控制流是由处理器适配器传递给处理器的，所以某种类型的处理器一定对应某个处理器适配器，这样就可以实现处理器适配器和处理器的任意插拔。

5.视图解析器

视图解析器用于映射一个逻辑视图名称到一个真正的视图。控制器在处理业务逻辑之后，通常会返回需要显示的数据模型和视图的逻辑名称，这样就需要一个支持目标视图的视图解析器解析出一个真正的视图，然后传递控制流给这个视图。

6.视图

视图用于把模型数据通过某种显示方式反馈给用户，通常通过执行JSP 页面来完成，也可以通过其他更复杂的显示技术来完成。

## 3.DispatcherServlet 的核心处理流程

( 1 ）通过处理器映射查找到支持这个操作的处理器，再通过这个处理器调用具体的操作方法，在调用处理器之前和之后都对应用在这个处理器中的拦截器进行了调用，可让定制化的处理器初始化或者析构资源。 ( 2 ）解析视图并且显示视图，发送HTTP 响应。

## 4.SpringMVC 常用注解

- @Controller： 声明该类为Spring MVC 中的控制器
- @RequestMapping： 用于声明映射Web 请求的地址和参数，包括访问路径和参数
- @ResponseBody： 支持将返问值放在Response Body 体中返问，通常用于返回JSON 数据到前端
- @ RequestBody： 允许Request 的参数在Request Body 体中
- @ Path Variable： 用于接收基于路径的参数，通常作为RESTful 接口的实现
- @RestController： 组合注解，相当于＠ Controller 和 @ ResponseBody的组合
- @ExceptionHandler： 用于全局控制器的异常处理
- @InitBinder： WebDataBinder 用来自动绑定前台请求的参数到模型（ Model ）中

## 5.如何开启注解处理器和适配器？

我们在项目中一般会在springmvc.xml 中通过开启 <mvc:annotation driven> 来实现注解处理器和适配器的开启。

## 6.组件说明

以下组件通常使用框架提供实现： DispatcherServlet：作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展 性 HandlerMapping：通过扩展处理器映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。 HandlAdapter：通过扩展处理器适配器，支持更多类型的处理器。 ViewResolver：通过扩展视图解析器，支持更多类型的视图解析，例如：jsp、freemarker、pdf、excel等。 组件： 1、前端控制器DispatcherServlet（不需要工程师开发）,由框架提供 作用：接收请求，响应结果，相当于转发器，中央处理器。有了dispatcherServlet减少了其它组件之间的耦合度。 用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求， dispatcherServlet的存在降低了组件之间的耦合性。 2、处理器映射器HandlerMapping(不需要工程师开发),由框架提供 作用：根据请求的url查找Handler HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式， 实现接口方式，注解方式等。 3、处理器适配器HandlerAdapter 作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler 通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。 4、处理器Handler(需要工程师开发) 注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。 由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。 5、视图解析器View resolver(不需要工程师开发),由框架提供 作用：进行视图解析，根据逻辑视图名解析成真正的视图（view） View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对 象，最后对View进行渲染将处理结果通过页面展示给用户。springmvc框架提供了很多的View视图类型，包括：jstlView、 freemarkerView、pdfView等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求 开发具体的页面。 6、视图View(需要工程师开发jsp...) View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）核心架构的具体流程步骤如下： 1、首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访 问点，进行全局的流程控制； 2、DispatcherServlet——>HandlerMapping， HandlerMapping 将会把请求映射为HandlerExecutionChain 对象（包含一个Handler 处 理器（页面控制器）对象、多个HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略； 3、DispatcherServlet——>HandlerAdapter，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计 模式的应用，从而很容易支持很多类型的处理器； 4、HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处理方法，完成功 能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）； 5、ModelAndView的逻辑视图名——> ViewResolver， ViewResolver 将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更 换其他视图技术； 6、View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技 术； 7、返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。下边两个组件通常情况下需要开发： Handler：处理器，即后端控制器用controller表示。 View：视图，即展示给用户的界面，视图中通常需要标签语言展示模型数据。

## 7.描述一下 DispatcherServlet 的工作流程？

① **发送请求**

用户向服务器发送 HTTP 请求，请求被 Spring MVC 的调度控制器 DispatcherServlet 捕获。

② **映射处理器**

DispatcherServlet 根据请求 URL ，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 **Handler** 对象以及 Handler 对象对应的**拦截器**），最后以 HandlerExecutionChain 对象的形式返回。

- 即 HandlerExecutionChain 中，包含对应的 **Handler** 对象和**拦截器**们。

```html
@Nullable
> HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
```

③ **处理器适配**

DispatcherServlet 根据获得的 Handler，选择一个合适的HandlerAdapter 。（附注：如果成功获得 HandlerAdapter 后，此时将开始执行拦截器的 `#preHandler(...)` 方法）。

提取请求 Request 中的模型数据，填充 Handler 入参，开始执行Handler（Controller)。 在填充Handler的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作：

- HttpMessageConverter ：会将请求消息（如 JSON、XML 等数据）转换成一个对象。
- 数据转换：对请求消息进行数据转换。如 String 转换成 Integer、Double 等。
- 数据格式化：对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等。
- 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到 BindingResult 或 Error 中。

Handler(Controller) 执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象。

```html
@Nullable
> ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
```

⑤ **解析视图**

根据返回的 ModelAndView ，选择一个适合的 ViewResolver（必须是已经注册到 Spring 容器中的 ViewResolver)，解析出 View 对象，然后返回给 DispatcherServlet。

```html
@Nullable
> View resolveViewName(String viewName, Locale locale) throws Exception;
```

⑥ ⑦ **渲染视图** + **响应请求**

ViewResolver 结合 Model 和 View，来渲染视图，并写回给用户( 浏览器 )。

```html
void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
```

在步骤 ③ 中，如果 Handler(Controller) 执行完后，如果判断方法有 `@ResponseBody` 注解，则直接将结果写回给用户( 浏览器 )。

但是 HTTP 是不支持返回 Java POJO 对象的，所以需要将结果使用 [HttpMessageConverter](http://svip.iocoder.cn/Spring-MVC/HandlerAdapter-5-HttpMessageConverter/) 进行转换后，才能返回。例如说，大家所熟悉的 [FastJsonHttpMessageConverter](https://github.com/alibaba/fastjson/wiki/在-Spring-中集成-Fastjson) ，将 POJO 转换成 JSON 字符串返回。

## 8.@RequestMapping 和 @GetMapping 注解的不同之处在哪里？

- `@RequestMapping` 可注解在类和方法上；`@GetMapping` 仅可注册在方法上。
- `@RequestMapping` 可进行 GET、POST、PUT、DELETE 等请求方法；`@GetMapping` 是 `@RequestMapping` 的 GET 请求方法的特例，目的是为了提高清晰度。

## 9.Spring MVC 的异常处理？

Spring MVC 提供了异常解析器 HandlerExceptionResolver 接口，将处理器( `handler` )执行时发生的异常，解析( 转换 )成对应的 ModelAndView 结果。代码如下：

```html
// HandlerExceptionResolver.java

public interface HandlerExceptionResolver {

    /**
     * 解析异常，转换成对应的 ModelAndView 结果
     */
    @Nullable
    ModelAndView resolveException(
            HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex);

}
```

- 也就是说，如果异常被解析成功，则会返回 ModelAndView 对象。

一般情况下，我们使用 `@ExceptionHandler` 注解来实现过异常的处理，

## 10. Spring MVC 拦截器？

- 一共有三个方法，分别为：
  - `#preHandle(...)` 方法，调用 Controller 方法之**前**执行。
  - `#postHandle(...)` 方法，调用 Controller 方法之**后**执行。
  - \#afterCompletion(...)方法，处理完 Controller 方法返回结果之后执行。
    - 例如，页面渲染后。
    - **当然，要注意，无论调用 Controller 方法是否成功，都会执行**。
- 举个例子：
  - 当俩个拦截器都实现放行操作时，执行顺序为 `preHandle[1] => preHandle[2] => postHandle[2] => postHandle[1] => afterCompletion[2] => afterCompletion[1]` 。
  - 当第一个拦截器 `#preHandle(...)` 方法返回 `false` ，也就是对其进行拦截时，第二个拦截器是完全不执行的，第一个拦截器只执行 `#preHandle(...)` 部分。
  - 当第一个拦截器 `#preHandle(...)` 方法返回 `true` ，第二个拦截器 `#preHandle(...)` 返回 `false` ，执行顺序为 `preHandle[1] => preHandle[2] => afterCompletion[1]` 。
- 总结来说：
  - `#preHandle(...)` 方法，按拦截器定义**顺序**调用。若任一拦截器返回 `false` ，则 Controller 方法不再调用。
  - `#postHandle(...)` 和 `#afterCompletion(...)` 方法，按拦截器定义**逆序**调用。
  - `#postHandler(...)` 方法，在调用 Controller 方法之**后**执行。
  - `#afterCompletion(...)` 方法，只有该拦截器在 `#preHandle(...)` 方法返回 `true` 时，才能够被调用，且一定会被调用。为什么“且一定会被调用”呢？即使 `#afterCompletion(...)` 方法，按拦截器定义**逆序**调用时，前面的拦截器发生异常，后面的拦截器还能够调用，**即无视异常**。