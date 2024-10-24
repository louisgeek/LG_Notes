# Kotlin 集合转换操作


## map
- 将数组类（如list, set, array等）中每个元素应用转换成新的元素添加到一个新的 List 中,最后返回这个新的 List

```
//Array 的扩展函数
public inline fun <T, R> Array<out T>.map(transform: (T) -> R): List<R> {
    return mapTo(ArrayList<R>(size), transform)
}
//new 出 ArrayList，将集合里所有转换后的元素 add 进去
public inline fun <T, R, C : MutableCollection<in R>> Array<out T>.mapTo(destination: C, transform: (T) -> R): C {
    for (item in this)
        destination.add(transform(item))
    return destination
}
```


## flatMap
- flatMap 的表现为 map （以集合作为映射结果）与 flatten 的连续调用

```
//Array 的扩展函数
public inline fun <T, R> Array<out T>.flatMap(transform: (T) -> Iterable<R>): List<R> {
    return flatMapTo(ArrayList<R>(), transform)
}
//new 出 ArrayList，将集合里所有转换后的元素的集合 addAdd 进去
public inline fun <T, R, C : MutableCollection<in R>> Array<out T>.flatMapTo(destination: C, transform: (T) -> Iterable<R>): C {
    for (element in this) {
        val list = transform(element)
        destination.addAll(list)
    }
    return destination
}
```