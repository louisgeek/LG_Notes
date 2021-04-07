Java 反射

- 就是把类中的各种元素映射成一个个对象

反射机制

- 实现了在程序运行状态下，能够动态获取对象的属性、调用对象的方法





- 提高了程序的灵活性和扩展性
- 降低耦合性
- 可以在不依赖目标类的情况下完成代码的编译





使用场景

- 可以做到在运行时下访问私有的成员和方法
- 开发通用框架，读取文本配置文件来完成不同配置的初始化
- 项目中动态加载 string 文件中的 fragment 进行实例化







```java
//
Class<?> clazz = Class.forName("com.louisgeek.OuterClass");
//用 com.louisgeek.OuterClass.InnerClass 会报 java.lang.ClassNotFoundException
Class<?> clazz = Class.forName("com.louisgeek.OuterClass$InnerClass");

Field[] fields = clazz.getDeclaredFields();
Field[] fields = clazz.getFields();

Method[] methods = clazz.getDeclaredMethods();
Method[] methods = clazz.getMethods();

Constructor<?>[] constructors = clazz.getDeclaredConstructors();
Constructor<?>[] constructors = clazz.getConstructors();

Annotation[] annotations = clazz.getAnnotations();
Annotation[] annotations = clazz.getDeclaredAnnotations();

Class<?>[] classes = clazz.getDeclaredClasses();
Class<?>[] classes = clazz.getClasses();


Package _package = clazz.getPackage();
Class<?>[] interfaces = clazz.getInterfaces();
int modifiers = clazz.getModifiers();

//
String typeName = clazz.getTypeName();
String name = clazz.getName();
String simpleName = clazz.getSimpleName();
//canonical 准确的
String canonicalName = clazz.getCanonicalName();
//================= Class.forName("com.louisgeek.OuterClass") ===========
typeName：com.louisgeek.OuterClass
name：com.louisgeek.OuterClass
simpleName：OuterClass
canonicalName：com.louisgeek.OuterClass
//================= Class.forName("com.louisgeek.OuterClass$InnerClass") ===========   
typeName：com.louisgeek.OuterClass$InnerClass
name：com.louisgeek.OuterClass$InnerClass
simpleName：InnerClass
canonicalName：com.louisgeek.OuterClass.InnerClass
```

