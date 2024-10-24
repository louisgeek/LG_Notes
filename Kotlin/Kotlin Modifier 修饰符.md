
# Kotlin Modifier 修饰符

## Access Modifier 访问控制修饰符（可见性修饰符）

- public > internal > protected > private
- private 只能被当前类自身所访问，protected 当前类或子类可访问，internal 同一模块内的类可访问，public 所有类都可见
- 和 Java 相比，默认的操作符不同，protected 区别比较大

1 public 公共，默认
2 internal 模块
3 protected  受保护
4 private 私有

## 非访问控制修饰符

1 open  可被继承，可被覆盖重写的
2 final 不可被继承，不可被覆盖重写的，默认是 final 的，可以省略
3 abstract 抽象的
4 const 编译时常量
5 external 外部的，用于 JNI

6 enum 枚举类
7 sealed 密闭类、密封类
8 data 数据类
9 annotation 注解类

10 override 重写
11 lateinit 延迟初始化

12 operator 操作符重载
13 infix 中缀
14 inline 内联
15 suspend 挂起

