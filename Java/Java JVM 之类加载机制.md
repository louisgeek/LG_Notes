类的加载

- 类初始化

指在程序运行期间，类从磁盘加载到内存中过程，经历了：加载、连接、初始化这三个阶段





加载阶段

java 文件编译成 class 文件



.class 文件的字节码

将类的.class文件中的二进制数据读入到内存中，并将其放在运行时数据区的方法区内



.class 来源

- 磁盘直接读取
- 网络下载
- rar，zip，jar 等文件中加载
- .java 文件动态编译成的 .class 文件



- java.lang.ClassLoader









（1）类名.class：JVM将使用类装载器，将类装入内存(前提是:类还没有装入内存)，不做类的初始化工作，返回Class 的对象。

（2）Class.forName("类名字符串")：装入类，并做类的静态初始化，返回Class的对象。

（3）实例对象.getClass()：对类进行静态初始化、非静态初始化；返回引用运行时真正所指的对象(子对象的引用会赋给父对象的引用变量中)所属的类的Class的对象。





- 方法区、Heap

- 动态代理模式
- AndroidDavilk与ART
- PathClassLoader、DexClassLoader与BootClassLoader
- 双亲委托机制



