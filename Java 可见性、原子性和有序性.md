# Java 可见性、原子性和有序性
- Java 多线程的三大特性
- Java 并发的三大特性
- Java 并发编程的三大核心问题

## 可见性
- 指在多个线程访问同一个变量的情况下，当一个线程修改了该共享变量的值后，其他线程能够立刻读到修改后的值
- 普通的共享变量不能保证可见性，因为该共享变量被修改后值是先写入到本地内存（线程的工作内存）里的，后续写入主内存的时间是不确定的，当其他线程去读取时，此时取到的很有可能还是原来的旧值，所以无法保证可见性
- 可以通过 volatile 关键字保证变量的可见性
- 可以通过 synchronized 或 Lock 保证可见性，因为 synchronized 或 Lock 可以保证同一时刻只有一个线程能访问共享资源，并在其释放锁之前将变量修改后的值刷新到主内存中
- 可以通过 Atomic 类型保证可见性

## 原子性
- 指一个或多个操作，要么全部执行，并且在执行过程中是不能被中断的，要么全部不执行（完全不发生）
- java 中对基本数据类型的变量的读取和赋值操作本身就是原子性操作，但是如果不用 volatile 关键字修饰的话，其他线程可能不能立马看到改后的新值，而对于复合操作（例如 i++，包含了三个步骤：读取 i 值、对 i 加 1、然后将新值写回 i，这三个步骤之间可能被其他线程打断，导致结果数据不一致），即便加了 volatile 关键字后，整体操作也做不到原子性
- 可以通过 synchronized 或 Lock 保证原子性，因为 synchronized 或 Lock 能够保证同一时刻只有一个线程访问这个代码块或方法
- 可以通过 Atomic 类型保证原子性

## 有序性
- 为了让机器指令能更符合 CPU 的执行特性，最大化发挥性能，在程序编译成机器码指令时可能适当的会出现指令重排的现象（指令重排序），然而这种重排序就有可能导致多线程中出现数据不一致问题
- 可以通过 volatile 关键字来保证变量的有序性（它能禁止指令重排序，提供了变量的读写操作的有序性保证）
- 可以通过 synchronized 和 Lock 来保证有序性，很显然 synchronized 或 Lock 保证同一时刻只有一个线程访问这个代码块或方法，相当于是让线程顺序执行同步代码，自然就保证了有序性
- Atomic 类型无法保证有序性

## 示例：i++ 保证安全
- synchronized 方案

```java
private int i = 0;

public synchronized void increment() {
    i++;
}
```

- Atomic 类型
```java
private AtomicInteger i = new AtomicInteger(0);

public void increment() {
    i.incrementAndGet();
}
```