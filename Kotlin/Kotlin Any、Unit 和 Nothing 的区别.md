
# Kotlin Any、Unit 和 Nothing 的区别

## Any
- Any 是一个 open 的类
- 所有不可空的类的顶级父类都是 Any，所有可空的类的顶级父类都是 Any?，Any? 是 Any 的父类（或者说 Any? 等价于 Any 和 null 的合集）
- Any 和 Any? 都相当于 Java 中的 Object 类（Object 是 Java 中所有引用类型的父类）


## Unit
- Unit 是一个 object 单例类
- 当一个函数没有写返回值的时候，默认返回 Unit
- Unit 感觉有点类似 Java 中的 void，但 Unit 可被访问到，而 void 不能
- Unit 的父类是 Any,而 Unit? 的父类是 Any?


## Nothing
- Nothing 是一个构造函数是私有的类
- Nothing 可被访问到，相当于 Java 中的 Void 类（而 void 不可被访问到），但结果永远不会返回，比如抛出异常
- Nothing 可以代替原方法的返回类型
- Nothing? 唯一允许的值是 null



kotlin.Any::class == java.lang.Object::class 成立
kotlin.Nothing::class == java.lang.Void::class 成立



常用的实现接口时候默认生成的 TODO("Not yet implemented")，方法返回值写的是 Nothing，抛出一个异常

```kotlin
/**
 * Always throws [NotImplementedError] stating that operation is not implemented.
 */

@kotlin.internal.InlineOnly
public inline fun TODO(): Nothing = throw NotImplementedError()

/**
 * Always throws [NotImplementedError] stating that operation is not implemented.
 *
 * @param reason a string explaining why the implementation is missing.
 */
@kotlin.internal.InlineOnly
public inline fun TODO(reason: String): Nothing = throw NotImplementedError("An operation is not implemented: $reason")

```

```kotlin
//看这里，按上面说的 testAbstract 没写返回值，那么默认返回 Unit，但是方法里又写了 TODO，它的返回值是 Nothing，那是不是相当于此时 Nothing 替换了 Unit
 override fun testAbstract(test: String) {
        TODO("Not yet implemented")
    }
```