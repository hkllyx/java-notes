# `CopyOnWriteArrayList` & `CopyOnWriteArraySet`

## `CopyOnWriteArrayList`

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final long serialVersionUID = 8673264195747942595L;

    /** 互斥锁 */
    final transient ReentrantLock lock = new ReentrantLock();

    /** 保存元素的数组，只能通过getArray/setArray访问 */
    private transient volatile Object[] array;

    ...
}
```

- 实现了`List`：支持列表操作
- 实现了`RandomAccess`：支持随机访问
- 内部使用数组保存元素，数组容量随元素变化（size），没有默认容量
- 写操作时，使用`ReentrantLock`保证线程安全，并修改原数组的拷贝然后替换，实现独写分离

### 构造器

```java
// 默认
public CopyOnWriteArrayList() {
    setArray(new Object[0]);
}

// 使用其他集合生成
public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;
    if (c.getClass() == CopyOnWriteArrayList.class)
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else {
        elements = c.toArray();
        if (c.getClass() != ArrayList.class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    setArray(elements);
}

// 使用数组生成
public CopyOnWriteArrayList(E[] toCopyIn) {
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}

final void setArray(Object[] a) {
    array = a;
}
```

### 添加元素

```java
// 新增元素到末尾
public boolean add(E e) {
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 复制array，且容量+1
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 将元素加到最后，然后将array替换为新数组
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        // 解锁
        lock.unlock();
    }
}

// 新增元素到指定位置
public void add(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+len);
        Object[] newElements;
        int numMoved = len - index;
        if (numMoved == 0)
            newElements = Arrays.copyOf(elements, len + 1);
        else {
            newElements = new Object[len + 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index, newElements, index + 1,
                             numMoved);
        }
        newElements[index] = element;
        setArray(newElements);
    } finally {
        lock.unlock();
    }
}

// 批量添加
public boolean addAll(Collection<? extends E> c) {
    Object[] cs = (c.getClass() == CopyOnWriteArrayList.class) ?
        ((CopyOnWriteArrayList<?>)c).getArray() : c.toArray();
    if (cs.length == 0)
        return false;
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        // 如果添加的集合是CopyOnWriteArrayList/ArrayList（底层也是数组），则将array设为cs
        if (len == 0 && (c.getClass() == CopyOnWriteArrayList.class ||
                         c.getClass() == ArrayList.class)) {
            setArray(cs);
        } else {
            // 否则复制一份&替换array
            Object[] newElements = Arrays.copyOf(elements, len + cs.length);
            System.arraycopy(cs, 0, newElements, len, cs.length);
            setArray(newElements);
        }
        return true;
    } finally {
        // 解锁
        lock.unlock();
    }
}

// 批量添加到指定位置
public boolean addAll(int index, Collection<? extends E> c) {
    Object[] cs = c.toArray();
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+len);
        if (cs.length == 0)
            return false;
        int numMoved = len - index;
        Object[] newElements;
        if (numMoved == 0)
            newElements = Arrays.copyOf(elements, len + cs.length);
        else {
            newElements = new Object[len + cs.length];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index,
                             newElements, index + cs.length,
                             numMoved);
        }
        System.arraycopy(cs, 0, newElements, index, cs.length);
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

// 如果元素不存在则添加，返回是否添加成功
public boolean addIfAbsent(E e) {
    // 获取快照
    Object[] snapshot = getArray();
    return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
        addIfAbsent(e, snapshot);
}

// 定位元素
private static int indexOf(Object o, Object[] elements, int index, int fence) {
    if (o == null) {
        for (int i = index; i < fence; i++)
            if (elements[i] == null)
                return i;
    } else {
        for (int i = index; i < fence; i++)
            if (o.equals(elements[i]))
                return i;
    }
    return -1;
}

private boolean addIfAbsent(E e, Object[] snapshot) {
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] current = getArray();
        int len = current.length;
        // 比较读取快照之后是否有改动
        if (snapshot != current) {
            // Optimize for lost race to another addXXX operation
            int common = Math.min(snapshot.length, len);
            // 当前array共有部分同快照不同且已存在e
            for (int i = 0; i < common; i++)
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            // 当前array非共有部分已存在e
            if (indexOf(e, current, common, len) >= 0)
                    return false;
        }
        // 复制当前array（而不是快照）
        Object[] newElements = Arrays.copyOf(current, len + 1);
        // 赋值并替换array
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        // 解锁
        lock.unlock();
    }
}

// 批量-如果元素不存在则添加，返回添加成功数
public int addAllAbsent(Collection<? extends E> c) {
    // 获取集合的数组拷贝
    Object[] cs = c.toArray();
    if (c.getClass() != ArrayList.class) {
        cs = cs.clone();
    }
    if (cs.length == 0)
        return 0;
    // 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        int added = 0;
        // 去除已存在的、重复的元素
        // uniquify and compact elements in cs
        for (int i = 0; i < cs.length; ++i) {
            Object e = cs[i];
            if (indexOf(e, elements, 0, len) < 0 &&
                indexOf(e, cs, 0, added) < 0)
                cs[added++] = e;
        }
        // 复制&替换
        if (added > 0) {
            Object[] newElements = Arrays.copyOf(elements, len + added);
            System.arraycopy(cs, 0, newElements, len, added);
            setArray(newElements);
        }
        return added;
    } finally {
        // 解锁
        lock.unlock();
    }
}
```

### 删除元素

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

## `CopyOnWriteArraySet`

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    private final CopyOnWriteArrayList<E> al;

    ...
}
```

- 继承自`AbstractSet`：支持集操作
- 内部使用`CopyOnWriteArrayList`保存元素

## `CopyOnWriteArrayList`优缺点

### 优点

- 线程安全：写的时候加互斥锁
- 适合读多写少的并发场景。比如白名单，黑名单等场景

### 缺点

- 数据一致性：读写分离，读到的数据可能是旧数据（脏读）
- 内存占用：写操作复制数组，内存消耗较大
