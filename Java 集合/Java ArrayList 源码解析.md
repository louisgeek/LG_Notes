ArrayList

```java
//继承自 AbstractList<E> 
//实现了 List<E> 
//实现了 RandomAccess 接口，支持随机访问
//实现了 Cloneable 接口,代表可复制的
//实现了 Serializable 可被序列化
public class ArrayList<E> extends AbstractList<E>
implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

## 全局变量

```java
 	/**
 	  * Default initial capacity.
     *  定义数组的默认初始容量是 10 
     */
    private static final int DEFAULT_CAPACITY = 10;
    /**
    共享空数组实例
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
    /**
    共享数组 默认容量 
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    // Android-note: Also accessed from java.util.Collections
    //transient 代表不序列化
    transient Object[] elementData; // non-private to simplify nested class access
    /**
     * The size of the ArrayList (the number of elements it contains).
     * 数组里存放元素的数量
     * @serial
     */
    private int size;
```

## 构造函数

```java
   /**
     * Constructs an empty list with the specified initial capacity.
     *	指定初始容量的构造
        如果在明确容量的情况下可调用该构造，省去 ArrayList 自动扩容而产生的消耗
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //指定 initialCapacity 大于 0 
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //等于 0 直接赋值空数组实例
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //小于 0 
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
    /**
     * Constructs an empty list with an initial capacity of ten.
       无参构造 
     */
    public ArrayList() {
        //直接指向 共享的默认容量的空数组
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     * 带集合参数的构造
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
        //直接转换成数组进行赋值
        elementData = c.toArray();
        // elementData.length 长度赋值给 size ，同时判断是不是不为 0 
        if ((size = elementData.length) != 0) {
            //Object[].class 可能有 bug 吧
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                //？
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            //直接赋值空数组实例
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

## 增

```java
 /**
     * Appends the specified element to the end of this list.
     *往数组尾增加元素
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        //确认容量
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
  /**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        //校验 index 
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
 		//同样也要确认容量
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //Object src, int srcPos,Object dest, int destPos, int length
        //源数组 源数组的 pos 目标数组 拷贝长度
        //假设 size= 8 index=3    [aa,bb,cc,dd,ee,ff,gg,hh]
        //elementData，3 (dd)，elementData，4，5 (变成了[aa,bb,cc,  ,dd,ee,ff,gg,hh])
        //其实就是通过拷贝把目标索引的值和它后面的所有值往后挪了一位
         System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        //[aa,bb,cc,element,dd,ee,ff,gg,hh]
        elementData[index] = element;
        size++;
    } 
 /**
     * Appends all of the elements in the specified collection to the end of
     * this list, in the order that they are returned by the
     * specified collection's Iterator.  The behavior of this operation is
     * undefined if the specified collection is modified while the operation
     * is in progress.  (This implies that the behavior of this call is
     * undefined if the specified collection is this list, and this
     * list is nonempty.)
     *
     * @param c collection containing elements to be added to this list
     * @return <tt>true</tt> if this list changed as a result of the call
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(Collection<? extends E> c) {
        //集合直接转成数组
        Object[] a = c.toArray();
        int numNew = a.length;
        //确认容量
        ensureCapacityInternal(size + numNew);  // Increments modCount
        //直接把源数组 a 从索引 0 开始复制存到原 elementData 最后一个索引，即 size 的地方，相当于把所有元素按顺序添加到数组后面
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        //如果添加是空集合 返回false
        return numNew != 0;
    }
   /**
     * Inserts all of the elements in the specified collection into this
     * list, starting at the specified position.  Shifts the element
     * currently at that position (if any) and any subsequent elements to
     * the right (increases their indices).  The new elements will appear
     * in the list in the order that they are returned by the
     * specified collection's iterator.
     *
     * @param index index at which to insert the first element from the
     *              specified collection
     * @param c collection containing elements to be added to this list
     * @return <tt>true</tt> if this list changed as a result of the call
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        //校验 index
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
   		//集合直接转成数组
        Object[] a = c.toArray();
        int numNew = a.length;
         //确认容量
        ensureCapacityInternal(size + numNew);  // Increments modCount
		//前面已经校验，index 肯定小于等于 size 且大于等于 0 
        int numMoved = size - index;
        //如果等于0 就意味着  size = index ，也就说明不需要挪位，直接走下面的 System.arraycopy
        //挪位进行空出 numNew 数量的位置
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);
		//拷贝的方式把对应空位补上
        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
         //如果添加是空集合 返回false
        return numNew != 0;
    }
```

### ArrayList#ensureCapacityInternal

```java
    private void ensureCapacityInternal(int minCapacity) {
        //判断是否指向 DEFAULTCAPACITY_EMPTY_ELEMENTDATA ，结合上下文换句话说是否是无参构造出来的
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            //10 和 1 比较得到 10
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }
    
    
```

### ArrayList#ensureExplicitCapacity

```java
 private void ensureExplicitCapacity(int minCapacity) {
         //AbstractList 里的 protected transient int modCount = 0;
        modCount++;
		//容量不够 需要扩容
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

### ArrayList#grow

```java
/**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        //记录旧的容量
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //假设 oldCapacity = 10
        //1111 = 1010 + 0101
        //15 = 10 + 5
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //新的容量必须 >= minCapacity
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //新的容量也不能超过 MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8 
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            //如果大于 MAX_ARRAY_SIZE 继续处理
            //判断如果 minCapacity < 0 抛出 OutOfMemoryError 异常
            //如果 minCapacity > MAX_ARRAY_SIZE 返回 Integer.MAX_VALUE
            //如果 minCapacity <= MAX_ARRAY_SIZE 返回 MAX_ARRAY_SIZE
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```









## 总结



由于利用数组本身实现增删元素操作比较麻烦，而 ArrayList 的实现恰恰可以看成是对数组操作的一种封装



插入时会判断数组容量是否足够，不够的话会进行扩容,每次扩容的容量是当前的1.5倍

- 扩容就是新建一个新的数组，然后将老的数据里面的元素复制到新的数组里面(所以增加较慢)
- 基于数组拷贝实现















