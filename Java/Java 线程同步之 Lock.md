# Java 线程同步之 Lock

## Lock 简介
- 一般使用 Lock 接口的实现类 ReentrantLock，Lock 可以比 synchronized 更加灵活
- Lock 是一个接口 - JUC并发工具集
- 使用 Lock 的时候必须手动释放，即使发生异常了也不会自动释放锁，所以一般都是逻辑写在 try catch 中，将 unlock 解锁操作放在 finally 中，防止死锁
- Lock 是可中断锁，是可以有能力让等待锁的线程响应中断的
- Lock 是可以知道有没有成功获取锁
- Lock 实现默认情况下是非公平锁，但可以设置成公平锁
- 可以提高多个线程进行读操作的效率


## ReentrantLock
- 可重入锁（递归锁）
- 排他锁


## ReadWriteLock 
- 一般使用 ReadWriteLock 接口的实现类 ReentrantReadWriteLock 
- 特别适合读多写少的场景，大大提升了效率
- ReadWriteLock 是一个接口 - JUC并发工具集
- 提供了 readLock 和 writeLock 方法相当于提供两把锁
- ReentrantReadWriteLock.ReadLock 和 ReentrantReadWriteLock.WriteLock 分别是读锁和写锁，其中读锁是共享锁；写锁是排他锁
- 同一时刻允许多个线程进行读操作，此时所有线程的写操作会被阻塞，同一时刻只允许一个线程进行写操作，此时其他线程的读操作会被阻塞
