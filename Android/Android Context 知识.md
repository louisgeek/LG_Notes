

Context

- 环境、上下文、场景
- 是一个抽象类，把很多种场景抽象成一类
- 应用程序环境的全局信息接口
- Activity、Service、Application 都间接继承自 Context
- 启动 Activity、Service、Br、访问资源、调用系统级服务
- 显示 Toast、创建 Dialog 时需要传入





ContextWrapper

- Wrapper 包装
- ContextWrapper 就是一个包装类、装饰类
- 装饰模式，包装了 mBase 一个 ContextImpl，扩展了 ContextImpl 的功能



ContextImpl

- 抽象类 Context 的实现类



### Application Context 创建

```
ActivityThread#handleLaunchActivity
- ActivityThread#performLaunchActivity
-- LoadedApk#makeApplication
--- ContextImpl#createAppContext 得到一个 ContextImpl 类型的 appContext
--- Instrumentation#newApplication 内部利用反射方式创建 Application 得到 app
---- Application#attach 
----- ContextWrapper#attachBaseContext 内部就是把 appContext 赋给 ContextWrapper 的 mBase
--- appContext.setOuterContext(app)
```



Application Context 获取

```
Context#getApplicationContext
ContextWrapper#getApplicationContext
ContextImpl#getApplicationContext
LoadedApk#getApplication
```

Activity Context 创建

```
ActivityThread#handleLaunchActivity
- ActivityThread#performLaunchActivity
- ActivityThread#createBaseContextForActivity 得到一个 ContextImpl 类型 appContext
- Instrumentation#newActivity 得到 Activity 类型 activity
- appContext.setOuterContext(activity)
- Activity#attach
-- ContextThemeWrapper#attachBaseContext 内部也是把 appContext 赋给 ContextWrapper 的 mBase

```



Service Context 创建

```
ActivityThread#handleCreateService
- ContextImpl#createAppContext 得到一个 ContextImpl 类型 context
- context.setOuterContext(service)
- Service#attach
-- ContextWrapper#attachBaseContext 内部也是把 context 赋给 ContextWrapper 的 mBase
```

