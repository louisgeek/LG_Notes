```
ContextWrapper#startService
包装类方法直接调用
Context#startService 其实就是 ContextImpl#startService
内部调用
ContextImpl#startServiceCommon
内部其实就是调用了 AMS 的方法
ActivityManager.getService().startService


```





```java
public class ContextWrapper extends Context{
	Context mBase;
	@Override
	public ComponentName startService(Intent service){
		return mBase.startService(service);
	}
}
```

- Wrapper 包装
- ContextWrapper 就是一个包装类、装饰类
- 装饰模式，包装了 mBase 其实是一个 ContextImpl，扩展了 ContextImpl 的功能



ActivityThread#performLaunchActivity

里有个 createBaseContextForActivity 得到一个 ContextImpl 类型的 appContext 

传入 Activity#attach 里，将 ContextImpl 赋值给 ContextWrapper 的 mBase





