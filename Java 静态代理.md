# Java 静态代理
- 静态代理指程序运行之前所需要的代理类就已经被创建了
- 通常情况下代理类、目标类都需要实现同一个接口
- 代理类内部维护了目标类的引用，通过给代理类增加功能逻辑实现在不改变目标类的情况下扩展目标类的功能，真正的业务逻辑功能还是由目标类来实现，代理类只是调用目标类的相关方法，符合开闭原则
- 一般需要为每个目标类都创建代理类和接口，导致类的数量大大增加
- 一般情况下接口一旦修改，代理类和目标类都需要修改

## 例子
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
    //...
    System.out.print("登录前逻辑");
    //真正的业务逻辑功能还是由目标类来实现
    mUserLogin.doLogin();
    System.out.print("登录后逻辑");     
    //...
  }
}
```

使用

```java
public class ProxyTest {
	//
	public static void main(String[] args){
     ILogin iLogin = new UserLoginProxy();
     iLogin.doLogin();
	}
}
```




