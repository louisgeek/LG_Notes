> 代码基于 org.greenrobot:eventbus:3.2.0



https://github.com/greenrobot/EventBus

https://greenrobot.org/eventbus



基于**发布/订阅事件**总线框架，基于观察者模式 



使用 @Subscribe 注解方法，方面名随意推荐 Event 结尾，一目了然

```java
@Subscribe
public void BaseEvent(String event) {
    //
}
```
@Subscribe 注解
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Subscribe {
    //默认 ThreadMode.POSTING
    ThreadMode threadMode() default ThreadMode.POSTING;
    /**
     * If true, delivers the most recent sticky event (posted with
     * {@link EventBus#postSticky(Object)}) to this subscriber (if event available).
     */
    //是否支持接收粘性事件 默认 false
    boolean sticky() default false;
    /** Subscriber priority to influence the order of event delivery.
     * Within the same delivery thread ({@link ThreadMode}), higher priority subscribers will receive events before
     * others with a lower priority. The default priority is 0. Note: the priority does *NOT* affect the order of
     * delivery among subscribers with different {@link ThreadMode}s! */
    //优先级
    int priority() default 0;
}
```

关键属性

- 线程模式

- 粘性事件

- 优先级

  

```java
EventBus.getDefault().register(this);
```

## EventBus#getDefault()

```java
//volatile
static volatile EventBus defaultInstance;
……
//就是通过 双重校验并加锁的单例模式 获取到 EventBus 的实例
public static EventBus getDefault() {
        EventBus instance = defaultInstance;
        if (instance == null) {
            synchronized (EventBus.class) {
                instance = EventBus.defaultInstance;
                if (instance == null) {
                    instance = EventBus.defaultInstance = new EventBus();
                }
            }
        }
        return instance;
    }
```



## EventBus#register(this)

//this 通常传入 Activity 实例，如 MainActivity.this

```java
//subscriber 订阅者
public void register(Object subscriber) {
    //订阅者类的类对象 com.xxx.MainActivity.class
        Class<?> subscriberClass = subscriber.getClass();
    //通过反射获取订阅者类中的所有方法，然后找到以 @Subscribe 注解的方法，添加到 subscriberMethods 列表中
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
            //循环 subscriberMethods 调用 subscribe
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
```

### SubscriberMethod 类

```
public class SubscriberMethod {
    final Method method;
    final ThreadMode threadMode;
    final Class<?> eventType;
    final int priority;
    final boolean sticky;
    /** Used for efficient comparison */
    String methodString;
    ……
    }
```

### SubscriberMethodFinder#findSubscriberMethods()

SubscriberMethodFinder 类 ，在 EventBus 实例化的时候创建

```java
……
Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();
……
  List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
    //METHOD_CACHE 缓存有的话，直接 get 得到 List<SubscriberMethod>
        List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
        if (subscriberMethods != null) {
            return subscriberMethods;
        }
		//EventBusBuilder.ignoreGeneratedIndex()  构造时候传入，默认 false
        if (ignoreGeneratedIndex) {
            subscriberMethods = findUsingReflection(subscriberClass);
        } else {
            //默认
            subscriberMethods = findUsingInfo(subscriberClass);
        }
        if (subscriberMethods.isEmpty()) {
            throw new EventBusException("Subscriber " + subscriberClass
                    + " and its super classes have no public methods with the @Subscribe annotation");
        } else {
            //缓存起来
            METHOD_CACHE.put(subscriberClass, subscriberMethods);
            return subscriberMethods;
        }
    }
    ……
