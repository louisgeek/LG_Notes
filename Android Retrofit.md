Retrofit

- 支持同步、异步
- 支持多种数据格式的解析
- 通过注解配置网络请求参数
- 采用大量设计模式简化使用
- 功能模块高度封装、解耦性强
- 遵循 Restful API 设计风格
- 支持 RxJava

通过创建一个接口，在接口中写出所有需要请求网络资源的方法，不需要去实现该接口，通过编写注释的方式，让Retrofit 通过解析注解来获得指定的参数，以及实现具体的请求逻辑







RequestBuilder#build 调用 OkHttp 真正的发起网络请求



### Retrofit 和 OkHttp 的区别?

Retrofit 引用了 OkHttp，Retrofit 专注于请求接口的封装，OkHttp 专注于网络请求的高效


## 涉及设计模式
- create 方法非常典型的采用代理模式（JDK 动态代理）
- Builder 建造者模式
- 网络请求工厂用了工厂方法模式

## 动态代理
- Retrofit 内部就是通过 JDK 动态代理的方式（Proxy.newProxyInstance）生成接口对象的
```java
//Retrofit#create 传入接口生成动态代理对象
public <T> T create(final Class<T> service) {
    //校验
    validateServiceInterface(service);
    //JDK 动态代理
    //java.lang.reflect.Proxy
    return (T)Proxy.newProxyInstance(
            service.getClassLoader(),
            new Class<?>[] {service},
            new InvocationHandler() {
              private final Platform platform = Platform.get();
              private final Object[] emptyArgs = new Object[0];

              @Override
              public @Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args) throws Throwable {
   // If the method is a method from Object then defer to normal invocation.
              if (method.getDeclaringClass() == Object.class) {
                  return method.invoke(this, args);
              }
              args = args != null ? args : emptyArgs;
              return platform.isDefaultMethod(method)
                    ? platform.invokeDefaultMethod(method, service, proxy, args)
                    : loadServiceMethod(method).invoke(args);
              }
            });
  }
```
