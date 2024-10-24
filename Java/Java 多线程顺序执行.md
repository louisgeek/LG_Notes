# 多线程顺序执行

## 1 最基础的就是按顺序手动串行调用
## 2 利用 Executors.newSingleThreadExecutor() 单线程池来保证
## 3 Thread#join - 使调用者所处的线程转换为 State.WATING 状态，直到 join 者执行完，底层使用 Object#wait 方法来实现，wait 方法要求处在同步方法或同步代码块中，而 join 里有 synchronized(lock) 逻辑
## 4 CountDownLatch   - JUC并发工具集
- 1 定义 1 个信号，其他地方 await 等待，直到 countDown 执行，计数器到 0 后通知所有 await 继续执行  ----跑步指令枪打出去后所有人才开始跑
- 2 定义 N 个信号，总的一个地方 await 等待，各个线程 countDown 执行，计数器到 0 后通知 await 继续执行 ----汇总若干个线程的结果后再继续执行一个线程  ----跑步所有人跑完了整体才能收场，不过有 await 带超时参数方法，给个超时时间，不无限制等待；另外线程要注意异常捕获，出异常了也需要减计数，建议在 finally 里执行 countDown