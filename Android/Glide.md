可以进行图片缓存，还支持Gif、WebP、缩略图，甚至是Video

- 绑定生命周期：可以使加载图片的生命周期动态管理起来
- 高效的缓存策略：支持内存、Disk缓存，并且Picasso只会缓存原始尺寸的图片，内Glide缓存的是多种规格，也就是Glide会根据你ImageView的大小来缓存相应大小的图片尺寸
- 内存开销小：默认的Bitmap格式是RGB_565格式，而Picasso默认的是ARGB_8888格式，内存开销小一半





Glide内存缓存如何控制大小







如果自己去实现图片库，怎么做？



##### 写个图片浏览器，说出你的思路？