```

#### subscriberMethodFinder.findUsingInfo(Class<?> subscriberClass)

```java
    private List<SubscriberMethod> findUsingInfo(Class<?> subscriberClass) {
    //假设 FindState[] 数组 FIND_STATE_POOL里有缓存，取缓存，没有之间返回 new FindState()
        FindState findState = prepareFindState();
        //FindState.subscriberClass = subscriberClass
        //FindState.clazz = subscriberClass
        findState.initForSubscriber(subscriberClass);
        while (findState.clazz != null) {
            //初始时返回 null
            findState.subscriberInfo = getSubscriberInfo(findState);
            if (findState.subscriberInfo != null) {
                SubscriberMethod[] array = findState.subscriberInfo.getSubscriberMethods();
                for (SubscriberMethod subscriberMethod : array) {
                    if (findState.checkAdd(subscriberMethod.method, subscriberMethod.eventType)) {
                        findState.subscriberMethods.add(subscriberMethod);
                    }
                }
            } else {
                //初始时 findState.subscriberInfo 为 null，所以走这里
                //这里就是通过反射获取到所有 DeclaredMethods / Methods，然后循环得到 public 的，不是 static 的，不是 abstract 的 （还有不是 BRIDGE 、SYNTHETIC ），同时有 @Subscribe 注解的且一个参数的方法
                //然后构造： method方法名，eventType( class XxxEvent )，threadMode，优先级，是否为sticky方法
                //findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                findUsingReflectionInSingleClass(findState);
            }
            //检查父类  会跳过 java. javax. android. androidx.开头的类
            findState.moveToSuperclass();
        }
        return getMethodsAndRelease(findState);
    }
```

#### subscriberMethodFinder.getSubscriberInfo(FindState findState)

```java
 private SubscriberInfo getSubscriberInfo(FindState findState) {
        if (findState.subscriberInfo != null && findState.subscriberInfo.getSuperSubscriberInfo() != null) {
            SubscriberInfo superclassInfo = findState.subscriberInfo.getSuperSubscriberInfo();
            if (findState.clazz == superclassInfo.getSubscriberClass()) {
                return superclassInfo;
            }
        }
        if (subscriberInfoIndexes != null) {
            for (SubscriberInfoIndex index : subscriberInfoIndexes) {
                SubscriberInfo info = index.getSubscriberInfo(findState.clazz);
                if (info != null) { 
                    return info;
                }
            }
        }
        return null;
    }
```

#### subscriberMethodFinder.getMethodsAndRelease(FindState findState)

```java
 private List<SubscriberMethod> getMethodsAndRelease(FindState findState) {
     //此时 findState.subscriberMethods 已经 add 过了
     //复制 findState 里的 subscriberMethods 到新的 subscriberMethods 里
        List<SubscriberMethod> subscriberMethods = new ArrayList<>(findState.subscriberMethods);
     //恢复 findState 
        findState.recycle();
        synchronized (FIND_STATE_POOL) {
            for (int i = 0; i < POOL_SIZE; i++) {
                if (FIND_STATE_POOL[i] == null) {
                    FIND_STATE_POOL[i] = findState;
                    break;
                }
            }
        }
        return subscriberMethods;
    }
