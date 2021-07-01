

- Wrapper 包装者
  - 也可以用 Decorator 作后缀







- 被装饰的类和装饰的类继承同一个类
- 装饰的类的被装饰的对象一定是外部传入的，一般是构造函数传入
- 功能的拓展，比继承更为灵活








- android.content.ContextWrapper

```java
//装饰类 ContextWrapper 继承 Context
public class ContextWrapper extends Context {
    //被装饰的类继承 Context，这里是 Context 本身
    @UnsupportedAppUsage
    Context mBase;
	//被装饰的类作为参数通过构造函数传入
    public ContextWrapper(Context base) {
        mBase = base;
    }
  }
```

