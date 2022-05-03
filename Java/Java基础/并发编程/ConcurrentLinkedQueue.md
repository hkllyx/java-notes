# `ConcurrentLinkedQueue`

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {

    /**
     * 基本不变原则：
     * 1. 队列中一定会存在且仅存在一个结点，该结点的next指向null，但tail不一定指向它
     * 2. 队列中所有item != null的节点，head一定能够到达；CAS设置node.item = null，意味着这个节点被删除
     */

    /**
     * 队列中第一个存活（没有被删除）的结点，能O(1)效率达到
     * 不变性（invariants）
     *     1. 所以存活的结点都能由head通过succ()抵达
     *     2. head != null
     *     3. (tmp = head).next != tmp || tmp != head (其实就是head.next != head)
     * 可变性（Non-invariants）
     *     1. head.item可能是null, 也可能不是null
     *     2. 允许tail滞后于head, 也就是调用succ()方法, 从head不可达tail
     */
    private transient volatile Node<E> head;

    /**
     * 队列中最后一个元素，唯一一个Node.next = null的结点，能O(1)效率达到
     * 不变性（invariants）
     *     1. tail节点通过succ()方法一定到达队列中的最后一个节点（node.next = null）
     *     2. tail != null
     * 可变性（Non-invariants）
     *     1. tail.item可能是null, 也可能不是null
     *     2. 允许tail滞后于head, 也就是调用succ()方法,从head不可达tail
     *     3. tail.next可能指向tail
     */
    private transient volatile Node<E> tail;

```

## 内部嵌套类`Node`

```java
private static class Node<E> {
    // volatile
    volatile E item;
    volatile Node<E> next;

    /**
     * Constructs a new node. Uses relaxed write because item can
     * only be seen after publication via casNext.
     */
    Node(E item) {
        // 使用Unsafe将对象放到内存中
        UNSAFE.putObject(this, itemOffset, item);
    }

    // CAS设置Item
    boolean casItem(E cmp, E val) {
        return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
    }

    // 延迟设置next
    void lazySetNext(Node<E> val) {
        UNSAFE.putOrderedObject(this, nextOffset, val);
    }

    // CAS设置next
    boolean casNext(Node<E> cmp, Node<E> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }

    // Unsafe mechanics

    private static final sun.misc.Unsafe UNSAFE;
    private static final long itemOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = Node.class;
            itemOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("item"));
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

## 构造方法

```java
// 默认
public ConcurrentLinkedQueue() {
    // 初始时，head和tail都指向一个item = null的结点
    head = tail = new Node<E>(null);
}

// 使用一个Collection生成
public ConcurrentLinkedQueue(Collection<? extends E> c) {
    Node<E> h = null, t = null;
    for (E e : c) {
        checkNotNull(e);
        Node<E> newNode = new Node<E>(e);
        // 头部结点为null，说明放入的是第一个元素，使该元素为头部和尾部
        if (h == null)
            h = t = newNode;
        // 头部不为null，则说明队列不为空，将元素放到尾部的后面，然后尾部后移
        else {
            t.lazySetNext(newNode);
            t = newNode;
        }
    }
    // 如果头部为null，说明Collection为空
    if (h == null)
        h = t = new Node<E>(null);
    head = h;
    tail = t;
}
```

## 特别的头部更新方法

导致一个结点指向自身的罪魁祸首。

```java
// 将head通过CAS设为p，如果设置成功，令原head的next指向自己
final void updateHead(Node<E> h, Node<E> p) {
    if (h != p && casHead(h, p))
        h.lazySetNext(h);
}

private boolean casHead(Node<E> cmp, Node<E> val) {
    return UNSAFE.compareAndSwapObject(this, headOffset, cmp, val);
}
```

## 加入元素

```java
public boolean offer(E e) {
    // 不能放入null
    checkNotNull(e);
    // 保证成结点
    final Node<E> newNode = new Node<E>(e);

    // 死循环，放入成功才结束
    for (Node<E> t = tail, p = t;;) {
        Node<E> q = p.next;
        // q == null表示是最后一个元素
        if (q == null) {
            // 将q的next使用设为插入结点，
            if (p.casNext(null, newNode)) {
                // 和其他线程竞争且竞争成功，将新结点插入了队列
                // 此时如果p != t，表明有其他线程插入了
                if (p != t) // hop two nodes at a time
                    casTail(t, newNode);  // Failure is OK.
                return true;
            }
            // Lost CAS race to another thread; re-read next
            // 设置q的next和其他线程竞争且竞争失败，重试
        }
        // p == q，即p = p.next，说明p是head，且在另一个线程被删除了
        else if (p == q)
            /* t != (t = tail) 可以认为是先比较t != tail，后令t = tail
             * 如果t != tail，说明另一个线程将tail改变了，且指向新的队尾，令p = t
             * 如果t = tail，说明另一个线程没有改变tail，
             * 此时tail = t = p，被删除了，不可达，所以令p等于head
             */
            p = (t != (t = tail)) ? t : head;
        else
            /* Check for tail updates after two hops.
             * 如果p != t，说明另一个线程在原tail后新增了元素（q），p执行新增元素，
             * 如果t != tail，说明另一个线程将tail改变了，且被指向新的队尾令p = t
             * 如果t = tail，说明另一个线程没有改变tail，而此时tail指向队尾，所以令p = q
             */
            p = (p != t && t != (t = tail)) ? t : q;
    }
}
```

## 移除元素

```java
public E poll() {
    restartFromHead:
    // 外部死循环，成功才退出
    for (;;) {
        // 内部死循环
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;

            // 如果item不为null，则p没有被移除，将p.item其设为null，表示移除了p
            if (item != null && p.casItem(item, null)) {
                // Successful CAS is the linearization point
                // for item to be removed from this queue.
                // p != h，说明p已经后移了
                if (p != h) // hop two nodes at a time
                    // 如果p是尾部结点，则让head为p，否则为p.nxet
                    updateHead(h, ((q = p.next) != null) ? q : p);
                return item;
            }
            // 如果q.next为null，说明q为尾部结点
            else if ((q = p.next) == null) {
                // 通过CAS将head设为p，令原h的next指向自身
                updateHead(h, p);
                return null;
            }
            // 如果p = q = p.next，说明p已经被别的线程移除了，跳出到外部循环，重新开始移除操作
            else if (p == q)
                continue restartFromHead;
            // 上述结果都不是，则将p后移，p = p.next
            else
                p = q;
        }
    }
}
```

[ConcurrentLinkedQueue源码分析 (基于Java 8)](https://www.jianshu.com/p/08e8b0c424c0)
