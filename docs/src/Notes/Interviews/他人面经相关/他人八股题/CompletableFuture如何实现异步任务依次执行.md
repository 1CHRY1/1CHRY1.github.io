### 来自拼多多提前批一面

#### CompletableFuture如何实现多个异步任务依赖执行？

CompletableFuture 实现多个异步任务依赖执行，主要是通过链式回调机制来完成的。每一个异步任务完成后，可以注册一个回调函数，通过 thenApply、thenCompose、thenCombine 等方法将前一个任务的结果传递给下一个任务，从而形成任务之间的依赖关系。这种回调链实际上构建了一棵有向图，每个节点表示一个任务，节点之间通过依赖关系连接。任务的执行依赖于其前置任务完成，CompletableFuture 会在前置任务完成时自动触发后续任务的执行，执行线程默认来自 ForkJoinPool.commonPool，也可以自定义线程池。最终，所有任务执行完成后，主线程可以通过 join 或 get 方法等待并获取最终结果。这种机制无需显式地管理线程或阻塞等待，有效提升了异步编程的效率和可读性。

---

多个异步任务依赖执行的方式有几种：
1. 串行依赖
```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> {
    return "Hello";
});

CompletableFuture<String> f2 = f1.thenApply(result -> {
    return result + " World";
});

System.out.println(f2.join()); // Hello World
```
使用thenApply或thenCompose，thenApply可以将上一个任务的结果作为参数，thenCompose可以返回另一个CompletableFuture

2. 并行依赖
```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "A");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "B");
CompletableFuture<String> f3 = CompletableFuture.supplyAsync(() -> "C");

CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2, f3);

all.thenRun(() -> {
    try {
        System.out.println(f1.get() + f2.get() + f3.get());
    } catch (Exception e) {
        e.printStackTrace();
    }
}).join();
```
使用allOf，适合将全部都完成再继续的任务

3. 竞争依赖
```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "First");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "Second");

CompletableFuture<Object> any = CompletableFuture.anyOf(f1, f2);
any.thenAccept(result -> System.out.println("Winner: " + result)).join();
```
谁先完成就用谁的结果，忽略其他未完成任务

最后使用thenCombine（合并两个任务结果），thenAcceptBoth（消费两个任务结果），runAfterBoth（两个任务都完成后执行最后一个无参任务），这里底层依赖ForkJoinPool执行异步任务，通过回调链在任务完成时触发后续任务，保证依赖顺序

因为JDK8为避免创建太多线程，所以默认使用公共线程池来调度异步任务，ForkJoinPool适合大量小任务且支持工作窃取机制，效率高，当然也可以通过传入自定义ExecutorService来指定线程池。

CompletableFuture 利用 ForkJoinPool 默认执行异步任务，通过每个任务阶段内部维护回调链（Completion List），来注册依赖任务。每个阶段完成后，自动触发其下游的回调逻辑，并将结果传递下去。这样就能实现多个异步任务的合并、依赖、串行与并行等操作，并且天然保证依赖顺序。