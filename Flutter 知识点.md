# Flutter 知识点
- Flutter 是一个由 Google 开发的开源 UI 工具包，用于构建跨平台移动应用、Web 应用、桌面应用和嵌入式应用，它使用的是 Dart 编程语言，具有快速的热重载功能和丰富的现代化 UI 组件，使得开发者能够轻松构建高性能、美观的应用程序


https://docs.flutter.cn/get-started/flutter-for/android-devs


## Flutter 特点
- Hot Reload 热重载
- Flutter 中一切皆是 Widget 呈现的，大致对应 Android 中的 View（不完全对应），也不会直接绘制任何内容，而是 UI 及其底层创建真正视图对象的语义的描述
- 使用自己的渲染引擎来绘制 UI，能够保证高性能，媲美原生应用的速度
- 在 Flutter 中，Widget 是不可变的，无法被直接更新，你需要操作 Widget 的状态
底层渲染引擎 Skia

在 Flutter 中，通过 组合 更小的 Widget 来创建自定义 Widget（而不是继承它们，这和 Android 中实现一个自定义的 ViewGroup 有些类似）
暂不支持从 Flutter 中直接调用 Native 代码
Skia绘制，实现跨平台应用层渲染一致性

## Flutter 官方 package
https://pub.dev/packages?q=publisher%3Aflutter.dev&sort=like

##  Dart 官方 package
https://pub.dev/packages?q=publisher%3Adart.dev&sort=like


Widget是用来描述Element配置信息的，Widget是不可变的，因此状态改变时，需要重新构建UI。

. 什么是StatelessWidget和StatefullWidget？他们之间的区别是什么？
StatelessWidget是一旦构建后状态就不能改变的Widget，无生命周期的回调。
StatefulWidget是一旦构建Widget的状态还会发生改变的Widget。
StatefulWidget有一个状态类State，维护了可变状态。当状态发生变化时，StatefulWidget 将会重建其对应的 State 对象。
生命周期不同，调用build()方法次数和时机不同。

作者：渚清与沙白
链接：https://www.jianshu.com/p/625af6bdca35
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


## StatelessWidget

## StatefulWidget


## 解决项目移动后编译报错的一种方式
删除 pubspec.lock 文件
执行 flutter pub get 命令

## 通过 async/await 来执行异步任务


## http

## Dio



flutter的channel通信有哪几种？你用的哪种？插件你如何实现的？



使用 isolate 的具体场景: 可能需要几百毫秒的
1、JSON解析: 解码JSON，这是HttpRequest的结果，可能需要一些时间，可以使用封装好的 isolate 的 compute 顶层方法。
2、加解密: 加解密过程比较耗时
3、图片处理: 比如裁剪图片比较耗时
4、从网络中加载大图 



Flutter 是如何与原生Android、iOS进行通信的？
Flutter 通过 PlatformChannel 与原生进行交互，其中 PlatformChannel 分为三种：

BasicMessageChannel ：用于传递字符串和半结构化的信息。
MethodChannel ：用于传递方法调用（method invocation）。
EventChannel : 用于数据流（event streams）的通信。
 


6. 什么是Flutter中的异步编程？有哪些常用的异步编程模式？
Flutter中的异步是向事件队列中插入任务。
Future、Stream
Future一次只支持一个任务，Stream可以支持多个任务。
async和await、await for是异步的语法糖。



JIT 和 AOT

