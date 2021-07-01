单例模式

- 属于创建型模式

- 自行完成实例化，私有化构造函数

单例模式的目标

- 实例唯一性
- 线程安全性

任何情况都需要确保一个类只存在一个实例，不会因为多线程的访问而导致创建多个实例，同时也不会因为多线程而引入新的效率问题

## 1 饿汉式

```java
//原理：通过 JVM 在加载类的时候来完成静态对象的初始化，而这个过程本身就是线程安全的（类初始化锁保证线程安全），无法实现懒加载，完全依赖虚拟机加载类的策略进行加载
//1.1 静态常量
public class Singleton {
    //基于 Class Loader 类加载机制
	private final static Singleton INSTANCE = new Singleton();
	// 私有构造方法，防止被外部直接实例化 
	private Singleton(){}
	public static Singleton getInstance(){
    	return INSTANCE;
	}
}
//1.2 静态全局变量
public class Singleton {
    private static Singleton sInstance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {
        return sInstance;
    }
}
//1.3 静态全局变量 静态代码块
public class Singleton {
	private static Singleton sInstance;
	static {
   	 	sInstance = new Singleton();
	}
	private Singleton() {}
	public Singleton getInstance() {
    	return sInstance;
	}
}
```

- 通过变通避免了多线程的同步问题
- 延长了类加载的时间，效率比较低
- 已经加载，如果最终未使用，就造成了浪费资源

## 2 懒汉式

```java
//2.1 线程不安全
public class Singleton {
    private static Singleton sInstance;
    private Singleton() {}
    /**
    *只适合在单线程情况下使用，如果是多线程情况下，一个线程进入 if (singleton == null) 判断语句块，还没来得及往下执行，另一个线程也通过了这个 if 判断语句，此时就会产生多个实例
    */
    public static Singleton getInstance() {
        if (sInstance == null) {
            sInstance = new Singleton();
        }
        return sInstance;
    }
}
//2.2 上面的例子 getInstance() 方法加 synchronized 关键字，线程安全
public class Singleton {
    private static Singleton sInstance;
    private Singleton() {}
    //对 getInstance() 进行了线程同步，每次 getInstance 要进行同步，大大增加了开销
    public synchronized static Singleton getInstance() {
        if (sInstance == null) {
            sInstance = new Singleton();
        }
        return sInstance;
    }
}
//2.3 实例化外面加了 synchronized(Singleton.class) { } 代码块，线程不安全
public class Singleton {
	private static Singleton sInstance;
	private Singleton() {}
	public static Singleton getInstance() {
    if (sInstance == null) {
        //多个线程仍旧能够通过 if 判断，虽然同步了实例化的代码，但还是会多次实例化
        synchronized (Singleton.class) {
            sInstance = new Singleton();
        }
    }
    return sInstance;
	}
}
//2.4 上面的例子 if 判断外面加 synchronized(Singleton.class) {} 代码块，其实和 2.2 一样，线程安全
public class Singleton {
    private static Singleton sInstance;
    private Singleton() { }
    public static Singleton getInstance() {
        synchronized (Singleton.class) {
            if (sInstance == null) {
                sInstance = new Singleton();
            }
        }
        return sInstance;
    }
}
```

## 3  双重检查锁

```java
// Double-Checked Locking 双重检查锁 DCL，进行了两次判空
public class Singleton {
    //volatile 关键字声明，会在编译时加 lock，禁止了指令重排序
    private static volatile Singleton sInstance;
    private Singleton() {}
    public static Singleton getInstance() {
        //判断一次避免不必要的同步锁
        if (sInstance == null) {
            synchronized (Singleton.class) {
                if (sInstance == null) {
                    sInstance = new Singleton();
                }
            }
        }
        return sInstance;
    }
}
```

- 线程安全，延迟加载，效率较高

- 需要使用 volatile 的原因

  系统执行 sInstance = new Singleton() 这段代码不是一次性完成的，大概分为三步指令

  1 为需要创建的 Singleton 实例分配内存
  2 调用对应 Singleton 的构造方法
  3 将 sInstance 指向前面分配的内存空间，此时 sInstance 不为 null 了

  而指令重排序可能会出现先执行 3 再执行 2 的情况，所以当一个线程执行了 1 和 3 但还没执行 2 (后面走完就会创建一个实例)，而此时另一个线程就能通过空判断而创建了另一个实例

  

  

## 4 静态内部类

```java
//
public class Singleton {
    private Singleton() {}
	//静态内部类
    private static class SingletonInstanceHolder {
        //内部类的加载机制，类的静态属性只会在第一次加载该类的时候进行初始化
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() {
        return SingletonInstanceHolder.INSTANCE;
    }
}
```

- 避免了线程不安全，延迟加载，效率高

## 5 枚举式

```java
public enum Singleton {
    INSTANCE;
    public void doSome1(){
    }
}
```

- 避免了线程不安全
- 防止反序列化重新创建新的对象



注意

1 以上方案不考虑反序列化的情况

- 反序列化时会调用 readResolve() 方法重新生成一个实例，所以需要覆写直接返回单例对象

2 通过用 Map 存值的方式也可以实现单例的概念

3 Android 有自带的 Singleton 抽象类

```java

/**
 * Singleton helper class for lazily initialization.
 *
 * Modeled after frameworks/base/include/utils/Singleton.h
 *
 * @hide
 */
//源码位置 /frameworks/base/core/java/android/util/Singleton.java
public abstract class Singleton<T> {
    private T mInstance;

    protected abstract T create();

    public final T get() {
        synchronized (this) {
            if (mInstance == null) {
                mInstance = create();
            }
            return mInstance;
        }
    }
}
```



思考：在 Java 中构造一个普通对象的成本比较低，那为什么针对单例模式要如此纠结是否是懒加载呢？

- 加载时：通常单例的生命周期较长，假设单例类本身在初始化的时候涉及大量业务逻辑，比较复杂或耗时，那是不是不太适合懒加载，反而需要提前加载呢 ？
- 加载后：假设单例类初始化后持有了大量资源，如果初始化后却没能使用上了就显得很浪费，那不管用没用上内存的风险一直都存在着，是否就意味着本身就需要进行内存优化了呢？

所以如果把单例类设计得比较轻，是否就不用过多去纠结懒加载的问题呢？