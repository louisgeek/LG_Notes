## 内部类（Inner Class）

- 不能定义静态属性、静态方法
- 内部类可以直接访问外部类的属性，即便是 private 修饰的，但是外部类不可以直接访问内部类的元素
- 内部类会自动捕获一个外部类的引用，所以内部类能够访问外部类元素，实际上是通过所持有的外部类引用来访问的

### 成员内部类

```java
public class OuterClass {
    //成员变量Field
    private String outerField ;
    public void outerMethod() {
        System.out.println(outerField);
    }
    //OuterClass$InnerClass
    public class InnerClass {
        private String innerField;
        private void innerMethod() {
            //直接访问属性
            System.out.println(outerField);
            System.out.println(OuterClass.this.outerField);
            //直接访问方法
            outerMethod();
            OuterClass.this.outerMethod();
        }
    }
}
```

实例化内部类

```java
OuterClass outerClass = new OuterClass();
//外部类的实例 outerClass 直接.new
OuterClass.InnerClass innerClass = outerClass.new InnerClass();

//或者
OuterClass.InnerClass innerClass2 = new OuterClass().new InnerClass();
```

### 方法(局部)内部类

```java
public class OuterClass {
    private String outerField;
    public void outerMethod() {
        System.out.println(outerField);
        //LocalVariable 局部变量
        String localField ;
        //方法(局部)内部类定义在外部类的方法中，参考局部变量，不能使用修饰符
        class MethodInnerClass {
            private void methodInnerMethod() {
                System.out.println(outerField);
            }
        }
        MethodInnerClass methodInnerClass = new MethodInnerClass();
        methodInnerClass.methodInnerMethod();
    }
    //OuterClass$InnerClass
    public class InnerClass {
        private String innerField;
        private void innerMethod() {
            System.out.println(outerField);
            System.out.println(OuterClass.this.outerField);
            outerMethod();
            OuterClass.this.outerMethod();
        }
    }
}
```

### Anonymous 匿名内部类

- 其实就是一个没有名字的方法(局部)内部类
- 没有名字所以没有构造方法
- 必须继承一个抽象类或者实现一个接口





## 静态嵌套(内部)类（Static Nested Class）









local class

member class



