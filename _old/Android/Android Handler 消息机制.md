## Handler 

- Android 是基于消息驱动机制运行的

Android 中定义的一套消息处理机制，为了能够实现线程间的通讯，封装了一套消息创建、传递、处理的机制，**非堵塞的消息传递机制**，可以将子线程的信息发送到主线程使用，比如更新 UI

Handler 处理器

Message 消息

MessageQueue 消息队列，单向链表方式存储消息

Looper 轮询器



## 基本用法

### 创建 Handler

```java
//Handler 无参构造函数，内部默认 mLooper = Looper.myLooper(); 这样不够明确，如果子线程调用，而 Looper 此时如果没有初始化，那么就抛异常了
//@Deprecated
Handler handler = new Handler();
//显示传入 Looper 
Handler handler = new Handler(Looper.myLooper());
//传入 Looper 和 Handler#Callback
Handler handler = new Handler(Looper.myLooper(), Handler#Callback);
//主线程
Handler uiHandler = new Handler(Looper.getMainLooper());
```
### 创建 Message

```java
Message msg = new Message();
//传入 handler 后和下面写法一致，有缓存机制，避免重复创建实例对象
Message msg = Message.obtain(handler);
//内部直接调用 Message.obtain(this);
Message msg = handler.obtainMessage();
```



### 收消息

#### 1 重写 Handler#handleMessage

##### 1.1 静态类

```java
static class MyHandler extends Handler{
   	  //构造方法
      public MyHandler() {
          //过时
            super();
        }
		//构造方法
        public MyHandler(@NonNull Looper looper) {
            super(looper);
        }
	//重写 handleMessage
    @Override
    public void handleMessage(@NonNull Message msg) {
        //内部空方法
        super.handleMessage(msg);
        //
    }
}
//实例化
private  MyHandler mMyHandler = new MyHandler(Looper.myLooper());
```

##### 1.2 匿名类

```java
Handler handler = new Handler(Looper.myLooper()) {
    @Override
    public void handleMessage(@NonNull Message msg) {
        //内部空方法
        super.handleMessage(msg);
        //
    }
};
```

#### 2 实现 Handler#Callback 接口 Callback#handleMessage 方法

- Handler 传入 Callback 接口

```java
Handler handler= new Handler(Looper.myLooper(),new Handler.Callback() {
      @Override
      public boolean handleMessage(@NonNull Message msg) {
         return false;
      }
});
```

#### 3 Handler#post 传入 Runnable

- 发送的同时通过 Runnable 处理收到的消息，具体说明见下文【发消息 2】
- 意味着该方法实现收发一体



### 发消息

#### 1 Handler#sendMessage 方式

```java
handler.sendMessage(msg);
handler.sendMessageDelayed(obtain,time);
```

#### 2 Handler#post 方式

- 不需要准备 Message 实例

```java
//handler.post
public final boolean post(@NonNull Runnable r) {
    //sendMessageDelayed 和上文说的 sendMessage 里的原理一致
    return  sendMessageDelayed(getPostMessage(r), 0);
}
//handler.postDelayed
public final boolean postDelayed(@NonNull Runnable r, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}
//Runnable 封装成 Message 的 callback
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    //Message#callback 被赋值，对应上文【收消息 3】
    m.callback = r;
    return m;
}
```

#### 3 Activity#runOnUiThread

- 内部也是调用 Handler#post

```java
 public final void runOnUiThread(Runnable action) {
        if (Thread.currentThread() != mUiThread) {
            mHandler.post(action);
        } else {
            action.run();
        }
}
```



## 消息分发

```java
 /**
     * Handle system messages here.
     */
    public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) {
       //内部直接调用 message.callback.run(); 所以就是处理上文 Handler#post 之类发送的消息
       //对应上文【3 Handler#post 传入 Runnable】
          handleCallback(msg);
        } else {
            //Handler#Callback 接口
            if (mCallback != null) {
                //对应上文【2 实现 Handler#Callback 接口 Callback#handleMessage 方法】
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            //对应上文【1 重写 Handler#handleMessage】
            handleMessage(msg);
        }
    }
```

- 先处理 Message#callback 的 Runnable ，如果有就直接处理 Runnable 后结束，如果没有就判断有没有 Handler#Callback 接口，如有则判断其返回值，返回是 true 就不继续执行了，返回 false 那么就继续执行 Handler#handleMessage 方法



