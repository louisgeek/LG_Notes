# Android 内存泄漏
- 可以简单理解为当一个生命周期较长的对象引用了生命周期较短的对象，而当该生命周期短的对象在生命周期结束后本该被回收的却未能被回收，从而就导致了内存泄露
- 比如：单例类引用了 Activity 的情况，当该 Activity 在生命周期结束后本该被回收的却未能被回收，就导致了内存泄露


## 导致 OOM 内存溢出
- Out Of Memory 指程序在申请内存时，当下没有足够的内存空间供其使用，会导致程序崩溃或异常终止的现象
- 如果程序未能及时释放那些已经不再使用的内存，那么就会导致内存泄露，随着泄露内存的增长，最终一定会导致 OOM 的发生





在 JVM 中，对于对象的回收 GC 是基于**可达性分析**。简单来说，就是从 GC Root 出发，被引用的对象均被标记为存活，而没有被引用的对象，则被标记为垃圾，即可以被 GC 回收。







### 单例类引用 Activity
- 单例类是生命周期比 Activity 长



### 子线程耗时操作运行未结束
- Activity - 子线程



### Handler 涉及的内存泄漏

#### 发送延迟消息没有及时发完

- Activity - 匿名 Handler - Message - MessageQueue - Looper 

- 推断：主线程 Looper 一直在 loop ，Looper 持有 MessageQueue，MessageQueue 持有 Message，Message 持有 Handler，匿名 Handler 持有 Activity 引用





静态（static）变量中持有生命周期短的对象；

匿名内部类中执行耗时任务（Handler）；

非静态内部类（MyThread）；

异步处理耗时任务，在finish时未终止(new Thread)；

资源对象未关闭或者释放（Cursor、Bitmap）；



 

## 分析工具





### Heap Dump

- Android Studio 自带的 Heap Dump 堆转储工具

Profiler - Sessions 里选择指定进程 - Memory 

有两个按钮，force garbage collection GC 按钮和 Dump Java heap 捕获堆转储文件按钮

前者能触发一次手动 GC ，一般我们在我们捕获堆转储文件之前，推荐点一下 GC，能把一些弱引用给回收，防止给我们的分析带来干扰，后者可以生成 hprof 文件，这个文件会展示 Java 堆的使用情况，点击这个按钮后，Android Studio 会帮我们生成这个堆转储文件并且可以进行分析



结果页面会显示存在的 Leaks，点击去然后选择对应的 Class - 选择对应的 Instance - 可以分别查看 Fields 和 References，这样就可以查看到内存泄漏类的具体引用路径了



### LeakCanary

- Square 提供的内存泄漏检测框架库