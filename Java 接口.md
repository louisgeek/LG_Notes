
# 接口
- 接口不能直接使用，必须有一个实现类来实现该接口，实现类必须实现接口中所有的抽象方法，除非这个实现类自己就是抽象类
- 接口默认方法、静态方法都可以有多个，默认方法通过实例对象调用，静态方法通过接口名调用，可以直接用接口名.方法名调用静态方法
- 接口可以看成是对类某种行为的抽象，所以接口和实现类之间的关系显得并没有很强的关联性


## 标记接口
没有包含任何内容的接口通常称为标记接口，实现它一般用来表示这个实现类是一个这种的类，最常见的就是 Serializable 接口，用来标记它的实现类是可以序列化的


## 接口成员
- 常量
- 抽象方法
- 默认方法 jdk 1.8
- 静态方法 jdk 1.8
- 私有方法 jdk 1.9
- 私有静态方法 jdk 1.9


```java
//接口都是隐式抽象的，默认用 abstract 来修饰 interface，abstract 可省略
//可以用 public 或者 default(这个指的是默认不写修饰符，写了编译器反而会报错的，注意和方法的 default 之间的区别) 来修饰 interface
public abstract interface TestJava { 

    //接口中的成员变量只能用 public static final 来修饰的，可省略，所以就是是常量，（其实这里可以看出接口可以用作常量类来使用，但不推荐）
    public static final int a = 0;
    public static final String b = null;

    //接口中的方法默认是 public 和 abstract 的，均可省略
    public abstract void testAbstract(String test);

    //Java 1.8 以后接口里可以有 default 方法，方便后期对接口作升级，就不需要被迫写一个空实现了，默认方法可以被接口实现类重写
    public default void testDefault(String test) {

    }

    //Java 1.8 以后接口里可以有 static 方法，为接口提供了一种不必创建对象就能调用方法能力，方便在不同实现类中重复调用这个方法，静态方法不可以被接口实现类重写
    public static void testStatic(String test) {

    }

    //Java 1.9 以后接口里可以有 private 方法，方便接口内部写复用的代码
    private void testPrivate(String test) {

    }

    //Java 1.9 以后接口里可以有 private static 方法
    private static void testPrivateStatic(String test) {

    }

}
```

## 抽象方法
抽象方法不能用 protected 修饰

## 接口默认方法
- 默认方法具有类优先原则，如果一个类 A 和一个接口 B 具有相同名称和参数的方法，如果此时有一个类继承 A 和实现接口 B，那么接口中的那个默认方法会被忽略
- 默认方法可能会出现接口冲突，如果一个接口 A 和一个接口 B 具有相同名称和参数的方法 ，如果此时有一个类都实现了接口 A 和 B，那么该类必须覆写该方法显式解决处理这个冲突
 


## 接口静态方法
- 静态方法不能被继承及重写