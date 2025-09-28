### 来自网络面经

#### 介绍一下SpringIoC

SpringIoC即控制反转，将对象创建的控制权从应用代码交由 Spring 容器托管，由容器负责 Bean 的实例化和依赖管理。这样做降低了组件之间的耦合度，提高系统的可维护性与扩展性。Spring通过构造器、Setter或注解方式注入将依赖自动注入到Bean中，解除手动new的职责。

SpringBean的生命周期：
1. IoC 容器首先加载配置信息（XML、注解、Java Config），并将其解析为 BeanDefinition 对象，注册到 BeanDefinitionRegistry 中
2. 容器根据 BeanDefinition 调用构造器创建 Bean 实例。
3. Spring 使用反射或 setter 等方式填充属性。如果依赖其他 Bean，会先去获取并注入它们。
4. 若 Bean 实现了某些 Aware 接口，Spring 会回调它们。
5. 初始化前处理，初始化后处理和初始化回调条古
6. Bean此时以及就绪，可以被业务逻辑调用
7. 容器关闭时会被销毁