# `ThreadLocal`

该类提供了线程局部(thread-local)变量。这些变量不同于它们的普通对应物，因为访问某个变量（通过其`get/set`方法）的每个线程都有自己的局部变量，它独立于变量的初始化副本。`ThreadLocal`实例通常是类中的`private static`字段，它们希望将状态与某一个线程（例如，用户ID或事务ID）相关联。

也就是说：通常情况下，我们创建的变量是可以被任何一个线程访问并修改的，而使用`ThreadLocal`创建的变量只能被当前线程访问，其他线程则无法访问和修改。

## 实现原理

`ThreadLocal`有一个嵌套类`ThreadLocalMap`，是一个特制的`HashMap`，它使用所在线程的弱引用作为`Map`的键，线程中`set`方法设置的值作为`Map`的值生成`Entry`。

当一个线程调用`ThreadLocal`的`get`方法时，实际上是以该线程作为键在`Map`中查找对应的值。也就是说，一个线程对应一个值，且这些值不可被其他线程访问。当然，如果使用的是`InheritableThreadLocal`，一个线程可以继承定义该线程的线程（父线程）的`ThreadLocal`值。

**每一个`ThreadLocal`都对应一个`ThreadLocalMap`。**

## 源码分析

```java
public void set(T value) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 获取该线程对应的ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    // 如果ThreadLocalMap存在，将对应的Entry值更新
    // 否则创建一个新ThreadLocalMap，并将键值对插入
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

```java
public T get() {
    Thread t = Thread.currentThread();
    // 获取对应的ThreadLocalMap
    `ThreadLocal`Map map = getMap(t);
    if (map != null) {
        // map查找Entry
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // map不存在则使初始值创建一个map，并返回初始值
    return setInitialValue();
}

private T setInitialValue() {
    // 将初始值加入Map
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    // 返回初始值
    return value;
}
```

# 内部嵌套类`ThreadLocalMap`

`ThreadLocalMap`是一个定制的散列映射，只适合维护线程本地值。在`ThreadLocal`类之外不能进行任何操作。类是包私有的，允许在类线程中声明字段。为了帮助处理非常大且长期存在的使用，哈希表的`Entry`继承自`WeakReference`。但是，由于不使用引用队列，所以只有当表开始耗尽空间时，才保证删除旧的`Entry`。

```java
static class ThreadLocalMap {

        /**
         * 继承自WeakReference，使用一个ThreadLocal对象作为键，
         * 且键为弱引用，当哈希表空间不足时会被回收
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** 与当前`ThreadLocal`关联的值 */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * 初始容量，必须是2的幂
         */
        private static final int INITIAL_CAPACITY = 16;

        ...

        /**
         * ThreadLocalMap是延时加载（lazy-load）,只有有Entry时才初始化
         */
        ThreadLocalMap(`hreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

        /**
         * 继承父类的ThreadLocal值，只有在createInheritedMap方法被调用时使用
         */
        private ThreadLocalMap(ThreadLocalMap parentMap) {
            Entry[] parentTable = parentMap.table;
            int len = parentTable.length;
            setThreshold(len);
            table = new Entry[len];

            for (int j = 0; j < len; j++) {
                Entry e = parentTable[j];
                if (e != null) {
                    @SuppressWarnings("unchecked")
                    ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                    if (key != null) {
                        Object value = key.childValue(e.value);
                        Entry c = new Entry(key, value);
                        int h = key.threadLocalHashCode & (len - 1);
                        while (table[h] != null)
                            h = nextIndex(h, len);
                        table[h] = c;
                        size++;
                    }
                }
            }
        }
    }
}
```

# `InheritableThreadLocal`

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T>
```

该类扩展了`ThreadLocal`，为子线程提供从父线程那里继承的值：在创建子线程时，子线程会接收所有可继承的线程局部变量的初始值，以获得父线程所具有的值。通常，子线程的值与父线程的值是一致的；但是，通过重写这个类中的`childValue`方法，子线程的值可以作为父线程值的一个任意函数。

当必须将变量（如用户ID和事务ID）中维护的每线程属性（per-thread-attribute）自动传送给创建的所有子线程时，应尽可能地采用可继承的线程局部变量，而不是采用普通的线程局部变量。
