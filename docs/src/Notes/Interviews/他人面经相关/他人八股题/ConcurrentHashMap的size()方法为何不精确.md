### 来自拼多多提前批一面

#### ConcurrentHashMap的size()方法为何不精确？替代方案？

因为ConcurrentHashMap在1.7前使用的分段锁，1.7后使用节点级锁，使用size()方法不会全局加锁。在1.7中是遍历所有Segment的计数值并相加，1.8中是遍历所有bin的count并相加，若统计时不阻塞写的操作可能在遍历过程中会有线程在put或者remove导致数据的变化。

代替方案的话可以在访问size()的时候对ConcurrentHashMap外部加锁，或者在put和remove的时候手动维护一个AtomicLong计数器保证精确，或者将当前chmp的数据拷贝出来统计这个时刻的数据量，使用`map.entrySet().stream().count()方法`