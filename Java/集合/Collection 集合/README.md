

 Collection 集合



Collection

- List
  - java.util.ArrayList
  - Vetor
  - LinkedList
  - CopyOnWriteArrayList

- Set
  - HashSet
  - LinkedHashSet
- TreeSet
  
  
  
  ！！！！
  
- java.util.Collections.SynchronizedCollection
  - java.util.Collections.SynchronizedList
    - java.util.Collections.SynchronizedRandomAccessList
  
- java.util.Collections.SynchronizedSet



## List

- 不推荐 for 和 get 进行循环变量
- 推荐用 Iterator 迭代器进行遍历
- 可以直接使用 for each 遍历 List，Java 编译器会自动改用 Iterator 

| List     | ArrayList          | Vector         | CopyOnWriteArrayList | LinkedList         |
| -------- | ------------------ | -------------- | -------------------- | ------------------ |
| 底层实现 | 数组               | 数组           |                      | 链表               |
| 线程安全 | 线程不安全         | 线程安全       |                      |                    |
| 效率     | 效率高             | 效率低         |                      |                    |
|          | 查快，增删慢  改？ | 查快，增删改慢 |                      | 查慢，增删快  改？ |
| 增加     |                    |                |                      |                    |
| 删除     |                    |                |                      | 快                 |
| 修改     |                    |                |                      |                    |
| 查找     | 快                 | 快             |                      | 慢                 |





## ArrayList 和 Vector 的区别

- Vector 底层原理和 ArrayList 基本一致，都是采用数组实现

- ArrayList 是不保证线程安全的，Vector 是线程安全的
- Vector 的增删改查相关的方法都加了 synchronized 关键字，这些方法都是同步的，比较耗时，效率不高





## ArrayList 与 LinkedList 的区别

- ArrayList 底层采用数组实现，LinkedList 底层采用双向循环链表数据结构实现

- 通常情况下首选 ArrayList，经常需要增加和删除操作的时候选 LinkedList
- LinkedList 查找操作需要从头开始查找元素，速度较慢







| Set      | HashSet        | LinkedHashSet  | TreeSet        |
| -------- | -------------- | -------------- | -------------- |
| 线程安全 | 线程不安全     | 线程安全       |                |
| 效率     | 效率高         | 效率低         |                |
| 增删改查 | 查快，增删改慢 | 查快，增删改慢 | 查慢，增删改快 |
| 顺序     | 无序           |                | 有序           |







## Collection#toArray

- ArrayList#toArray

```java
 /**
     * Returns an array containing all of the elements in this list
     * in proper sequence (from first to last element).
     *
     * <p>The returned array will be "safe" in that no references to it are
     * maintained by this list.  (In other words, this method must allocate
     * a new array).  The caller is thus free to modify the returned array.
     *
     * <p>This method acts as bridge between array-based and collection-based
     * APIs.
     *
     * @return an array containing all of the elements in this list in
     *         proper sequence
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * Returns an array containing all of the elements in this list in proper
     * sequence (from first to last element); the runtime type of the returned
     * array is that of the specified array.  If the list fits in the
     * specified array, it is returned therein.  Otherwise, a new array is
     * allocated with the runtime type of the specified array and the size of
     * this list.
     *
     * <p>If the list fits in the specified array with room to spare
     * (i.e., the array has more elements than the list), the element in
     * the array immediately following the end of the collection is set to
     * <tt>null</tt>.  (This is useful in determining the length of the
     * list <i>only</i> if the caller knows that the list does not contain
     * any null elements.)
     *
     * @param a the array into which the elements of the list are to
     *          be stored, if it is big enough; otherwise, a new array of the
     *          same runtime type is allocated for this purpose.
     * @return an array containing the elements of the list
     * @throws ArrayStoreException if the runtime type of the specified array
     *         is not a supertype of the runtime type of every element in
     *         this list
     * @throws NullPointerException if the specified array is null
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```

- toArray() 会丢失类型信息
- 如果传入的数组不够大，那么`List`内部会创建一个新的刚好够大的数组，填充后返回；如果传入的数组比`List`元素还要多，那么填充完元素后，剩下的数组元素一律填充`null`



## java.util.ArrayList 和数组之间互转

java.util.ArrayList -> 数组

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
//List 直接转成数组
String[] array = list.toArray(new String[list.size()]);
```



数组 -> java.util.ArrayList 

```java
String[] array = new String[] {"1", "2", "3","4"};
//注意这里返回的是 java.util.Arrays.ArrayList 而不是 java.util.ArrayList
//可以调 set，get 等，而不能调 add，remove 等，因为没有实现会报 UnsupportedOperationException
List<String> listTemp = Arrays.asList(array);
List<String> list = new ArrayList<>(listTemp);
```





## Collections



### Collections#binarySearch





### Collections#addAll



### Collections#reverse



### 空集合

```
List<String> list = Collections.emptyList();
Map<String,String> map = Collections.emptyMap();
Set<String> set = Collections.emptySet();
```

- 空集合是不可变集合

### synchronized

```
Collection<String> synchronizedCollection = Collections.synchronizedCollection(collection);
Map<String,String> synchronizedMap = Collections.synchronizedMap(map);
List<String> synchronizedList = Collections.synchronizedList(list);
Set<String> synchronizedSet = Collections.synchronizedSet(set);
```





Java 官方推荐用 Deque 替代 Stack 