# `ConcurrentHashMap`

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    private static final long serialVersionUID = 7249069246763182397L;

    ...
```

- 实现了`AbstractMap`：支持映射操作
- 线程安全：通过自旋锁和CAS

## 常量

```java
/**
 * 最大的可能的Hash表容量
 * 这个值必须正好是1<<30，以保持在Java数组分配和索引边界内，以两个表大小的幂
 * 而且还需要进一步使用，因为32位Hash的前2位需用于控制目的
 */
private static final int MAXIMUM_CAPACITY = 1 << 30;

/** Hash表默认容量，必须是2次幂，从1~MAXIMUM_CAPACITY */
private static final int DEFAULT_CAPACITY = 16;

/** 最大的数组长度，用于toArray及相关方法 */
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/** Hash表默认并发级别。未使用，兼容以前版本 */
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

/**
 * Hash表默认加载因子，只在表初始话时才可改变。
 * 通常不直接乘以浮点值，而是用类似{@code n - (n >>> 2)}的表达式来调整阈值
 */
private static final float LOAD_FACTOR = 0.75f;

/**
 * Hash桶（Hash表的一个结点）结构使用红黑树而非链表（树化）的阈值。
 * 当向桶中添加至少阈值个元素的时候才会出现链表转化为树，阈值应> 2且<= 8
 * 以便和树收缩为链表时的判断相啮合
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * Hash桶结构resize时将红黑树收缩（拆分）为链表（取消树化）的阈值。
 * 必须< TREEIFY_THRESHOLD
 * 且在<= 6的情况下进行收缩判断
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 桶树化树化需要的最小表容量
 * 如果未达到这个容量，应当调整表容量来减少Hash冲突
 * 值应当>= 4 * TREEIFY_THRESHOLD以避免调整表容量和树化阈值之间的冲突
 */
static final int MIN_TREEIFY_CAPACITY = 64;

/**
 * 每个转移步骤的最小rebinnings数
 * 细分范围来允许有多个resizer线程，这个值作为一个下界来避免resizer出现过多内存争用
 * Minimum number of rebinnings per transfer step. Ranges are
 * 值应当>= DEFAULT_CAPACITY
 */
private static final int MIN_TRANSFER_STRIDE = 16;

/** 在sizeCtl中用于生成stamp的bit数。对于32位数组必须>= 6 */
private static int RESIZE_STAMP_BITS = 16;

/** 调整容量的最大线程数，必须符合32 - RESIZE_STAMP_BITS */
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

/** 在sizeCtl中记录大小的bit移位 */
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

/* Node累哈希字段的编码（encoding） */
static final int MOVED     = -1; // hash for forwarding nodes
static final int TREEBIN   = -2; // hash for roots of trees
static final int RESERVED  = -3; // hash for transient reservations
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

/** CPU核心数，对某些大小设置限制 */
static final int NCPU = Runtime.getRuntime().availableProcessors();

/** 序列化兼容 */
private static final ObjectStreamField[] serialPersistentFields = {
    new ObjectStreamField("segments", Segment[].class),
    new ObjectStreamField("segmentMask", Integer.TYPE),
    new ObjectStreamField("segmentShift", Integer.TYPE)
};
```

## 字段

```java
/**
 * 表：Hash桶数组，大小总是2的幂。由迭代器直接访问
 * 懒加载，在第一次插入元素时初始话
 */
transient volatile Node<K,V>[] table;

/** 下一个表，仅在调整表容量时非null */
private transient volatile Node<K,V>[] nextTable;

/**
 * 基本计数值
 * 主要在没有争用时使用，但也可以在表初始化竞争时用作回退，使用CAS更新
 */
private transient volatile long baseCount;

/**
 * 用于控制表的初始话和容量调整
 * 为负数表示正则初始话或者容量调整，-1表示初始话，否则为-（1 + 正在调整大小的线程）
 * 当表为null时，默认是0或者为创建时要使用的初始
 * 表初始化后，保存要调整表大小的下一个元素计数值
 */
private transient volatile int sizeCtl;

/** 调整大小时要拆分的下一个表索引（+1）。 */
private transient volatile int transferIndex;

