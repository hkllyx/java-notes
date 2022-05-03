# `TreeMap` & `TreeSet`

## 红黑树

[红黑树思维导图](../../../resource/xmind/Red-Black%20Tree.xmind)

## `TreeMap`

```java
public class TreeMap<K,V> extends AbstractMap<K,V>
                          implements NavigableMap<K,V>,
                                     Cloneable,
                                     java.io.Serializable {
    // 比较器，只能初始化一次，不可更改
    private final Comparator<? super K> comparator;
    // 树的根结点
    private transient Entry<K,V> root;
    // Map的长度
    private transient int size = 0;
    private transient int modCount = 0;
    ...
    // 表示结点的颜色
    private static final boolean RED   = false;
    private static final boolean BLACK = true;
    ...
}
```

- 继承了`AbstractMap`：支持`Map`操作
- 实现了`NavigableMap`：支持有序操作并能获取指定元素最近的元素
- 实现了`Cloneable`：支持拷贝
- 实现了`Serializable`：支持序列化
- 使用**红黑树**保存元素
- 保存的元素必须实现了`Comparable`接口，或指定比较器`Comparator`

### 内部静态类`TreeMap.Entry`

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    // 指向左子结点
    Entry<K,V> left;
    // 指向右子结点
    Entry<K,V> right;
    // 指向父结点
    Entry<K,V> parent;
    // 颜色默认为黑色
    boolean color = BLACK;

    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }

    ...
}
```

### 构造器

```java
public TreeMap() {
    comparator = null;
}

// 指定比较器
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}

// 用其他Map生成
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}

// 用其他有序Map生成
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

