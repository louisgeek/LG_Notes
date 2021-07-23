

```java
public static HomeFragment newInstance(String param1, String param2) {
        HomeFragment fragment = new HomeFragment();
        Bundle args = new Bundle();
        args.putString(ARG_PARAM1, param1);
        args.putString(ARG_PARAM2, param2);
        fragment.setArguments(args);
        return fragment;
}
```



全局变量



单例类成员变量



## Activity 调用 Fragment 

Activity 里一般持有实例

Activity 里通过 Fragment 的 id 或 tag 获取到 Fragment 的实例





### Fragment 调用 Activity 

Fragment 通过 getActivity() 获取到对应 Activity 的实例