## 基本原理


```java
//ActivityThread.java 的 main() 方法调用了 Looper.prepareMainLooper()
public static void main(String[] args){

//UI 线程默认已经调用以下方法创建了 Looper 对象，是系统自动调用的
Looper.prepareMainLooper();
……
//调用 loop() 进入死循环，轮询获取消息
Looper.loop();
//正常不会走到这
throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

### 先看 Looper 的 prepareMainLooper 方法

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
private static Looper sMainLooper;
//系统自动调用，不需要主动调用
@Deprecated
public static void prepareMainLooper() {
    	//直接调用了 prepare(false);
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
}
public static void prepare() {
  //无参方法传入true,默认允许退出
  prepare(true);
}
// prepare(false)，所以主线程传入的 false 是不允许退出的
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        //知识点：每个线程只能创建一个 Looper，再次调用 prepare 就会抛出异常
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    //实例化一个 Looper 存入 ThreadLocal
    sThreadLocal.set(new Looper(quitAllowed));
}
```

- prepare 在一个线程内只能调用一次，通过 sThreadLocal 判断

--> new Looper(quitAllowed)

```java
//Looper 构造函数时候初始化对应的 MessageQueue 对象
//知识点：一个 Looper 对应一个 MessageQueue
private Looper(boolean quitAllowed) {
    //传入的 quitAllowed 是在 MessageQueue.quit(boolean safe) 方法里使用的
        mQueue = new MessageQueue(quitAllowed);
    //Looper 和当前线程绑定
        mThread = Thread.currentThread();
}
```

那 Looper 如何和 Handler  、Message 联系起来，先看下 handler 发送 Message

handler.sendMessage(Message msg)

```java
public final boolean sendMessage(@NonNull Message msg) {
        return sendMessageDelayed(msg, 0);
}
```

```java
  public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
      //SystemClock.uptimeMillis() 从开机到现在的毫秒数（手机睡眠的时间不包括在内）
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
```

```java
public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
    //mQueue 的赋值来自 mLooper.mQueue 
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
}
```

handler.enqueueMessage 里面就是调用了 queue.enqueueMessage(msg, uptimeMillis) 方法

```java
    boolean enqueueMessage(Message msg, long when) {
        ……
       //多个线程给 MessageQueue 发消息，加锁保证线程安全，
        synchronized (this) {
		……
        }
        return true;
    }
```

而 Message 类里还有一个 Message 类型名为 next 的变量，用于存放下一个待处理的 Message，MessageQueue 按照  uptimeMillis 的顺序，通过 for (;;) 和一系列判断把多个 Message 组成一个单向链表

### 接下来看 Looper 的 loop 方法

loop 里也有一个 for (;;) 不断循环去取 Message msg = queue.next(); 

```java
//myLooper() 调用的就是 sThreadLocal.get();
final Looper me = myLooper();
    ……
for (;;){
    //next()内部也通过 synchronized 加锁，确保取的时候线程安全
    Message msg = queue.next();
    ……
// target 就是 Handler
msg.target.dispatchMessage(msg);
 ……
//
msg.recycleUnchecked();
}
```

Handler.dispatchMessage(Message msg)

```java
 public void dispatchMessage(@NonNull Message msg) {
 //上文说到 Message 的 callback ，即 Runnable ，如果不为空意味着是 post 方式发送
        if (msg.callback != null) {
        //handleCallback 之间调用了 message.callback.run() ，也就是 Runnable.run()
            handleCallback(msg);
        } else {
        //Handler 的 mCallback 是通过 Handler 的构造函数传入赋值的 
            if (mCallback != null) {
                //mCallback 的 handleMessage 
                if (mCallback.handleMessage(msg)) {
//如果返回 true 就 return 了，不会继续走下面 Handler 的  handleMessage 方法了
                    return;
                }
            }
            //Handler 的  handleMessage 
            handleMessage(msg);
        }
    }
```



## ThreadLocal

- 是一个线程内部的数据存储类，数据存取时做到了线程隔离
- 提供了一个全局的哈希表，用于实现指定线程的数据控制
- 推荐主动调用 ThreadLocal.remove 方法，防止内存泄漏



## HandlerThread





## IdleHandler





## Handler 的同步屏障机制





## 问答

### 子线程中创建 Handler 为啥会报错，主线程中创建为啥不报错？

