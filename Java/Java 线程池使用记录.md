
1 预览送流背景
- 设备给 sdk 不断送流，再让 sdk 给 app
- sdk 给 app 的 onStream 每上来一次都是一个子线程
- sdk 给 app 的 onStream 是同步等待的，会等上一个 onStream 里的逻辑执行完才继续上来

```java
 interface StreamCallback {
        void onStream(byte[] bytes);
 }
```

- 所以此时如果 dealStream 过于耗时，就不能保证画面流畅了，会卡卡的，但是注意上来的 onStream 传出的数据是最新的
```java
 StreamCallback streamCallback = new StreamCallback() {
            @Override
            public void onStream(byte[] bytes) {
                //处理逻辑
                dealStream(bytes);
            }
        };
streamSdk.setStreamCallback(streamCallback);
```


2 改造1
- 引入单线程池，接管原来 onStream 的子线程的逻辑任务
- 所以这样的话 onStream 没啥耗时的，逻辑很快就执行完了，那么 sdk 给 app 的 onStream 送上来就很快了
- 但是注意 bytes 的使用，直接传入 dealStream 使用的话，这样的表现就是画面延迟很大很大，因为当下执行的 dealStream 可能用的是比较旧的数据
```java
//
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
//
private void testPrivate(String test) {
        StreamCallback streamCallback = new StreamCallback() {
            @Override
            public void onStream(byte[] bytes) {
            //Runnable 局部匿名内部类
            singleThreadExecutor.execute(new Runnable() {
                    @Override
                    public void run() {
                        //处理逻辑，内部类并不是直接使用传递进来的参数，而是将传递进来的参数通过自己的构造器备份到自己内部（copy local variable 机制），表面看是同一个变量 bytes，实际调用的是自己的属性而不是外部类方法的参数，java7 这么写会报错：“Variable 'bytes' is accessed from within inner class, needs to be declared final”，需要改成 “ public void onStream(final byte[] bytes) ”，而 java8 不会报错是因为 jvm 自动帮你在底层添加了 final 修饰符，相当于 java8新增的语法糖
                        dealStream(bytes);
                    }
                  });
             }
        };
}

```

2 改造2
- 引入全局变量 lastBytes 做记录
- 这样基本能满足使用
```java
//
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
byte[] lastBytes = new byte[0];
//
private void testPrivate(String test) {
        StreamCallback streamCallback = new StreamCallback() {
            @Override
            public void onStream(byte[] bytes) {
                //
                lastBytes = bytes;
                //
                singleThreadExecutor.execute(new Runnable() {
                    @Override
                    public void run() {
                        //处理逻辑
                        dealStream(lastBytes);
                    }
                });
             }
        };
}

```

3 改造2
- 引入定时器，处理逻辑不直接依赖回调
- 这样也能满足使用
```java
//
ScheduledExecutorService singleThreadScheduledExecutor = Executors.newSingleThreadScheduledExecutor();
byte[] lastBytes = new byte[0];
//
private void testPrivate(String test) {
    singleThreadScheduledExecutor.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                //
                if (lastBytes.length > 0){
                    //处理逻辑
                    dealStream(lastBytes);
                }
            }
        }, 0, 20, TimeUnit.MILLISECONDS);

    //
    StreamCallback streamCallback = new StreamCallback() {
         @Override
         public void onStream(byte[] bytes) {
                //
                lastBytes = bytes;
               
        }
    };
}

```