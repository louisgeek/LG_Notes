1 饿汉式

```kotlin
object Singleton {
    fun doSome1() {
        println("test")
    }
}
```

Kotlin 调用方式  `Singleton.doSome1()`

Java 调用方式  `Singleton.INSTANCE.doSome1()`

Android Studio 通过 Tools -> Kotlin -> Show Kotlin Bytecode 查看编译后的字节码，点 Decompile 转成反编译后的 Java 代码

```java
//final 声明类不可变性
public final class Singleton {
   public static final Singleton INSTANCE;
   public final void doSome1() {
      String var1 = "test";
      boolean var2 = false;
      System.out.println(var1);
   }
   private Singleton() {}
   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
   }
}
```

- 可以看出是饿汉式的 Java 单例模式
- 线程安全
- 无法进行传参

2 懒汉式

```kotlin
//2.1 线程不安全
class Singleton private constructor() {
    companion object {
        private var instance: Singleton? = null
        @JvmStatic
        fun getInstance(): Singleton {
            if (instance == null) {
                instance = Singleton()
            }
            return instance as Singleton
        }
    }
    fun doSome1() {
        println("test")
    }
}
```
```java
//2.1 反编译后的 Java 代码
public final class Singleton {
   private static Singleton instance;
   public static final Singleton.Companion Companion = new Singleton.Companion((DefaultConstructorMarker)null);
   public final void doSome1() {
      String var1 = "test";
      boolean var2 = false;
      System.out.println(var1);
   }
   private Singleton() {
   }
   // $FF: synthetic method
   public Singleton(DefaultConstructorMarker $constructor_marker) {
      this();
   }
   @JvmStatic
   @NotNull
   public static final Singleton getInstance() {
      return Companion.getInstance();
   }
   public static final class Companion {
      @JvmStatic
      @NotNull
      public final Singleton getInstance() {
         if (Singleton.instance == null) {
            Singleton.instance = new Singleton((DefaultConstructorMarker)null);
         }
         return Singleton.instance;
      }
      private Companion() {
      }
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

```kotlin
//2.2 线程安全，2.1 例子加上 @Synchronized 注解
class Singleton private constructor() {
    companion object {
        private var instance: Singleton? = null
        @JvmStatic
        @Synchronized // 加了 @Synchronized 
        fun getInstance(): Singleton {
            if (instance == null) {
                instance = Singleton()
            }
            return instance as Singleton
        }
    }
    fun doSome1() {
        println("test")
    }
}
```

```java
//2.2 反编译后的 Java 代码
public final class Singleton {
   private static Singleton instance;
   public static final Singleton.Companion Companion = new Singleton.Companion((DefaultConstructorMarker)null);
   public final void doSome1() {
      String var1 = "test";
      boolean var2 = false;
      System.out.println(var1);
   }
   private Singleton() {
   }
   // $FF: synthetic method
   public Singleton(DefaultConstructorMarker $constructor_marker) {
      this();
   }
    //增加了 synchronized 关键字
   @JvmStatic
   @NotNull
   public static final synchronized Singleton getInstance() {
      return Companion.getInstance();
   }
   public static final class Companion {
       //增加了 synchronized 关键字
      @JvmStatic
      @NotNull
      public final synchronized Singleton getInstance() {
         if (Singleton.instance == null) {
            Singleton.instance = new Singleton((DefaultConstructorMarker)null);
         }
         return Singleton.instance;
      }
      private Companion() {
      }
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```
```kotlin
//2.3 kotlin get() 特性写法 
//线程安全
class Singleton private constructor() {
    companion object {
        private var instance: Singleton? = null
            get() {
                if (field == null) {
                    field = Singleton()
                }
                return field;
            }
        //Companion 里已经生成一个 private final Singleton getInstance 方法，所以改用 gainInstance 这个命名
        @JvmStatic
        @Synchronized
        fun gainInstance(): Singleton? {
            return instance
        }
    }
    fun doSome1() {
        println("test")
    }
}
```

```java
//2.3 反编译后的 Java 代码
public final class Singleton {
   private static Singleton instance;
   public static final Singleton.Companion Companion = new Singleton.Companion((DefaultConstructorMarker)null);
   public final void doSome1() {
      String var1 = "test";
      boolean var2 = false;
      System.out.println(var1);
   }
   private Singleton() {
   }
   // $FF: synthetic method
   public Singleton(DefaultConstructorMarker $constructor_marker) {
      this();
   }
   @JvmStatic
   @Nullable
   public static final synchronized Singleton gainInstance() {
      return Companion.gainInstance();
   }
   public static final class Companion {
      private final Singleton getInstance() {
         if (Singleton.instance == null) {
            Singleton.instance = new Singleton((DefaultConstructorMarker)null);
         }
         return Singleton.instance;
      }
      private final void setInstance(Singleton var1) {
         Singleton.instance = var1;
      }
      @JvmStatic
      @Nullable
      public final synchronized Singleton gainInstance() {
         return ((Singleton.Companion)this).getInstance();
      }
      private Companion() {
      }
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}

```

```kotlin
//2.4 kotlin lazy 特性写法
//线程不安全
class Singleton private constructor(){
    companion object {
        @JvmStatic
        val instance by lazy(LazyThreadSafetyMode.NONE) {
            Singleton()
        }
    }
}
```

```java
//2.4 反编译后的 Java 代码
public final class Singleton {
   @NotNull
   private static final Lazy instance$delegate;
   public static final Singleton.Companion Companion = new Singleton.Companion((DefaultConstructorMarker)null);
   private Singleton() {
   }
   static {
      instance$delegate = LazyKt.lazy(LazyThreadSafetyMode.NONE, (Function0)null.INSTANCE);
   }
   // $FF: synthetic method
   public Singleton(DefaultConstructorMarker $constructor_marker) {
      this();
   }
   @NotNull
   public static final Singleton getInstance() {
      return Companion.getInstance();
   }
   public static final class Companion {
      /** @deprecated */
      // $FF: synthetic method
      @JvmStatic
      public static void instance$annotations() {
      }
      @NotNull
      public final Singleton getInstance() {
         Lazy var1 = Singleton.instance$delegate;
         Singleton.Companion var2 = Singleton.Companion;
         Object var3 = null;
         boolean var4 = false;
         return (Singleton)var1.getValue();
      }
      private Companion() {
      }
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

3 双重检查锁

```kotlin
//3.1  @Volatile 注解
class Singleton private constructor() {
    companion object {
        @Volatile
        private var instance: Singleton? = null
        @JvmStatic
        fun getInstance(): Singleton {
            if (instance == null) {
                synchronized(this) {
                    if (instance == null) {
                        instance = Singleton()
                    }
                }
            }
            return instance as Singleton
        }
    }
    fun doSome1() {
        println("test")
    }
}
```

```java
//3.1 反编译后的 Java 代码
public final class Singleton {
   private static volatile Singleton instance;
   public static final Singleton.Companion Companion = new Singleton.Companion((DefaultConstructorMarker)null);
   public final void doSome1() {
      String var1 = "test";
      boolean var2 = false;
      System.out.println(var1);
   }
   private Singleton() {
   }
   // $FF: synthetic method
   public Singleton(DefaultConstructorMarker $constructor_marker) {
      this();
   }
   @JvmStatic
   @NotNull
   public static final Singleton getInstance() {
      return Companion.getInstance();
   }
   public static final class Companion {
      @JvmStatic
      @NotNull
      public final Singleton getInstance() {
         if (Singleton.instance == null) {
            boolean var1 = false;
            boolean var2 = false;
            synchronized(this) {
               int var3 = false;
               if (Singleton.instance == null) {
                  Singleton.instance = new Singleton((DefaultConstructorMarker)null);
               }
               Unit var5 = Unit.INSTANCE;
            }
         }
         return Singleton.instance;
      }
      private Companion() {
      }
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

```kotlin 
//3.2 kotlin lazy 特性写法
class Singleton private constructor() {
    companion object {
        @JvmStatic
        val instance by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
            Singleton()
        }
    }
}
```

```java
//3.2 反编译后的 Java 代码
public final class Singleton {
   @NotNull
   private static final Lazy instance$delegate;
   public static final Singleton.Companion Companion = new Singleton.Companion((DefaultConstructorMarker)null);
   private Singleton() {
   }
   static {
      instance$delegate = LazyKt.lazy(LazyThreadSafetyMode.SYNCHRONIZED, (Function0)null.INSTANCE);
   }
   // $FF: synthetic method
   public Singleton(DefaultConstructorMarker $constructor_marker) {
      this();
   }
   @NotNull
   public static final Singleton getInstance() {
      return Companion.getInstance();
   }
   public static final class Companion {
      /** @deprecated */
      // $FF: synthetic method
      @JvmStatic
      public static void instance$annotations() {
      }
      @NotNull
      public final Singleton getInstance() {
         Lazy var1 = Singleton.instance$delegate;
         Singleton.Companion var2 = Singleton.Companion;
         Object var3 = null;
         boolean var4 = false;
         return (Singleton)var1.getValue();
      }
      private Companion() {
      }
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

4 object 对象声明 和 companion object 伴生对象

```kotlin
class Singleton private constructor() {
    //伴生对象
    companion object {
        @JvmStatic //方便 java 调用
        fun getInstance(): Singleton {
            return SingletonInstanceHolder.instance
        }
    }
    //对象声明
    private object SingletonInstanceHolder {
        val instance = Singleton()
    }
    fun doSome1() {
 		println("test")
    }
}
```

```java
//4 反编译后的 Java 代码
public final class Singleton {
   public static final Singleton.Companion Companion = new Singleton.Companion((DefaultConstructorMarker)null);
   public final void doSome1() {
      String var1 = "test";
      boolean var2 = false;
      System.out.println(var1);
   }
   private Singleton() {
   }
   // $FF: synthetic method
   public Singleton(DefaultConstructorMarker $constructor_marker) {
      this();
   }
   @JvmStatic
   @NotNull
   public static final Singleton getInstance() {
      return Companion.getInstance();
   }
   private static final class SingletonInstanceHolder {
      @NotNull
      private static final Singleton instance;
      public static final Singleton.SingletonInstanceHolder INSTANCE;
      @NotNull
      public final Singleton getInstance() {
         return instance;
      }
      static {
         Singleton.SingletonInstanceHolder var0 = new Singleton.SingletonInstanceHolder();
         INSTANCE = var0;
         instance = new Singleton((DefaultConstructorMarker)null);
      }
   }
   public static final class Companion {
      @JvmStatic
      @NotNull
      public final Singleton getInstance() {
         return Singleton.SingletonInstanceHolder.INSTANCE.getInstance();
      }
      private Companion() {
      }
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

