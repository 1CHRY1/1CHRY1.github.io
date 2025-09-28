### 来自拼多多提前批一面

#### ThreadLocal内存泄漏的根本原因？JDK改进方案？

ThreadLocal是Java提供的一种线程局部变量解决方案。可以**为每个线程都维护一个独立的变量副本**，线程之间互不干扰，并且实现线程隔离，避免多线程并发访问共享变量导致数据竞态或同步问题。每个线程都有一个内部的ThreadLocalMap存放该线程所有ThreadLocal变量副本，ThreadLocal实例本身相当于一个键，可以在当前线程中查找对应的值。

ThreadLocal 是为每个线程维护变量副本的机制。多个线程可以共享同一个 ThreadLocal 实例，但每个线程会在自己独立的 ThreadLocalMap 中存储该变量的副本。

Thread & ThreadLocalMap结构：
```shell
Thread (长期存活，特别是线程池线程)
 └─ ThreadLocalMap (线程内存储结构)
       ├─ Entry[] table (数组结构)
            ├─ key: ThreadLocal 弱引用 (WeakReference<ThreadLocal<?>>)
            └─ value: 线程私有变量，强引用 Object
```

ThreadLocalMap中的key是对ThreadLocal实例的弱引用，如果程序不再持有某个ThreadLocal实例的key，GC会回收该弱引用对应的ThreadLocal对象，这也将导致ThreadLocalMap中对应的Entry的key变成null。

而虽然key被回收，但Entry仍是强引用value的，因此value仍存在内存中，且无法通过 ThreadLocalMap的正常接口访问或清理。

弱线程是线程池中的线程，则线程不会结束，也不会释放ThreadLocalMap，也就是弱不调用remove()函数，其中的value一直无法释放。

因此JDK的改进方案有几点：
1 对于key的泄漏，使用了弱引用来解决，ThreadLocal类中的Entry继承了弱引用类，并在key中使用。因此当外部不再强引用这个ThreadLocalMap中的key时，该弱引用对象会被清除。
2 对于value的泄漏，ThreadLocal实现了两大清理方式：**探测式清理**和**启发式清理**，以及**手动清理**
- 探测式清理，也就是当线程调用get,set,remove方法时会触发对ThreadLocalMap的清理，移除那些已经被垃圾回收的key键及其对应的value值。
- 启发式清理，也就是在set方法中有一个阈值，默认为Entry数组长度的1/4，当ThreadLocalMap中的Entry被删除并且剩余Entry数量大于这个阈值时会触发启发式清理操作。
- 手动清理：即使用完ThreadLocal后显示调用remove()方法确保不再需要的值能被完全回收


