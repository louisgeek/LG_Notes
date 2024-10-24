



```groovy
android {

//    viewBinding.enabled = true
    viewBinding {
        enabled = true
    }
    
}
```





```
/build/generated/data_binding_base_class_source_out
```



```java
private ActivityMainBinding binding;
  
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
	binding = ActivityMainBinding.inflate(getLayoutInflater());
    setContentView(binding.getRoot());
}
  
```



```
FragmentViewBindBinding binding;
@Override
public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
    binding = FragmentViewBindBinding.inflate(inflater, container, false);
    return binding.getRoot();
}
```



```
@NonNull
@Override
public ViewHolder onCreateViewHolder(@NonNull ViewGroup viewGroup, int i) {
    ItemViewBinding binding = ItemViewBinding.inflate(LayoutInflater.from(mContext));
    return new MyViewHolder(binding);
}
```





## 总结



- 空安全

- 类型安全

由于是扫描 layout 文件生成 Binding 后缀的代码类，id 和类型都是对应的，使用的时候不会出现空安全和类型安全问题，代码里用的仍然是 findViewById



