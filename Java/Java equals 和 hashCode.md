## equals 方法



引用类型在作比较时需要覆写 equals 方法，String、Integer 等类 Java 已经覆写好了 equals 方法，所以可以直接用



### equals 方法覆写原则

引用类型实际相等的逻辑

用 instanceof 先判断传入的 Object 是不是当前类型，如果是继续比较，否则返回 false

判断时，基本数据类型可以直接用 == 比较，引用类型推荐用 Objects.equals() 进行比较，自带判空逻辑





### equals 方法条件

- 假设 x y z 不为 null，x.equals(null) 返回 false

Reflexive 自反性   x.equals(x) 必须为 true

Symmetric 对称性  如果 x.equals(y) 为 true，那么 y.equals(x) 也必须为 true

Transitive 传递性    如果 x.equals(y) 为 true，y.equals(z)  也为 true，那么 x.equals(z) 也必须为 true

Consistent 一致性  只要 x 和 y 状态不变，则 x.equals(y) 总是一致地返回 true 或 false

 



## hashCode 方法