```

### EventBus#subscribe(Object subscriber, SubscriberMethod subscriberMethod)

```java
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
      //class XxxEvent
        Class<?> eventType = subscriberMethod.eventType;
    //包装了，订阅者、订阅者的方法集
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
    //Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType 是一个 HashMap，存放着 eventType(事件类) ->
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    //初始的时候为 null 
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<>();
            //存入 subscriptionsByEventType 
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }
		//如果初始的时候 size = 0 
       //第二次走的时候 size = 1 
        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            //0 == 0 成立，那么 add
            //0 == 1 不成立；或者当前优先级大的 add   
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }
		//Map<Object, List<Class<?>>> typesBySubscriber 是一个 HashMap，存放着订阅者 -> eventType(事件类)的集合
        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<>();
            //typesBySubscriber 存入
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
    //add eventType
        subscribedEvents.add(eventType);
		//如果是粘性的
        if (subscriberMethod.sticky) {
            //EventBusBuilder.eventInheritance() 构造时候传入，默认 true
            if (eventInheritance) {
                // Existing sticky events of all subclasses of eventType have to be considered.
                // Note: Iterating over all events may be inefficient with lots of sticky events,
                // thus data structure should be changed to allow a more efficient lookup
                // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
                // Map<Class<?>, Object> stickyEvents 是一个 ConcurrentHashMap<>() ，EventBus 构造的时候初始化
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    //判断 candidateEventType 是不是 eventType 的子类
                    if (eventType.isAssignableFrom(candidateEventType)) {
                        Object stickyEvent = entry.getValue();
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
    }

```

EventBus#checkPostStickyEventToSubscription(Subscription newSubscription, Object stickyEvent)

```java
   private void checkPostStickyEventToSubscription(Subscription newSubscription, Object stickyEvent) {
        if (stickyEvent != null) {
            // If the subscriber is trying to abort the event, it will fail (event is not tracked in posting state)
            // --> Strange corner case, which we don't take care of here.
            //直接调用 postToSubscription
            postToSubscription(newSubscription, stickyEvent, isMainThread());
        }
    }
```

先看 EventBus.getDefault().post(Object event)

```java
   public void post(Object event) {
       //currentPostingThreadState 是一个 ThreadLocal<PostingThreadState>
        PostingThreadState postingState = currentPostingThreadState.get();
        List<Object> eventQueue = postingState.eventQueue;
        eventQueue.add(event);
		//是否在不在发送中
        if (!postingState.isPosting) {
            //MainThreadSupport.AndroidHandlerMainThreadSupport 实现类构造传入 Looper.getMainLooper() 
            //然后和 Looper.myLooper() 作比较判断是否是主线程
            postingState.isMainThread = isMainThread();
            //标记发送中
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
                //循环发送事件
                while (!eventQueue.isEmpty()) {
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }
```

EventBus#postSingleEvent(Object event, PostingThreadState postingState)

```java

  private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        Class<?> eventClass = event.getClass();
        boolean subscriptionFound = false;
       //EventBusBuilder.eventInheritance() 构造时候传入，默认 true
        if (eventInheritance) {
            //取出XxxEvent及其父类和接口的class列表 里面也有 eventTypesCache 缓存
            List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
            int countTypes = eventTypes.size();
            for (int h = 0; h < countTypes; h++) {
                Class<?> clazz = eventTypes.get(h);
                subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
            }
        } else {
            subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
        }
        if (!subscriptionFound) {
            if (logNoSubscriberMessages) {
                logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
            }
            if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                    eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
    }

```

EventBus#postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass)

```java
 private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
        CopyOnWriteArrayList<Subscription> subscriptions;
        synchronized (this) {
            //取出 subscriptions ，和 subscribe 存入相对应
            subscriptions = subscriptionsByEventType.get(eventClass);
        }
        if (subscriptions != null && !subscriptions.isEmpty()) {
            for (Subscription subscription : subscriptions) {
                postingState.event = event;
                postingState.subscription = subscription;
                boolean aborted;
                try {
                    //也调用 postToSubscription
                    postToSubscription(subscription, event, postingState.isMainThread);
                    aborted = postingState.canceled;
                } finally {
                    postingState.event = null;
                    postingState.subscription = null;
                    postingState.canceled = false;
                }
                if (aborted) {
                    break;
                }
            }
            return true;
        }
        return false;
    }

```

EventBus#postToSubscription(Subscription subscription, Object event, boolean isMainThread)

```java
 private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                //直接 method.invoke
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                     //直接 method.invoke
                    invokeSubscriber(subscription, event);
                } else {
                    //通过handler去发送
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case MAIN_ORDERED:
                if (mainThreadPoster != null) {
                    mainThreadPoster.enqueue(subscription, event);
                } else {
                    // temporary: technically not correct as poster not decoupled from subscriber
                    invokeSubscriber(subscription, event);
                }
                break;
                
            case BACKGROUND:
                //BACKGROUND中的任务是一个接着一个的去调用
                if (isMainThread) {
                    //线程池去调用
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    //直接 method.invoke
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                //ASYNC则会即时异步运行
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }

```

EventBus#unregister(Object subscriber)

```java
   public synchronized void unregister(Object subscriber) {
       //取出 eventType(事件类)的集合
        List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
        if (subscribedTypes != null) {
            for (Class<?> eventType : subscribedTypes) {
                unsubscribeByEventType(subscriber, eventType);
            }
            typesBySubscriber.remove(subscriber);
        } else {
            logger.log(Level.WARNING, "Subscriber to unregister was not registered before: " + subscriber.getClass());
        }
    }
```

EventBus#unsubscribeByEventType(Object subscriber, Class<?> eventType)

```java
    private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
        List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions != null) {
            int size = subscriptions.size();
            for (int i = 0; i < size; i++) {
                Subscription subscription = subscriptions.get(i);
                if (subscription.subscriber == subscriber) {
                    subscription.active = false;
                    subscriptions.remove(i);
                    i--;
                    size--;
                }
            }
        }
    }
```

