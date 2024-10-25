## Fragment 生命周期

- https://developer.android.google.cn/guide/fragments/lifecycle

- 11个

```
onAttach —— onCreate —— onCreateView —— onActivityCreated —— onStart —— onResume 
—— onPause —— onStop —— onDestroyView —— onDestroy —— onDetach
```



![Fragment生命周期](http://upload-images.jianshu.io/upload_images/1477714-fddcd4bf7894d26d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2 经典场景
场景1：第一次启动 A
```
A：onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume
```

场景2：在 A 上启动 B
```
A：onPause -> onStop -> onDestroyView
B：onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume
```
场景3：从 B 返回 A
```
A：onCreateView -> onActivityCreated -> onStart -> onResume
B：onPause -> onStop -> onDestroyView -> onDestroy -> onDetach
```
场景4：锁屏与解锁

```
//锁屏
onPause -> onStop
//解锁
onStart -> onResume
```
场景5 点击 HOME 键、来电

```
//Home键，来电
onPause -> onStop
//回到应用
onStart -> onResume
```



## 2 Activity 和 Fragment 生命周期关系



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210330234704796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdWlzZ2Vlaw==,size_16,color_FFFFFF,t_70#pic_center)





![关系](http://upload-images.jianshu.io/upload_images/1477714-4cc76d4d6bc25ff0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Activity 开始创建带动 Fragment
- Fragment 开始销毁带动 Activity 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210330104852627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdWlzZ2Vlaw==,size_16,color_FFFFFF,t_70#pic_center)



## 3 碎片状态
1 运行状态
2 暂停状态
3 停止状态
4 销毁状态