# Android webview 加载 html页面 实现 不同分辨率 不同 dpi 缩放自适应处理

两种情况一起使用  实现  不同分辨率 不同 dpi 缩放自适应处理

```
//webview 需要配置
mWebView.getWebSetting().setUseWideViewPort(true);//让webview读取网页设置的viewport，pc版网页
```

1、同分辨率  不同dpi 缩放自适应处理  ( 也可以在android端 注入相关js 代码)
```
<script type="text/javascript">
<!-- 针对 Android 【不同 dpi】 适配处理 同分辨率 不同 dpi 显示一致 -->
var ua = navigator.userAgent;
if (/Android (\d+\.\d+)/.test(ua)){
<!-- 需要在页面加载时候 生效 才能有效 -->
var devicePixelRatio = window.devicePixelRatio;
var deviceScale = 1/devicePixelRatio;
document.write('<meta name="viewport" content="width=device-width,initial-scale='+deviceScale+',minimum-scale='+deviceScale+',maximum-scale='+deviceScale+',user-scalable=no">');
}
</script>
```

图片涉及隐私  已打码，点击查看大图

![这里写图片描述](http://img.blog.csdn.net/20180209183416944?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUmljaGllWmh1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![图片涉及隐私  已打码](http://img.blog.csdn.net/20180209182502126?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUmljaGllWmh1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



2、同dpi  不同分辨率 缩放自适应处理 

```
   @Override
            public void onPageFinished(WebView webView, String url) {
                super.onPageFinished(webView, url);
               
      
                /**
                 * 针对 web 页面 【不同分辨率】 适配处理   同dpi 不同分辨率 显示一致
                 *  zoom = 100%    分辨率 1920 * 1080   Ui设置设计出来合适的 也是body标签默认未设置zoom属性的或者100%  例子 <body style="zoom: 50%;">
                 *  zoom = 66.67%     分辨率 1280 * 720
                 *  zoom = 71.14%     分辨率 1366 * 768
                 *  zoom = 106.67%     分辨率 2048 * 1152
                 *  zoom = 83.33%     分辨率 1600 * 900
                 *  zoom = 50%     分辨率 960 * 540
                 */
                float widthPixels = getResources().getDisplayMetrics().widthPixels;
                float heightPixels = getResources().getDisplayMetrics().heightPixels;
                Log.d("zfq", "widthPixels:" + widthPixels);
                Log.d("zfq", "heightPixels:" + heightPixels);
                float zoomPercent = widthPixels / 1920 * 100;
                Log.d("zfq", "zoomPercent:" + zoomPercent);
                String javaScriptInterfaceName_zoomWeb_NeedInjectJsStr =
                        "javascript:(function(){  document.body.style.zoom='" + zoomPercent + "%'; })()";
                mWebView.loadUrl(javaScriptInterfaceName_zoomWeb_NeedInjectJsStr);
            }
```

图片涉及隐私  已打码，点击查看大图

![这里写图片描述](http://img.blog.csdn.net/20180209183434975?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUmljaGllWmh1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![图片涉及隐私  已打码](http://img.blog.csdn.net/20180209182518925?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUmljaGllWmh1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)




