## HashMap

- 线程不安全的



```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    
    
}
```



## 构造函数

```java
 
 	/**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
	//用于记录下一个扩容的容量，一般是 capacity * loadFactor
    int threshold;

    /**
     * The load factor for the hash table.
     *
     * @serial
     */
	//装载因子、装载系数
    final float loadFactor;

	/**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
	//传入指定的 initialCapacity 和 loadFactor
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        //MAXIMUM_CAPACITY = 1 << 30  //左移30 就是 1073741824
        //为啥设定左移 30？
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
	//只传入指定的 initialCapacity
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        //DEFAULT_LOAD_FACTOR = 0.75f
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```



### Float#isNaN(float)

```java
/**
     * Returns {@code true} if the specified number is a
     * Not-a-Number (NaN) value, {@code false} otherwise.
     *
     * @param   v   the value to be tested.
     * @return  {@code true} if the argument is NaN;
     *          {@code false} otherwise.
     */
    public static boolean isNaN(float v) {
        //比较有意思！
        return (v != v);
    }
```





### HashMap#tableSizeFor

```java
 /**
     * Returns a power of two size for the given target capacity.
     */
//传入 initialCapacity
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        //无符号右移
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```



### HashMap#putMapEntries

```java
 /**
     * Implements Map.putAll and Map constructor
     *
     * @param m the map
     * @param evict false when initially constructing this map, else
     * true (relayed to method afterNodeInsertion).
     */
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            //定义 transient Node<K,V>[] table;
            if (table == null) { // pre-size
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold)
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                //下文再说
                putVal(hash(key), key, value, false, evict);
            }
        }
    }

```



#### 顺便提一下 HashMap#putAll

```java
   /**
     * Copies all of the mappings from the specified map to this map.
     * These mappings will replace any mappings that this map had for
     * any of the keys currently in the specified map.
     *
     * @param m mappings to be stored in this map
     * @throws NullPointerException if the specified map is null
     */
    public void putAll(Map<? extends K, ? extends V> m) {
        //就是调用 putMapEntries,不过 evict 传的是 true
        putMapEntries(m, true);
    }
```



## HashMap#put

```java
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```



### HashMap#hash

```java
/**
 * Computes key.hashCode() and spreads (XORs) higher bits of hash
 * to lower.  Because the table uses power-of-two masking, sets of
 * hashes that vary only in bits above the current mask will
 * always collide. (Among known examples are sets of Float keys
 * holding consecutive whole numbers in small tables.)  So we
 * apply a transform that spreads the impact of higher bits
 * downward. There is a tradeoff between speed, utility, and
 * quality of bit-spreading. Because many common sets of hashes
 * are already reasonably distributed (so don't benefit from
 * spreading), and because we use trees to handle large sets of
 * collisions in bins, we just XOR some shifted bits in the
 * cheapest possible way to reduce systematic lossage, as well as
 * to incorporate impact of the highest bits that would otherwise
 * never be used in index calculations because of table bounds.
 */
static final int hash(Object key) {
    int h;
    //根据key 的hashcode计算在哈希表中的存储位置
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



### HashMap#putVal

```java
/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //table 未被初始化或者数组长度为 0
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;//初始化 
    //如果数组最后一个元素 p 为 null ？
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);//把新值放在数组最后一个元素的位置
    else {
        //如果数组最后一个元素 p 不为空
        Node<K,V> e; K k;
        if (p.hash == hash &&
            //最后一个元素 p 的 key 等于即将要存放进去的 key，或者最后一个元素 p 的 key 和 即将要存放进去的 key 之间满足 equals 条件，ps：为啥加个中间变量 k ？直接写 p.key 不香么？
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;//存到局部变量 e
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
         //
         for (int binCount = 0; ; ++binCount) {
          if ((e = p.next) == null) {
          //最后一个元素 p 的 next 为 null
          p.next = newNode(hash, key, value, null);//把新值赋给数组最后一个元素的 next 变量
          //？
          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
               treeifyBin(tab, hash);
               break;
          }
          if (e.hash == hash &&
              ((k = e.key) == key || (key != null && key.equals(k))))
           break;
           p = e;
      }
  }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            //onlyIfAbsent 默认 false
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    //evict 默认 true
    afterNodeInsertion(evict);
    return null;
}
```



#### HashMap#resize

- 扩容，返回新的 Node 数组

```java
 /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                //超过了最大的情况就不需要扩容了，直接返回旧的 table 即旧的 Node 数组
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //oldCap 左移 1 位，？
            //ps:如果走到这,能保证 newCap 在 if (oldCap > 0) 该分支内肯定能赋一次 oldCap 的值
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;// threshold 不为 0 说明不是初始时候的情况，可以用作新的容量
        else {
            //oldCap<=0，oldThr<=0 说明是初始化的情况
            // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //走到这，上面的 if 能够保证 newCap 被赋大于 0 的值了，而 newThr 就不能保证了
        if (newThr == 0) {
            //newThr 初始化值为 0 ，后续操作也没有赋值为负的情况，所以判断一次是否是 0，如果等于 0 说明需要初始化
            float ft = (float)newCap * loadFactor;
            //newCap 和下一次需要扩容的容量 ft 都要满足小于最大的容量限制，否则直接赋值 Integer.MAX_VALUE，？
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //赋值给全局变量进行保存
        threshold = newThr;
        //直接 new 出新的容量的 Node 数组
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //赋给全局变量进行保存
        table = newTab;
        //
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```





哈希表

- 又叫散列表



什么是哈希冲突



为啥 DEFAULT_INITIAL_CAPACITY = 1<<4  不直接用 16 表示？



初始容量是多少？什么时候初始化的







如何扩容，合适触发扩容？DEFAULT_LOAD_FACTOR 加载因子为何是 0.75?







HashMap 的数据结构是什么？





hash过程是怎样的？





hash 冲突时怎么搞？

- 给链表插入数据，1.7头插法，1.8尾插法。
