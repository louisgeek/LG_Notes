Launcher 

- 启动器，是一个应用程序，可以供用户启动 APP
- 安卓系统的桌面，可以显示 APP 快捷图标和桌面组件
- Launcher 的入口是 AMS 的 systemReady 方法



SystemServer#startBootstrapServices

- 完成了启动 AMS 和 PackageManagerService

SystemServer#startOtherServices

- ActivityManagerService#systemReady 传入一个 Callback
- ActivityStackSupervisor#resumeFocusedStackTopActivityLocked
- ActivityStack#resumeTopActivityUncheckedLocked
- ActivityStack#resumeTopActivityInnerLocked
- ActivityStackSupervisor#resumeHomeStackTask
- ActivityManagerService#startHomeActivityLocked
- ActivityStarter#startHomeActivityLocked
- mSupervisor.moveHomeStackTaskToTop







Launcher 中图标的显示过程

Launcher#onCreate

```
LauncherAppState app = LauncherAppState.getInstance();
...
mModel = app.setLauncher(this);
...
mModel.startLoader(xxx)
```



- LauncherAppState#setLauncher(this)

- ```java
  LauncherModel setLauncher(Launcher launcher){
  ...
  mModel.initialize(launcher); //传入 launcher
  return mModel;
  }
  ```

- LauncherModel#initialize

  ```
  public void initialize(Callbacks callbacks){
  mCallbacks  = new WeakReference<Callbacks>(callbacks);//封装成 WeakReference
  }
  ```

  

  LauncherModel#startLoader

  ```
  mLoaderTask = new LoaderTask(mAPP,isLaunching);// LoaderTask 实现了 Runnable
  mWorker.post(mLoaderTask);// mWorker 是一个 Handler
  ```

- LauncherModel$LoaderTask#run

  - LoaderTask#loadWorkSpace
  - LoaderTask#bindWorkSpace
  - LauncherModel#loadAllApps

LauncherModel#loadAllApps

- callbacks.bindAllApplications(added)

  

  Launcher#bindAllApplications

  - AllAppsContainerView#setApps 把 List<AppInfo> 类型的 apps 设置到 AlphabeticalAppsList 上
  - AllAppsContainerView#onFinishInflate 

  - 内部 findViewById 找到一个 apps_list_view 的 AllAppsRecyclerView
    - 然后填充数据、setAdapter 把所有图标显示到列表上

  
