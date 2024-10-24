

## ArrayList 和 Vector 的区别

Vector 底层原理和 ArrayList 基本一致，都是采用数组实现    都是使用数组方式存储数据

ArrayList 和 Vector 底层都是数组实现的，ArrayList 和 Vector 的绝大部分方法都是一样的，甚至连方法名都差不多,只是因为 Vector 的方法大多都添加了 synchronized 方法锁



## ArrayList

- 继承 AbstractMap
- 线程不安全的，效率较高
- 内部 Object[] elementData 数组加了 transient
- 默认初始容量为 10，每次扩充容量扩充为原来的1.5倍



## Vector（不推荐使用）

- 继承 Dictionary
- 方法都加了 synchronized 关键字，都是同步的，线程安全的，效率较低
- 内部 Object[] elementData 数组不加 transient
- 当 capacityIncrement 扩容增量大于0时，扩充为原有容量加上capacityIncrement的容量值。否则采取在原有容量基础上扩充为原来的1.5倍





ArrayList提供的writeObject和readObject方法来实现定制序列化，而Vector只是提供了writeObject方法，并没有完全实现定制序列化



