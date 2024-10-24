原理是 **Hook** Android 系统的核心进程 **Zygote** 来达到修改程序运行过程和结果

所有的进程都是直接或者间接地由 **init进程fork出来的**

安装 Xposed Installer 之后，系统app_process将被替换，然后利用Java的Reflection机制覆写内置方法，实现功能劫持



支持 ART虚拟机









Zygote进程加载XposedBridge.jar，将所有需要替换的Method通过JNI方法hookMethodNative指向Native方法xposedCallHandler，这个方法再通过调用handleHookedMethod这个Java方法来调用被劫持的方法转入Hook逻辑。对方法的Hook和Replace，配合反射技术，修改目标代码逻辑





银行APP检测



