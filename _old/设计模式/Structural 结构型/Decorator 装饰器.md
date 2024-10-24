

- Decorator







- Wrapper 包装者
  - 通常推荐用 Decorator 作后缀







- 被装饰的类和装饰的类继承同一个类
- 装饰的类的被装饰的对象一定是外部传入的，一般是构造函数传入
- 功能的拓展，比继承更为灵活



装饰器的透明实现：跟被装饰的类的接口一致，但很多都是半透明的



Context

是一个抽象类，声明了各种抽象方法，所以可以将它看成一个接口






- ContextWrapper

```java
/**
 * Proxying implementation of Context that simply delegates all of its calls to
 * another Context.  Can be subclassed to modify behavior without changing
 * the original Context.
 * Context 的代理实现，简单地将其所有的调用委托给另外一个 Context，可以被子类化以修改行为而不改变原始的 Context
 */
//装饰类 ContextWrapper 继承 Context
public class ContextWrapper extends Context {
    //被装饰的类继承 Context，这里是 Context 本身
    @UnsupportedAppUsage
    Context mBase;
	//被装饰的类作为参数通过构造函数传入
    public ContextWrapper(Context base) {
        mBase = base;
    }
    /**
     * Set the base context for this ContextWrapper.  All calls will then be
     * delegated to the base context.  Throws
     * IllegalStateException if a base context has already been set.
     *
     * @param base The new base context for this wrapper.
     */
    protected void attachBaseContext(Context base) {
        if (mBase != null) {
            throw new IllegalStateException("Base context already set");
        }
        mBase = base;
    }

    /**
     * @return the base context as set by the constructor or setBaseContext
     */
    public Context getBaseContext() {
        return mBase;
    }

    @Override
    public AssetManager getAssets() {
        return mBase.getAssets();
    }

    @Override
    public Resources getResources() {
        return mBase.getResources();
    }

    @Override
    public PackageManager getPackageManager() {
        return mBase.getPackageManager();
    }

    @Override
    public ContentResolver getContentResolver() {
        return mBase.getContentResolver();
    }

    @Override
    public Looper getMainLooper() {
        return mBase.getMainLooper();
    }

    @Override
    public Executor getMainExecutor() {
        return mBase.getMainExecutor();
    }

    @Override
    public Context getApplicationContext() {
        return mBase.getApplicationContext();
    }

    @Override
    public void setTheme(int resid) {
        mBase.setTheme(resid);
    }
    
    ……
    
} 
```



ContextImpl

Context 的真正实现类



ContextThemeWrapper



Activity



Application



Service



ContextImpl.java$ReceiverRestrictedContext

是一个注册广播和绑定服务收限制的实现类（复写了对应方法，加入了限制）

静态注册的广播 BroadcastReceiver#onReceive 回调的 context 参数就是 ReceiverRestrictedContext 实例，所以 onReceive的 context 不能再注册广播和绑定服务





