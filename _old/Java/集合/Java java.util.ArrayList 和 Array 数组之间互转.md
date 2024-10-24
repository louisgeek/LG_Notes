# java.util.ArrayList 和 Array 数组之间互转



## java.util.ArrayList -> Array 数组

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
//List 直接转成数组
String[] array = list.toArray(new String[list.size()]);
```



## Array 数组 -> java.util.ArrayList 

```java
String[] array = new String[] {"1", "2", "3","4"};
//注意这里返回的是 java.util.Arrays.ArrayList 而不是 java.util.ArrayList
//可以调 set，get 等，但不能调 add，remove 等，因为没有实现会报 UnsupportedOperationException
List<String> listTemp = Arrays.asList(array);
List<String> list = new ArrayList<>(listTemp);
```



