# Android SharedPreferences
- SharedPreferences 的定位是轻量级数据存储，用于存储简单的数据结构
- 以键值对形式存储在 xml 文件里
- 可以保证线程安全（内部有很多 synchronized 的逻辑）、不过进程间是不安全的（SharedPreferences 有内存缓存）
- commit 有返回值，apply 无返回值
- 官方推荐考虑改用 apply 替代 commit
- apply 异步保存、commit 同步提交，commit 将修改的数据提交到内存，然后同步提交到硬盘，apply 是将修改的数据提交到内存，然后异步的提交到硬盘（可能是 Aty 的 onStop）
- 读写性能差，只支持全量更新，单个文件如果过大，所以可能引起 ANR