集合

- 一个 Java 对象可以在内部持有若干个其他 Java 对象，并且对外提供访问的接口



数组也可以看作是一种集合

- 但是数组初始化后大小就固定了，不可变了
- 只能按照索引取数据



所以需要一些功能更加强大的集合：List，Set，Map 等



List

- 有顺序的列表集合，元素可以重复，可以添加 null



Set

- 元素唯一无重复的集合



Map

- 通过键值对形成的映射表集合，key 不会重复，value 可以重复，没有顺序





由于长期的迭代和历史原因，有些集合类不推荐使用：Hashtable，Vector，Stack 等





Collections 集合工具类的 Collections#SynchronizedMap

并发包下的 ConcurrentHashMap





### Collections#SynchronizedSet





