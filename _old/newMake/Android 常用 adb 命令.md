启动 app

```shell
adb shell am start -n "com.bsoft.mob.testapp/com.bsoft.mob.testapp.MainActivity" -a android.intent.action.MAIN -c android.intent.category.LAUNCHER
```

```shell
adb shell am start -n "com.bsoft.mob.neb.webview.remote_http.debug/com.bsoft.mob.neb.SplashActivity" -a android.intent.action.MAIN -c android.intent.category.LAUNCHER
```

提取 apk 备份

```shell
adb -s 10.0.4.66  shell pm list packages
//获取包名的地址
adb -s 10.0.4.66 shell pm path com.bsoft.mob.others
//导出 文件 到电脑路径 D:/apks 必须是已存在的目录
adb -s 10.0.4.66  pull /data/app/com.bsoft.mob.others-1/base.apk d:/apks
//ps 用于快速复制
adb shell pm list packages
adb shell pm path com.hikvision.thermal.beta
adb pull /data/app/com.bsoft.mob.others-1/base.apk d:/apks
```

卸载 app

```shell
adb -s 10.0.4.66 shell pm list packages
// -k 保存数据和缓存
adb -s 10.0.4.66 uninstall -k com.louisgeek.testapp
```

root 下卸载 内置 系统 app

```shell
adb root 
//remount successful  //表示成功
adb remount
adb shell pm list package
adb shell pm path com.bsoft.mob.ienr
adb shell rm -rf  xxx.apk 
//重启设备-- 可选
adb shell reboot
//-ps 删除某文件 BBB.xxx 
mount -o remount ,rw /
mount -o remount ,rw /system
rm /AAA/CCC/BBB.xxx
//如果还是提示只读 到目录里执行和删除
cd /AAA/CCC
mount -o remount ,rw /AAA/CCC
rm BBB.xxx
```

查看内存信息

```shell
adb shell cat /proc/meminfo
```

获取 设备屏幕信息

```shell
adb shell dumpsys window displays
```

获取 设备屏幕分辨率

```shell
adb shell wm size
```

获取设备屏幕 density / dpi

```shell
adb shell wm density
```

获取厂商

```shell
adb shell getprop ro.product.manufacturer
```

获取型号

```shell
adb shell getprop ro.product.model
```

获取 cpu 架构  （armeabi-v7a  2011 年起  arm64-v8a  2014 年起 ）

```
adb shell getprop ro.product.cpu.abi
```

获取以太网 mac

```shell
adb shell cat /sys/class/net/eth0/address
```

清除缓存

```shell
adb shell pm clear com.louisgeek.xxx
```

重启设备

```shell
adb -s 11.25.69.45 reboot
```

部分设备出现 failed to authenticate to xxx (待验证)

```shell
adb kill-server
adb connect
```

命令 模拟 android studio 运行 app

```shell
adb push E:\LouisWork\Louis\LouisAppBackup\app\build\outputs\apk\debug\app-debug.apk /data/local/tmp/com.louisgeek.appbackup
//
adb shell pm install -t -r "/data/local/tmp/com.louisgeek.appbackup"
```

记录 logcat 日志到 文件

```shell
//存放在启动 adb 的目录里
adb logcat > logs.txt
```

导出日志报告

```shell
adb bugreport D:\bugs
```
