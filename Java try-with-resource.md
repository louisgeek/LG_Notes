try-with-resouce写法是在JDK1.7引入的一个语法糖，用来进行io数据流关闭的简易写法







数据库操作、IO操作等需要使用结束close()的对象必须在try-catch-finally 的finally中close()，如果有多个IO对象需要close()，需要分别对每个对象的close()方法进行try-catch,防止一个IO对象关闭失败其他IO对象都未关闭。
Android Studio 3.0已经支持任意版本的try-with-resource语法，对于AutoCloseable对象，可以使用该语法简化操作





Kotlin中有没有类似的语法呢？实际上也是有的，有一个内联的扩展方法，use，它就是来做相同的事情的。



kotlin.io.CloseableKt#use

