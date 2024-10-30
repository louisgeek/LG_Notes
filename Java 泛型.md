泛型的作用与定义
通配符与嵌套
泛型上下边界
RxJava中泛型的使用分析



用 List 举例



## List，List<?>，List<Object>

List

- 原生类型、原始类型

List<?>

- 通配符类型

List<Object>

- 参数化类型，实参是 Object



```java
		List list = new ArrayList();
        List<?> arrayList = new ArrayList();
        List<Object> objList = new ArrayList();
        List<String> strList = new ArrayList();
        List<Integer> intList = new ArrayList();
		//================== List ================
        //List 接收
        list = arrayList;
        list = objList;
        list = strList;
        list = intList;
        //List add
        list.add(new Object());
        list.add("11");
        list.add(22);
		//================== List<?> ================
		//List<?> 接收
        arrayList = list;
        arrayList = objList;
        arrayList = strList;
        arrayList = intList;
        //List<?> add
        arrayList.add(new Object());//编译报错
        arrayList.add("11");//编译报错
        arrayList.add(22);//编译报错
		//================== List<Object> ================
		//List<Object> 接收
        objList = list;
        objList = arrayList;//编译报错
        objList = strList;//编译报错
        objList = intList;//编译报错
        //List<Object> add
        objList.add(new Object());
        objList.add("11");
        objList.add(22);
```

- List<?> 无法 add
- List<Object> 只能接收 List

- String 是 Object 的子类，但 List<String> 并不是参数化类型 List<Object> 的子类
- 但 List<String> 是原生类型 List 的子类







## 泛型通配符

?





## 有界通配符



Collections#copy

```
//PECS 原则
//src   producer extends
//dest  consumer super
public static <T> void copy(List<? super T> dest, List<? extends T> src){

}
```



### <? extends T>





### <? super T>





## 泛型擦除

 

```java
public  void method(List list) {

}
public  void method(List<Object> list) {

}
public  void method(List<?> list) {

}
public  void method(List<String> list) {

}
```

- 其中任意 2 个同时写即编译报错

