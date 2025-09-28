### 来自网络面经

#### SpringMVC常用的一些注解及其原理是什么？

1. 控制器相关注解：
`@Controller`: 将一个类标识为Spring MVC控制器。
`@RestController`: 是 `@Controller` 的变体，用于标识RESTful风格的控制器，常用于返回JSON等。

2. 请求映射注解：
`@RequestMapping`: 映射请求路径和HTTP方法到处理方法。
`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`: 缩写形式的 `@RequestMapping`，用于指定不同的HTTP方法。

3. 请求参数注解：
`@RequestParam`: 用于将请求参数绑定到方法的参数。
`@PathVariable`: 用于将URI模板变量绑定到方法的参数。

4. 请求体注解：
`@RequestBody`: 用于将请求的内容体（JSON、XML等）绑定到方法的参数。

5. 表单提交注解：
`@ModelAttribute`: 用于将请求参数绑定到模型对象。
`@ModelAttribute`（在方法参数上）: 用于将方法的返回值添加到模型中。

6. Session 相关注解：
`@SessionAttribute`: 用于将模型中的属性暴露给Session。
`@SessionAttributes`: 指定模型属性应该存储在会话中，以供多个请求之间共享。

7. 响应体注解：
`@ResponseBody`: 表示方法返回的对象直接写入响应体，而不是解析为视图。
`@ResponseStatus`: 指定响应的HTTP状态码。

8. 异常处理注解：
`@ExceptionHandler`: 定义处理特定异常的方法。
`@ControllerAdvice`: 用于定义全局的异常处理器。

9. 重定向和视图注解：
`@RedirectAttributes`: 用于在重定向中传递数据。
`@ModelAttribute`（在方法上）: 用于将方法的返回值添加到模型中，以供视图使用。

10. 文件上传相关注解：
`@RequestPart`: 用于将请求的一部分（通常是文件）绑定到方法的参数。

**原理：**
请求映射注解（如@RequestMapping）：
- 启动阶段：RequestMappingHandlerMapping会扫描所有控制器 Bean 中的方法，解析@RequestMapping及其变体（@GetMapping 等）中定义的请求路径（value/path）、HTTP 方法（method）、请求头（headers） 等信息，**生成一个 “请求条件→处理方法” 的映射表**（存储在内存中）。例如，@GetMapping("/users/{id}")会被解析为 “GET 方法 + 路径/users/{id}” 对应某个具体的控制器方法。
- 运行阶段：当请求到达DispatcherServlet（前端控制器）时，DispatcherServlet会调用RequestMappingHandlerMapping，根据请求的路径和 HTTP 方法，从映射表中匹配到对应的处理方法。

请求参数 / 路径变量注解（如@RequestParam）
- @RequestParam：由RequestParamMethodArgumentResolver处理，从请求的查询参数（如?name=xxx）中提取指定名称的参数值，转换为方法参数的类型（如String→Integer）。
- @PathVariable：由PathVariableMethodArgumentResolver处理，从请求路径的URI 模板变量（如/users/{id}中的id）中提取值，绑定到参数。
- 解析过程：当DispatcherServlet找到匹配的处理方法后，会调用HandlerAdapter（处理器适配器），由其触发参数解析器，完成请求数据到方法参数的绑定。
...

总而言之，核心原理是：
**启动时：**通过扫描组件识别注解，解析元数据（如路径、参数规则），建立映射关系（如请求→方法）并注册到容器；
**运行时：**通过处理器映射器找到匹配的处理方法，再通过参数解析器、返回值处理器等组件，基于注解规则完成数据绑定、转换和响应，同时结合 AOP 处理异常等横切逻辑。