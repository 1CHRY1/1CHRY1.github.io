### 来自拼多多提前批一面

#### 如何用CAS实现一个无锁栈？ABA问题如何规避？

```java
public class LockFreeStack<E> {

    private static class Node<E> {
        final E item;
        Node<E> next;

        Node(E item) {
            this.item = item;
        }
    }

    private final AtomicReference<Node<E>> top = new AtomicReference<>();

    public void push(E item) {
        Node<E> newNode = new Node<>(item);
        Node<E> oldTop;
        do {
            oldTop = top.get();
            newNode.next = oldTop;
            // 若CAS失败则说明其他线程同时修改了栈顶，需要重试
        } while (!top.compareAndSet(oldTop, newNode));
    }

    public E pop() {
        Node<E> oldTop;
        Node<E> newTop;
        do {
            oldTop = top.get();
            if (oldTop == null)
                return null;
            newTop = oldTop.next;
        } while (!top.compareAndSet(oldTop, newTop));
        return oldTop.item;
    }

}
```

ABA问题描述：当线程读取top=A，准备CAS更新为B，并记录了此时B的地址；在此期间另一个线程把A和B弹出(Top变成C)，又重新压入旧的A节点(Top变成A)，此时CAS看到top还是A，认为没变，实际上栈结构已经被改动而导致数据结果不一致（CAS修改了栈顶指针，让栈链表与C断开了引用关系，虽然某个线程的局部变量还持有A或C，但在全局数据结构中这些节点已经不可达）。

可以通过禁止节点复用并且每次push都new一个节点，使用AtomicStampedReference每次修改版本号，或者添加时间戳的方式避免对象的复用。

