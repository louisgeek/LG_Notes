##  Android 系统架构

 从下到上分为五层

### 1 Linux Kernel Linux 内核层

Android 系统核心服务基于 Linux 内核

为 Android 设备的各个硬件提供了各种底层专用驱动

内存管理，进程管理，网络协议

- PowerManagement 电源管理
- Drivers
   如 Audio Driver、Camera Driver、USB Driver、Display Driver、Bluetooth Driver、Binder(IPC) Driver 等



### 2 Hareware Abstraction Layer 硬件抽象层 HAL

位于硬件电路和系统内核之间的接口层，Android 5.0 新增，对 Linux 内核驱动程序的封装，向上提供接口，屏蔽底层的实现细节

如 Audio、Camera、Bluetooth 等

- 将硬件抽象化，隐藏各平台硬件接口细节，保护了硬件厂商的知识产权
- 为操作系统提供虚拟硬件平台，使其具有硬件无关性，可实现跨平台移植
   软硬件测试可以基于硬件接口层来完成，让软硬件并行测试成为了可能



### 3 系统运行库层

- Native C/C++ Libraries C/C++ 程序库 
  系统库提供了一系列系统功能，能被 Android 系统中的不同组件所使用，通过应用程序框架可以为开发者提供服务，另外可以通过 Android NDK 为开发者提供了可以直接使用系统资源的能力
  如 SQLite、OpenGL ES、Media Framework、Suface Manager、Webkit、SSL 等

-  Android Runtime Android 运行时

    -  Core Libraries
    
        提供了 Java SE API 的绝大数功能，也提供了 Android 的核心 API，允许开发者用 Java 编写 Android 应用
        
    - ART(以前是 Dalvik Virtual Machine)
    
        使得每个Android 程序拥有一个独立的进程中，都拥有自己的虚拟机实例
    
        完成生命周期管理、堆栈管理、内存管理、垃圾回收
    
        - Dalvik Virtual Machine（Android 5.0 后被 ART 取代）
    
          运行时编译：Dalvik 虚拟机每次运行 字节码 都需要用 Just In Time 及时编译器 JIT 转换成 机器码 
    
        - Android Runtime（Android 4.4 开始发布）
    
          安装时编译：ART 每次安装时会进行一次 Ahead Of Time 预编译 AOT ，将字节码预编译成 机器码 存在本地，做到空间换时间，所以耗费更多存储空间，安装时间拉长
    
    
    
    

### 4 Java API Framework

这一层主要提供一些 Java 编写的 API 以供开发 APP 时调用

如 Activity、ContentProviders、Notification、Window



### 5 System Apps

如 Email、Camera、Brower 等




![Android 系统架构](https://gitee.com/louisgeek/LG_Notes/raw/master/images/android_xitongjiagou.png)
