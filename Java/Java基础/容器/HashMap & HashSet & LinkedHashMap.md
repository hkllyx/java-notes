# `HashMap` & `HashSet` & `LinkedHashMap`

## `HashMap`

```java
public class HashMap<K,V> extends AbstractMap<K,V>
                          implements Map<K,V>,
                                     Cloneable,
                                     Serializable {
    ...
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    static final int MAXIMUM_CAPACITY = 1 << 30;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    static final int TREEIFY_THRESHOLD = 8;
    static final int UNTREEIFY_THRESHOLD = 6;
    static final int MIN_TREEIFY_CAPACITY = 64;
    ...
    // hash表，存放hash桶（值保存了桶的第一个元素）。 长度是2的n次幂，或者初始化时为长度0
    transient Node<K,V>[] table;
    transient Set<Map.Entry<K,V>> entrySet;
    transient int size;
    transient int modCount;
    // 哈希表内元素数量的阈值，当哈希表内元素数量超过阈值时，会发生扩容，即resize()。
    int threshold;
    // 用于计算哈希表元素数量的阈值。threshold = 哈希表.length * loadFactor;
    final float loadFactor;
    ...
}
```

- 实现了`Map`并继承自`AbstractMap`：支持映射相关操作
- 实现了`Cloneable`：支持拷贝
- 实现了`Serializable`：支持序列化
- 默认初始容量为`16`，默认加载因子为`0.75f`
- 使用静态内部`Node`数组保存hash表（拉链法）
- 当碰撞比较大时，hash桶结构转变成红黑树来保证时间效率

### 静态内部类`HashMap.Node`

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash; // 节点的hash值
    final K key;    // 键
    V value;        // 值
    Node<K,V> next; // 链表后置结点

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    // 一个结点的hash值 = 键的hash值和值的hash值进行异或得来
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    // 修改值，返回旧值
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        // 如果比较对象是Map.Entry或其子孙类实例
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            // 比较二者的键和值是否相等
            if (Objects.equals(key, e.getKey())
                && Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

### 构造器

```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

// 指定初始值和加载因子
public HashMap(int initialCapacity, float loadFactor) {
    // 初始容量不能小于0
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +initialCapacity);
    // 指定的初始容量可以大于最大容量，但实际上会替换成最大容量MAXIMUM_CAPACITY
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 加载因子不能等于或小于0
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    // 阈值必须是2的n次幂
    this.threshold = tableSizeFor(initialCapacity);
}

// 只指定初始容量
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 使用另一个Map来生成
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

// 计算表容量，保证容量为>= cap的最小2的n次幂
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    // 计算结束后二进制形式从第一个非0位开始全为1，再加1就是2的n次幂
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### 放入键值对

```java
// 放入一对键值对，已存在该键时改变旧值
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 放入一对键值对，已存在该键时不会改变旧值
public V putIfAbsent(K key, V value) {
    return putVal(hash(key), key, value, true, true);
}

// 不直接使用对象的hashCode()作为hash值，而是将其高16位不变，低16位和高16位进行异或操作得出hash值。
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

// 内部放入方法
// onlyIfAbsent：如果为true，若已存在该键，则不替换已存在的值
// evict：只有在表的创建/初始化模式时才为false
final V putVal(int hash, K key, V value, booleanonlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 如果当前表未初始化或为空，扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash是一个快速取模的方法，相当于hash % n
    // 如果需要使用的位置上元素为null，说明没有发生hash碰撞，直接放入
    if ((p = tab[i = (n - 1) & hash]) == null) {
        tab[i] = newNode(hash, key, value, null);
    // 如果发生了hash碰撞
    } else {
        Node<K,V> e; K k;
        // 用k保存需要表中已存在的结点（与新放入结点发生碰撞的结点）
        // 如果k的键同需放入结点的键
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果键不等，且当前桶的结构为红黑树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 如果键不等，且当前桶的结构为链表
        else {
            // 遍历链表
            for (int binCount = 0; ; ++binCount) {
                // e指向自身的后置结点，若为null，说明当前链表遍历到了结尾
                if ((e = p.next) == null) {
                    // 将需放入的结点追加到链表的最后
                    p.next = newNode(hash, key, value, null);
                    // 如果追加后链表长度达到限值7，则将桶结构转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 停止循环
                    break;
                }
                // 如果遍历到的结点e的键同需放入的结点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 停止循环
                    break;
                // p指向自身的后置结点
                p = e;
            }
        }
        // 如果e不为null，说明需放入的结点的键同于链表中某一结点的键
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // 如果需要更换为新值或需放入的结点的值为null
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 拓展方法，用作LinkedHashMap及其他子类重写使用。
            afterNodeAccess(e);
            // 直接返回旧值，不执行后续代码
            return oldValue;
        }
    }
    // 修改modCount
    ++modCount;
    // 更新size。如果发现size达到阈值，扩容
    if (++size > threshold)
        resize();
    // 实现的方法，用作LinkedHashMap及其他子类重写使用。
    afterNodeInsertion(evict);
    return null;
}