```java
public Handler(@Nullable Callback callback, boolean async) {
 		……
        //myLooper() 调用的就是 sThreadLocal.get();
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            // mLooper 为空就会报错，提示需要先执行 Looper.prepare() 方法
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

结合上文，主线程中已经默认调用了 Looper.prepareMainLooper() 所以不会报错，而子线程没有就会报错

### 子线程中怎么去使用 Handler？

结合上文，子线程中使用 Handler 需要先执行两个操作：Looper.prepare()  和 Looper.loop()  ，因为上述 mLooper 为空时候，即 sThreadLocal.get() 为空，而 Looper.prepare()  就是 sThreadLocal.set(new Looper(quitAllowed)) ，另外调用 Looper.loop()  开启消息循环**等待接收消息**，因此最终不需要的时候别忘了终止 Looper，调用`Looper.myLooper().quit()`

### 子线程是不是一定不可以更新UI？一定不能在主线程中进行网络操作？

硬要在子线程中更新UI也是可以的，但是能更新的是子线程创建的UI，直接去更新主线程创建的UI就会报错

```
//ViewRootImpl.checkThread()
void checkThread() {
//因为检测的是否是当前线程
    if (mThread != Thread.currentThread()) {
        throw new CalledFromWrongThreadException(
                "Only the original thread that created a view hierarchy can touch its views.");
    }
}
```

还有在 onCreate 中创建子线程直接去更新UI也不会报错

旧版安卓在主线程中进行网络操作是不会报错的，新的理解加入了校验所以报错，所以生活可以把校验去掉硬在主线程里进行网络操作

### 主线程如何通过 Handler 给子线程发消息？

//**主线程延时给子线程发消息**

//通过 **HandlerThread**  





### Message#what 的不同值，会影响 Message 在 MessageQueue 中的顺序么？



MessageQueue 中的 Message是如何排列的？Msg 的 runnable 对象可以外部设置么，比如说不用Handler#post 系列方法（反射可以实现）；





### handler 如何实现延时发消息 postDelay ？

也是通过 sendMessageDelayed 方式实现的



###  View.post  和 Handler.post 的区别

上文说过了 Handler.post 接下来看下 View.post 原理

View.post

```java
    public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }
		//mRunQueue = new HandlerActionQueue();
        // Postpone the runnable until we know on which thread it needs to run.
        // Assume that the runnable will be successfully placed after attach.
        getRunQueue().post(action);
        return true;
    }
    
      public boolean postDelayed(Runnable action, long delayMillis) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.postDelayed(action, delayMillis);
        }

        // Postpone the runnable until we know on which thread it needs to run.
        // Assume that the runnable will be successfully placed after attach.
        getRunQueue().postDelayed(action, delayMillis);
        return true;
    }
```

View.post 最终也是通过 Handler.post 来执行消息的，执行过程如下：

1. 如果在 performTraversals 前调用 View.post，则会将消息进行保存，之后在 dispatchAttachedToWindow 的时候通过 ViewRootImpl 中的 Handler 进行调用。
2. 如果在 performTraversals 以后调用 View.post，则直接通过 ViewRootImpl 中的 Handler 进行调用。

同时这也就是为什么 View.post 里可以拿到 View 的宽高信息呢？

因为 View.post 的 Runnable 执行的时候，已经执行过 performTraversals 了，也就是 View 的 measure layout draw 方法都执行过了，自然可以获取到 View 的宽高信息了







### Looper在主线程中死循环，为啥不会 ANR ，如何响应用户事件？

Looper 阻塞 - 主线程也阻塞- **ActivityThread**  main函数也阻塞，不会走到最后一步抛异常

应用启动时，可不止一个**ActivityThread**  main线程，还有其他两个**Binder线程**：**ApplicationThread** 和 **ActivityManagerProxy**，用来和系统进程进行通信操作，接收系统进程发送的通知。





### handler 引起的内存泄漏，如何避免？

- 静态内部类 、WeakReference 弱引用

静态内部类，Handler 就不能调用 Activity 里的非静态方法了，所以加上 WeakReference  弱引用

有延时消息的销毁时候及时调用 handler.removeCallbacksAndMessages(null)



### 为啥 Message 消息对象持有构造它的 Handler 对象？





### Looper 阻塞的时候 采用了 epoll 机制 什么原理？





















 







