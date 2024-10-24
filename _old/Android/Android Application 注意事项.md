# 在 Application 的业务初始化代码加入进程判断，确保只在自己需要的进程初始化。特别是后台进程减少不必要的业务初始化。

在 Application 的业务初始化代码加入进程判断，确保只在自己需要的进程
初始化。特别是后台进程减少不必要的业务初始化。
正例：
public class MyApplication extends Application {
@Override
public void onCreate() {
//在所有进程中初始化
....
//仅在主进程中初始化
if (mainProcess) {
...
}
//仅在后台进程中初始化
if (bgProcess) {
...
}
}
}