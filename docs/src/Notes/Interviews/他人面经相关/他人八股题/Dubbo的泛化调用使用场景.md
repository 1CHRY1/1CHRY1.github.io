### 来自拼多多提前批一面

#### Dubbo的泛化调用使用场景？如何实现服务降级？

Dubbo 的泛化调用功能允许调用方在无需依赖目标服务的编译期接口和模型类的情况下，动态调用 RPC 服务。它通过 GenericService 对象及 $invoke(methodName, parameterTypeNames, args) 方法，传入服务接口全名、方法名、参数类型和参数值即可完成调用。非常适用于网关服务或 RPC 测试平台这样的通用场景，避免频繁依赖 SDK，提升灵活性与解耦度。

在服务降级方面，Dubbo 提供了原生的 Mock 模式支持，例如 mock="force:return null" 可以直接命中降级逻辑返回指定值或 mock="fail:return default" 用于失败后回退。此外可使用注解或 XML 配置界定哪些接口或方法需要降级。在更复杂治理场景下，可结合 Sentinel 或 Hystrix 实现限流、熔断与 fallback 策略，保障服务可用性，尤其在依赖服务不稳定时非常有效。

------

Dubbo的泛化调用指的是：允许客户端在不依赖服务提供方接口和模型类的情况下，动态调用服务。通过传入服务全限定名 interface、方法名和参数类型及参数值，就可以完成调用

使用场景有：
- **服务网关**：作为统一入口，网关不应该依赖具体服务接口 SDK，否则每次新增服务就要改代码并部署。泛化调用让网关动态调用任何服务，无需依赖接口。
- **RPC测试平台**：用户输入接口信息后，通过泛化调用即可测试调用，而无需提前引入接口依赖。

实现原理：
- 客户端通过 ReferenceConfig<GenericService> reference.setGeneric(true) 获取 GenericService 实例。调用 $invoke(method, parameterTypeNames, args) 即可发起调用。
- 底层通过动态代理和反射机制，将方法调用封装为通用 RPC 请求，服务器端通过反射找到对应方法执行并返回结果。参数和返回值使用通用格式（例如 Map）封装。

Dubbo的服务降级：
1. Dubbo原生的降级机制——Mock机制，可以通过XML或配置注册中心动态覆盖
2. 面向降级治理平台：Sentinel/Hystrix，Dubbo本身提供基础的Mock降级，若需要复杂的熔断、限流、回退逻辑，可结合 Sentinel 或 Hystrix 等工具
3. 使用场景：
- 非关键服务临时中断：如日志、监控等可降级处理，返回默认值避免影响主业务。
- 异常/超时处理：若服务响应慢或异常，通过 Mock 返回默认值快速恢复体验。
- 整合限流熔断平台治理：如流量激增或依赖不稳定时，使用 Sentinel 降级与限流保护系统稳定性。