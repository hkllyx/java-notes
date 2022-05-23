# `PriorityQueue`

```java
public class PriorityQueue<E> extends AbstractQueue<E>
                              implements java.io.Serializable {
    ...
    private static final int DEFAULT_INITIAL_CAPACITY = 11;
    /**
     * 优先级队列使用二叉平衡堆表示：queue[n]的子结点为queue[2n + 1]和queue[2n + 2]。
     * 如果队列不为空，最小值的元素位于queue[0]（堆顶）。
     */
    transient Object[] queue; // non-private to simplify nested class access

    /** 队列元素的个数 */
    private int size = 0;

    /** 比较器，不指定时为null */
    private final Comparator<? super E> comparator;

    /**
     * The number of times this priority queue has been
     * <i>structurally modified</i>.  See AbstractList for gory details.
     */
    transient int modCount = 0; // non-private to simplify nested class access
```

- 继承自`AbstractQueue`：支持队列相关操作
- 实现了`java.io.Serializable`：支持序列化
- 内部使用数组来保存元素，且使用数组构建二叉平衡堆（最小堆）
- 默认初始容量为`11`

## 构造器

```java
public PriorityQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}

// 指定初始容量
public PriorityQueue(int initialCapacity) {
    this(initialCapacity, null);
}

// 指定比较器
public PriorityQueue(Comparator<? super E> comparator) {
    this(DEFAULT_INITIAL_CAPACITY, comparator);
}

// 指定初始容量和比较器
public PriorityQueue(int initialCapacity,
                     Comparator<? super E> comparator) {
    // Note: This restriction of at least one is not actually needed,
    // but continues for 1.5 compatibility
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}

// 使用其他结合构建
@SuppressWarnings("unchecked")
public PriorityQueue(Collection<? extends E> c) {
    if (c instanceof SortedSet<?>) {
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
        this.comparator = (Comparator<? super E>) ss.comparator();
        initElementsFromCollection(ss);
    }
    else if (c instanceof PriorityQueue<?>) {
        PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
        this.comparator = (Comparator<? super E>) pq.comparator();
        initFromPriorityQueue(pq);
    }
    else {
        this.comparator = null;
        initFromCollection(c);
    }
}

@SuppressWarnings("unchecked")
public PriorityQueue(PriorityQueue<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initFromPriorityQueue(c);
}

@SuppressWarnings("unchecked")
public PriorityQueue(SortedSet<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initElementsFromCollection(c);
}
```

## 入队

```java
public boolean offer(E e) {
    // 不能存放null
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    // 如果超出数组容量，扩容
    if (i >= queue.length)
        grow(i + 1);
    // 队列长度+1
    size = i + 1;
    // 如果队列长度为0，说明加入的是第一个元素，放在第一个
    // 否则将元素放到最后，然后进行调整
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e);
    return true;
}

// 扩容，参见ArrayList
private void grow(int minCapacity) {
    int oldCapacity = queue.length;
    // Double size if small; else grow by 50%
    int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                     (oldCapacity + 2) :
                                     (oldCapacity >> 1));
    // overflow-conscious code
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    queue = Arrays.copyOf(queue, newCapacity);
}

// siftUp调整
// 将x放到数组的k位
private void siftUp(int k, E x) {
    // 功能类似，只是比较时所用比较工具不同
    if (comparator != null)
        siftUpUsingComparator(k, x);
    else
        siftUpComparable(k, x);
}


@SuppressWarnings("unchecked")
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        // 找到父结点
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        // 如果当前插入结点大于当前位置的父结点，停止循环
        if (comparator.compare(x, (E) e) >= 0)
            break;
        // 如果小于父结点，则将父结点保存到当前位置
        queue[k] = e;
        k = parent;
    }
    // 循环结束后，插入结点的值大于父结点的值
    queue[k] = x;
}
```

## 出队

```java
@SuppressWarnings("unchecked")
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    // 返回第一个元素
    E result = (E) queue[0];
    E x = (E) queue[s];
    // 将最后一个元素设为null
    queue[s] = null;
    // 如果最后一个元素不为0，即出队前元素不止一个，进行调整
    if (s != 0)
        siftDown(0, x);
    return result;
}

// siftDown调整
private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);
    else
        siftDownComparable(k, x);
}


@SuppressWarnings("unchecked")
private void siftDownComparable(int k, E x) {
    // 调整的结点，出队时为最后一个结点
    Comparable<? super E> key = (Comparable<? super E>)x;
    // 一半的位置，该位置的子结点时最后元素的位置
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {
        // 获取子结点，初始时为左子结点
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        // 获取右子结点位置
        int right = child + 1;
        // 如果右子结点存在，且左子结点元素小于右子结点元素
        if (right < size
            && ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            // 令子结点为右子结点
            c = queue[child = right];
        // 如果调整结点小于子结点，停止循环
        if (key.compareTo((E) c) <= 0)
            break;
        // 否则令调整结点所占的位置等于子结点，并将调整结点的位置设为子结点位置
        queue[k] = c;
        k = child;
    }
    // 调整之后调整结点的子结点大于调整结点
    queue[k] = key;
}
```
