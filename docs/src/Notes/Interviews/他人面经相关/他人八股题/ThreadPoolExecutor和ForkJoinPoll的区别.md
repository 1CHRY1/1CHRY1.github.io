### 自己编的

#### ForkJoinPool与ThreadPoolExecutor的比较

ForkJoinPool 采用的是分治任务模型，而 ThreadPoolExecutor 采用的是简单并行任务模型。这两种任务模型各有优劣，ForkJoinPool 的优势在于能够处理分治任务，而 ThreadPoolExecutor 的优势在于能够处理简单并行任务。

以计算1到1亿的和为例：
```java
public interface Calculator { 
     /** 
      * 把传进来的所有numbers 做求和处理 
      * 
      * @param numbers 
      * @return 总和 
      */ 
     long sumUp(long[] numbers); 
}

// 使用ThreadPoolExecutor线程池实现
public class ExecutorServiceCalculator implements Calculator { 
 
    private int parallism; 
    private ExecutorService pool; 
 
    public ExecutorServiceCalculator() { 
        // CPU的核心数 默认就用cpu核心数了 
        parallism = Runtime.getRuntime()availableProcessors();  
        pool = ExecutorsnewFixedThreadPool(parallism); 
    } 
 
    // 1. 处理计算任务的线程 
    private static class SumTaskimplements Callable<Long> { 
        private long[] numbers; 
        private int from; 
        private int to; 
 
        public SumTask(long[] numbers,int from, int to) { 
            this.numbers = numbers; 
            this.from = from; 
            this.to = to; 
        } 
 
        @Override 
        public Long call() { 
            long total = 0; 
            for (int i = from; i <= to; ++) { 
                total += numbers[i]; 
            } 
            return total; 
        } 
    } 
 
    // 2. 核心业务逻辑实现 
    @Override 
    public long sumUp(long[] numbers) { 
        List<Future<Long>> results = newArrayList<>(); 
 
        // 2.1 数字拆分 
        // 把任务分解为 n 份，交给 n 个线处理   4核心 就等分成4份呗 
        // 然后把每一份都扔个一个SumTask程 进行处理 
        int part = numbers.length /parallism; 
        for (int i = 0; i < parallism; ++) { 
            int from = i * part; //开始置 
            int to = (i == parallism -1) ? numbers.length - 1 : (i+ 1) * part - 1; //结束位置 
 
            //扔给线程池计算 
            results.add(pool.submit(newSumTask(numbers, from,to))); 
        } 
 
        // 2.2 阻塞等待结果 
        // 把每个线程的结果相加，得到最终果 get()方法 是阻塞的 
        // 优化方案：可以采CompletableFuture来优化  JDK1.8新特性 
        long total = 0L; 
        for (Future<Long> f : results) { 
            try { 
                total += f.get(); 
            } catch (Exception ignore) { 
            } 
        } 
 
        return total; 
    } 
}

// 使用ForkJoinPool的实现
public class ForkJoinCalculator implements Calculator{

    private ForkJoinPool pool;

    private static class SumTask extends RecursiveTask<Long> {
        private long[] numbers;
        private int from;
        private int to;

        public SumTask(long[] numbers, int from, int to) {
            this.numbers = numbers;
            this.from = from;
            this.to = to;
        }

        @Override
        protected Long compute() {
            if (to - from < 6) {
                long total = 0;
                for (int i = from; i <= to; i++) {
                    total += numbers[i];
                }
                return total;
            } else {
                int middle = (from + to) / 2;
                SumTask taskLeft = new SumTask(numbers, from, middle);
                SumTask taskRight = new SumTask(numbers, middle+1, to);
                taskLeft.fork();
                taskRight.fork();
                return taskLeft.join() + taskRight.join();
            }
        }
    }

    public ForkJoinCalculator() {
        pool = new ForkJoinPool();
    }

    @Override
    public long sumUp(long[] numbers) {
        Long result = pool.invoke(new SumTask(numbers, 0, numbers.length-1));
        return result;
    }

    @Override
    public void shutdown() {
        pool.shutdown();
    }

}
```