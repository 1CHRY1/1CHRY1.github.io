### 来自拼多多提前批一面面经

#### volatile能否保证数组元素的可见性?如何解决？

volatile 修饰数组变量只能保证引用本身的可见性，不能保证数组元素的可见性。修改元素值不会触发 volatile 的内存语义，可能被其他线程看到旧值。

解决方法包括：使用 AtomicIntegerArray 或 AtomicReferenceArray，用锁同步访问，或在 Java 9+ 使用 VarHandle 进行 volatile 元素访问。

```java
// 使用AtomicIntegerArray
AtomicIntegerArray arr = new AtomicIntegerArray(10);
arr.set(0, 42);
int value = arr.get(0);

// 手动加锁
synchronized(lock) {
    arr[0] = 42;
}

// 使用VarHandle (Java 9 以上)
// setVolatile可以将值直接写入主内存并刷新写缓冲区，getVolatile可以从主内存读取最新值，阻止重排序
import java.lang.invoke.MethodHandles;
import java.lang.invoke.VarHandle;
int[] arr = new int[10];
VarHandle ARRAY_HANDLE = MethodHandles.arrayElementVarHandle(int[].class);
ARRAY_HANDLE.setVolatile(arr, 0, 42);
int value = (int) ARRAY_HANDLE.getVolatile(arr, 0);
```

AtomicIntegerArray 用volatile语义操作数组元素，保证可见性和原子性；
synchronized 通过锁的内存语义保证修改对所有线程可见，并可实现多元素原子更新；
VarHandle（Java 9+）可对数组单个元素执行 volatile 读写，是更灵活的低层 API。