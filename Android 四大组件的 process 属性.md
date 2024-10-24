# Android 四大组件的 process 属性
- 在 AndroidManifest.xml 文件中的 <activity>, <service>, <receiver> 或 <provider> 标签中，可以添加 android:process 属性并设置为一个字符串，该字符串表示组件应该运行的进程名
- 如果不指定 android:process 属性情况下，组件默认都运行在应用程序的主进程中，默认进程的名称通常与应用的包名相同
- 通常以冒号开头表示是应用的私有进程


```xml
<activity
    android:name=".MainActivity"
    android:process=":my_main_act">
</activity>

<service
     android:name=".MainService"
     android:process=":my_main_svc">  
 </service>
```

## android:process=":my_main_act"
```shell
packageName=com.louisgeek.louispro processName=com.louisgeek.louispro:my_main_act
```

## android:process=".my_main_act"
```shell
packageName=com.louisgeek.louispro processName=.my_main_act
```

## android:process="my_main_act"
```
运行报错
```

## android:process="" 和不指定 android:process 效果一致

```shell
packageName=com.louisgeek.louispro processName=com.louisgeek.louispro
```

