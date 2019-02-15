### 构造方法

```java
/**
 * 默认初始容量大小
 */
private static final int DEFAULT_CAPACITY = 10;
    
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 1.默认构造函数，使用初始容量 10 构造一个空列表(无参数构造)
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
    
/**
 * 2.带初始容量参数的构造函数（用户自己指定容量）
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) { // 初始容量大于 0
        // 创建 initialCapacity 大小的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {//初始容量等于 0
        // 创建空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else { // 初始容量小于 0，抛出异常
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}

/**
 * 3.构造包含指定 collection 元素的列表，这些元素利用该集合的迭代器按顺序返回
 *   如果指定的集合为 null，throws NullPointerException
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
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

​	<font color=red>*注意*</font> ：以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为 `10`。



### 扩容机制

#### add 方法

```java
/**
 * 将指定的元素追加到此列表的末尾。 
 */
public boolean add(E e) {
    // 添加元素之前，先调用 ensureCapacityInternal 方法
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 这里看到 ArrayList 添加元素的实质就相当于为数组赋值
    elementData[size++] = e;
    return true;
}
```

#### ensureCapacityInternal 方法

```java
// 得到最小扩容量
private void ensureCapacityInternal(int minCapacity) {
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
}
```

```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 如果 elementData 为空数组（如刚通过无参构造创建完成），返回初始默认容量 10
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    // 否则返回传入参数，即 size + 1
    return minCapacity;
}
```

#### ensureExplicitCapacity 方法

```java
// 判断是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        //调用 grow 方法进行扩容，调用此方法代表已经开始扩容了
        grow(minCapacity);
}
```

​	如果使用无参构造创建了一个 ArrayList，则 add 进第 1 个元素到 ArrayList 时，elementData.length 为 0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为10。此时，`minCapacity - elementData.length > 0 ` 成立，所以会进入 `grow(minCapacity)` 方法。当 add 第2个元素时，minCapacity 为 2，此时 e lementData.length（容量）在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0 ` 不成立，所以不会进入 （执行）`grow(minCapacity)` 方法。添加第 3、4... 到第 10 个元素时，依然不会执行 grow 方法，数组容量都为 10。直到添加第 11 个元素，minCapacity （为 11）比 elementData.length（为10）要大，进入 grow 方法进行扩容。

#### grow 方法

```java
/**
 * 要分配的最大数组大小
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * ArrayList 扩容的核心方法
 */
private void grow(int minCapacity) {
    // oldCapacity 为旧容量，newCapacity 为新容量
    int oldCapacity = elementData.length;
    // 将 oldCapacity 右移一位，其效果相当于 oldCapacity / 2，
    // 位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的 1.5 倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
   // 如果新容量大于 MAX_ARRAY_SIZE，进入(执行) hugeCapacity() 方法来比较 minCapacity 和 MAX_ARRAY_SIZE
   // 如果 minCapacity 大于最大容量，则新容量则为 Integer.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE，即 Integer.MAX_VALUE - 8
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

#### hugeCapacity 方法

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
    return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```



### Arrays.copyOf() 和 System.arraycopy()

#### Arrays.copyOf 方法

```java
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    // copyOf() 内部实际调用了 System.arraycopy() 方法
    System.arraycopy(original, 0, copy, 0,Math.min(original.length, newLength));
    return copy;
}
```

#### System.arraycopy 方法

```java
/**
 * @param  src      the source array.
 * @param  srcPos   starting position in the source array.
 * @param  dest     the destination array.
 * @param  destPos  starting position in the destination data.
 * @param  length   the number of array elements to be copied.
 */
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

#### 联系与区别

**联系**

​	copyOf() 内部实际调用了 System.arraycopy() 方法。

**区别**

​	arraycopy() 需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置；copyOf() 是系统自动在内部新建一个数组，并返回该数组。



### ensureCapacity 方法

```java
/**
 * 如有必要，增加此 ArrayList 实例的容量，以确保它至少可以容纳由 minCapacity 参数指定的元素数
 *
 * @param  minCapacity   所需的最小容量
 */
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```

​	此方法在 ArrayList 内部没有被调用过，是提供给用户调用的。最好在插入大量数据之前调用 `ensureCapacity` 方法，以减少增量重新分配的次数，即减少调用 `grow()` （扩容）的次数。