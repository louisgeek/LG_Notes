> 源码基于 com.github.bumptech.glide:glide:3.8.0





```java
Glide.with(holder.id_iv.getContext())
                .load(dataItem.imageUrl)
                .override(150, 100)
                .into(new SimpleTarget<GlideDrawable>() {
                    @Override
                    public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
                        holder.id_iv.setImageDrawable(resource);
                    }
                });
```









线程





线程切换

采用哦那个





格式解码





图像大小

Glide 会自动根据 ImageView 的大小来决定加载图片的大小



缓存

内存缓存

磁盘缓存

网络





内存溢出







内存泄漏







快速滑动



RequestManager

RequestManagerFragment

- SupportRequestManagerFragment



```
public class SupportRequestManagerFragment extends android.support.v4.app.Fragment
```

```
public class RequestManagerFragment extends android.app.Fragment
```



RequestManagerRetriever







1 Glide 如何关联生命周期的？

Glide#with 方法内会创建并添加一个 Fragment ，通过在其内部进行主动回调各个生命周期的方式实现监听，关联的 FragmentRequestManager 就能感知生命周期了，然后它里面由把事件传递给了 TargetTracker 

TargetTracker 统一管理所有 Target

Glide 加载图片的时候，最终调用的 into(ImageView view) 其实就是封装了一个 ImageViewTarget，ImageViewTarget 继承 ViewTarget，它又继承 BaseTarget ，然后 BaseTarget 实现了 Target 接口

