
# Java 修饰符

## 访问控制修饰符

- public > protected > default > private
- private 只能被当前类自身所访问，default 同一包中的类可访问，protected 同一包中的类和其他包中的子类可访问，public 所有包中的类都可见


1 public 公共访问
- 类、方法/构造方法、成员变量

2 protected 受保护访问
- 方法/构造方法、成员变量

3 (default) 默认缺省访问，不写 default 关键字
- 类、方法/构造方法、成员变量、代码块

4 private 私有访问
- 方法/构造方法、成员变量



## 非访问控制修饰符

1 static 静态的 
- 类(内部类)、方法、成员变量、代码块

2 final 不可改变的
- 类、方法、成员变量、局部变量

3 abstract 抽象的
- 类、方法

4 synchronized 同步的
- 方法、代码块

5 native 本地的，用于 JNI
- 方法