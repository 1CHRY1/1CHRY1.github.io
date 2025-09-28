#### Java 8中Stream的并行处理原理？ForkJoinPool工作窃取机制？

Java 8 并行流基于 Spliterator 拆分数据源、ForkJoinPool 分配任务执行、并在多线程中并行计算再合并结果。
ForkJoinPool 使用工作窃取机制：每个线程有自己的双端队列，本地任务从队尾取，其他线程空闲时从队头窃取任务执行，这样既能保持本地任务的缓存友好性，又能平衡负载，提高 CPU 利用率。

---

Java的Stream API是一种流式操作集合的方法，提供了丰富的集合处理操作如过滤、映射、排序等，还支持延迟加载和并行处理，能够简化代码并提高编码效率。

Java8的stream其实是一个声明式数据处理流水线，核心思想有两个：
- 内部迭代(Internal Iteration)：只声明要做什么，具体遍历、并行、优化由Stream内部负责。
- 惰性求值(Lazy Evaluation)：中间操作不立即执行，只有终结操作时才触发整个流水线。

| 阶段       | 作用            | 示例                            |
| -------- | ------------- | ----------------------------- |
| **数据源**  | 提供流的输入        | 集合、数组、IO、生成器等                 |
| **中间操作** | 转换、过滤、排序等（惰性） | `filter()`、`map()`、`sorted()` |
| **终结操作** | 触发执行，产生结果（急性） | `collect()`、`forEach()`       |

Stream底层是由java.util.stream.Stream接口实现的，定义了中间/终结操作的方法，实现类有：
- AbstractPipeline：抽象基类，所有流实现都继承它，维护了一个流水线链表结构，记录操作顺序；
- ReferencePipeline：针对引用类型的流实现
- IntPipeline/LongPipeline/DoublePipeline：针对基本类型流的优化实现（避免装箱拆箱）

流的执行逻辑：
```java
// 创建，内部创建Spliterator（分割迭代器）描述数据源的遍历方式
Stream<String> s = list.stream();

// 中间操作，每个中间操作都返回一个新的流水线节点，但不会执行
Stream<String> s2 = s.filter(x -> x.length() > 3).map(String::toUpperCase);

// 终结操作，从流水线的源头开始，数据被 Spliterator 拆分、逐个元素按顺序流过中间操作，最后被收集到结果中。所有中间操作会被合并成一个遍历过程，避免多次循环
List<String> result = s2.collect(Collectors.toList());

// 并行流处理，可以将数据源递归拆分成两半，拆分出来的任务提交到ForkJoinPool.commonPool()中，各个线程并行执行流水线中间操作并在最终阶段合并
list.parallelStream()
```
Spliterator 的核心作用是在普通流中顺序遍历，在并行流中递归trySplit拆分数据源，也会保存一些特性来优化执行计划

总结Stream执行路径：
顺序流：
```java
list.stream().filter(...).map(...).collect(...) 
```
- stream() → 创建 Spliterator + 第一个流水线节点。
- filter() → 创建第二个流水线节点（链表结构）。
- map() → 创建第三个流水线节点。
- collect()：
调用 AbstractPipeline.evaluate()。
创建遍历任务 → 从 Spliterator 拿元素 → 依次执行 filter、map → 传给 Collector。
收集结果并返回。

并行流：
```java
list.parallelStream().filter(...).map(...).collect(...)
```
- parallelStream() → 创建 Spliterator。
- filter() / map() → 构建流水线链表。
- collect()：
调用 AbstractPipeline.evaluate()。
Spliterator.trySplit() 把数据切成多份。
为每份数据创建子任务（ForkJoinTask）放入 ForkJoinPool.commonPool()。
各个线程并行执行流水线操作。
最后合并子任务结果。

Java8并行流基于三大核心：
- Spliterator：用于分割数据源（split + iterate），把集合拆分成更小的部分，直到每个部分适合单线程处理。
- ForkJoinPool
并行流的默认线程池（ForkJoinPool.commonPool()），默认线程数 = CPU核心数 - 1。
每个任务（task）由 ForkJoinPool 中的工作线程（worker）执行。
- ForkJoinTask（RecursiveTask/RecursiveAction）
并行流的任务会被包装成 ForkJoinTask，通过递归切分再合并结果。

执行流程为：先创建Spliterator将数据源拆分成多块，将任务提交到ForkJoinPool使每块数据对应一个子任务，ForkJoinPool中多个工作线程并行处理子任务后，将所有线程处理完的结果合并。

这里的任务执行适合CPU密集型且数据量大的场景，小任务的并行流使用反而会有额外线程调度开销，且不适合强顺序依赖的任务。

ForkJoinPool的核心结构是每个线程有一个双端队列存放任务，提交的子任务放在自己队列的队尾，执行任务时，从队尾取任务（pop）。窃取任务时，从别人的队列队头偷（steal）。

流程上看，每个工作线程先处理自己队列中的任务，若自己的队列空了，就随机选择另一个线程的队列，从队头偷一个任务来执行，这样可以减少线程空闲，提高 CPU 利用率，避免某个线程任务过多而另一个线程空闲。

```java
public final ForkJoinTask<V> fork() {
    Thread t;
    // 如果当前线程是 ForkJoinWorkerThread
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
        // 放进当前线程的工作队列（队尾）
        ((ForkJoinWorkerThread)t).workQueue.push(this);
    else
        // 如果不是池里的线程（例如 main 线程）
        ForkJoinPool.common.externalPush(this);
    return this;
}

class MyTask extends RecursiveTask<Integer> {
    int start, end;

    MyTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if (end - start <= 10) {
            // 任务足够小，直接计算
            int sum = 0;
            for (int i = start; i <= end; i++) sum += i;
            return sum;
        } else {
            // 任务太大，拆成两半
            int mid = (start + end) / 2;
            MyTask left = new MyTask(start, mid);
            MyTask right = new MyTask(mid + 1, end);

            right.fork();          // 异步执行右半
            int leftResult = left.compute(); // 递归处理左半
            int rightResult = right.join();  // 等右半完成
            return leftResult + rightResult;
        }
    }
}
```

为什么本地LIFO而窃取是FIFO呢？因为需要将自己执行的任务刚刚fork（将大任务拆分成小任务）出来的子任务一般更可能命中CPU缓存，且防止窃取到刚刚fork出来、还依赖其他任务结果的任务，减少竞争和依赖冲突

