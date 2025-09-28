### 来自网络面经

#### synchronized锁和ReentrantLock的区别是什么？

**区别一、用法不同**
sychronized可以用来修饰普通方法、静态方法和代码块，而ReentrantLock只能用于代码块。

**区别二、获取锁和释放锁的方式不同**
synchronized会自动加锁和释放锁，而ReentrantLock需要手动加锁和释放锁

**区别3：锁类型不同**
synchronized 属于非公平锁，而 ReentrantLock 既可以是公平锁也可以是非公平锁。默认情况下 ReentrantLock 为非公平锁，使用`new ReentrantLock(true)`可以创建一个公平锁。

**区别4：响应中断不同**
ReentrantLock 可以使用 lockInterruptibly 获取锁并响应中断指令，而 synchronized 不能响应中断，也就是如果发生了死锁，使用 synchronized 会一直等待下去，而使用 ReentrantLock 可以响应中断并释放锁，从而解决死锁的问题

ReeantrantLock可以使用lockInterruptibly()实现一个可以响应中断的锁，并用thread.interrupt()执行中断。

**区别5：底层实现不同**
synchronized是JVM 层面通过监视器(Monitor)实现的，而 ReentrantLock 是通过 AQS(AbstractQueuedSynchronizer)程序级别的 API 实现。