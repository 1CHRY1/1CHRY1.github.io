
# Vert.x 概述

## 1. 什么是 Vert.x？
Vert.x（http://vertx.io/）是一个基于 JVM 的轻量级、高性能应用平台，非常适合现代移动端后台、互联网和企业应用架构。

Vert.x 框架基于事件驱动和异步机制，依托全异步 Java 服务器 Netty，并扩展了许多其他特性，因其轻量、高性能和多语言支持而深受开发者青睐。

**官方定义**：Vert.x 是一个用于在 JVM 上构建响应式应用的工具包。

## 2. 优势
- **异步非阻塞**：Vert.x 是一个异步非阻塞框架，确保高性能。
- **多语言支持**：支持运行在 JVM 上的多种编程语言。
- **无需中间件**：基于 Netty，Vert.x 在构建 Web 项目时无需依赖中间件。
- **简洁生态**：与 Spring 庞大的生态系统不同，Vert.x 提供数据库操作、Redis 操作、Web 客户端操作以及 NoSQL 数据库等常用功能，简洁清新但功能足够。
- **为微服务而生**：提供多种组件来构建基于微服务的应用程序，通过 EventBus 实现服务间交互，并使用 Hazelcast 实现分布式功能。

## 3. 自带特性
- **多语言支持**：支持主流 JVM 语言（如 Java、JavaScript、Groovy、Ruby 和 Ceylon）编写 Vert.x 应用。
- **简单性**：代码完全基于异步事件驱动，类似 Node.js，开发者无需关注线程同步或锁，程序异步执行且通信无阻塞。
- **扩展性**：基于 Actor 模型，程序由独立单元组成，可水平扩展和动态替换，实现近乎无限的扩展。
- **并发性**：不同于 JDK 的并发机制，Vert.x 避免了锁、同步块和死锁等概念，内置三种线程池处理并发，开发者可专注于业务逻辑。
- **模块化**：内置强大的模块管理机制，Vert.x 业务逻辑可打包为模块，部署到 Maven 仓库或 Vert.x 模块注册中心，与主流开发流程无缝集成。
- **分布式消息传输**：基于分布式 EventBus 实现 Actor 模型，Vert.x 实例间通过内置通信接口实现消息同步与传输，类似 Erlang 的 ping-pong 自检消息。
- **WebSocket 支持**：兼容 WebSocket 协议和 SockJS。
- **丰富的 IO 支持**：异步模型支持 TCP、UDP、FileSystem、DNS、EventBus 和 SockJS。
- **异步无锁编程**：适合高并发移动互联网场景，Vert.x 的 Reactor 模型优于传统多线程模型，且无需高超的并发控制技巧。
- **成熟的生态系统**：隶属 Eclipse 基金会，异步驱动支持 Postgres、MySQL、MongoDB、Redis 等常用组件，并有多个生产环境应用案例。

## 4. Reactor 模式
与传统 Java 框架的多线程模型相比，Vert.x 的 Netty 是 Reactor 模式的 Java 实现。传统服务器（如 Tomcat）在 100 个并发长请求下容易阻塞，而 Vert.x 将长任务委托给其他线程执行，避免阻塞当前线程，与 Node.js 的原理类似。

## 5. 基本概念
### Verticle
Verticle 是 Vert.x 中可部署运行的最小代码块，可理解为最小化的业务处理引擎。一个应用程序可由单个 Verticle 或通过 EventBus 通信的多个 Verticle 组成。Verticle 部署后会调用其 `start` 方法开始业务逻辑处理，完成后调用 `stop` 方法销毁。

Verticle 运行在 Vert.x 实例中，一个 Vert.x 实例可承载多个 Verticle，每个 Vert.x 实例运行在独立的 JVM 实例中。一台服务器可运行一个或多个 Vert.x 实例（建议实例数与 CPU 核数相关）。

### Module
Vert.x 应用由一个或多个模块实现，每个模块由多个 Verticle 组成。模块类似于 Java 包，包含特定业务逻辑或可重用的公共服务。模块可发布到 Maven 仓库或 Vert.x 模块注册中心，以 zip 格式打包为二进制文件。这种模块化开发支撑了整个 Vert.x 生态系统。

### Vert.x 实例
Verticle 运行在 Vert.x 实例上，Vert.x 实例是运行在某个 JVM 上的 Vert.x 对象实例。多个 Verticle 可运行在同一个 Vert.x 实例上，而 Vert.x 实例可运行在单个或多个 JVM 上，通过分布式 EventBus 维持通信。Vert.x 实例通过 `vertx` 命令行启动，可指定实例数量。

### Event Loop（事件循环）
Event Loop 是 Vert.x 启动的事件处理线程，也是 Vert.x 的外部事件入口。一个 Vert.x 实例包含一个或多个事件循环线程，最大线程数为主机有效 CPU 核数。每个 Vert.x 实例内部维护的线程数与 CPU 核数一致，这些线程称为事件循环（Event Loop）。类似 iOS 的事件监控线程，Event Loop 捕获外部事件（如 Socket 数据读取、定时器触发、HTTP 请求等），并分配到指定的 Handler 接口处理。相关处理在同一个 Event Loop（即单线程）中完成，Handler 接口是线程同步的，符合 Reactor 模式。

**注意**：不要在 Event Loop 中编写阻塞代码，例如：
- `Thread.sleep()`
- `Object.wait()`
- `CountDownLatch.await()`

长时间的计算或 IO 阻塞（如 JDBC 查询）也应避免。Vert.x 提供阻塞任务处理机制，将耗时任务委托给长任务线程，避免阻塞 Event Loop。

### Standard Verticle
Standard Verticle 被分配到一个 Event Loop 线程，其 `start` 方法在该 Event Loop 中执行。调用 Core API 并传入处理器时，Vert.x 保证使用相同的 Event Loop 执行处理器。这确保 Verticle 实例的所有代码在同一 Event Loop 中运行（除非开发者创建新线程）。开发者可按单线程方式编写代码，Vert.x 负责线程和扩展问题，避免 `synchronized`、`volatile`、竞态条件和死锁等问题。

### Event Bus
Event Bus 是 Vert.x 的核心，用于集群中容器间及 Verticle 间的通信。

### Shared Data（共享数据）
Vert.x 提供简单的共享 Map 和 Set，解决 Verticle 间的数据共享问题。数据存储在不可变数据结构中，各实例通过 API 直接访问，避免通过消息传输共享数据。

### Worker Verticle（阻塞处理）
Worker Verticle 类似于 Standard Verticle，但由 Worker Pool 中的线程执行，而非 Event Loop。Worker Verticle 设计用于运行阻塞式代码，不会阻塞 Event Loop。也可通过内联阻塞式代码避免使用 Worker Verticle。

部署 Worker Verticle 示例：
```java
DeploymentOptions options = new DeploymentOptions().setWorker(true);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```
Worker Verticle 不会被多个线程同时执行，但可在不同时间由不同线程执行。

### Multi-threaded Worker Verticle
Multi-threaded Worker Verticle 类似于 Worker Verticle，但可由不同线程同时执行。需使用标准 Java 多线程技术保持状态一致性，适用于高级场景，大多数应用无需使用。

