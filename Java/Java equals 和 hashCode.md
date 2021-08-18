# equals 和 hashCode



## ==

- Java 的操作符

- 对于基本数据类型 == 判断两边的值是否相等

- 对于引用类型来说 == 判断是两边的引用是否相等，即是否指向同一块内存区域



## Object#equals

```java
 	/**
     * Indicates whether some other object is "equal to" this one.
     * <p>
     * The {@code equals} method implements an equivalence relation
     * on non-null object references:
     * <ul>
     * <li>It is <i>reflexive</i>: for any non-null reference value
     *     {@code x}, {@code x.equals(x)} should return
     *     {@code true}.
     * <li>It is <i>symmetric</i>: for any non-null reference values
     *     {@code x} and {@code y}, {@code x.equals(y)}
     *     should return {@code true} if and only if
     *     {@code y.equals(x)} returns {@code true}.
     * <li>It is <i>transitive</i>: for any non-null reference values
     *     {@code x}, {@code y}, and {@code z}, if
     *     {@code x.equals(y)} returns {@code true} and
     *     {@code y.equals(z)} returns {@code true}, then
     *     {@code x.equals(z)} should return {@code true}.
     * <li>It is <i>consistent</i>: for any non-null reference values
     *     {@code x} and {@code y}, multiple invocations of
     *     {@code x.equals(y)} consistently return {@code true}
     *     or consistently return {@code false}, provided no
     *     information used in {@code equals} comparisons on the
     *     objects is modified.
     * <li>For any non-null reference value {@code x},
     *     {@code x.equals(null)} should return {@code false}.
     * </ul>
     * <p>
     * The {@code equals} method for class {@code Object} implements
     * the most discriminating possible equivalence relation on objects;
     * that is, for any non-null reference values {@code x} and
     * {@code y}, this method returns {@code true} if and only
     * if {@code x} and {@code y} refer to the same object
     * ({@code x == y} has the value {@code true}).
     * <p>
     * Note that it is generally necessary to override the {@code hashCode}
     * method whenever this method is overridden, so as to maintain the
     * general contract for the {@code hashCode} method, which states
     * that equal objects must have equal hash codes.
     *
     * @param   obj   the reference object with which to compare.
     * @return  {@code true} if this object is the same as the obj
     *          argument; {@code false} otherwise.
     * @see     #hashCode()
     * @see     java.util.HashMap
     */
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

- Object 的 equals 就是用 == 判断的





## equals 方法

一般情况下，equals 方法被赋予的意义是比较两个对象的值是否相等，所以应该按需去重写 equals 方法

引用类型在做比较时需要重写 equals 方法，而 String、Integer 等类默认已经重写好了 equals 方法，所以可以直接使用



### equals 方法重写原则

引用类型前置条件用 instanceof 先判断传入的 Object 是不是当前类型，如果是继续比较，否则直接返回 false，继续判断时，基本数据类型可以直接使用 == 比较，引用类型推荐用 Objects#equals (自带判空逻辑) 进行比较





### equals 方法条件

- 假设 x y z 不为 null，x.equals(null) 返回 false

Reflexive 自反性   x.equals(x) 必须为 true

Symmetric 对称性  如果 x.equals(y) 为 true，那么 y.equals(x) 也必须为 true

Transitive 传递性    如果 x.equals(y) 为 true，y.equals(z)  也为 true，那么 x.equals(z) 也必须为 true

Consistent 一致性  只要 x 和 y 状态不变，则 x.equals(y) 总是一致地返回 true 或 false

 







## hashCode 方法

需要一并重写hashCode方法
