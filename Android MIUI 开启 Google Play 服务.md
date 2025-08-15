# Android MIUI 开启 Google Play 服务
- MIUI 版本号 14.0.2.0 对应 Android 版本 12
- Android 安全更新 2023-01-01
- 测试手机型号 Redmi Note 9 Pro
- 认证型号 M2007J17C

1 自备梯子

2 下载 Google Play Store 
- 可以从 https://play-store.en.divxland.org 下载得到 Google-Play-Store_47.5.19-29_0_PR_793029697.apk 安装到手机

3 更新 Google Play services
- 打开 Google Play Store 点击右上角的更多按钮（三个点）会显示两个菜单（“自动更新 Google 应用”和“Play 保护机制”）
- 关掉 Google Play Store 再次打开，然后点击下方“登录”按钮显示“正在核对信息”，期间会提示下载更新之类的提示，进行允许同意（如果有），然后紧接着会显示出登录的账号密码输入框（可以不登录）
- 尝试多次关掉 Google Play Store 再打开点击右上角的更多按钮（三个点），直到出现显示 3 个菜单（“更新”、“Play 保护机制”和“设置”）的情况
- 此时点击“更新”按钮，进入“可更新页面”，待更新下方会显示有个“Google Play服务”，点击对应右侧“更新”按钮，等待更新安装成功，Google Play Store 会自动退出
- 验证是否成功可用，另外整个过程中会自动将 MIUI 设置-> 账号与同步 -> 谷歌基础服务中的谷歌基础服务开关勾选上（无需手动打开）