### 放入键值对

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    // 如果根结点为null
    if (t == null) {
        // 键如果为null，抛出NPE，如果不为null但未实现Comparable，抛出CCE
        compare(key, key); // type (and possibly null) check

        // 令新增的键值对为根结点
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    // 如果指定了比较器
    if (cpr != null) {
        // 遍历树
        do {
            parent = t;
            // 使用比较器比较两个结点的键
            cmp = cpr.compare(key, t.key);
            // 如果值为负，说明插入点在当前结点的左侧
            if (cmp < 0)
                t = t.left;
            // 如果值为负，说明插入点在当前结点的右侧
            else if (cmp > 0)
                t = t.right;
            // 如果值为0，则当前结点的键就是指定键，替换该结点的值为指定值
            else
                return t.setValue(value);
        } while (t != null);
    // 如果没有指定比较器
    } else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        // 如果没有相同的键，则一定遍历到一条路径的结尾才停止
        do {
            parent = t;
            // 使用自己实现Comparable接口时重写的compare方法
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 遍历结束后，parent是新增结点的父结点，而新增结点一定插在一条路径的最后一个
    Entry<K,V> e = new Entry<>(key, value, parent);
    // 新增结点新增到父结点的左侧
    if (cmp < 0)
        parent.left = e;
    // 新增结点新增到父结点的右侧
    else
        parent.right = e;
    // 调整二叉搜索树为红黑树
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}

// 调整二叉搜索树为红黑树
private void fixAfterInsertion(Entry<K,V> x) {
    /* 将当前结点设为红色
     * 因为插入的结点一定是一条路径的最后一个，所以的子结点都是null
     * 设为null可以保证
     */
    x.color = RED;

    // 如果当前结点不为null和根结点，且其父结点也是红色
    // 根据红黑树的特点：红色结点的子结点一定是黑色。父亲结点时红色，所以祖父结点一定是黑色
    while (x != null && x != root && x.parent.color == RED) {
        // 如果父结点是祖父结点的左结点
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            // 如果祖父结点的右结点（伯父结点）是红色
            if (colorOf(y) == RED) {
                // 将父结点和伯父结点设为黑色
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                // 将祖父结点的颜色设为红色
                setColor(parentOf(parentOf(x)), RED);
                // 将当前结点设为祖父结点，继续下一个循环
                x = parentOf(parentOf(x));
            // 如果伯父结点颜色是黑色
            } else {
                // 如果当前结点为父结点的右结点
                if (x == rightOf(parentOf(x))) {
                    // 将当前结点设为父结点
                    x = parentOf(x);
                    // 左旋当前结点。旋转后，当前结点是原先结点左子结点
                    rotateLeft(x);
                }
                // 将父结点设为黑色，祖父结点设为红色
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                // 右旋祖父结点
                rotateRight(parentOf(parentOf(x)));
            }
        // 如果父结点是祖父结点的右结点
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            // 如果祖父结点的左结点（叔父结点）是红色
            if (colorOf(y) == RED) {
                // 将父结点和叔父结点设为黑色
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                // 将祖父结点设为红色
                setColor(parentOf(parentOf(x)), RED);
                // 将当前结点设为祖父结点，继续下一个循环
                x = parentOf(parentOf(x));
            // 如果叔父结点为黑色
            } else {
                // 如果当结点是父结点的左结点
                if (x == leftOf(parentOf(x))) {
                    // 将当前结点设为父结点
                    x = parentOf(x);
                    // 右旋当前结点。旋转后，当前结点是原先结点的右子结点
                    rotateRight(x);
                }
                // 将父结点设为黑色，祖父结点设为红色
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                // 左旋祖父结点
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    // 根据红黑树的性质，根结点一定要是黑色
    root.color = BLACK;
}

// 左旋
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> r = p.right;
        // 将r的左子结点作为p的右子结点
        p.right = r.left;
        if (r.left != null)
            r.left.parent = p;

        // 将r代替p的位置
        r.parent = p.parent;
        if (p.parent == null)
            root = r;
        else if (p.parent.left == p)
            p.parent.left = r;
        else
            p.parent.right = r;

        // 将r的左结点设为p
        r.left = p;
        p.parent = r;
    }
}

// 右旋
private void rotateRight(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> l = p.left;
        //
        p.left = l.right;
        if (l.right != null) l.right.parent = p;
        l.parent = p.parent;
        if (p.parent == null)
            root = l;
        else if (p.parent.right == p)
            p.parent.right = l;
        else p.parent.left = l;
        l.right = p;
        p.parent = l;
    }
}
```

```java
// 放入一个Map的所有键值对
public void putAll(Map<? extends K, ? extends V> map) {
    int mapSize = map.size();
    // 如果源Map不问空且是有序的Map
    if (size==0 && mapSize!=0 && map instanceof SortedMap) {
        Comparator<?> c = ((SortedMap<?,?>)map).comparator();
        // 如果源Map的比较器同当前的Map
        if (c == comparator || (c != null && c.equals(comparator))) {
            ++modCount;
            try {
                buildFromSorted(mapSize, map.entrySet().iterator(),
                                null, null);
            } catch (java.io.IOException cannotHappen) {
            } catch (ClassNotFoundException cannotHappen) {
            }
            return;
        }
    }
    super.putAll(map);
}
```

```java
// 从排序数据中构建
// 基于排序数据的线性时间树构建算法
private void buildFromSorted(int size,
                             Iterator<?> it,
                             java.io.ObjectInputStream str,
                             V defaultVal)
    throws  java.io.IOException, ClassNotFoundException {
    this.size = size;
    root = buildFromSorted(0, 0, size-1, computeRedLevel(size),
                           it, str, defaultVal);
}

private static int computeRedLevel(int sz) {
    int level = 0;
    for (int m = sz - 1; m >= 0; m = m / 2 - 1)
        level++;
    return level;
}


