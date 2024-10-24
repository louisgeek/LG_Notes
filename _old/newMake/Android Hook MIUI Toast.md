## Hook MIUI Toast

新版 MIUI （测试版本是 MIUI 12.5.7）中，部分 APP（测试 targetSdkVersion 设置成 29及以下 的 APP 会出现）弹出 Toast 的时候提示内容会有应用名加冒号的前缀
[![带应用名加冒号的前缀](http://10.1.65.34/group1/M00/13/A3/CgFBImIZq0CATKB6AABjj8YMl_A753.png "带应用名加冒号的前缀")](http://10.1.65.34/group1/M00/13/A3/CgFBImIZq0CATKB6AABjj8YMl_A753.png "带应用名加冒号的前缀")
## 利用 Hook 之动态代理方式处理去掉前缀

### 分析源码寻找合适的 Hook 点

- 如果直接 Hook Toast#mNextView 或者 Hook Toast#mTN 针对每个 Toast 的实例都需要处理一次，太麻烦了，尽可能的找到最合适的点

### Hook Toast#sService

- sService 是静态变量，可实现只处理一次

```java
 /**
     * 可以在 MyApplication#onCreate() 方法中调用执行
     * @throws Exception
     */
    public static void initFix() throws Exception {
        //从 Toast 反射获取 getService 方法
        Method getServiceMethod = Toast.class.getDeclaredMethod("getService", null);
        getServiceMethod.setAccessible(true);
        //先调用一次 Toast#getService 让静态变量 Toast#sService 先初始化
        //invoke 执行 getService 有返回 INotificationManager，因为是获取静态变量所以传 null
        Object service = getServiceMethod.invoke(null);
        //打印 android.app.INotificationManager$Stub$Proxy@10c594e
        Log.e(TAG, "onCreate service: " + service);

        //再或者直接再通过反射从 Toast 中取 sService 静态变量（需要先初始化再获取）
        Field sServiceField = Toast.class.getDeclaredField("sService");
        sServiceField.setAccessible(true);
        //因为取静态变量所以可以传 null
        Object sService = sServiceField.get(null);
        //打印 android.app.INotificationManager$Stub$Proxy@10c594e
        Log.e(TAG, "onCreate sService: " + sService);
        //以上两种取 service/sService 的方式任选其一，这里选用 sService

        //生成代理类对象
        Object sServiceProxy = Proxy.newProxyInstance(TextView.class.getClassLoader(),
                sService.getClass().getInterfaces(), new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        //Android 11 是 service.enqueueToast(pkg, mToken, tn, mDuration, displayId)
                        Log.e(TAG, "invoke: method " + method);
                        if ("enqueueToast".equals(method.getName())) {
                            Log.d(TAG, "enqueueToast");
                            //android.widget.Toast.TN 类名在不同版本是否一致？
                            Class<?> tnClass = Class.forName(Toast.class.getName() + "$TN");
                            Field mNextViewField = tnClass.getDeclaredField("mNextView");
                            mNextViewField.setAccessible(true);
                            //不同版本中 TN 参数所在位置不同
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
                                                        //最少的放后面
                                                        .replace(appName, "");
                                                //toastText = toastText.substring(appName.length() + 1);
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
        //替换成代理类对象
        //Toast#sService 是静态对象所以可以为 null
        sServiceField.set(null, sServiceProxy);
        //
    }
```

- 由于源码版本的不尽相同，需要做好各版本的兼容及测试

[![去掉了应用名加冒号的前缀](http://10.1.65.34/group1/M00/13/A3/CgFBImIZr46ALn83AABctqUdI_I811.png "去掉了应用名加冒号的前缀")](http://10.1.65.34/group1/M00/13/A3/CgFBImIZr46ALn83AABctqUdI_I811.png "去掉了应用名加冒号的前缀")

## 总结
通过上述简单的案例场景，阐述了如何通过 Hook 技术绕过系统的限制，实现某些功能或者是改动了系统决定的一些效果，常用的 Hook 技术还有很多，能够利用的场景和案例也非常的多，此处也是为了抛砖引玉，同时也启发自己去思考发现更多的实际应用场景，促使开发效率更高，使得产品更稳定、易用。
