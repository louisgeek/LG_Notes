# Android Studio 配置和技巧

1 用 xmlns:tools=”http://schemas.android.com/tools”命名空间

tools:text=”XXX”来预览文字效果，发布不影响APP，也方便阅读



2 Logcat 打印输出颜色分明设置 
打开Setting->Editor->Colors & Fonts->Android Logcat 
选新Logcat风格修改Foregound颜色（我AS是2.0 p4要取消勾选 Use inherited attributes再设置） 
分别选一个分类的信息修改对应的颜色 建议颜色值

```xml
V BBBBBB
D 0070BB
I 48BB31
W BBBB23
E FF0006
A 8F0005
```





3 自动导包 
打开Setting->Editor->General->Auto Import 
勾选Java下3个复选框



4 代码提示不区分大小写 
打开Setting->Editor->General->CodeCompletion 
选择case sensitive completion下拉的none



5 显示行号 
打开Setting->Editor->General->Appearance 
勾选show line numbers



6 提示api用法 悬浮框 
Ctrl+Q  



7 提示方法参数
Ctrl+P



8 cannot run program “git.exe”…

打开Settings -> Version Control -> Git 之后，在选项 “Path to Git Executable” 你可以看到 “git.exe” , 填入当前安装git的安装的目录的\bin\git.exe，后面可以用test按钮测试下是否可用



9 变量名 带m前缀 静态变量s前缀
在 Settings - Editor - Code Style - Java 的最后一个 Tab ：Code Generation 里有个 Naming 表格，其中 Field 的 Name prefix 列填入 m 就可以让 member 变量以 m 开头时自动提示了
Static Filed 前缀为:s



10 文件头注释自动生成 模板
打开  Settings -> Editor -> Code Style -> File And Code Templates -> Include -> File Header
填写自己的模板



11 去掉第一次初始化功能

进入刚安装的Android Studio目录下的bin目录。找到idea.properties文件，用文本编辑器打开。 在idea.properties文件末尾添加一行： disable.android.first.run=true



12 所有 Android Studio 版本的下载地址汇总

https://developer.android.google.cn/studio/archive.html#android-studio-3-0?utm_source=androiddevtools&utm_medium=website

