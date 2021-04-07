## Intent Bundle

```java
public static void actionShow(Context context,String device_id,String device_Name) {
        Intent intent = new Intent(context, HomeActivity.class);
        Bundle bundle = new Bundle();
 		bundle.putString("device_id", device_id);
 		bundle.putString("device_Name", device_Name);
        intent.putExtras(bundle);
        context.startActivity(intent);
}
```

## 全局变量





## 单例类成员变量