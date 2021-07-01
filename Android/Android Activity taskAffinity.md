任务栈的名称默认采用 该栈中根 Activity 的 taskAffinity ，未显示声明 taskAffinity 的 Activity 都采用 Application 的 taskAffinity 值，如果也没显示声明的话默认就是该应用的包名



当启动模式是 standard 或 singleTop 时，显示声明 taskAffinity 是无效的，会跟随源 Activity 

除了 singleTask 和 singleInstance 模式外，FLAG_ACTIVITY_NEW_TASK 也能令 taskAffinity 生效

 adb shell dumpsys activity 