Intent

- Bundle



文件共享



Messenger



AIDL



ContentProvider





Socket













Binder

- 进程间通信机制，也是一个驱动

- Binder.java 继承 Ibinder接口，实现跨进程能力



有很多现成的 IPC 通信方式，为啥 Android 自己又设计了一套 Binder 机制？

效率

稳定性

安全性