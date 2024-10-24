# Java 动态代理
- 动态代理指在程序运行时通过反射机制等技术动态创建出代理对象
- 使用 Proxy#newInstance 来创建指定接口的代理实现类
- 通过 Proxy.newProxyInstance 方法产生的代理类，当调用接口的任何方法时都会调用 InvocationHandler#invoke 方法，可以在这个方法中可以拿到传入的参数，注解等
- 只能实现基于接口的动态代理，因为 Java 是单继承的，它在动态生成 $ProxyX 代理类的时候已经继承 Proxy 类了
- 可以减少类的数量，降低工作量
- 减少对业务接口的依赖，降低耦合，便于后期维护
- 但是因为使用了反射，所以会在运行时消耗一定的性能


  并有一个带 InvocationHandler 参数的构造方法，使用 super 调用父类 Proxy 类的构造方法

```java
private InvocationHandler mInvocationHandler = new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return method.invoke(xxx,args);
        }
    };
//
Class<?> clazz = Proxy.getProxyClass(Toast.class.getClassLoader(), Toast.class);
Toast toastProxy = (Toast) clazz.getConstructor(InvocationHandler.class)
                    .newInstance(mInvocationHandler);
//类加载器
//对象的接口???
//调用处理器，代理对象中的方法被调用时都会执行 invoke 方法，可以对所有被代理对象的方法进行拦截
Toast toastProxy2 = (Toast) Proxy.newProxyInstance(Toast.class.getClassLoader(),
                    new Class<?>[]{Toast.class}, mInvocationHandler);
```


## 使用场景
- 记录日志，调用方法后记录日志
- 监控性能，统计执行方法消耗的时间
- 权限控制，调用方法前校验是否有权限