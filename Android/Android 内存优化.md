### 内存优化



用 StringBuilder、StringBuffer 代替 String

用 ArrayMap、SparseArray 替换 HashMap

避免内存泄漏

- 集合类泄漏(集合一直引用着被添加进来的元素对象)
- 单例/静态变量造成的内存泄漏(生命周期长的持有了生命周期短的引用)
- 匿名内部类/非静态内部类
- 资源未关闭造成的内存泄漏
  - 网络,文件等流忘记关闭
  - 手动注册广播时,退出时忘记unregisterReceiver()
  - Service执行完成后忘记stopSelf()
  - EventBus等观察者模式的框架忘记手动解除注册

