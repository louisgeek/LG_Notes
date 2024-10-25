# Java 线程同步之 synchronized

## synchronized 关键字
- 是一种重量级锁
- 通过监视器实现线程同步操作
- 分为对象锁和类锁


## 线程安全
- 单线程不存在线程安全问题，而多个线程可能会存在同时访问同一个资源（变量、对象、文件、数据库，这种资源又叫临界资源、共享资源）的情况，会导致程序运行结果不是预期结果


## 问题示例

```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
    public void methodTest() {
        for (int i = 0; i < 10000; i++) {
            count++;
        }
        System.out.println(this + " " + Thread.currentThread() + " count " + +count);
    }
}
```

```java
  public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        new Thread(myRunnable).start();
        new Thread(myRunnable).start();
}

//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@54998dc8 Thread[Thread-1,5,main] count 15621
//com.louisgeek.MyRunnable@54998dc8 Thread[Thread-0,5,main] count 16073

```
- 两个线程用的是同一个 MyRunnable 实例对象
- count 结果不是预期值



## 解决方案
- 同步互斥访问
- 在同一时刻，确保最多只能有一个线程访问该临界资源，当一个线程在访问临界资源的代码时上加一个锁，当用完临界资源后释放锁，从而让其他线程能够继续访问该资源，java 可以通过 synchronized 或 Lock 来实现同步互斥访问



### 对象锁
1. 加入 synchronized(this) 代码块锁（对象锁）

```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
    public void methodTest() {
        //此时的 this 就是 MyRunnable.this
        synchronized(this) {
            for (int i = 0; i < 10000; i++) {
                count++;
            }
            System.out.println(this + " " + Thread.currentThread() + " count " + +count);
        }
    }
}

//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@44f12c70 Thread[Thread-0,5,main] count 10000
//com.louisgeek.MyRunnable@44f12c70 Thread[Thread-1,5,main] count 20000

```



2. 也可以加入 synchronized 方法锁（对象锁）

```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
  public synchronized void methodTest() {
        for (int i = 0; i < 10000; i++) {
            count++;
        }
        System.out.println(this + " " + Thread.currentThread() + " count " + +count);
    }
}
//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@406fd94d Thread[Thread-0,5,main] count 10000
//com.louisgeek.MyRunnable@406fd94d Thread[Thread-1,5,main] count 20000

```
- 方法锁默认的锁对象是 this，也就是 MyRunnable.this
- 同一时刻只有一个线程访问这个方法



### 类锁

- 修改使用多个 MyRunnable 实例对象
```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
    public void methodTest() {
        //此时的 this 就是 MyRunnable.this
        synchronized(this) {
            for (int i = 0; i < 10000; i++) {
                count++;
            }
            System.out.println(this + " " + Thread.currentThread() + " count " + +count);
        }
    }
}
```

```java
public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        MyRunnable myRunnable2 = new MyRunnable();
        new Thread(myRunnable).start();
        new Thread(myRunnable2).start();
}
//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@141cded9 Thread[Thread-1,5,main] count 12370
//com.louisgeek.MyRunnable@519868d6 Thread[Thread-0,5,main] count 14802

```
- count 结果又不是预期值了




1. 修改加入 synchronized(MyRunnable.class) 代码块锁（类锁）

```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }
    public void methodTest() {
        synchronized (MyRunnable.class) {
            for (int i = 0; i < 10000; i++) {
                count++;
            }
            System.out.println(this + " " + Thread.currentThread() + " count " + +count);
        }
    }
}

//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@2034c894 Thread[Thread-0,5,main] count 10000
//com.louisgeek.MyRunnable@2cf062b0 Thread[Thread-1,5,main] count 20000
```


2. 也可以加入 synchronized 静态方法锁（类锁）

```java
public class MyRunnable implements Runnable {
    private static int count;
    @Override
    public void run() {
        methodTest();
    }

    public static synchronized void methodTest() {
    for (int i = 0; i < 10000; i++) {
                count++;
     }
    System.out.println(this + " " + Thread.currentThread() + " count " + +count);
 }
}

//------------ 打印结果 ------------
//com.louisgeek.MyRunnable@5024c814 Thread[Thread-1,5,main] count 10000
//com.louisgeek.MyRunnable@30f16220 Thread[Thread-0,5,main] count 20000
```










