## OkHttp  

- 功能全面，满足了绝大部分网络请求的需求
- 内置 ConnectionPool 连接池来复用连接以提高效率
- 默认支持 gzip 压缩来降低传输内容的大小
- 提供了对 HTTP 响应的缓存机制，可以避免不必要的网络请求
- 支持最新的 HTTP 协议版本 HTTP/2 和 SPDY
- 网络出现问题时，OkHttp 会自动重试一个主机的多个 IP 地址
- 拦截器模式的设计使其非常方便得进行功能扩展





## 使用

- 采用了建造者模式

```java
OkHttpClient okHttpClient = OkHttpClient.Builder().build();
Request request = new Request.Builder().url(mUrl).build();
//Call 是接口，实现类是 RealCall
Call call = okHttpClient.newCall(request);
//同步请求，需要放到子线程里调用
Response response = call.execute();
//异步请求
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
    }
    @Override
    public void onResponse(Call call, Response response) throws IOException {
        //不是主线程，需要使用 Handler 切换到 UI 线程
        Log.e(TAG, "onResponse: " + response.body().string() );
    }
});
```







![OkHttp 流程图](https://gitee.com/louisgeek/LG_Notes/raw/master/images/okhttp_lc.png)



## OkHttpClient

- 带 Builder

- 维护了一个  Dispatcher 调度器





## Dispatcher 调度器

- 分发器，任务调度核心类，负责分发执行任务，控制请求并发数和执行时机
- 通过 RealCall#execute 和 RealCall#enqueue 方法分别管理同步和异步请求，管理每一个请求任务的状态，这两个方法会最终调用 RealCall#getResponseWithInterceptorChain 方法，从拦截器链中获取返回结果
- 内部维护了一个 ThreadPoolExecutor 线程池用于执行异步请求
- 内部维护了三个 Deque 队列



## 拦截器

- 采用了责任链模式

实现拦截器的分层设计，每一个拦截器对应一个功能，充分实现了功能解耦，易维护

不管是同步请求还是异步请求，最后都会走 okhttp3.RealCall#getResponseWithInterceptorChain 方法

### client.interceptors()

- 传入的自定义应用拦截器
- 每个 HTTP 响应都只会调用一次，即使 HTTP 响应是从缓存中获取
- 不关心 headers 
- 可以通过不调用 Chain#proceed 方法来终止请求，也可以通过多次调用 Chain#proceed 方法来进行重试

- 如 TokenInterceptor ，请求耗费的时间统计



### RetryAndFollowUpInterceptor

- 网络请求失败后进行重试
- 重定向时直接发起新的请求



#### StreamAllocation

负责管理 Connections 、Streams 、Calls ，针对一个 RealCall 通 while (true) 循环 findConnection 寻找一个 RealConnection，获得一个 Stream（HttpCode），然后调用 RealConnection#connect 调用 Platform#connectSocket 使用 socket 与服务端建立连接



### BridgeInterceptor

- 把原始代码写的 Request 内容转换成服务端可接受格式的 Request ，如果有 RequestBody 进行设置 Content-Type、Content-Length、按需设置 Transfer-Encoding 支持 chunked
- 设置 Accept-Encoding 支持 gzip 压缩传输，并且在接收到内容后进行解压
- 设置 Cookie
- 设置 Connection 支持 Keep-Alive
- 设置 User-Agent、Host



### CacheInterceptor

- 管理服务器返回内容的缓存，由内存策略决定



#### CacheStrategy.Factory

- Date
- Expires
- Last-Modified 用时间戳标识资源是否被修改
- ETag  用资源标识码标识资源是否被修改
- Age







### ConnectInterceptor

- 为请求找到合适的连接，复用已有连接或重新创建的连接，由连接池决定
- 通过 StreamAllocation#newStream 获得一个 Stream（HttpCode）
- 然后 RealConnection#connect 调用 Platform#connectSocket 使用 socket 与服务端实现连接





### client.networkInterceptors()

- 传入的自定义网络拦截器
- 调用执行中的自动重定向和重试所产生的响应也会被调用，而如果响应来自缓存，则不会被调用

- 如 HttpLoggingInterceptor



### CallServerInterceptor

- 向服务器发起真正的访问请求，获取 response
- 利用 okhttp3.internal.http.HttpCodec#writeRequestHeaders 写入请求头到服务器

- 利用 okhttp3.RequestBody#writeTo 写入 okio.BufferedSink 写入请求体到服务器
- 利用 okhttp3.internal.http.HttpCodec#readResponseHeaders  从服务端读取响应头

- 利用 okhttp3.internal.http.HttpCodec#openResponseBody 从服务端读取响应体





## 缓存

- 采用了策略模式

缓存策略

DiskLruCache





## 总结



#### Request 和 Call 的区别？

- Request 表示 Http 的 Request 请求，用来封装网络请求信息及请求体，跟 Response 相对应
- Call 表示请求执行器，负责发起网络请求



#### OKHttp 针对网络层有哪些优化？







#### OKHttp 是如何处理网络缓存的？







#### OKHttp 和 HttpUrlConnection 的关系?







#### StreamAllocation 为什么在除自定义拦截器以外第一个拦截器中就进行创建？

便于取消请求以及出错释放资源





#### client.interceptors() 和  client.networkInterceptors() 的区别?

应用拦截器是最先执行的拦截器

网络拦截器位于 ConnectInterceptor 和 CallServerInterceptor 之间

因为应用拦截器在 RetryAndFollowUpInterceptor 之前，所以一旦发生错误重试或网络重定向，网络拦截器可能会执行多次(因为相当于进行了二次请求)，但是应用拦截器只走了一次

另外如果在 CacheInterceptor 中命中了缓存，那就意味着不需要继续走网络请求了，所以这时候就不会执行网络拦截器了





#### 其他一些涉及到的设计模式？

单例模式：线程池
观察者模式：各种回调监听
外观模式：OkHttpClient 封装了很多类对象
工厂模式：Socket 的生产