@SuppressWarnings("unchecked")
private final Entry<K,V> buildFromSorted(int level,
                                         int lo,
                                         int hi,
                                         int redLevel,
                                         Iterator<?> it,
                                         java.io.ObjectInputStream str,
                                         V defaultVal)
    throws  java.io.IOException, ClassNotFoundException {
    /*
     * Strategy: The root is the middlemost element. To get to it, we
     * have to first recursively construct the entire left subtree,
     * so as to grab all of its elements. We can then proceed with right
     * subtree.
     *
     * The lo and hi arguments are the minimum and maximum
     * indices to pull out of the iterator or stream for current subtree.
     * They are not actually indexed, we just proceed sequentially,
     * ensuring that items are extracted in corresponding order.
     */

    if (hi < lo) return null;

    int mid = (lo + hi) >>> 1;

    Entry<K,V> left  = null;
    // 递归低侧
    if (lo < mid)
        left = buildFromSorted(level+1, lo, mid - 1, redLevel,
                               it, str, defaultVal);

    // extract key and/or value from iterator or stream
    K key;
    V value;
    if (it != null) {
        if (defaultVal==null) {
            Map.Entry<?,?> entry = (Map.Entry<?,?>)it.next();
            key = (K)entry.getKey();
            value = (V)entry.getValue();
        } else {
            key = (K)it.next();
            value = defaultVal;
        }
    } else { // use stream
        key = (K) str.readObject();
        value = (defaultVal != null ? defaultVal : (V) str.readObject());
    }

    Entry<K,V> middle =  new Entry<>(key, value, null);

    // color nodes in non-full bottommost level red
    if (level == redLevel)
        middle.color = RED;

    if (left != null) {
        middle.left = left;
        left.parent = middle;
    }

    if (mid < hi) {
        Entry<K,V> right = buildFromSorted(level+1, mid+1, hi, redLevel,
                                           it, str, defaultVal);
        middle.right = right;
        right.parent = middle;
    }

    return middle;
}
```

### 移除键值对

```java
// 以键移除一对键值对
public V remove(Object key) {
    // 根据键获取目标结点
    Entry<K,V> p = getEntry(key);
    // 如果不存在，直接返回null
    if (p == null)
        return null;

    // 删除结点，返回旧值
    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}

// 移除一个结点
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // If strictly internal, copy successor's element to p and then make p point to successor.
    // 此if语句结束后，p一定目标结点或其子孙结点。但p可能是目标结点，也可能是目标结点后继结点
    if (p.left != null && p.right != null) {
        // 找到目标结点的后继，后继结点如果存在一定是目标结点子孙结点
        Entry<K,V> s = successor(p);
        // 令后继结点的键值替换目标结点
        p.key = s.key;
        p.value = s.value;
        // 令p指向后继结点
        p = s;
    }

    // 令替换结点为p的子结点中更小的一个
    // p一定目标结点或其子孙结点，所有如果替换结点存在，那它一定是目标结点的子孙结点
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    // 如果替换结点存在，用替换结点完全替换p（p会从树中移除）
    if (replacement != null) {
        replacement.parent = p.parent;
        // 如果后p的父结点为null，
        // 说明后p是根结点，所以令替换结点替换为新的根结点
        if (p.parent == null)
            root = replacement;
        // 如果p是其父结点的左子结点，则令，p的父结点的左子结点为替换结点
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        // 如果p是其父结点的右子结点，则令，p的父结点的右子结点为替换结点
        else
            p.parent.right = replacement;

        // 替换已经完成，p已经从树中移除，所以删除p的向外指针
        p.left = p.right = p.parent = null;

        /* 当p为红色时，其父结点和子结点一定都是黑色的，
         * 替换结点为任何颜色均不会出现两个红色结点相邻，且删除后路径上黑色结点数不会有变化
         * 当p为黑色时，就有可能会出现违背红黑树性质4或5的情形
         */
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    // 如果p没有子结点，且为根结点，那么p一定是目标结点。
    // 此时，树中只存在目标结点，删除后树为空，直接令根结点为空即可
    } else if (p.parent == null) {
        root = null;
    // p没有子结点，p也不为根节点。此时p仍可能为目标结点或其后继结点
    } else {
        // 如果p为黑色，删除可能导致树不平衡，需要修复
        if (p.color == BLACK)
            fixAfterDeletion(p);

        // p为红色时，直接从树中移除，不会影响数平衡
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}

// 获取后继结点
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    // 如果右子结点存在，直接从右子结点开始找
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        // 如果右子结点没有左子结点，那么后继就是右子结点
        // 否则，一定是最左侧的结点
        while (p.left != null)
            p = p.left;
        return p;
    // 若没有右子结点，向上查找
    } else {
        // 如果当前结点在父结点左侧，那后继结点就是父结点
        // 如果是在右侧，说明父结点仍比当前结点小，继续查询祖父结点，
        // 直至一个结点是其父结点的左子结点或已不存在父结点为止
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}

