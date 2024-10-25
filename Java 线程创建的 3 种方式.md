线程创建的 3 种方式



1 继承 Thread 类，重写 run 方法

无法实现资源共享，是一种特殊的 Runnable （ Thread 也是实现 Runnable 的）

```java
	 MyThread myThread = new MyThread();
     myThread.start();
```



2  实现 Runnable 接口，实现 run 方法

可实现资源共享

```java
 MyRunnable myRunnable = new MyRunnable();
 Thread thread = new Thread(myRunnable);
 thread.start();
```
2.1  实现 Runnable 接口比继承 Thread 类的优势 
① 适合多个相同的程序代码的线程去处理统一资源的情况 
② 可以避免由于 java 单继承特征带来的局限性 
③ 增强开发程序的健壮性，代码可以多个线程共享，代码和数据是独立的

3  实现 Callable 接口

可选配合 ExecutoreService 提供了submit() 方法，传递一个Callable，有返回值Future，调用Future对象的get()方法，会阻塞直到 call 执行完成

```java
    Callable<String> callable = new Callable<String>() {
            @Override
            public String call() throws Exception {
                //后台执行耗时
                Thread.sleep(5*1000);
                return "some text";
            }
        };
        FutureTask<String> futureTask = new FutureTask<String>(callable);

         // new Thread(futureTask,"有返回值").start();
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        executorService.submit(futureTask);
        System.out.println( "==== start");
        try {
            //调用 get()方法后，当前线程开始阻塞，直到 call 方法结束返回结果
            System.out.println( "取到返回值 ===="+ futureTask.get());
            System.out.println( "==== other 1");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println( "==== other 2");
```
3.1 Callable 和 Runnable 的区别
Callable 类似于 Runnable 接口，和 FutureTask 配合使用可以得到别的线程任务方法的返回值。Callable 和 Runnable 不同：
①  Callable 规定的方法是 call()，而 Runnable 规定的方法是 run()

② Callable 的任务执行后可返回值，而 Runnable 的任务没有有返回值

③ call() 方法可抛出异常，而 run() 方法是不能抛出异常