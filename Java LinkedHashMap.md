
LinkedHashMap 查询与 HashMap 性能一致，而插入和删除操作略慢于 HashMap，因为需要维护双向链表，迭代性能显著优于 HashMap，因为直接遍历链表而非哈希桶

synchronizedMap  或使用 ConcurrentLinkedHashMap




可通过构造函数参数accessOrder切换为访问顺序模式



适用场景包括：需要保留插入顺序（如日志记录）、按访问频率排序（如 LRU 缓存 最近最少使用），以及需要高效遍历的场景