// 删除后修复红黑树（待分析...）
private void fixAfterDeletion(Entry<K,V> x) {
    while (x != root && colorOf(x) == BLACK) {
        // 如果x是其父结点的左结点
        if (x == leftOf(parentOf(x))) {
            Entry<K,V> sib = rightOf(parentOf(x));

            // 如果兄结点为红色
            if (colorOf(sib) == RED) {
                // 将兄结点设为黑色，父结点设为红色
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                // 左旋父结点
                rotateLeft(parentOf(x));
                // 重新寻找兄结点
                sib = rightOf(parentOf(x));
            }

            // 如果兄结点的子结点都为黑色
            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                // 如果兄结点的左子结点为红色，右子结点为黑色
                if (colorOf(rightOf(sib)) == BLACK) {
                    // 将兄结点的左子结点设为黑色
                    setColor(leftOf(sib), BLACK);
                    // 将兄结点设为红色
                    setColor(sib, RED);
                    // 右旋兄结点
                    rotateRight(sib);
                    // 重新寻找兄结点
                    sib = rightOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(rightOf(sib), BLACK);
                rotateLeft(parentOf(x));
                x = root;
            }
        } else { // symmetric
            Entry<K,V> sib = leftOf(parentOf(x));

            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateRight(parentOf(x));
                sib = leftOf(parentOf(x));
            }

            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateLeft(sib);
                    sib = leftOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(leftOf(sib), BLACK);
                rotateRight(parentOf(x));
                x = root;
            }
        }
    }

    // 将x设为黑色
    setColor(x, BLACK);
}
```

### `SortedMap`

```java
public interface SortedMap<K,V> extends Map<K,V> {
    /** 返回比较器 */
    Comparator<? super E> comparator();

    /**
     * 返回此Map中Key范围从fromKey（包含）到toKey（排除）的部分的视图
     * 如果fromKey == toKey，返回空视图
     * 返回的Map拥有此Map的所有操作
     */
    SortedMap<K,V> subMap(K fromKey, K toKey);

    /** 返回此Map中Key范围小于toKey的部分的视图 */
    SortedMap<K,V> headMap(K toKey);

    /** 返回此Map中Key范围大于等于fromKey   的部分的视图 */
    SortedMap<K,V> tailMap(K fromKey);

    /** 返回第一个（最小）的key */
    K firstKey();

    /** 返回最后一（最大）的key */
    K lastKey();

    ...
}
```

### `NavigableMap`

```java
public interface NavigableMap<K,V> extends SortedMap<K,V> {
    /** 返回集合中严格小于给定key的最大key对应的Entry，如果没有这样的key，则返回null */
    Map.Entry<K,V> lowerEntry(K key);

    /** 返回集合中严格小于给定key的最大key，如果没有这样的key，则返回null */
    K lowerKey(K key);

    /** 返回集合中小于等于给定key的最大key对应的Entry，如果没有这样的key，则返回null */
    Map.Entry<K,V> floorEntry(K key);

    /** 返回集合中小于等于给定key的最大key，如果没有这样的key，则返回null */
    K floorKey(K key);

    /** 返回集合中大于等于给定key的最小key对应的Entry，如果没有这样的key，则返回null */
    Map.Entry<K,V> ceilingEntry(K key);

        /** 返回集合中大于等于给定key的最小key，如果没有这样的key，则返回null */
    K ceilingKey(K key);

    /** 返回集合中严格大于给定key的最小key对应的Entry，如果没有这样的key，则返回null */
    Map.Entry<K,V> higherEntry(K key);

    /** 返回集合中严格大于给定key的最小key，如果没有这样的key，则返回null */
    K higherKey(K key);

    /** 返回第一个（最小）key对应的Entry，没有则返回null */
    Map.Entry<K,V> firstEntry();

    /** 返回最后一个（最大）key对应的Entry，没有则返回null */
    Map.Entry<K,V> lastEntry();

