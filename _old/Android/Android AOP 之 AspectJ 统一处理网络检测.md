## 利用 Eclipse AspectJ

project gradle

```groovy
//采用 AspectJX 来快速配置 Eclipse AspectJ
dependencies {
     classpath "com.android.tools.build:gradle:4.1.2"
     //add
     classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.10'
}
```
module gradle
```groovy
plugins {
    id 'com.android.application'
    //add
    id 'android-aspectjx'
}
dependencies {
    //add
    implementation 'org.aspectj:aspectjrt:1.9.5'
}
```

定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CheckNetwork {
    String tip() default "未连接网络";
}
```

定义切面类

```java
@Aspect
public class CheckNetworkAspect {
    private static final String TAG = "CheckNetworkAspect";

    @Pointcut("execution(@com.louisgeek.aop.CheckNetwork * *(..))")
    public void pointcutCheckNetwork() {
        Log.e(TAG, "pointcutCheckNetwork: ");
    }

    @Around("pointcutCheckNetwork()")
    public Object aroundCheckNetwork(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        CheckNetwork checkNetwork = signature.getMethod().getAnnotation(CheckNetwork.class);
        if (checkNetwork == null) {
            return null;
        }
        Context context = AspectTool.getContext(joinPoint.getThis());
        if (context == null) {
            return null;
        }
        String tip = checkNetwork.tip();
        if (TextUtils.isEmpty(tip)) {
            tip = "连接网络异常";
        }
        if (NetworkTool.isConnected(context)) {
            Log.e(TAG, "aroundCheckNetwork: 网络已连接 ");
            joinPoint.proceed();
        } else {
            Toast.makeText(context, tip, Toast.LENGTH_SHORT).show();

        }
        return null;
    }
}
```

使用方式

```java
id_tv_1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.e(TAG, "onClick: ");
                checkNetworkTest();
            }
});
        //
@CheckNetwork(tip = "网络开了小差")
public void checkNetworkTest() {
   Toast.makeText(this, "继续执行", Toast.LENGTH_SHORT).show();
}     
```



## 封装成 library 使用

project.gradle 

```groovy
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
}
```

project.gradle

```groovy
 dependencies {
        classpath "com.android.tools.build:gradle:4.1.2"
		//add
        classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.10'

}
```

3 module gradle

```groovy
plugins {
    id 'com.android.application'
    //add
    id 'android-aspectjx'
}

```

4 module gradle [![](https://jitpack.io/v/louisgeek/LG_AOP.svg)](https://jitpack.io/#louisgeek/LG_AOP)

```groovy
dependencies {
	...
	//add 
    implementation 'com.github.louisgeek:LG_AOP:x.x.x'
}
```

- 使用方式与上文一致

