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





### 涉及设计模式

- create 方法非常典型的采用动态代理模式
- Builder 建造者模式
- 网络请求工厂用了工厂方法模式





### Retrofit 和 OkHttp 的区别?

Retrofit 引用了 OkHttp，Retrofit 专注于请求接口的封装，OkHttp 专注于网络请求的高效
