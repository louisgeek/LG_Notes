## 静态代理

- 程序运行之前代理类就已经被创建了

- 代理类、目标类都需要实现同一个接口

- 代理类内部维护目标类的引用，通过给代理类增加功能逻辑实现在不改变目标类的情况下扩展目标类的功能，真正的业务逻辑功能还是由目标类来实现，代理类只是调用目标类的相关方法，符合开闭原则

  

ILogin 接口

```java
public interface ILogin {
    void doLogin();
}
```

UserLogin 目标类

```java
public class UserLogin implements ILogin {

  @Override
  public void doLogin(){
      System.out.print("用户登录逻辑");
  }   
}
```

UserLoginProxy 代理类

```java
public class UserLoginProxy implements ILogin {
  private UserLogin mUserLogin;
  public UserLoginProxy() {
    mUserLogin = new UserLogin();
  }
  @Override
  public void doLogin(){
      ...
    System.out.print("登录前逻辑");
    mUserLogin.doLogin();
    System.out.print("登录后逻辑");     
      ...
  }
}
```

客户端

```java
public class ProxyTest {
	//
	public static void main(String[] args){
    	 ILogin iLogin = new UserLoginProxy();
         iLogin.doLogin();
	}
}
```

- 需要为每个目标类都创建代理类和接口，导致类的数量大大增加
- 接口一旦修改，代理类和目标类都得修改





## 动态代理

- 在程序运行时通过反射机制动态得创建代理对象



### JDK 动态代理

- 使用 Proxy#newInstance 来创建指定接口的代理实现类

- 通过`Proxy.newProxyInstance`产生的代理类，当调用接口的任何方法时，都会调用`InvocationHandler#invoke`方法，在这个方法中可以拿到传入的参数，注解等

- 只能实现基于接口的动态代理，因为 Java 时单继承的，它动态生成 $ProxyX 代理类已经继承 Proxy 类了

  并有一个带 InvocationHandler 参数的构造方法，使用 super 调用父类 Proxy 类的构造方法

```java
private InvocationHandler mInvocationHandler = new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return method.invoke(xxx,args);
        }
    };
//
Class<?> clazz = Proxy.getProxyClass(Toast.class.getClassLoader(), Toast.class);
Toast toastProxy = (Toast) clazz.getConstructor(InvocationHandler.class)
                    .newInstance(mInvocationHandler);
//
Toast toastProxy2 = (Toast) Proxy.newProxyInstance(Toast.class.getClassLoader(),
                    new Class<?>[]{Toast.class}, mInvocationHandler);
```



- 减少类的数量，降低工作量
- 减少对业务接口的依赖，降低耦合，便于后期维护
- 使用反射会在运行时会消耗一定的性能







### GCLIB

