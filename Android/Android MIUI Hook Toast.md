## 查看分析源码寻找合适的 Hook 点

如果直接 Hook Toast#mNextView 或者 Hook Toast#mTN 针对每个 Toast 的实例都需要处理一次，太麻烦了

，尽可能的找到最合适的点



 Hook Toast#sService

- sService 是静态变量，可以实现只需要处理一次


```java
//先调用一次 Toast#getService 让静态变量 Toast#sService 先初始化
//从 Toast 反射获取静态变量 sService
Method getServiceMethod = Toast.class.getDeclaredMethod("getService", null);
getServiceMethod.setAccessible(true);

//invoke 执行 getService 有返回 INotificationManager
Object service = getServiceMethod.invoke(null);
//打印 android.app.INotificationManager$Stub$Proxy@10c594e
Log.e(TAG, "onCreate service: " + service);

//或者直接从 Toast 中取 sService 静态变量（需要先初始化再获取）
Field sServiceField = Toast.class.getDeclaredField("sService");
sServiceField.setAccessible(true);
//因为取静态变量所以可以传 null
Object sService = sServiceField.get(null);
//打印 android.app.INotificationManager$Stub$Proxy@10c594e
Log.e(TAG, "onCreate sService: " + sService);
//生成代理类
Object sServiceProxy = Proxy.newProxyInstance(TextView.class.getClassLoader(),
   sService.getClass().getInterfaces(), new InvocationHandler() {
   @Override
   public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
       //Android 11 service.enqueueToast(pkg, mToken, tn, mDuration, displayId)
       Log.e(TAG, "invoke: method " + method);
       if ("enqueueToast".equals(method.getName())) {
       Log.d(TAG, "enqueueToast");
       //android.widget.Toast.TN 类名在不同版本是否一致？
       Class<?> tnClass = Class.forName(Toast.class.getName() + "$TN");
       Field mNextViewField = tnClass.getDeclaredField("mNextView");
       mNextViewField.setAccessible(true);
       //不同版本 TN 参数位置不同
       Object tn = null;
       for (int i = 0; i < args.length; i++) {
         Object arg = args[i];
         Log.e(TAG, i + "invoke: arg " + arg);
         if ((Toast.class.getName() + "$TN").equals(arg.getClass().getName())) {
         tn = arg;
         Log.e(TAG, i + "invoke: tn " + tn);
         }
       }
       if (tn != null) {
       //从 tn 对象中获取 mNextView 对象
       Object mNextViewObject = mNextViewField.get(tn);
       Log.e(TAG, "invoke: mNextView " + mNextViewObject);
       if (mNextViewObject instanceof LinearLayout) {
        	LinearLayout mNextView = (LinearLayout) mNextViewObject;
            for (int i = 0; i < mNextView.getChildCount(); i++) {
            View view = mNextView.getChildAt(i);
            if (view instanceof TextView) {
            TextView textView = (TextView) view;
            String toastText = textView.getText().toString();
            String appName = textView.getContext().getString(R.string.app_name);
            if (toastText.contains(appName)) {
            //空格 中文冒号 英文冒号
            toastText = toastText.replace(appName + " ", "")
                                 .replace(appName + "：", "")
                                 .replace(appName + ":", "")
                                 //最短的放后面
                                 .replace(appName, "");
//                      toastText = toastText.substring(appName.length() + 1);
                        textView.setText(toastText);
                                                }
                                            }
                                        }
                                    }
                                }
                            }
       
    //调用 sService 本身的方法
    return method.invoke(sService, args);
	//return null;
  }
});

//
//Toast#sService 是静态对象所以可以为 null
sServiceField.set(null, sServiceProxy);
```

