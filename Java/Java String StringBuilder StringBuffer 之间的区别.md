# String StringBuilder StringBuffer 之间的区别



## String 字符串常量

- String 类被 final 修饰
- String 类中维护的 char[] 数组也是 final 修饰的

String 类是无法被继承和改变的，是不可变对象，每次对 String 对象进行改变，如字符串拼接、截取等操作都会生成一个新的 String 对象后将指针指向新的 String 对象，所以效率比较低，性能最差，对于要经常改变内容的字符串不推荐使用 String 类



## StringBuilder 字符串变量（非线程安全）

- 继承自 AbstractStringBuilder

AbstractStringBuilder 类中维护了一个 char[] 数组，没加 final 修饰，初始化数组容量为 16

操作时当 char[] 的空间不足时会进行扩容，其中 Arrays#copyOf 利用 native 方法 System#arraycopy  进行 copy 操作，并不会像 String 类那样直接生成新的 String 对象，所以性能较好

调用 toString 方法时会使用通过创建 String ，内部利用 Arrays#copyOfRange 进行生成一个 String 的对象返回

- 不保证线程安全，适合在单线程的情况下使用

## StringBuffer 字符串变量（线程安全）

- 继承自 AbstractStringBuilder

StringBuffer 和 StringBuilder 逻辑基本一样，不过 StringBuffer 方法带有 synchronized 锁，能够保证线程安全，适合多线程的情况下使用，所以性能稍比 StringBuilder 差点

调用 toString 方法时如果 toStringCache 不为 null 则会使用 toStringCache 这个 char[] 数组共享赋值给 String 的 char[] 数组进行生成一个 String 的对象返回，其中 toStringCache 每次记录的是最后一次 toString 执行后的值



## 总结

性能上通常 StringBuilder > StringBuffer > String

- String 适用于少量的字符串操作的情况，StringBuilder 适用于单线程下在字符缓冲区进行大量操作的情况，StringBuffer 适用于多线程下在字符缓冲区进行大量操作的情况
- StringBuilder 每次调用 toString 都需要复制数组，而 StringBuilder 只有当 toStringCache 是空的时候（当字符串发生变化后会置空）才需要复制数组，否则直接共享赋值