/** 调整大小货创建CounterCells时用的自旋锁（通过CAS） */
private transient volatile int cellsBusy;

/** 计数单元数组，非空时大小为2的幂 */
private transient volatile CounterCell[] counterCells;

// 视图
private transient KeySetView<K,V> keySet;
private transient ValuesView<K,V> values;
private transient EntrySetView<K,V> entrySet;
```

## 静态工具方法

```java
/** 分散hash值 */
static final int spread(int h) {
    // 低16未和高16位进行异或，在和HASH_BITS取与
    return (h ^ (h >>> 16)) & HASH_BITS;
}

//
// 返回给定所需容量的表大小的2次幂
private static final int tableSizeFor(int c) {
    int n = c - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

/** 如果对象实现了Comparable接口，返回其类对象，否则返回null */
static Class<?> comparableClassFor(Object x) {
    if (x instanceof Comparable) {
        Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
        if ((c = x.getClass()) == String.class) // bypass checks
            return c;
        if ((ts = c.getGenericInterfaces()) != null) {
            for (int i = 0; i < ts.length; ++i) {
                if (((t = ts[i]) instanceof ParameterizedType) &&
                    ((p = (ParameterizedType)t).getRawType() == Comparable.class) &&
                    (as = p.getActualTypeArguments()) != null &&
                    as.length == 1 && as[0] == c) // type arg is c
                    return c;
            }
        }
    }
    return null;
}

/** 如果x和k属于同一个类的实例，返货k和x的比较值，否则返回0 */
@SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
static int compareComparables(Class<?> kc, Object k, Object x) {
    return (x == null || x.getClass() != kc ? 0 :
            ((Comparable)k).compareTo(x));
}
```

## 静态Hash桶元素访问方法

```java
// 获取第i位
@SuppressWarnings("unchecked")
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

// CAS自旋如果第i位为c，则替换为v
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}

static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}

// Unsafe机制
private static final sun.misc.Unsafe U;
private static final long SIZECTL;
private static final long TRANSFERINDEX;
private static final long BASECOUNT;
private static final long CELLSBUSY;
private static final long CELLVALUE;
private static final long ABASE;
private static final int ASHIFT;

static {
    try {
        U = sun.misc.Unsafe.getUnsafe();
        Class<?> k = ConcurrentHashMap.class;
        SIZECTL = U.objectFieldOffset
            (k.getDeclaredField("sizeCtl"));
        TRANSFERINDEX = U.objectFieldOffset
            (k.getDeclaredField("transferIndex"));
        BASECOUNT = U.objectFieldOffset
            (k.getDeclaredField("baseCount"));
        CELLSBUSY = U.objectFieldOffset
            (k.getDeclaredField("cellsBusy"));
        Class<?> ck = CounterCell.class;
        CELLVALUE = U.objectFieldOffset
            (ck.getDeclaredField("value"));
        Class<?> ak = Node[].class;
        ABASE = U.arrayBaseOffset(ak);
        int scale = U.arrayIndexScale(ak);
        if ((scale & (scale - 1)) != 0)
            throw new Error("data type scale not a power of two");
        ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
    } catch (Exception e) {
        throw new Error(e);
    }
}
```

## 内部静态类`Node`

Hash表中桶结构为链表时的结点。

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;

    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }

    public final K getKey()       { return key; }
    public final V getValue()     { return val; }
    // hash值时key和val的hash值异或结果
    public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
    public final String toString(){ return key + "=" + val; }
    public final V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    public final boolean equals(Object o) {
        Object k, v, u; Map.Entry<?,?> e;
        return ((o instanceof Map.Entry) &&
                (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                (v = e.getValue()) != null &&
                (k == key || k.equals(key)) &&
                (v == (u = val) || v.equals(u)));
    }

    /**
     * Virtualized support for map.get(); overridden in subclasses.
     */
    Node<K,V> find(int h, Object k) {
        Node<K,V> e = this;
        if (k != null) {
            do {
                K ek;
                if (e.hash == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
            } while ((e = e.next) != null);
        }
        return null;
    }
}
```

## 内部静态类`TreeNode`

Hash表中桶结构为树时的结点。