    /** 返回并移除第一个（最小）key对应的Entry，没有则返回null */
    Map.Entry<K,V> pollFirstEntry();

    /** 返回最后一个（最大）key对应的Entry，没有则返回null */
    Map.Entry<K,V> pollLastEntry();

    /**
     * 返回此Map中包含的Entry（key）的逆序视图
     * 如果在对任意一个Map进行迭代时修改了其中的任何一个集合（除了通过该迭代器自身删除），则迭代的结果是未定义的
     */
    NavigableMap<K,V> descendingMap();

    NavigableSet<K> navigableKeySet();

    NavigableSet<K> descendingKeySet();

    /** 返回key大于（或等于）fromKey、小于（或等于）toKey的子Map */
    NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                             K toKey,   boolean toInclusive);

    /** 返回key小于（或等于）toKey的子Map */
    NavigableMap<K,V> headMap(K toKey, boolean inclusive);


    /** 返回key大于（或等于）fromKey的子Map */
    NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);

    ...
}
```

## `TreeSet`

```java
public class TreeSet<E> extends AbstractSet<E>
                                implements NavigableSet<E>,
                                Cloneable,
                                java.io.Serializable {
    private transient NavigableMap<E,Object> m;
    private static final Object PRESENT = new Object();
}
```

- 继承了`AbstractSet`：支持`Set`操作
- 实现了`NavigableSet`：支持有序操作并能获取指定元素最近的元素
- 实现了`Cloneable`：支持拷贝
- 实现了`Serializable`：支持序列化
- 使用`NavigableMap`保存元素
- 保存的元素必须实现了`Comparable`接口，或指定比较器`Comparator`

### 构造器

```java
public TreeSet() {
    this(new TreeMap<E,Object>());
}

// 指定比较器
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

// 使用一个有序Map生成
public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}

// 使用其他集合生成
public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}

// 使用NavigableMap生成
// 包可见构造器，一般情况下用户不可使用
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}
```

### `SortedSet`

```java
public interface SortedSet<E> extends Set<E> {
    /** 返回比较器 */
    Comparator<? super E> comparator();

    /** 返回大于fromElement小于toElement的子集 */
    SortedSet<E> subSet(E fromElement, E toElement);

    /** 返回小于toElement的子集 */
    SortedSet<E> headSet(E toElement);

    /** 返回大于fromElement的子集 */
    SortedSet<E> tailSet(E fromElement);

    /** 返回当前集合中的第一个（最低的）元素 */
    E first();

    /** 返回当前集合中的最后一个（最高的）元素 */
    E last();

    ...
}
```

### `NavigableSet`

```java
public interface NavigableSet<E> extends SortedSet<E> {
    /** 返回集合中严格小于给定元素的最大元素，如果没有这样的元素，则返回null */
    E lower(E e);

    /** 返回集合中小于或等于给定元素的最大元素，如果没有这样的元素，则返回null */
    E floor(E e);

    /** 返回集合中大于或等于给定元素的最小元素，如果没有这样的元素，则返回null */
    E ceiling(E e);

    /** 返回集合中严格大于给定元素的最小元素，如果没有这样的元素，则返回null */
    E higher(E e);

    /** 返回并删除第一个（最低的）元素，如果该集合为空，则返回null。 */
    E pollFirst();

    /** 返回并删除最后（最高）的元素，如果该集合为空则返回null */
    E pollLast();

    Iterator<E> iterator();

    /**
     * 返回此集合中包含的元素的逆序视图
     * 如果在对任意一个集合进行迭代时修改了其中的任何一个集合（除了通过该迭代器自身删除），则迭代的结果是未定义的
     */
    NavigableSet<E> descendingSet();

    /**
     * 按降序返回集合中元素的迭代器
     * 等同descendingSet().iterator()
     */
    Iterator<E> descendingIterator();

    /** 返回大于（或等于）fromElement、小于（或等于）toElement的子集 */
    NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                           E toElement,   boolean toInclusive);

    /** 返回大于（或等于）fromElement的子集 */
    NavigableSet<E> tailSet(E fromElement, boolean inclusive);

    ...
}
```
