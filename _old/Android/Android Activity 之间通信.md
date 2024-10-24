

## 常规方法

### startActivity + Intent 

- 可选 Bundle

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



### startActivityForResult + onActivityResult + setResult

startActivityForResult 

```java
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
intent.putExtra("main_test_str", "testMain");
startActivityForResult(intent, REQUEST_CODE_1);
```
onActivityResult
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        //
        if (resultCode == Activity.RESULT_OK) {
            //
            if (requestCode == REQUEST_CODE_1) {
                //
                String second_test_str = data.getStringExtra("second_test_str");
                id_tv_main.setText("来自second的数据：" + second_test_str);
            }

        }
}
```

getIntent

```java
String main_test_str = getIntent().getStringExtra("main_test_str");
id_tv_second.setText("来自main的数据："+main_test_str);
```
setResult
```java
Intent intent = new Intent();
intent.putExtra("second_test_str", "testSecond");
setResult(Activity.RESULT_OK, intent);
//
finish();
```



### Activity Result API

- 无需定义 resultCode 和返回后判断 resultCode，其实就是通过 ActivityResultContract 把业务细分了

#### ActivityResultContracts.StartActivityForResult

registerForActivityResult

```java
//Input Intent
//Output ActivityResult
private ActivityResultLauncher<Intent> mStartActivityForResultLauncher = registerForActivityResult(new ActivityResultContracts.StartActivityForResult(),
            new ActivityResultCallback<ActivityResult>() {
                @Override
                public void onActivityResult(ActivityResult result) {
                    int resultCode = result.getResultCode();
                    Intent data = result.getData();
                    if (resultCode == Activity.RESULT_OK) {
                        String second_test_str = data.getStringExtra("third_test_str");
                        id_tv_main.setText("来自third的数据：" + second_test_str);
              }

          }
});
```

launch

```java
Intent intent = new Intent(MainActivity.this, ThirdActivity.class);
intent.putExtra("main_test_str", "testMain");
mStartActivityForResultLauncher.launch(intent);
```

getIntent

```java
String main_test_str = getIntent().getStringExtra("main_test_str");
id_tv_third.setText("来自main的数据："+main_test_str);
```

setResult

```java
Intent intent = new Intent();
intent.putExtra("third_test_str", "testThird");
setResult(Activity.RESULT_OK, intent);
//
finish();
```

#### 自定义 ActivityResultContract 协议

```java
//Input Boolean
//Output String
public class MyActivityResultContract extends ActivityResultContract<Boolean, String> {
    @NonNull
    @Override
    public Intent createIntent(@NonNull Context context, Boolean input) {
        Intent intent = new Intent();
        intent.putExtra("in_test", input);
        return intent;
    }
	//如果需要外部处理 resultCode 可以考虑使用
	//return new ActivityResult(resultCode, intent);
	//ActivityResultContracts.StartActivityForResult 就是这么处理的
    @Override
    public String parseResult(int resultCode, @Nullable Intent intent) {
        if (resultCode == Activity.RESULT_OK) {
            if (intent != null) {
                boolean get_test = intent.getBooleanExtra("get_test", false);
                return get_test ? "T" : "F";
            }
        }
        return null;
    }
}
```

registerForActivityResult

```java
private ActivityResultLauncher<Boolean> mMyActivityResultContractLauncher = registerForActivityResult(
            new MyActivityResultContract(), new ActivityResultCallback<String>() {
                @Override
                public void onActivityResult(String result) {
                    Toast.makeText(MainActivity.this, "传入Boolean值对应字符串是（T/F）："+result, Toast.LENGTH_SHORT).show();
                }
});
```

launch

```java
mMyActivityResultContractLauncher.launch(false);
```

其他逻辑和上文一致



### ActivityResultRegistry 

- 支持不在 Activity 和 Fragment 中使用 registerForActivityResult

### ActivityResultLauncher#unregister

- Activity 和 Fragment 中使用为什么不需要手动调用 unregister，ComponentActivity 和 Fragment 已经实现了 LifecycleObserver，自动处理了





## 全局变量







## 单例类成员变量







## EventBus

- 这个很灵活，不容易跟踪排查问题