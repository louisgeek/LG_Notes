

# HashMap 和 Hashtable 的区别



HashMap 和 HashTable 都是基于哈希表实现的，其内部每个元素都是 key-value 键值对，HashMap 和 HashTable 都实现了 Map、Cloneable、Serializable 接口。



## HashMap

- 继承 AbstractMap
- 线程不安全，效率较高
- 允许空的 key 和 value 值
- 默认初始大小为 16，每次扩充容量变为原来的 2 倍



## HashTable（不推荐使用）

- 继承 Dictionary
- 线程安全，效率较低
- 不允许空的 key 和 value 值
- 默认初始大小为 11，每次扩充容量变为原来的 2n+1