JVM 的内存分类



 

stack 栈

- 每一个线程有一个栈

- 存放基本数据类型
- 存放引用类型对象的引用



heap 堆

- JVM 中只有一个堆区

- 存放引用类型对象自身



method area 方法区

- 又叫静态区

- JVM 中只有一个方法区

- 存放所有的class，静态变量

  



方法区存放类的元数据(Class)、堆中存放new出来的对象(Class实例)、栈中存放引用变量

```java
Person person = new Person();
//person 进栈
//new Person() 进堆
//Person 类加载字节码文件，编译的 Person 类进方法区


```

栈指向堆、堆指向方法区