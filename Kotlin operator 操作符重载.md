
# Kotlin operator 操作符重载
- 对已有的运算符重载后赋予它们新的含义（对预定义的操作符进行自定义实现）
- 约定：通过调用特定的函数，来实现特定的语言特性，这种技术称之为约定（实现名为 plus 特殊方法的类，可以在该类的对象上使用 + 运算符），约定的方法都需要使用 operator 关键字修饰



## 1 val pointC = pointA.plus(pointB)

```
class Point(val x: Int,val y: Int) {
    //重载赋予新的含义
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

```
```

    fun test() {
        val point1 = Point(100, 100)
        val point2 = Point(200, 200)
        //
        val point3 = point1.plus(point2)
        //等价于
        val point4 = point1 + point2
 
    }
```

## 2 val pointE = pointD.plus(intValue)

修改 Point 类,假设 x、y 统一改变一个数值

``` 
class Point(val x: Int,val y: Int) {
    //重载赋予新的含义
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
     //
    operator fun plus(xyDis: Int): Point {
        return Point(x + xyDis, y + xyDis)
    }
}

```


```
fun test() {
        val point5 = Point(100, 100)

        val point6 = point5.plus(100)
        //等价于
        val point7 = point5 + 100
 
    }
```

## 3 val pointG = intValue.plus(pointF)

//创建 IntExt.kt 增加如下
```
 operator fun Int.plus(point: Point): Point {
        return Point(this + point.x,this + point.y)
  }
   
```

```
  val point8 = Point(100, 100)

  val point9 = 100.plus(point8)
  //等价于
  val point10 = 100 + point8

```


## 算术运算符

```

a + b	a.plus(b)
a - b	a.minus(b)
a * b	a.times(b)
a / b	a.div(b)
a % b	a.rem(b)
a..b	a.rangeTo(b)

``` 

##  in 操作符

```
a in b	b.contains(a)
``` 


## 索引访问操作符

```
a[i]	  a.get(i)
a[i, j]	  a.get(i, j)

a[i] = b	   a.set(i, b)
a[i, j] = b	   a.set(i, j, b)

``` 

## 调用操作符
```
a()     	a.invoke()
a(i)	    a.invoke(i)
a(i, j)	    a.invoke(i, j)

``` 

## 比较操作符

```
a > b	a.compareTo(b) > 0
a < b	a.compareTo(b) < 0

``` 

```
class Child(val name: String, val age: Int) {
    //根据年龄做比较，年龄越大，返回 Int 正值 
    operator fun compareTo(other: Child): Int {
        return this.age.compareTo(other.age)
    }
}

```


## 属性委托操作符

```
operator fun getValue 

operator fun setValue
```



PS:协程用 + 号组合传值
```kotlin
viewModelScope.launch(CoroutineName("doSomeIO") + Dispatchers.IO) {

}
```
就是因为 CoroutineContext 重载了 plus 操作符

```kotlin
public operator fun plus(context: CoroutineContext): CoroutineContext = {

}
```