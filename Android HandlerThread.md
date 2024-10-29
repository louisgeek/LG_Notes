# Android HandlerThread
- HandlerThread 继承 Thread，组合了一个 Handler 和 Looper



流程

- 实例化 

  ```java
  HandlerThread handlerThread = new HandlerThread("name");
  ```

- 启动子线程

- ```java
  handlerThread.start() 
  ```

- 创建 Handler，传入 handlerThread 的 Looper 进行关联

  ```java
  //无参方法已过时
  Handler workHandler = new Handler(handlerThread.getLooper());
  //可选覆写 handleMessage 方法
   Handler workHandler = new Handler(handlerThread.getLooper()){
              @Override
              public void handleMessage(@NonNull Message msg) {
                  super.handleMessage(msg);
              }
  };
  //带 callback
  Handler workHandler2 = new Handler(handlerThread.getLooper(), new Handler.Callback() {
              @Override
              public boolean handleMessage(@NonNull Message msg) {
                  //如果返回 true ，那么就不会调用 Handler 内部的 handleMessage 方法处理
                  return false;
       }
  });
  ```

  

  

- 发消息

  ```java
  workHandler.sendMessage(msg);
  ```

  

- 退出

  ```java
  HandlerThread.quit();
  //
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
              handlerThread.quitSafely();
  }
  ```

  

