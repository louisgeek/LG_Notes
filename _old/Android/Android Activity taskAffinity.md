taskAffinity

- 任务栈的名称默认采用该栈中根 Activity 的 taskAffinity，未显式声明 taskAffinity 的 Activity 都采用 Application 的 taskAffinity 值，如果 Application 也没显式声明的话默认就是该应用的包名



当 Activity 启动模式是 standard 或 singleTop 时，显式声明 taskAffinity 是不起作用的，除了 singleTask 和 singleInstance 模式外，FLAG_ACTIVITY_NEW_TASK 也能使 taskAffinity 配置生效



将系统中所有存活中的 Activity 信息打印到控制台

```shell
adb shell dumpsys activity activities
//adb shell dumpsys activity
```

 