```java
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;

    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }

    Node<K,V> find(int h, Object k) {
        return findTreeNode(h, k, null);
    }

    /**
     * Returns the TreeNode (or null if not found) for the given key
     * starting at given root.
     */
    final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
        if (k != null) {
            TreeNode<K,V> p = this;
            do  {
                int ph, dir; K pk; TreeNode<K,V> q;
                TreeNode<K,V> pl = p.left, pr = p.right;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.findTreeNode(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
        }
        return null;
    }
}
```

## 构造器

```java
// 默认，初始容量为16（第一次resize时调整）
public ConcurrentHashMap() {
}

// 指定初始容量
public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}

// 用其他Map生成
public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
    this.sizeCtl = DEFAULT_CAPACITY;
    putAll(m);
}

// 指定初始容量和加载因子
public ConcurrentHashMap(int initialCapacity, float loadFactor) {
    this(initialCapacity, loadFactor, 1);
}

// 指定初始容量、加载因子和并发几倍
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins
        initialCapacity = concurrencyLevel;   // as estimated threads
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}
```

## 添加元素

```java
// 直接添加
public V put(K key, V value) {
    // 调用内部方法
    return putVal(key, value, false);
}

final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 不接受null key和value
    if (key == null || value == null) throw new NullPointerException();
    // 计算key的hash
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

## 删除元素

```java
// 移除指定位置元素
public E remove(int index) {
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        // 如果移除的元素是最后一个，不用移动其他元素，直接复制&替换
        // 否则还需要指定位置之后的元素
        if (numMoved == 0)
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            setArray(newElements);
        }
        return oldValue;
    } finally {
        // 解锁
        lock.unlock();
    }
}

// 移除指定元素1次
public boolean remove(Object o) {
    // 取快照
    Object[] snapshot = getArray();
    // 移除元素位置
    int index = indexOf(o, snapshot, 0, snapshot.length);
    return (index < 0) ? false : remove(o, snapshot, index);
}

private boolean remove(Object o, Object[] snapshot, int index) {
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] current = getArray();
        int len = current.length;
        // 此时array已有改动
        if (snapshot != current) findIndex: {
            int prefix = Math.min(index, len);
            // 共有部分存在指定元素，修正index
            for (int i = 0; i < prefix; i++) {
                if (current[i] != snapshot[i] && eq(o, current[i])) {
                    index = i;
                    break findIndex;
                }
            }
            // 不存在相同元素
            if (index >= len)
                return false;
            // index元素不变
            if (current[index] == o)
                break findIndex;
            // 查找剩余不同部分是否存在指定元素
            index = indexOf(o, current, index, len);
            if (index < 0)
                return false;
        }
        // 复制&替换
        Object[] newElements = new Object[len - 1];
        System.arraycopy(current, 0, newElements, 0, index);
        System.arraycopy(current, index + 1,
                         newElements, index,
                         len - index - 1);
        setArray(newElements);
        return true;
    } finally {
        // 解锁
        lock.unlock();
    }
}

// 批量删除
public boolean removeAll(Collection<?> c) {
    if (c == null) throw new NullPointerException();
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        if (len != 0) {
            // temp array holds those elements we know we want to keep
            // 新建数组，添加c中不存在的元素（不需要删除的元素）
            int newlen = 0;
            Object[] temp = new Object[len];
            for (int i = 0; i < len; ++i) {
                Object element = elements[i];
                if (!c.contains(element))
                    temp[newlen++] = element;
            }
            // 复制&替换array
            if (newlen != len) {
                setArray(Arrays.copyOf(temp, newlen));
                return true;
            }
        }
        return false;
    } finally {
        // 解锁
        lock.unlock();
    }
}

// 保存指定集合元素（交集）
public boolean retainAll(Collection<?> c) {
    if (c == null) throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        if (len != 0) {
            // temp array holds those elements we know we want to keep
            // 新建数组，添加c中存在的元素（需要保存的元素）
            int newlen = 0;
            Object[] temp = new Object[len];
            for (int i = 0; i < len; ++i) {
                Object element = elements[i];
                if (c.contains(element))
                    temp[newlen++] = element;
            }
            if (newlen != len) {
                setArray(Arrays.copyOf(temp, newlen));
                return true;
            }
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```
