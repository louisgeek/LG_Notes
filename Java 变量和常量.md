# Java 变量和常量

# 变量

## 1 成员变量（全局变量），又叫类的属性
- 定义在类中，方法/构造方法、代码块以外

### Instance Variables 实例变量
- 不用 static 关键字声明
- 可用访问修饰符
- 具有默认值
- 存在堆内存中

### Class Variables 类变量（静态变量）

- 用 static 关键字声明
- 可用访问修饰符
- 具有默认值
- 存在方法区（共享数据区）中

## 2 Local Variables 局部变量

- 定义在方法/构造方法、代码块中
- 不能用访问修饰符
- 没有默认值，所以声明后必须经过初始化才能使用
- 存在栈内存中




# 常量

## 1 直接(字面)常量，又称常量值

```
1,11,0,-88,0125,-013,0x100,-0x16,697L,1.75e5,4.3,-2.3,69.7F,'a','测',"a","abc","测试",true,false,null
```


## 2 符号常量

- final 关键字声明的(变量)，被赋值后就不能改变了

```java
public class Test {
    //成员常量
    final int value = 1;
    //静态常量
    static final double PI = 3.14;
    public void method() {
        //局部常量
        final String localValue = "test";
    }
}
```

### 成员常量

### 局部常量

### 静态常量


总的来说常量可以看成是 final 修饰的变量
