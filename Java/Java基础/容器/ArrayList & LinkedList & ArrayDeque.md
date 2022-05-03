# `ArrayList` & `LinkedList` & `ArrayDeque`

## `ArrayList`

```java
public class ArrayList<E> extends AbstractList<E>
                          implements List<E>,
                                     RandomAccess,
                                     Cloneable,
                                     java.io.Serializable {
    ...
    /** 默认容量为10 */
    private static final int DEFAULT_CAPACITY = 10;
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    /** 保存元素的数组 */
    transient Object[] elementData;
    /** 列表大小 */
    private int size;
    ...
}
```

- 实现了`List`并继承自`AbstractList`：支持列表操作
- 实现了`RandomAccess`：支持随机访问，通过数组下标实现
- 实现了`Cloneable`：支持拷贝
- 实现了`java.io.Serializable`：支持序列化
- 内部使用`Object[]`保存数据
- 默认容量为`10`（懒加载，实际上初始默认容量为0，第一次扩展时如果需要的容量< 10才扩展为10）
- 扩容时增长一半

### 构造器

```java
// 默认
public ArrayList() {
    // 令elementData等于默认容量的空数组
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 指定初始容量
public ArrayList(int initialCapacity) {
    // 如果指定初始容量大于0，则令初始容量为指定容量
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    // 如果指定初始容量等于0，则令elementData等于默认的空数组
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    // 如果指定初始容量小于0，抛出异常
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
    }
}

// 使用其他集合生成
public ArrayList(Collection<? extends E> c) {
    // 令elementData为指定集合生成的数组
    elementData = c.toArray();
    // 将size设置为elementData的长度
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

### 增加元素

#### 新增元素到末尾

```java
public boolean add(E e) {
     // 确保内部数组elementData容量足够，能添加一个元素
     // size + 1可能溢出，溢出后为0x10000000
     ensureCapacityInternal(size + 1);  // Increments modCount!!
     // 将新增元素放在size位上，并size增大一位
     elementData[size++] = e;
     return true;
}

public boolean addAll(Collection<? extends E> c) {
     Object[] a = c.toArray();
     int numNew = a.length;
     // 确保内部数组elementData容量足够，能添加集合中所以元素。
     // size + numNew可能溢出，溢出值为负
     ensureCapacityInternal(size + numNew);  // Increments modCount
     // 将集合中的元素添加到内部数组中
     System.arraycopy(a, 0, elementData, size, numNew);
     size += numNew;
     return numNew != 0;
}

