# Android_Note

[Java_Note](https://github.com/louisgeek/Java_Note)

[Kotlin_Note](https://github.com/louisgeek/Kotlin_Note)





Android面试官装逼失败之：Activity的启动模式https://www.jianshu.com/p/b3a95747ee91

我打赌你一定没搞明白的Activity启动模式https://www.jianshu.com/p/2a9fcf3c11e4

任务栈？返回栈？启动模式？https://mp.weixin.qq.com/s/9fwQscSZ10pFdmi7bUFaaA

- 设计模式

  Android进阶之大话设计模式 http://mobile.51cto.com/android-419145.htm

- Android Studio

  跳过sdk检查更新

  ```
  在Android Studio\bin\idea.properties中添加disable.android.first.run=true
  ```

  

  最强 Android Studio 使用技巧和快捷键 http://www.open-open.com/lib/view/open1458715872710.html


  

- 四大组件 Activity 

  - 主线程 

    ComponentActivity.java 中 post 主线程方法 
    
    ```java
    new Handler(Looper.getMainLooper()).post(new Runnable()(){
        
    });
    ```
    
    



- Fragment 

  onActivityResult() inside Nested Fragment is now called on Support Library rev 23.2 and above https://inthecheesefactory.com/blog/onactivityresult-nested-fragment-support-library-v23.2/en

- navigation

  

- TabLayoutMediator

  viewpager2 & tablayout


 
Service执行的操作最多是20s，BroadcastReceiver是10s，Activity是5s

- 可以在xml中设置Service所在的进程，让Service在另外的进程中执行。







UI



  

- ListView

  ArrayIndexOutOfBoundsException问题：http://blog.csdn.net/yak262/article/details/8825409

- RecyclerView 

  - RecyclerView 封装

    封装那些事-RecyclerView封装实践 http://www.jianshu.com/p/a6f158d1a9c9

    真实项目运用-RecyclerView封装 http://www.jianshu.com/p/2f2996ef2c75

- 自定义view

   

- ~~百分比布局~~ 改用约束布局

Android UI性能优化详解 http://mrpeak.cn/android/2016/01/11/android-performance-u





- Material Design

material配色 https://www.materialpalette.com/

material 组件
MaterialShapeDrawable
MaterialShapeUtils
MaterialToolbar

- 动画

  https://github.com/lgvalle/Material-Animations

  演示View的平移、缩放动画，activity进入和退出动画，界面间元素共享，并且开发者在README中，对动画原理进行了精讲，是学习动画很好的项目，项目代码量比较少，也很适合新手学习

  

  

  

- ViewBinding

  https://developer.android.google.cn/topic/libraries/view-binding

  

  

  

- Demo

1、google 官方demo https://developer.android.com/samples/index.html

1、google 架构组件demo https://github.com/android/architecture-components-samples

 

4、泡在网上的日子里的完整项目   http://www.jcodecraeer.com/plus/list.php?tid=31&codecategory=22000

http://mobdevgroup.com/platform/android/project

5、第三方热门开源库

6、Android系统源码


  如何看懂源代码--(分析源代码方法)

  http://www.cnblogs.com/ToDoToTry/archive/2009/06/21/1507760.html

- arch项目

  BaseArchitecture

  台湾人写的 MVP     dragger.android  rxjava room 等   中等简洁

  Ganker

  大部分采用谷歌的思维和代码  搭配学习谷歌 sample demo

  https://github.com/xinghongfei/LookLook

  可以阅读知乎日报，网易头条，每日推送一张妹子图片和视频，是一个精美的阅读软件。遵循Google Meterial 设计风格，加入了一些5.0以上的新特性，阅读体验绝不逊色于官方的app。

  https://github.com/googlesamples/android-UniversalMusicPlayer

  这个开源项目展示了如何实现一个横跨各种Android平台的音乐播放器，包括手机，平板，汽车，手表，电视等。Google官方推出，跨平台开发必看项目

  https://github.com/nickbutcher/plaid

  由谷歌工程师开发，展示Google Material风格设计，项目代码量大，但是结构清晰，还是很好理解的。

- ML Kit

https://developers.google.cn/ml-kit

https://github.com/googlesamples/mlkit/tree/master/android/vision-quickstart

------

 


其他附加知识

- RESTful API 

  RESTful API 设计指南 http://www.androidchina.net/3749.html

- H5

  H5 调用摄像头 

  https://webrtc.github.io/samples/src/content/devices/input-output/

  https://www.jb51.net/html5/722394.html

- 正则表达式

  https://deerchao.net/tutorials/regex/regex.htm
  http://www.runoob.com/regexp/regexp-syntax.html





（1）类名.class：JVM将使用类装载器，将类装入内存(前提是:类还没有装入内存)，不做类的初始化工作，返回Class 的对象。

（2）Class.forName("类名字符串")：装入类，并做类的静态初始化，返回Class的对象。

（3）实例对象.getClass()：对类进行静态初始化、非静态初始化；返回引用运行时真正所指的对象(子对象的引用会赋给父对象的引用变量中)所属的类的Class的对象。

- 动态代理模式
- AndroidDavilk与ART
- PathClassLoader、DexClassLoader与BootClassLoader