// 扩容
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;

    int newCap, newThr = 0;
    // 如果旧容量大于0
    if (oldCap > 0) {
        // 如果旧表容量超出MAXIMUM_CAPACITY
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 令当前表的阈值为2^31 - 1，直接返回旧表，不再扩容
            threshold = Integer.MAX_VALUE;
            return oldTab;
        // 新容量为旧容量的2倍
        } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY
            && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    // 如果当前表是空的，但旧阈值大于0。说明初始化时指定了容量、阈值的情况
    } else if (oldThr > 0) {
        // 令新容量等于旧阈值
        newCap = oldThr;
    // 如果当前表是空的，阈值也为0。代表是初始化时没有指定任何容量和阈值
    } else {
        // 新容量等于16
        newCap = DEFAULT_INITIAL_CAPACITY;
        // 新阈值等于0.75 * 16 = 12
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }

    // 如果新阈值为0，对应当前表为空但有阈值
    if (newThr == 0) {
        // 用新容量和加载因子计算阈值
        float ft = (float)newCap * loadFactor;
        // 越界修正。阈值不能超出2^31 - 1
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    // 令当前表的阈值等于新阈值
    threshold = newThr;

    // 使用新容量创建新表
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    // 将当前表设为新表
    table = newTab;
    // 将旧表中的结点添加到新表中
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            // 保存当前遍历的结点
            Node<K,V> e;
            // 如果当前结点不为null
            if ((e = oldTab[j]) != null) {
                // 将旧表当前位置设为null，便于GC
                oldTab[j] = null;
                // 如果当前链表中就一个元素（没有发生哈希碰撞）
                if (e.next == null) {
                    /* 直接将这个元素放置在新的表里。
                     * 注意这里取下标是用哈希值与表的长度 - 1。
                     * 由于表的长度是2的n次方形式，这么做相当于一个取模运算。但是效率更高
                     */
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果发生了哈希碰撞，且桶的结构为红黑树
                } else if (e instanceof TreeNode) {
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 如果发生了哈希碰撞，且桶的结构为链表
                } else { // preserve order
                    // 低位（表中的第一位，table[0]）。则仍放在原位
                    Node<K,V> loHead = null, loTail = null;
                    // 高位（非表中第一位）。因为当前表容量为旧表容量的2倍，所以后移旧容量位，即放在旧容量+j处。
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 遍历当前结点所在的链表
                    do {
                        // 使用next保存当前结点的后置结点
                        next = e.next;
                        // 如果当前结点在旧表中存放在低位
                        if ((e.hash & oldCap) == 0) {
                            // 如果低位链表尾部为null（低位链表为空），将当前结点设为低位链表头部
                            if (loTail == null)
                                loHead = e;
                            // 否则让低位链表尾部后置结点等于当前结点
                            else
                                loTail.next = e;
                            // 将当前结点设为低位链表尾部。
                            loTail = e;
                        // 高位同理
                        } else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 低位放置在原处
                    if (loTail != null) {
                        loTail.next = null; // 方便GC
                        newTab[j] = loHead;
                    }
                    // 高位后移旧容量位
                    if (hiTail != null) {
                        hiTail.next = null; // 方便GC
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

```java
// 放入一个`Map`的所有键值对
public void putAll(Map<? extends K, ? extends V> m) {
    putMapEntries(m, true);
}

final void putMapEntries(Map<? extends K, ? extends V> m,boolean evict) {
    // 获取源Map的长度
    int s = m.size();
    // 如果源Map长度大于0
    if (s > 0) {
        // 如果当前表为初始化
        if (table == null) { // pre-size
            // 使用源Map的长度和加载因子计算新阈值
            float ft = ((float)s / loadFactor) + 1.0F;
            // 修正阈值
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            // 如果新阈值大于当前阈值，更新当前阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        // 如果当前表已经初始化，且源Map长度超出了阈值，则进行扩容
        } else if (s > threshold)
            resize();
        // 遍历源Map，将其中的Entry加入到当前表中
        // 注意：不是直接放入源Map的对象，而是新建一个等价的Entry
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            // 调用内部放入方法
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

### 移除键值对

```java
// 以键移除一对键值对
public V remove(Object key) {
    Node<K,V> e;
    // 删除键相等的结点，删除时需要移动其他结点
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

// 以键和值移除一对键值对
public boolean remove(Object key, Object value) {
    // 将值匹配设为true
    return removeNode(hash(key), key, value, true, true) != null;
}

// 移除一个结点
// matchValue：若为true，只在值相同时才移除
// movable：若为true，删除时移动其他节点。用于使用红黑树保存结点时的情形
final Node<K,V> removeNode(int hash, Object key, Objectvalue,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    // 如果当前表不为空，且存在hash值相同的结点。用p指向该结点
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        // node用于指向键同于指定删除的键的结点
        Node<K,V> node = null, e; K k; V v;
        // 如果p的键和指定删除的键相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            // node直接指向该结点
            node = p;
        // 如果p的键和指定删除的键不同，且p的键不为null
        else if ((e = p.next) != null) {
            // 如果当前桶的结构是红黑树
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            // 如果当前桶的结构是链表
            else {
                // 遍历当前链表。结束后，node指向键同于指定删除的键的结点或null
                // 当node为null时p指向链表最后一位，否则指向node的前一位
                do {
                    // 若链表中存在键同于指定删除的键的结点
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        // node直接指向该结点，结束循环
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 如果存在键同于指定删除的键的结点，且不需要匹配对应的值或对应的值匹配
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            // 红黑树操作
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            // 链表操作
            // 如果node等于p，说明需要删除的结点为链表的第一位
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            // 修改modCount
            ++modCount;
            // size减小一位
            --size;
            // 实现的方法，用作LinkedHashMap及其他子类重写使用。
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

### 替换一个键值对的值

```java
// 不指定旧值
public V replace(K key, V value) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) != null) {
        V oldValue = e.value;
        e.value = value;
        afterNodeAccess(e);
        return oldValue;
    }
    return null;
}

// 指定旧值
public boolean replace(K key, V oldValue, V newValue) {
    Node<K,V> e; V v;
    if ((e = getNode(hash(key), key)) != null &&
        ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
        e.value = newValue;
        afterNodeAccess(e);
        return true;
    }
    return false;
}
```

### 获取信息

```java
public V get(Object key) {
    Node<K,V> e;
    // 调用内部查询方法
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 如果当前桶不为空，且存在该hash值相同的结点
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 如果当前桶第一个结点的键等于key，则直接返回该键对应的值
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 否则遍历链表
        if ((e = first.next) != null) {
            // 红黑树操作
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 链表操作
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

// 指定默认值后根据键获取值
@Override
public V getOrDefault(Object key, V defaultValue) {
    Node<K,V> e;
    // 其余同get(Object)
    // 但如果当前Map中不存在该键，则返回指定的默认值
    return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
}
```

### 获取所有键的`Set`

```java
public Set<K> keySet() {
    // keySet在父类AbstractMap中定义
    Set<K> ks = keySet;
    if (ks == null) {
        // 令ks为成员内部类KeySet的对象
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}

// 成员内部类KeySet没有显示构造器，实际上不包含任何元素
final class KeySet extends AbstractSet<K> {
    public final int size()                 { return size; }
    public final void clear()               {HashMap.this.clear(); }
    public final Iterator<K> iterator()     { return new KeyIterator(); }
    public final boolean contains(Object o) { returncontainsKey(o); }
    public final boolean remove(Object key) {
        return removeNode(hash(key), key, null, false, true) != null;
    }
    public final Spliterator<K> spliterator() {}
    public final void forEach(Consumer<? super K> action) {}
}
```

### 获取所有值的`Collection`

```java
public Collection<V> values() {
    Collection<V> vs = values;
    if (vs == null) {
        vs = new Values();
        values = vs;
    }
    return vs;
}

final class Values extends AbstractCollection<V> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<V> iterator()     { return new ValueIterator(); }
    public final boolean contains(Object o) { return containsValue(o); }
    public final Spliterator<V> spliterator() {
        return new ValueSpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super V> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e.value);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}
```

### 获取所有键值对（`Entry`）

```java
public Set<Map.Entry<K,V>> entrySet() {
    Set<Map.Entry<K,V>> es;
    return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
}
```

### 查看是否包含一个键

```java
public boolean containsKey(Object key) {
    // 调用内部查询方法
    return getNode(hash(key), key) != null;
}
```

### 查看是否包含一个值

```java
public boolean containsValue(Object value) {
    Node<K,V>[] tab; V v;
    // 外部循环遍历表
    if ((tab = table) != null && size > 0) {
        // 内部循环遍历桶
        for (int i = 0; i < tab.length; ++i) {
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                if ((v = e.value) == value ||
                    (value != null && value.equals(v)))
                    return true;
            }
        }
    }
    return false;
}
```

[面试必备：HashMap源码解析（JDK8）](https://blog.csdn.net/zxt0601/article/details/77413921)

## `HashSet`

```java
public class HashSet<E> extends AbstractSet<E>
                        implements Set<E>,
                                   Cloneable,
                                   java.io.Serializable {
    ...
    // 使用HashMap保存元素
    private transient HashMap<E,Object> map;
    // 虚值，用于生成Map的键值对。没有意义
    private static final Object PRESENT = new Object();
    ...
}
```

- 实现了`Set`并继承自`AbstractSet`和：支持`Set`操作
- 实现了`Cloneable`：支持拷贝
- 实现了`java.io.Serializable`：支持序列化
- 不支持随机访问：没有`get`、`set`等随机访问相关方法

### 构造器

```java
public HashSet() {
    map = new HashMap<>();
}

// 使用其他集合生成
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

// 指定内部Map的初始容量及加载因子
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

// 只指定内部Map的初始容量
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

// 默认访问权限构造方法，用户一般无法调动
// dummy用于和其他参数列表为（int, float）的构造方法区分。没有实际意义
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

### 增加一个元素

```java
public boolean add(E e) {
    // 用HashMap的键来存储元素，值设为PRESENT
    return map.put(e, PRESENT)==null;
}
```

### 移除一个元素

```java
public boolean remove(Object o) {
    // 删除成功时HashMap返回PRESENT
    return map.remove(o)==PRESENT;
}
```

### 判断`Set`是否包含一个元素

```java
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

## `LinkedHashMap`

```java
public class LinkedHashMap<K,V> extends HashMap<K,V>
                                implements Map<K,V> {
    ...
    // 双端链表的头部
    transient LinkedHashMap.Entry<K,V> head;
    // 双端链表的尾部
    transient LinkedHashMap.Entry<K,V> tail;
    /* 迭代器迭代时的顺序。
     * 如果为true，使用hash表的连接顺序（无序）
     * 如果为false，则为插入顺序
     */
    final boolean accessOrder;
}
```

- 继承自`HashMap`，功能基本同`HashMap`
- 可以维护一个保存了插入顺序的双端链表
- 可以用于实现LRU（Least Recently Used）缓存

### 静态内部类`LinkedHashMap.Entry`

```java
// 继承自`HashMap.Node`
static class Entry<K,V> extends HashMap.Node<K,V> {
    // 有一个指向前和指向后的引用
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

### 构造器

```java
// 维护插入顺序，指定初始容量和加载因子
public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

// 维护插入顺序，只指定初始容量（使用默认加载因子）
public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

// 维护插入顺序，不指定初始容量和加载因子（使用默认加载因子）
public LinkedHashMap() {
    super();
    accessOrder = false;
}

// 维护插入顺序，使用另一个Map来生成
public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}

// 自主选择遍历顺序，指定初始容量和加载因子
public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

### 插入顺序链表维护

#### `afterNodeAccess()`

```java
// 只要访问了一个结点，该结点就会被移动到双端链表的尾部
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        // p指向当前结点
        // b指向当前节点的前面结点
        // a指向当前节点的后面结点
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        // 先将p移出链表
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;

        // 再将p添加到链表尾部
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

#### `afterNodeInsertion()`

```java
// evict：是否是新建/初始化模式
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        // 移出最旧的结点
        removeNode(hash(key), key, null, false, true);
    }
}

protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

#### `afterNodeRemoval()`

```java
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    // 将p移出双端链表
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

### LUR简单实现

```java
public class SimpleLRU<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public SimpleLRU(int capacity) {
        // accessOrder = true，最后访问的entry放在list最后
        super(16, 0.75f, true);
        this.capacity = capacity;
    }

    public SimpleLRU() {
        this(128);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 如果当前size > capacity，删除最早的entry
        return super.size() > capacity;
    }
}
```
