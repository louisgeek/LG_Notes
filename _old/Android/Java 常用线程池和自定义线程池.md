# 1 通过 Executors 工厂类来创建线程池

| 创建线程池的方法 |                             |
| ------------------------------ | --------------------------- |
| newFixedThreadPool        | 有固定线程数的线程池，如果没有任务执行，那么线程会一直等待 |
| newCachedThreadPool | 线程池里有很多线程需要同时执行，老的可用线程将被新的任务触发重新执行，如果线程超过60秒内没执行，那么将被终止并从池中删除 |
| newWorkStealingPool | 创建一个拥有多个任务队列（以便减少连接数）的线程池    |
| newScheduledThreadPool   | 该线程池可用于定时或周期性任务的执行 |
| newSingleThreadExecutor  | 只有一个线程的线程池，所有提交的任务是顺序执行 |
| newSingleThreadScheduledExecutor       | 只有一个线程，该线程池可用于定时或周期性任务的执行 |

1 各线程池适应场景总结
```
1.1  newFixedThreadPool：创建一个固定大小的线程池，因为采用无界的阻塞队列，所以实际线程数量永远不会变化，适用于可以预测线程数量的业务中，或者服务器负载较重，对当前线程数量进行限制
1.2 newCachedThreadPool：用来创建一个可以无限扩大的回收型线程池，适用于服务器负载较轻，执行很多短期异步任务
1.3 newWorkStealingPool：创建一个拥有多个任务队列的线程池，可以减少连接数，创建当前可用cpu数量的线程来并行执行，适用于大耗时的操作，可以并行来执行
1.4 newScheduledThreadPool：可以延时启动，定时启动的线程池，适用于需要多个后台线程执行周期任务的场景
1.5 newSingleThreadExecutor：创建一个单线程的，适用于需要保证顺序执行各个任务，并且在任意时间点，不会有多个线程是活动的场景
1.6 newSingleThreadScheduledExecutor：创建一个单线程的，适用于需要保证顺序执行各个任务，并且在任意时间点，不会有多个线程是活动的场景，同时可以延时启动，定时启动的线程池
```
2 采用线程池的优势
```
2.1 重用存在的线程，减少对象创建、消亡的开销，性能佳
2.2 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞
2.3 提供定时执行、定期执行、单线程、并发数控制等功能
```
3 方法列表：

```java
public static ExecutorService newFixedThreadPool(int nThreads)  
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)

public static ExecutorService newCachedThreadPool()
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory)

public static ExecutorService newWorkStealingPool()
public static ExecutorService newWorkStealingPool(int parallelism)

public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory)
            
public static ExecutorService newSingleThreadExecutor()
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory)

public static ScheduledExecutorService newSingleThreadScheduledExecutor()
public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory)
```

## 1.1 newFixedThreadPool

- 允 许 的 请 求 队 列 长 度 为 Integer.MAX_VALUE，可能会堆积大量的请求



参数 nThreads 指定线程池的最大线程数，核心线程数即为最大线程数，线程不会被回收，直到调用 shutdown / shutdownNow 方法回收，参数 threadFactory 是线程工厂类

```java
//实现方法
 public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    
 public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
```
## 1.2 newCachedThreadPool

- 允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程



如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。核心线程池大小为0，最大为Integer.MAX_VALUE，默认线程空闲存活时间是60秒


```java
//实现方法
 public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
    
      public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
    
```
## 1.3 newWorkStealingPool （ java 8 ）



创建一个拥有多个任务队列（以便减少连接数）的线程池，parallelism 参数 可选设置并行级别，默认有 Runtime.getRuntime().availableProcessors() 个线程同时执行 

```java
//实现方法
 public static ExecutorService newWorkStealingPool() {
        return new ForkJoinPool
            (Runtime.getRuntime().availableProcessors(),
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
     public static ExecutorService newWorkStealingPool(int parallelism) {
        return new ForkJoinPool
            (parallelism,
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```

## 1.4 newScheduledThreadPool

- 允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程

  ​

  该线程池可用于定时或周期性任务的执行

```java
//实现方法
 public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    
    
 public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory) {
        return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
    }
```




## 1.5 newSingleThreadExecutor

 - 允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求

  ​

  该线程池有且仅有一个线程执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行

```java
//实现方法
 public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
    
       public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```
## 1.6 newSingleThreadScheduledExecutor



只有一个线程，同时可以用来调度执行将来的任务

```java
//实现方法
 public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
        return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1));
    }
    
      public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory) {
        return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1, threadFactory));
    }
```





#  2 自定义 ThreadPoolExecutor 实现线程池

尽可能规避资源耗尽的风险

