Java parent-delegation model 双亲委托模型

- 描述类加载机制

类加载器在查找 Class 时候，先判断该类是否已经加载，如果没有，它首先不会自己尝试去加载这个类，而是委托父加载器去查找，往上递归查找到最上层 Bootstrap ClassLoader，如果找到就返回，如果还没找到则继续往下找，最后还没找到就回到了自身类加载器去查找

- 避免重复加载，判断已加载就直接读取已加载的 Class
- 更加安全，让很多系统类在 Java 虚拟机启动的时候就完成加载，如 java.lang.Integer 肯定被委托给 Bootstrap ClassLoader 加载了，而不会轻易被“通过定义同样路径的类”来实现冒充替换



父子加载器之前是 Composition 组合关系，而不是 Inheritance 继承关系

User ClassLoader -> Application ClassLoader -> Extention ClassLoader -> Bootstrap ClassLoader

Java 实现类

XxxClassLoader -> AppClassLoader -> ExtClassLoader -> BootstrapClassLoader

```
ClassLoader#loadClass
- ClassLoader#findLoadedClass 先判断是否已经加载
- 如果为未被加载，parent 不为 null，则调用 parent 的 loadClass 方法去加载，否则利用 Bootstrap ClassLoader 处理，方法是 findBootstrapClassOrNull，因为它是最上面的加载器，没有 parent
- 如果以上没加载成功，抛出异常，然后通过 findClass 去处理，需要自己实现加载，这是推荐的做法，而不是直接去重写 loadClass
```





Tomcat破坏双亲委派原则，提供隔离的机制，为每个web容器单独提供一个WebAppClassLoader加载器,为了实现隔离性，优先加载 Web 应用自己定义的类，所以没有遵照双亲委派的约定，每一个应用自己的类加载器——WebAppClassLoader负责加载本身的目录下的class文件，加载不到时再交给CommonClassLoader加载，这和双亲委派刚好相反

