# Android ThreadLocal
- ThreadLocal 是一种用于实现线程局部变量的工具类，为每个线程提供独立的变量副本，可以在多线程环境下实现线程隔离，防止因多线程操作共享资源而引发数据竞争冲突和线程安全问题，提高程序的并发性能（通过空间换时间的方式，避免同步锁的性能损耗，适用于需要线程隔离的场景，比如用户信息、会话管理、事务 ID 和日志跟踪 ID 传递等）
- 线程局部变量：ThreadLocal 为每个使用某个变量的线程都提供了一个该变量的独立副本，所以在不同部分的代码可以很方便地共享数据，从而简化线程范围内数据的传递和同步逻辑（特别是在一些复杂的方法调用链中避免逐层传递参数），提高代码的可读性和可维护性
- 线程隔离机制：ThreadLocal 为多线程环境下的变量管理提供了一种隔离机制，它提供的变量是线程私有的，会为每个线程都提供独立的变量副本，使得每个线程可以独立地修改自己的变量副本而不受其他线程影响和干扰，也无法看到其他线程对该变量值的修改

```java
//每个 Thread 线程内部都持有一个 ThreadLocalMap 对象，用于存储线程本地局部变量
public class Thread implements Runnable {
    //...
    //ThreadLocal$ThreadLocalMap 是一个键值对结构，类似于 Map
    ThreadLocal.ThreadLocalMap threadLocals = null;
    //...
}  
```

```java
//ThreadLocal$ThreadLocalMap 是 ThreadLocal 的一个静态内部类，以 ThreadLocal 实例对象作为 key 键，以实际存储的值（变量副本）作为 value 值
public class ThreadLocal<T> {
    //...
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            //ThreadLocal$ThreadLocalMap$Entry 继承自 WeakReference
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
    //...
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }
    public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null) {
             m.remove(this);
         }
    }
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
    //...
}   
```

## 总结
- ThreadLocal 提供了线程局部变量本地存储的功能，可以很方便地实现线程内数据共享，可以把 ThreadLocal（其实是 ThreadLocal$ThreadLocalMap）可以理解为是一个以线程（其实是弱引用的 ThreadLocal）为 key 键，任意对象为 value 值的存储结构
- 应用场景：适用于需要在线程作用域内部进行共享数据，但不适用于跨线程全局共享的情况，常见的应用场景包括数据库连接、事务管理和处理用户会话数据等，另外 Android Handler 消息机制里就涉及了用 ThreadLocal 来存取 Looper 对象
- 生命周期：ThreadLocal 变量的生命周期等同于线程的生命周期，当线程结束时 ThreadLocal 中的所有数据就可以被回收
- 内存泄漏：由于 ThreadLocalMap 的 Entry 的 key 键是弱引用的 ThreadLocal 对象，而 value 值是强引用，如果 ThreadLocal 对象被垃圾回收，但值仍然存在（未被清理，也无法被访问了），且当前线程还在运行，就可能会导致内存泄漏，应该在使用完 ThreadLocal 后及时主动调用 ThreadLocal#remove 方法清理数据（移除线程中的局部变量副本），避免发生内存泄漏，尤其是在使用线程池的情况下需要特别注意
