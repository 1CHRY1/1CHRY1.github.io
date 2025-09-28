### 来自虾皮二面

#### 介绍一下Spring的AOP？（Spring的AOP在Spring的各种中间件等内部有什么应用）

AOP也就是面向切面编程，是一种编程思想。在这种思想中功能可以分为核心业务和周边功能，通过AOP可以将这两种功能独立开发，并在业务场景中编织在一起。

然后AOP的实现主要依赖动态代理技术，也就是在程序运行的时候才生成代理对象，而不是编译时。在调用某个类方法的时候，其实是基于其代理类invocationHandler中的invoke方法实现的，在指定method，proxy和args的并通过反射方法执行类方法的前后可以分别织入前置、后置甚至是错误异常时的方法；SpringAOP支持默认的JDK动态代理（基于接口），同时也支持CGLIB动态代理（基于目标对象）

AOP的好处在于可以把与业务无关却被业务模块共同调用的逻辑封装起来，减少重复代码，降低耦合度，提高扩展性。

------

SpringAOP在Spring中还被用在事务、方法级安全、缓存、异步、方法校验等方面。

##### 事务 (@Transactional)

1. 参与组件
`TransactionInterceptor`：核心拦截器；基于方法/类上的事务元数据（传播、隔离、只读、回滚规则等）在调用前后开关事务。
`TransactionAttributeSource`：解析注解/XML 得到 TransactionAttribute。
`PlatformTransactionManager`：具体资源的事务实现（JDBC 用 DataSourceTransactionManager，JPA 用 JpaTransactionManager 等）。
`TransactionSynchronizationManager`：用 ThreadLocal 绑定当前线程的事务资源/同步回调（连接、会话等）

2. 关键点
- 事务是代理拦截实现的：只有通过代理的调用才生效；同类内部 this.method() 自调用会绕过代理导致失效；同时标准代理只拦截 public 方法。
- Spring 把参与事务的资源（连接、Session 等）放在 TransactionSynchronizationManager 的 ThreadLocal 里，所以跨线程默认不会继承事务上下文。
- 从 Spring 6 开始同一个拦截器也能根据返回类型区分命令式与反应式事务（reactive：返回 Publisher/Flow）。

##### 方法级安全 (@PreAuthorize/@PostAuthorize等)

1. 参与组件
启用：@EnableMethodSecurity。
MethodSecurityInterceptor：方法拦截器；委托 AccessDecisionManager、SecurityMetadataSource，并通过表达式处理器析 @PreAuthorize("#id == authentication.name") 等 SpEL 条件。
认证信息由 SecurityContextHolder（默认 ThreadLocal）提供。

2. 执行流程
进入 MethodSecurityInterceptor，获取该方法的权限元数据（注解/配置）。
通过 AccessDecisionManager（投票器/表达式）做鉴权，失败抛 AccessDeniedException；成功才执行目标方法；@PostAuthorize 在返回后再校验返回值策略。

##### 缓存 (@Cacheable/@CachePut/@CacheEvict)

1. 参与组件
- CacheInterceptor（继承 CacheAspectSupport）：核心拦截器；基于 CacheOperationSource 解析注解与 SpEL（key、condition、unless）。
- CacheManager/Cache：真正的缓存实现（ConcurrentMap、Redis、Caffeine、Ehcache…）。

2. 执行流程（@Cacheable 为例）
- 进入拦截器 → 解析 CacheOperation 与 Key（可自定义 KeyGenerator）。
- 命中直接返回缓存；未命中则 proceed() 调业务，得到结果后 put 入缓存（unless 条件可阻止写入）。
- @CachePut 强制执行并写入；@CacheEvict 在前/后删除条目或清空。

##### 异步 (@Async)

1. 参与组件
- 启用：@EnableAsync。
- AsyncAnnotationBeanPostProcessor：为带 @Async 的 Bean 添加 Advisor，若无现成代理则创建 AOP 代理。
- AsyncExecutionInterceptor/AnnotationAsyncExecutionInterceptor：把方法提交到 TaskExecutor（线程池）；返回 Future/CompletableFuture 时做适配；void 返回靠 AsyncUncaughtExceptionHandler 处理异常。

2. 执行特点
- 仍是代理拦截：自调用无效；默认只拦截 public；线程切换后不会继承调用线程里的事务/安全上下文（除非显式传播/包装）。

##### 方法参数/返回值校验 (@Validated)

1. 参与组件
- MethodValidationPostProcessor：一个 BeanPostProcessor，为 @Validated 的 Bean 织入方法级校验拦截器，委托底层 Bean Validation（Hibernate Validator）。
- 触发时机：调用前校验参数、调用后按需校验返回值，失败抛 ConstraintViolationException