private void ensureCapacityInternal(int minCapacity) {
    // 计算当前需要的最小容量
    // 确保内部数组容量满足计算出的容量
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果当前数组为空数组，则令最小容量为默认容量10和最小需求容量中的更大者
    // 即，当前数组为空数组时，新容量必须大于10（常说ArrayList默认初始容量为10的原因）
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    // 修改次数增1（fast-fail机制）
    modCount++;

    // overflow-conscious code
    // 若最小需要容量没有溢出，且大于内部数组的容量，
    // 若溢出，且溢出后减去内部数组长度后再度溢出为正数
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 新容量 = 旧容量 + 旧容量 / 2（扩大一半，近似1.5倍）
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    /* 若新容量和最小需求容量均没有溢出，则说明新容量仍小于最下需求容量
     * 若均溢出，则说明新容量溢出值更小，即不考虑溢出时，新容量仍小于最下需求容量
     * 若只有新容量溢出，说明扩大一半过多
     * 若只有最小需求容量溢出，说明扩大一半无法满足需求
     */
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 若新容量没有溢出，但超出最大数组长度
    // 若溢出，且溢出后减去MAX_ARRAY_SIZE再度溢出
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    // 溢出，抛出OOM异常
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    // 返回最小需求容量和MAX_ARRAY_SIZE中的更大值
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

#### 新增元素到指定位置

```java
public void add(int index, E element) {
    // 确保指定位置符合范围要求
    rangeCheckForAdd(index);

    // 确保内部容量能够添加一个元素
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
    elementData[index] = element;
    size++;
}

public boolean addAll(int index, Collection<? extends E> c) {
    // 确保指定位置符合范围要求
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    // 确保内部容量能够添加集合的所有元素
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}

private void rangeCheckForAdd(int index) {
    // 指定位置在0 ~ size之间（不包括0和size）
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

### 删除元素

#### 删除指定位置的元素

```java
public E remove(int index) {
    // 确保指定位置符合范围要求
    rangeCheck(index);

    modCount++;
    // 保存删除位置的元素
    E oldValue = elementData(index);

    // 移动元素
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work

    // 返回删除的元素
    return oldValue;
}
```

#### 删除值为指定值的元素

```java
public boolean remove(Object o) {
    // 指定对象为null
    if (o == null) {
        // 遍历内部数组
        for (int index = 0; index < size; index++)
            // 如果找到数组元素为null，删除并返回
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    // 指定对象不为null
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// 已保证不会越界，不用判断，也不需要取出该元素的情况下快速删除一个元素
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

#### 保存/删除同指定集合相交部分的元素

```java
public boolean retainAll(Collection<?> c) {
    // 要求指定集合不为null
    Objects.requireNonNull(c);
    // 批量删除
    return batchRemove(c, true);
}

public boolean removeAll(Collection<?> c) {
    // 要求指定集合不为null
    Objects.requireNonNull(c);
    // 批量删除
    return batchRemove(c, false);
}

// complement = true，删除不在指定集合中的元素（删除不相交部分）
//            = false，删除在指定集合中的元素（删除相交部分）
private boolean batchRemove(Collection<?> c, boolean complement) {
    // 用一个不可变局部变量指向内部数组
    final Object[] elementData = this.elementData;
    // r用于遍历内部数组，w用于向内部数组写入新元素
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            // 如果指定集合中包含当前元素，将r处元素移动至w处
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        // 如果由于异常r遍历中断，r为中断处
        if (r != size) {
            // 将中断处及之后的数据复制到w开始之后处
            // 即中断之后的元素不作处理，直接保留
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            // 移动w
            w += size - r;
        }
        // 如果内部数组为空，或有元素被移动
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            // 修改次数增加移动次数
            modCount += size - w;
            // 更新size
            size = w;
            modified = true;
        }
    }
    return modified;
}
```

#### 将内部数组长度剪切到列表长度

```java
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```

### 更新指定位置元素

```java
public E set(int index, E element) {
    // 确保指定位置符合范围要求
    rangeCheck(index);

    // 保存旧值
    E oldValue = elementData(index);
    // 将新值保存
    elementData[index] = element;
    // 返回旧值
    return oldValue;
}
```

### 获取信息

#### 获取指定位置的元素

```java
public E get(int index) {
    // 确保指定位置符合范围要求
    rangeCheck(index);

    // 返回指定位置元素元素
    return elementData(index);
}

@SuppressWarnings("unchecked")
E elementData(int index) {
    // 类型强制转换
    return (E) elementData[index];
}
```

#### 获取一个元素在列表中的位置

```java
// 获取中第一个
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        // 从头开始遍历
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}

// 列表中最后一个
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        // 从末尾开始遍历
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

#### 查看列表是否包含指定元素

```java
public boolean contains(Object o) {
    // 查看指定元素在列表中的第一个位置
    return indexOf(o) >= 0;
}
```

#### 获取子列表

```java
public List<E> subList(int fromIndex, int toIndex) {
    // 检查指定范围
    subListRangeCheck(fromIndex, toIndex, size);
    // 返回成员内部类SubList的实例。
    // 该内部类继承自AbstractList，且支持增删改查
    return new SubList(this, 0, fromIndex, toIndex);
}

static void subListRangeCheck(int fromIndex, int toIndex,int size) {
    if (fromIndex < 0)
        throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
    if (toIndex > size)
        throw new IndexOutOfBoundsException("toIndex = " + toIndex);
    if (fromIndex > toIndex)
        throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                           ") > toIndex(" + toIndex + ")");
}
```

### 转换成数组

```java
// 转换成Object数组
public Object[] toArray() {
    // 直接复制数组
    return Arrays.copyOf(elementData, size);
}

// 转换成指定类型数组
// 传入一个指定类型的数组（最好传入一个同size长的数组）
public <T> T[] toArray(T[] a) {
    // 如果传入数组长度小于列表长度，复制内部数组并强转成指定类型数组后直接返回
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    // 将内部数组复制到传入数组，并将传入数组超出内部数组部分设为null
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

## `LinkedList`

```java
public class LinkedList<E> extends AbstractSequentialList<E>
                           implements List<E>,
                           Deque<E>,
                           Cloneable,
                           java.io.Serializable {
    // 链表的长度
    transient int size = 0;
    // 指向第一个结点
    transient Node<E> first;
    // 指向最后一个结点
    transient Node<E> last;
    ...
}
```

- 实现了`List`并继承自`AbstractSequentialList`：支持有序列表操作
- 实现了`Deque`：支持双端队列操作
- 实现了`Cloneable`：支持拷贝
- 实现了`java.io.Serializable`：支持序列化

### 静态内部类`LinkedList.Node`

```java
private static class Node<E> {
    // 保存真正的元素
    E item;
    // 指向下一个结点
    Node<E> next;
    // 指向前一个结点
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 构造器

```java
// 默认
public LinkedList() {
}

// 使用其他集合生成
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### 增加元素

#### 在结尾增加一个元素

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

public void addLast(E e) {
    linkLast(e);
}

// 队列操作
public boolean offer(E e) {
    return add(e);
}

public boolean offerLast(E e) {
    addLast(e);
    return true;
}

void linkLast(E e) {
    final Node<E> l = last;
    // 生成一个新结点，结点保存的值为需要链接的元素
    // 向前的结点指向原尾部，向后的结点为null
    final Node<E> newNode = new Node<>(l, e, null);
    // 将尾部设为新增的结点
    last = newNode;
    // 如果原尾部为null，说明新增结点是第一个结点
    // 否则原尾部下一个结点指向新增的结点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    // 长度增大
    size++;
    // modCount增大
    modCount++;
}
```

#### 在开头增加一个元素

```java
public void addFirst(E e) {
    linkFirst(e);
}

public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

// 栈操作
public void push(E e) {
    addFirst(e);
}

private void linkFirst(E e) {
    final Node<E> f = first;
    // 生成一个新结点，结点保存的值为需要链接的元素
    // 向前的结点为null，向后的结点指向头部
    final Node<E> newNode = new Node<>(null, e, f);
    // 将头部设为新增的结点
    first = newNode;
    // 如果原头部为null，说明新增结点是第一个结点
    // 否则原头部前一个结点指向新增的结点
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

#### 在指定位置之前添加一个元素

```java
public void add(int index, E element) {
    // 检查指定位置是否符合当前链表范围
    checkPositionIndex(index);

    // 如果指定位置为size，说明是添加到尾部
    if (index == size)
        linkLast(element);
    else
        // 在指定位置的结点前添加
        linkBefore(element, node(index));
}

// 范围为 [0, size]
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}

Node<E> node(int index) {
    // assert isElementIndex(index);
    // 如果指定位置小于size的一半，从头部开始遍历，否则从尾部开始遍历
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    // 用后继结点找到前驱结点
    final Node<E> pred = succ.prev;
    // 用前驱和后继结点生成保存新增元素的结点
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 将后继结点的更新为新增结点
    succ.prev = newNode;
    // 如果前驱结点为null，说明添加到头部
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

#### 在结尾增加一个集合的所有元素

```java
public boolean addAll(Collection<? extends E> c) {
    // 调用addAll()方法，指定位置为size
    return addAll(size, c);
}
```

#### 在指定位置增加一个集合的所有元素

```java
public boolean addAll(int index, Collection<? extends E> c) {
    // 范围位置是否符合范围
    checkPositionIndex(index);

    // 如果指定集合为null，抛出NPE
    Object[] a = c.toArray();
    int numNew = a.length;
    // 如果新增元素长度为0，直接返回false
    if (numNew == 0)
        return false;

    // 找到指定位置的结点，设为后继，后继结点设为前驱
    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    // 使用前驱将集合添加到链表中
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        // 新建结点，向前指向前驱，向后指向null
        Node<E> newNode = new Node<>(pred, e, null);
        // 如果前驱为null，说明第一个元素添加到开头
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        // 前驱指向新增的结点
        pred = newNode;
    }

    // 如果后继结点为null，说明添加到结尾。遍历后前驱指向最后一个新增的结点
    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

### 删除元素

#### 删除结尾的一个元素

```java
public E removeLast() {
    // 如果尾部结点为null，抛出异常
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}

private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    // 获取尾部结点的前前一个结点
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    // 更新尾部结点为前一个结点
    last = prev;
    // 如果前一个结点为null，说明链表为空
    if (prev == null)
        first = null;
    else
        prev.next = null; // 便于GC
    size--;
    modCount++;
    return element;
}
```

#### 删除开头一个结点

```java
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

// 队列操作
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

// 栈操作
public E pop() {
    return removeFirst();
}
```

#### 删除指定位置结点

```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

#### 删除一个指定元素

```java
// 删除第一个
public boolean remove(Object o) {
    // 从头部开始遍历，删除第一个
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}

// 删除最后一个
public boolean removeLastOccurrence(Object o) {
    // 从尾部开始遍历
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

### 更新指定位置元素

```java
// 返回旧值
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

### 获取一个元素

#### 获取尾部元素

```java
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}

public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
```

#### 获取头部元素

```java
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

// 队列/栈操作（空队列返回null）
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

// 队列/栈操作（空队列返回抛出异常）
public E element() {
    return getFirst();
}
```

#### 获取指定位置元素

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

## `ArrayList` & `LinkedList`对比

- `ArrayList`底层是数组，长度有限制（`int`最大值0x7fffffff）。它可以快速（`O(1)`）获取元素（扩容、移动数组），但增删元素需要而外的空间进行复制
- `LinkedList`底层是链表，长度无限制。获取元素方面比较慢（`O(n)`），但可以方便增删（增加节点、缩减节点）
- 实际使用时，`LinkedList`在增加因为需要不断地创建对象导致其速度不一定快于，甚至慢于`ArrayList`，因此一般使用`ArrayList`，并在尽可能指定初始长度

## `ArrayDeque`

```java
public class ArrayDeque<E> extends AbstractCollection<E>
                           implements Deque<E>, Cloneable, Serializable
{
    /** 保存元素的数组 */
    transient Object[] elements; // non-private to simplify nested class access

    /** 队列头 */
    transient int head;

    /**
     * 队列尾
     * 当head < tail时，head ~ tail内的元素就是入队的元素
     * 当head > tail时，head ~ elements[elements.length - 1]、elements[0]内的元素就是入队的元素
     * 当head == tail时，说明elements已经放满，再添加元素需要扩容
     */
    transient int tail;

    /** 默认初始化容量 */
    private static final int MIN_INITIAL_CAPACITY = 8;

    ...
}
```

- 实现了`List`并继承自`AbstractList`：支持列表操作
- 实现了`Cloneable`：支持拷贝
- 实现了`Deque`：支持双端队列操作
- 实现了`java.io.Serializable`：支持序列化
- 内部使用`Object[]`保存数据
- 默认容量为`16`，最小初始容量为`8`
- 扩容时增长一倍

### 构造器

```java
public ArrayDeque() {
    elements = new Object[16];
}

public ArrayDeque(int numElements) {
    allocateElements(numElements);
}

public ArrayDeque(Collection<? extends E> c) {
    allocateElements(c.size());
    addAll(c);
}

private void allocateElements(int numElements) {
    elements = new Object[calculateSize(numElements)];
}

private static int calculateSize(int numElements) {
    int initialCapacity = MIN_INITIAL_CAPACITY;
    // Find the best power of two to hold elements.
    // Tests "<=" because arrays aren't kept full.
    if (numElements >= initialCapacity) {
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1);
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        initialCapacity++;

        if (initialCapacity < 0)   // Too many elements, must back off
            initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
    }
    return initialCapacity;
}
```

### 增加元素

#### 在结尾增加一个元素

```java
public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[tail] = e;
    // (tail + 1) & (elements.length - 1)相当于tail + 1对elements.length取模（参考HashMap）
    // 如过head ~ elements.length都放满了，开始放到0 ~ head之间
    // head == tail，放满了，容量扩展一倍
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        doubleCapacity();
}

public boolean offerLast(E e) {
    addLast(e);
    return true;
}

// 队列操作
public boolean offer(E e) {
    return offerLast(e);
}

private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // number of elements to the right of p
    int newCapacity = n << 1;
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    // head ~ n - 1的元素放到新数组的0 ~ r - 1
    // tail ~ head - 1的元素放到新数组的r ~ n - 1
    // 使旧数组的元素按head ~ tail顺序放置于新数组的前半部分
    System.arraycopy(elements, p, a, 0, r);
    System.arraycopy(elements, 0, a, r, p);
    elements = a;
    head = 0;
    tail = n;
}
```

#### 在开头增加一个元素

```java
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}

