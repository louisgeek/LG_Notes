# CopyOnWriteArrayList 和 Collections#SynchronizedList 的区别



## CopyOnWriteArrayList

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

}
```

在执行修改操作(add 和 remove 等方法)时，会 copy 一份新的数组进行相关的操作，在执行完修改操作后将原来集合指向新的集合

使用ReentrantLock可重入锁来保证不会有多个线程同时copy一个新的数组

private volatile transient Object[] array;//保证了线程的可见性，读写操作互不影响

## Collections#SynchronizedList

```java
static class SynchronizedList<E>
        extends SynchronizedCollection<E>
        implements List<E> {
       
}
```

### Collections#SynchronizedCollection

```java
//使用 iterator 遍历的时候仍然需要手动加锁
public Iterator<E> iterator() {
    //Collection
    return c.iterator(); // Must be manually synched by user!
}
```

- 大部分方法都加了对象锁，迭代方法没有加，需要在使用迭代的时候自行处理线程同步问题
- 由于遍历的时候需要加锁，增加了锁开销，性能上有一定影响

```java
//用法示例
List list = Collections.synchronizedList(new ArrayList());
 ...
synchronized (list) {
      Iterator i = list.iterator(); // Must be in synchronized block
      while (i.hasNext())
      foo(i.next());
}
```





## 总结

Collections#SynchronizedList 适合对数据时效要求较高的情况，但是因为读写全都加锁，所有效率较低
CopyOnWriteArrayList 效率较高，适合读多写少的场景，因为在读的时候读的是旧集合，所以它的实时性不高

 