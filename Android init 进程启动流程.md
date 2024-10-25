### init 进程启动流程

init.rc 配置文件中， AIL 语句中 Import 语句引入  init.${ro.zygote}.rc 文件，如 init.zygote64.rc

```
init.cpp#main
- mount 挂载启动所需的文件目录
- property_init
- signal_handler_init 子进程信号处理函数，会收到进程终止的消息，防止 init 进程的子进程成为僵尸进程
- start_property_service
- parser 解析 init.rc 配置文件
-- AIL 语句中 Service 语句完成 service.cpp#ServiceManager#AddService 加入配置的服务到 vector 
-- AIL 语句中 Action 语句完成 builtins.cpp#ServiceManager 进行 ForEach 遍历得到 Service,然后执行 service.cpp#Start
--- 利用 service.cpp#folk 函数创建 Service 子进程，service.cpp#execve 函数启动该子进程,并会进入到 Service 的 main 函数里，ps：如果 service 是 Zygote 类型，就相当于进入了 Zygote 进程的 main 函数里
```



#### init 进程

- 是系统中用户空间的第一个进程，进程号是 1

- 由多个源文件共同组成

- 入口函数是 main

system/core/init/init.cpp

```c++
int main(int argc,char** argv){
	//创建和挂载启动所需的文件目录
	mount(... ...)
	...
	//对属性服务进行初始化
	property_init();
	...
	//启动属性服务
	start_property_servce();
}
```

#### 僵尸进程

- 系统进程表是一项有限的资源，如耗尽则无法创建新的进程

- 父进程利用 fork 创建子线程，如子线程终止，父进程未知晓，系统进程表里还保留这该子进程的信息，该子进程就是僵尸进程

main 函数中有调用个 signal_handler_init 函数用于监听子进程终止的消息，以防出现僵尸进程

 

### init.rc 文件

是一种使用 Android Init Language 编写的脚本

system/core/rootdir/init.zygote64.rc 

- Zygote 的启动脚本

```
service zygote /system/bin/app_process64 ...
	class main ... 
	... 
```

service 是一个关键字，后面跟服务名称
class 就是 service 的 <option>

其中 class main 就是指 class 的 classname 是 main

```
service <name> <pathname> ...
	<option> ... 
	... 
```

所以 Zygote 是一种 service 进程

### 解析 init.rc 语句

AIL  五种类型语句

- Action
ActionParser

- Command

- Service
ServiceParser 服务解析器
根据参数创建 Service 对象
填充 Service 内容
ServiceManager 将 Service 对象 add 到 vector 类型的 service 链表中

- Option

- Import

  用于导入其他 rc 文件的配置

  如 init.rc 文件导入 init.zygote64.rc

  ```
   import /init.${ro.zygote}.rc
  ```

  

### init 进程启动 Zygote 



读取 rc 文件中的 Action 脚本

system/core/rootdir/init.rc

```
...
on nonencrypted
   ...
   class_start main
...
```

其中 class_start  就是 on 的 <command> ，对应 do_class_start 函数

do_class_start 函数里面就是遍历之前 ServiceManager 的 service 链表，取到 name 是 main 的服务，而这个服务就是 Zygote
判断服务是否禁用（init.zygote64.rc 里面定义），如不是则调用 Start 函数启动该服务
而 Start 函数里会判断服务是否已启动，是则 return false
如未启动则继续判断启动的执行文件是否存在，如不存在则 return false
有则继续利用 fork 函数创建子进程，其会有一个返回值 pid 
判断 pid 等于 0 的条件，代表代码逻辑在子进程中运行
再调用 execve 函数就启动了 Service 进程，并进入到 Service 的 main 函数
如果此时 Service 是 Zygote 服务，那么就是进入到  Zygote 的 main 函数
main 函数里调用 runtime（AppRuntime）的 start 函数完成 Zygote 服务的启动

 

### 属性服务

- 类似 Windows 系统里的注册表，它利用键值对记录用户、系统的软件信息
- 系统重启后仍旧能找到对应的信息进行工作的机制

上文提到 init 进程会走 property_init 初始化属性服务，然后 start_property_service 启动属性服务

#### start_property_service

##### property_set

```
property_set(“ro.property_service.version”,"2")
```

然后利用 create_socket 函数创建一个非阻塞的 Socket 返回 property_set_fd

##### listen

通过 listen 函数监听这个 property_set_fd，所以这个 Socket 是一个 server 型的 Socket
listen 函数第二个参数传入 8 代表最多同时为 8 个用户提供设置属性的服务

##### register_epllo_handler

```
//涉及 Linux 内核的 select/poll 机制，多路复用 I/O 接口
//新的 Linux 内核采用 epoll，select/poll 的增强版本
register_epoll_handler(property_set_fd,handle_property_set_fd)
```

register_epllo_handler 函数将 property_set_fd 放入 epoll 中来监听，当 property_set_fd 有数据到来时，
init 进程将会调用 handle_property_set_fd 函数进行处理

#### handle_property_set_fd 函数

新版本的代码是利用一个 handle_property_set 函数处理属性

##### handle_property_set 函数

里面开始判断属性，分为两种

- 控制属性

  名称以 ctl. 开头，如开机动画
  判断权限满足就继续利用 handle_control_message 函数处理

- 普通属性
  处理控制属性，其他都是普通属性

  利用 property_set 函数处理，先校验属性合法性，然后有则 _system_property_update 更新，无则 _system_property_add 添加，最后都走 property_change 函数

  另外针对 ro. 开头的是只读的，无法更新直接返回

  针对 persist.开头做了附加的处理

  



### 总结

init 进程

- Android 系统用户空间的第一个进程

- 创建和挂载启动所需的文件目录

- 初始化和启动属性服务

- 解析 init.rc 配置文件，并启动 Zygote 进程（ init.zygote64.rc 系列配置文件）

