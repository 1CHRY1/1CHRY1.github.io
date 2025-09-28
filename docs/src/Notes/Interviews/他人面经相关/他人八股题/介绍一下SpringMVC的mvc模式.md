### 来自虾皮二面

#### 介绍一下SpringMVC的mvc模式？

Spring MVC 是 Spring Framework 中实现的一种轻量级 Web MVC 机制，它遵循经典的 Model-View-Controller 架构模式，将应用划分为模型、视图和控制器三个职责单元——极大提高了应用的可维护性与可测试性。在 Spring 中，整个请求处理由 DispatcherServlet 担任前端控制器，负责接收所有 HTTP 请求；然后通过 HandlerMapping 映射器根据请求url找到具体的处理器，生成处理器的执行链HandlerExecutionChain一并返回给DispatcherServlet，它再根据处理器Handler获取处理器适配器 HandlerAdapter 使用 @Controller 和 @RequestMapping 注解定义的处理方法；处理完成后返回一个 ModelAndView，携带数据模型（Model）与视图名称；接着 ViewResolver 将逻辑视图名解析为具体视图（例如 JSP、Thymeleaf），并进行渲染。整个流程体现了控制流程清晰、视图隔离、灵活配置的优势，同时 Spring MVC 和 Spring IoC/AOP/事务等模块无缝整合，使得开发高可复用、高测试友好的 Web 应用变得更高效。

------

mvc也就是model-view-controller，是一种经典的软件架构模式。model是模型层，封装业务数据和简单逻辑，view是视图层，负责呈现业务的数据，而controller是控制器曾，可以接收用户的请求调用业务逻辑，最终生成适当的视图。

SpringMVC包括几个核心组件：
首先是DispatcherServlet，所有的HTTP请求都是以DispatcherServlet作为入口，由它拦截和接入
接着是HandlerMapping，DispatcherServlet会根据请求的数据、路径和方法使用HandlerMapping找到合适的Controller处理器方法，
然后是Controller，接收DS的请求逻辑并执行业务调用，返回一个ModelAndView对象
再后是ViewResolver，DS获取到ModelAndView对象后会使用ViewResolver将逻辑视图名解析为具体视图并返回给前端，最终渲染页面