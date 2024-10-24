## Android ClassLoader

- 用于加载 dex 文件

### 系统类加载器

BootClassLoader 

- Java 实现的类加载器
- 用来预加载常用类，它在 zygote 进程里的 preload 方法中创建
- 应用程序不能直接调用该加载器



DexClassLoader 扩展类加载器

- 用来加载 dex 文件和包含 dex 的 jar、apk 文件



PathClassLoader 应用程序类加载器

- 用来加载系统类和应用程序类
- 只能加载安装后的 apk 的 dex 文件
- 它在 zygote 进程里的 startSystemServer 方法中创建



### 自定义类加载器

Custom ClassLoader
