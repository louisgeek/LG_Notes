1.隐藏标题栏（要在setContentView()之前）

    this.requestWindowFeature(Window.FEATURE_NO_TITLE);
2.隐藏状态栏（全屏 要在setContentView()之前）

    this.getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
3.XML隐藏标题栏、隐藏状态栏（全屏 应用单个Activity）
在AndroidManifest.xml的配置文件里面的<activity>标签添加属性：

    android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
4.XML隐藏标题栏、隐藏状态栏（全屏 应用全局） 

    <!-- Application theme. -->
	<style name="AppTheme" parent="AppBaseTheme">
    <!-- All customizations that are NOT specific to a particular API-level can go here. -->
    <!-- 隐藏状态栏 -->
    <item name="android:windowFullscreen">true</item>
    <!-- 隐藏标题栏 -->
    <item name="android:windowNoTitle">true</item>
	</style>

5.


待续。。。
