## 进程

操作系统结构的基础

系统资源分配、调度和管理的基本单元

计算机中一个任务就是一个进程，指一个程序或者应用

程序的实体，一个程序至少包含一个进程

是线程的容器，一个进程至少包含一个线程

进程间相对独立，有自己独立的地址空间







## 线程

线程调度由操作系统控制

系统调度的最小单元

进程中子任务，可以看成轻量级的进程





### 线程的状态

- new 新建

线程被创建，调用start方法之前

- runnable 可运行

调用start方法之后

* blocked 阻塞

被锁阻塞

* waiting 等待

不活动，等待调度器

* timed waiting 超时等待

指定时间可自行返回的等待

* terminated 终止

线程完成或异常终止



![线程状态](https://gitee.com/louisgeek/LG_Notes/raw/master/images/xiancheng_zhuangtai.png)