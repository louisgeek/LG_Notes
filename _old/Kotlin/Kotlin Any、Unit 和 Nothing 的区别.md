

## Any

- 所有类的父类都是 Any
- 在运行时，自动映射成 java 的 Object 类（Object 是 Java 中所有引用类型的父类）



## Unit

- 当一个函数没有返回值时，默认返回 Unit

- 类似 Java 的 void ，但 Unit 可访问到，而 void 不能

- Unit 的父类是 Any，Unit? 的父类是 Any?

  

## Nothing

- 没有实例的类型，可被访问到，Nothing? 一直是 null
- 类似 Java 的 Void 
- throw Exception 就相当于一个返回 Nothing 的表达式，结果永远不会返回