public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

// 栈操作
public void push(E e) {
    addFirst(e);
}
```

### 删除元素

#### 删除结尾的一个元素

```java
public E pollLast() {
    // tail向前移动一位（如果到了-1），-1 & n = n，所以t为elements.length - 1（最后）
    int t = (tail - 1) & (elements.length - 1);
    @SuppressWarnings("unchecked")
    E result = (E) elements[t];
    if (result == null)
        return null;
    elements[t] = null;
    tail = t;
    return result;
}

public E removeLast() {
    E x = pollLast();
    if (x == null)
        throw new NoSuchElementException();
    return x;
}
```

#### 删除开头一个结点

```java
public E pollFirst() {
    int h = head;
    @SuppressWarnings("unchecked")
    E result = (E) elements[h];
    // Element is null if deque empty
    if (result == null)
        return null;
    elements[h] = null;     // Must null out slot
    head = (h + 1) & (elements.length - 1);
    return result;
}

// 队列操作
public E poll() {
    return pollFirst();
}

public E removeFirst() {
    E x = pollFirst();
    if (x == null)
        throw new NoSuchElementException();
    return x;
}

// 栈操作
public E pop() {
    return removeFirst();
}
```

### 获取一个元素

#### 获取尾部元素

```java
public E getLast() {
    @SuppressWarnings("unchecked")
    E result = (E) elements[(tail - 1) & (elements.length - 1)];
    if (result == null)
        throw new NoSuchElementException();
    return result;
}

public E peekLast() {
    return (E) elements[(tail - 1) & (elements.length - 1)];
}
```

#### 获取头部元素

```java
public E getFirst() {
    @SuppressWarnings("unchecked")
    E result = (E) elements[head];
    if (result == null)
        throw new NoSuchElementException();
    return result;
}

public E peekFirst() {
    // elements[head] is null if deque empty
    return (E) elements[head];
}

public E peek() {
    return peekFirst();
}

public E element() {
    return getFirst();
}
```
