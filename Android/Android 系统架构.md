 Android 系统架构

 自下而上分为五层

1 Linux Kernel Linux 内核层
Android 核心服务基于 Linux 内核
添加了 Android 专用驱动
 -- PowerManagement
 -- Drivers
 如 Audio、Camera、USB 等



2 Hareware Abstraction Layer 硬件抽象层 HAL
位于硬件电路和系统内核之间的接口层
 将硬件抽象化，隐藏各平台硬件接口细节，保护了硬件厂商的知识产权
 为操作系统提供虚拟硬件平台，使其具有硬件无关性，可实现跨平台移植
 软硬件测试可以基于硬件接口层来完成，让软硬件并行测试成为了可能
如 Audio、Camera、Bluetooth 等



3 系统运行库层
-- Native C/C++ Libraries C/C++ 程序库 
能被 Android 系统中不同组件所使用
能通过应用程序框架为开发者提供服务
如 SQLite、OpenGL ES、Media Framework 等

-  Android Runtime Android 运行时

    - ART
        Dalvik 每次运行 字节码 都需要用Just In Tim 及时编译器 JIT 转换成 机器码 
        ART 每次安装时会进行一次 Ahead Of Time 预编译 AOT ，将字节码预编译成 机器码存在本地，空间换时间
    - Core Libraries

    

4 Java API Framework
如 Activity、ContentProviders、Notification
Java 编写的 API 供开发应用程序时调用



5 System Apps
如 Email、Camera、Brower 等