```java
public  class ExecutorServiceHelper { 
/**
     *  获取活跃的 cpu数量
     */
    private static int NUMBER_OF_CORES = Runtime.getRuntime().availableProcessors();
    private final static BlockingQueue<Runnable> mWorkQueue;
    private final static long KEEP_ALIVE_TIME = 3L;
    private final static TimeUnit KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS;
    private static ThreadFactory mThreadFactory;

    static {
        mWorkQueue = new LinkedBlockingQueue<Runnable>();
 //默认的工厂方法将新创建的线程命名为：pool-[虚拟机中线程池编号]-thread-[线程编号]
        //mThreadFactory= Executors.defaultThreadFactory();
        mThreadFactory = new NamedThreadFactory();
//        System.out.println("NUMBER_OF_CORES:"+NUMBER_OF_CORES);
    }

    public  static  void  execute(Runnable runnable){
        if (runnable==null){
            return;
        }
        /**
         * 1.当线程池小于 corePoolSize 时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程。
         * 2.当线程池达到 corePoolSize 时，新提交任务将被放入 workQueue 中，等待线程池中任务调度执行
         * 3.当 workQueue 已满，且 maximumPoolSize > corePoolSize时，新提交任务会创建新线程执行任务
         * 4.当提交任务数超过 maximumPoolSize 时，新提交任务由 RejectedExecutionHandler 处理
         * 5.当线程池中超过 corePoolSize 线程，空闲时间达到 keepAliveTime 时，关闭空闲线程
         * 6.当设置 allowCoreThreadTimeOut(true) 时，线程池中 corePoolSize 线程空闲时间达到 keepAliveTime 也将关闭
         **/
         /**
   maximumPoolSize 推荐取值
    如果是 CPU 密集型任务，就需要尽量压榨CPU，参考值可以设为 NUMBER_OF_CORES + 1 或 NUMBER_OF_CORES + 2
    如果是 IO 密集型任务，参考值可以设置为 NUMBER_OF_CORES * 2
         */
        ExecutorService executorService = new ThreadPoolExecutor(NUMBER_OF_CORES,
                NUMBER_OF_CORES * 2, KEEP_ALIVE_TIME, KEEP_ALIVE_TIME_UNIT,
                mWorkQueue,mThreadFactory);
        executorService.execute(runnable);
    }


    private static class NamedThreadFactory implements ThreadFactory {
        private final AtomicInteger threadNumberAtomicInteger = new AtomicInteger(1);
        @Override
        public Thread newThread(Runnable r) {
            Thread thread=  new Thread(r,String.format(Locale.CHINA,"%s%d","NamedThreadFactory",threadNumberAtomicInteger.getAndIncrement()));
            /* thread.setDaemon(true);//是否是守护线程
            thread.setPriority(Thread.NORM_PRIORITY);//设置优先级 1~10 有3个常量 默认 Thread.MIN_PRIORITY*/
            return thread;
        }
    }
}
```

```java
	ExecutorServiceHelper.execute(new Runnable() {
			@Override
			public void run() {
				//do something
			}
		});
```

## 3 execute  和  submit

```java
execute()：Executor中声明的方法，向线程池提交一个任务，交由线程池去执行，没有返回值

submit() ：ExecutorService中声明的方法。向线程池提交一个任务，交由线程池去执行，可以接受 （ Callable.call() ）回调函数的返回值，适用于需要处理返回着或者异常的业务场景，实际上还是调用的execute()方法，只不过它利用了 Future 来获取任务执行结果

```



## 4 schedule

```java
schedule:  延时执行任务

scheduleAtFixedRate : 以固定频率来执行一个任务，按照上一次任务的发起时间计算下一次任务的开始时间

scheduleWithFixedDelay : 以上一次任务的结束时间计算下一次任务的开始时间，在你不能预测调度任务的执行时长时是很有用

```



## 5 线程池的关闭
```
shutdown() ：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
shutdownNow() ：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务
```
## 6  任务队列 BlockingQueue 

- BlockingQueue<Runnable> workQueue 取值

  ```java
  ArrayBlockingQueue ：有界的数组队列，先进先出，此队列创建时必须指定大小

  SynchronousQueue :  同步的阻塞队列，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务

  LinkedBlockingQueue ：基于链表的先进先出队列，可支持有界/无界的队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE

  PriorityBlockingQueue ：优先队列，可以针对任务排序

  ```

  ​

## 7 RejectedExecutionHandler  任务拒绝策略

```java
ThreadPoolExecutor.AbortPolicy : 直接丢弃后来的任务并抛出RejectedExecutionException异常。

ThreadPoolExecutor.DiscardPolicy ：直接丢弃后来的任务，但是不抛出异常。

ThreadPoolExecutor.DiscardOldestPolicy ：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）

ThreadPoolExecutor.CallerRunsPolicy ：直接调用 run 方法并且阻塞执

```


