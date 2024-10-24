# 函数式编程之 Functional Composition 函数组合

```kotlin
    private fun isOdd(x: Int) = x % 2 != 0 //不能被2整除
    private fun getLen(str: String) = str.length
    private fun <A, B, C> compose(odd: (B) -> C, getL: (A) -> B): (A) -> C {
        //return { a -> odd.invoke(getL.invoke(a)) }
        return { a -> odd(getL(a)) }
    }

    private fun testCompose() {
        //从右往左执行函数的顺序
        val odd = compose(::isOdd, ::getLen)
        val strList = listOf("a", "ab", "abc")
        //结果：testCompose: [a, abc]
        Log.e(TAG, "testCompose: ${strList.filter(odd)}")
    }
```