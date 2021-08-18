> 文中讨论的知识基于 Windows cmd  命令提示符在 `cmd /e:on` 命令扩展启用的情况下（默认是开启的 ，可以通过 `cmd /e:off` 关闭）

# CD

CHDIR 可以简写成 CD，全名 Change Directory ，有切换功能和显示功能

# 切换功能

`cd .` 切换到当前目录

```shell
C:\Users\louisgeek>cd .
C:\Users\louisgeek>
```

`cd ..` 切换到上一级目录(父目录)

```shell
C:\Users\louisgeek>cd ..
C:\Users>
```

`cd \` 切换到根目录 ，路径里的`\` 和 `/`好像可以混用？

```shell
C:\Users\louisgeek>cd \
C:\>
```

```shell
C:\Users\louisgeek>cd /
C:\>
```

`cd ..\..` 切换到上上级目录(父目录的父目录)

```shell
C:\Users\louisgeek\Desktop>cd ../..
C:\Users>
```

```shell
C:\Users\louisgeek\Desktop>cd ../../
C:\Users>
```

`cd <path> ` 切换到指定目录

```shell
//不区分大小写 会变成存在目录的实际大小写
C:\Users\louisgeek>cd desktop
C:\Users\louisgeek\Desktop>
```

`cd /d drive: ` 切换磁盘驱动器

```shell
C:\Users\louisgeek>cd /d d:
D:\>
```

`cd /d <drive>:<path>` 切换磁盘驱动器及指定目录

```shell
C:\Users\louisgeek\Desktop>cd /d d:hk
D:\HK>
```

# 显示功能

`cd drive:` 显示指定磁盘驱动器的当前目录

```shell
C:\Users\louisgeek\Desktop>cd d:
D:\
C:\Users\louisgeek\Desktop>cd c:
C:\Users\louisgeek\Desktop
```

`cd ` 不指定磁盘驱动器，直接显示当前磁盘驱动器的当前目录

```shell
C:\Users\louisgeek\Desktop>cd
C:\Users\louisgeek\Desktop
```



附 

`drive:` 切换磁盘驱动器

```shell
C:\Users\louisgeek>d:
D:\>
```

-  `cd <drive>:<path>`和`<drive>:` 搭配操作可以达到 `cd /d <drive>:<path>` 的效果，两者执行先后顺序可以交换

  



