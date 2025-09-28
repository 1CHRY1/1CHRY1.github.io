### 自己找的

#### ThreadLocal的源码解析相关内容

set(T value)方法，用于设置线程本地变量在当前线程ThreadLocalMap中对应的值

```java
    public void set(T value) {
        //获取当前线程对象
        Thread t = Thread.currentThread(); 

        //获取当前线程的ThreadLocalMap 成员
        ThreadLocalMap map = getMap(t);

        //判断map是否存在
        if (map != null) 
        { 
                //value被绑定到threadLocal实例
            map.set(this, value);   
        }
        else
        {
            // 如果当前线程没有ThreadLocalMap成员实例
            // 创建一个ThreadLocalMap实例，然后，作为成员关联到t（thread实例）
            createMap(t, value);
        }
    }

    // 获取线程t的ThreadLocalMap成员
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    // 线程t创建一个ThreadLocalMap成员
    //并为新的Map成员设置第一个Key-Value对，Key为当前的ThreadLocal实例
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

```

get()方法，获取线程本地变量在当前线程中对应的值

```java
public T get() {
    //获得当前线程对象
    Thread t = Thread.currentThread();
    //获得线程对象的ThreadLocalMap 内部成员
    ThreadLocalMap map = getMap(t);

   // 如果当前线程的内部map成员存在
   if (map != null) {
        // 以当前threadlocal为Key,尝试获得条目
        ThreadLocalMap.Entry e = map.getEntry(this);
       // 条目存在
        if (e != null) {
            T result = (T)e.value;
            return result;
        }
    }
    // 如果当前线程对应map不存在
    //或者map存在，但是当前threadlocal实例没有对应的Key-Value，返回初始值
    return setInitialValue();
}

// 设置threadlocal关联的初始值并返回
private T setInitialValue() {
    //调用初始化钩子函数，获取初始值
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

remove()方法，用于在当前线程的ThreadLocalMap中移除线程本地变量对应的值

```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

