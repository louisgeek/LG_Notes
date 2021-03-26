## 两种声明方式

- launchMode  

  在 Manifest.xml 清单文件中声明待启动 Activity 的 launchMode 属性

- Intent flag 

  在代码中通过 Intent 启动 Activity 时设置指定的 flag

> 清单文件的 launchMode 和 Intent flag 都不能完全代替对方，如果两种方式都设置了，Intent flag 优先级较高

###  launchModel 

> Task 任务(栈)  Tasks 任务(栈)组  PS：栈是其结构，加入该字眼方便后期理解 
>
> Back Stack 返回栈

大概可以分两种类型

1类 standard 和 singleTop，Activity 可以多次实例化，每个 Activity  实例可以归属任意一个任务栈，一个任务栈可以拥有多个 Activity 实例

2类 singleTask 和 singleInstance，只允许一个任务栈，只能保持一个 Activity 实例，并且始终在栈根位置

- standard 标准模式 

  默认模式，每次启动 Activity 都会新建一个新的 Activity 实例，待启动 Activity 会进入源 Activity 所属任务栈

- singleTop 栈顶复用模式

  如果待启动 Activity 已经位于源 Activity 所属的任务栈的栈顶时，不会创建新的 Activity 实例，而是直接使用栈顶的 Activity 实例，并回调其 `onNewIntent()`方法，`onCreate()` 和 `onStart()`不会被调用，直接调 `onResume()` ，否则就在栈顶创建一个新的 Activity  实例

  - 推送点击消息界面

- singleTask 栈内复用模式

  如果另外的任务栈中已存在待启动 Activity 的实例，则不会创建新的 Activity 实例，而是直接使用栈内的 Activity 实例，并回调其 `onNewIntent()` 方法，`onCreate()` 和 `onStart()`不会被调用，直接调 `onResume()` ，同时清空任务栈里该 Activity 实例上所有的 Activity 实例，Activity 一次只能有一个实例存在

  - 应用首页

- singleInstance 单实例模式

  首次启动时创建一个任务栈，该 Activity 实例始终在这个任务栈中，同时该任务栈中且只有这一个实例，如果存在则不新建，所以 Activity 实例和任务栈都是全局唯一的

  - 拨打电话界面、呼叫来电界面

  

  ### Intent flag

  - FLAG_ACTIVITY_NEW_TASK

    待启动 Activity 的启动模式是 singleInstance 或者 singleTask 时候，系统会自动加上 FLAG_ACTIVITY_NEW_TASK

    

    

  - FLAG_ACTIVITY_CLEAR_TOP

  - FLAG_ACTIVITY_SINGLE_TOP

    栈顶复用，和 singleTop 一致

    FLAG_ACTIVITY_CLEAR_TASK 

    这个属性必须和 FLAG_ACTIVITY_NEW_TASK 一起使用

  

  

  

  

  

  

  ## 启动模式的意义 ##

假设一个 Activity 频繁启动和退出，而每次启动都创建新实例 造成资源浪费，所以需要创造出启动模式去解决这个问题

任务栈







## 使用场景



A app MainAty  调转到  B app 的  MoreAty 点返回键 如何能够返回 B app MainAty 而不是 返回 A app 的 MainAty 



1 返回栈是什么？作用是什么？任务栈和返回栈有什么区别？

用来存放用户操作页面回退行为的，返回栈里存放的是可能来自不同任务栈里的 Activity 

2 前台任务栈 后台任务栈？

在不同任务栈种会存在重复的 singleTask 定义的 Activity 实例吗？

3 singleTop 只在当前任务栈种生效？

