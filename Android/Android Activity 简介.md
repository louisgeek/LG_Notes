11 

是一个界面 一个用户接口






 - Activity 的四种状态

   

  - 运行状态 **Active**

    Activity 获取了焦点

  - 暂停状态 **Paused**

     Activity失去了焦点，如显示了对话框，显示了另一个透明的 Activity

  - 停止状态 **Stopped**

     完全不可见的，但是内存里面的成员变量，状态信息等仍然存在，但是也没有被销毁

  - 销毁状态 **Killed**

    Activity 被销毁，内存里面的成员变量，状态信息等都会被一并回收

# Activity 的三个生存期

1.完整生存期 onCreate=onDestory
2.可见生存期 onStart=onStop
3.前台生存期 onResume===onPause





在任意位置关掉应用所有Activity & 如何在任意位置关掉指定的Activity

用Intent传递数据和Bundle传递数据的区别？为什么不用HashMap

从源码角度解析，Activity的启动流程

隐式启动和显示启动Activity的方式

Activity常用的标记位Flags

Activity之间如何通信

[startActivity()干了什么？](https://www.jianshu.com/p/89fd44083c1c)

隐式启动中Intent可以设置多个action,多个category吗 & 顺便讲讲它们的匹配规则

在Activity进行配置时，catrgory和action的区别是什么

了解scheme跳转协议吗？谈一谈

给Activity设置进入和退出的动画

使用Intent传递数据是否有限制 & 如果传递一个复杂的对象，例如一个复杂的控件对象应该怎么做

[setContentView干了什么](https://www.jianshu.com/p/d9d919608842)

可以多次调用setContentView方法吗？说说不同时机第二次调用setContentView会发生什么

onNewIntent()方法的认识？

你对Activity的Context的认识

Application中获取当前Activity实例

Activity进程优先级

出现ANR的条件有哪些 & 解决方案

Activity什么时候加载布局文件的？什么开始进行UI绘制？





----

分别在Activity里每一个生命周期函数里调用finish方法后，接下来Activity的生命周期如何回调

有什么方法可以启动一个没有在AndroidManifest.xml中注册过的Activity

activity中分别在onCreate，onStart，onResume中调用finish后的生命周期如何回调？



