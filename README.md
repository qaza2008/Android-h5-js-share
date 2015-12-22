title: Android与h5通过js调用实现分享
---

### 一.概述

 现在越来越多的app内嵌h5,有诸多好处:

*  HTML5和原生相比优势在于跨平台，成本低，快速迭代，动态执行
*  基于HTML5技术的应用流就会为我们的用户节省流量，带来便利同时也能节省我们手机上空间及电量“
*  避免频繁发版本

 那么本文限定于以下情况:

* app分享h5页面需要 `标题` , `url` ,`分享语` , `图标` ,正常情况下这些信息可以通过接口api获取
* 把`标题` , `url` ,`分享语` , `图标` 这些信息内嵌到h5的代码中当webview加载程序完成后
* 通过调用原生方法来与h5中的js通信,h5在通过调用js方法将需要的参数以json格式传到app上

![图片](https://github.com/qaza2008/pic_bed/blob/master/app与h5分享流程.png?raw=true)

### 二.代码示例

1. android 代码示例:

 ```
	WebView webView;
   @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
 			 webView = (WebView) findViewById(R.id.webView);
            webView.getSettings().setJavaScriptEnabled(true);
            webView.getSettings().setLoadsImagesAutomatically(true);
            webView.getSettings().setBlockNetworkImage(false);
            //解决html太大超出屏幕
            webView.getSettings().setLoadWithOverviewMode(true);
            webView.getSettings().setUseWideViewPort(true);
            webView.getSettings().setDefaultTextEncodingName("utf-8");

            webView.getSettings().setDomStorageEnabled(true);

            //这个类主要能帮助调用你的JavaScript函数中的任意活动方式
            //创建一个JavaScriptInterface实现类
            //前一个参数是绑定到JavaScript的类实例。
            //后一个参数用来显示JavaScript中的实例的名称。
            webView.addJavascriptInterface(new InJavaScriptLocalObj(), "loadShareInfo");

   			webView.setWebViewClient(new CustomWebViewClient());          
      		webView.loadUrl(url);

    }
    
     public class CustomWebViewClient extends WebViewClient {

        @Override
        public boolean shouldOverrideUrlLoading(android.webkit.WebView view, String url) {
            PaiPaiLog.i("orgUrl", url);

            return dispathUrlAction(url);
        }

        public void onPageStarted(WebView view, String url, Bitmap favicon) {
            PaiPaiLog.d("MyWebViewClient", "onPageStarted");
            super.onPageStarted(view, url, favicon);
        }

        public void onPageFinished(WebView view, String url) {
            PaiPaiLog.d("MyWebViewClient", "onPageFinished ");
            //当h5代码加载完成后,通过webview调用 js 方法:getShareInfo()
            view.loadUrl("javascript:getShareInfo()");
            super.onPageFinished(view, url);
        }


    }

 	private Handler mHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                if (msg.what == 0) {
                    String json = (String) msg.obj;
                    if (!TextUtils.isEmpty(json)) {
                        shareEntity = JsonTools.getCollFromJson(json, ShareEntity.class);
                        iv_cdl_share.setVisibility(View.VISIBLE);
                    } else {

                        shareEntity = null;
                        iv_cdl_share.setVisibility(View.GONE);
                    }
                }
            }
        };

      public class InJavaScriptLocalObj {
            @JavascriptInterface
            public void getInfo(String json) {
                if (TextUtils.isEmpty(json)) {
                    return;
                }
                PaiPaiLog.i("getInfo", " json  " + json);
                /*
       {"url":"http://www.paipai.com/m2/2015/rongmomo/index.html","title":"荣耀暑飙，疯狂暴走赢荣耀7！","desc":"荣耀暑飙，疯狂暴走赢荣耀7，不服来战！","img":"http://www.paipai.com/m2/2015/rongmomo/image/100X100.jpg"}
*/
//因为这里不是UI线程,所以通过Handler将数据传递出去,方便分享按钮展示或者隐藏.
                mHandler.sendMessage(mHandler.obtainMessage(0, json));


            }

         }


```


2. Js部分

```
var optionToAndroid = {
  
      					"url": "http://www.paipai.com/m2/2015/rongmomo/index.html",
  
      					"title": "荣耀暑飙，疯狂暴走赢荣耀7！",
  
      					"desc": "荣耀暑飙，疯狂暴走赢荣耀7，不服来战！",
  
      					"img": "http://www.paipai.com/m2/2015/rongmomo/image/100X100.jpg",
  
      				};
  
      
  
      			var optionToAndroidJson = {"url":optionToAndroid.url,"title":optionToAndroid.title,"desc":optionToAndroid.desc,"img":optionToAndroid.img};
  
      			
      			function getShareInfo(){
  
      				window.loadShareInfo.getInfo(JSON.stringify(optionToAndroidJson));
  
      			}
  


```

3. 交互说明

*  `android` webview加载完url数据 调用    `view.loadUrl("javascript:getShareInfo()");`方法
*  `js` 调用 `function getShareInfo()`,实际上调用 `window.loadShareInfo.getInfo(JSON.stringify(optionToAndroidJson));
`,`window` 代表android平台,`loadShareInfo`代表`InJavaScriptLocalObj` 对象,调用 `getInfo` 后将数据传递给android 平台.
* `android` 得到json,解析,并且展示分享按钮.

4. 注意事宜  

*  @JavascriptInterface 这个注解用在 getInfo()方法上,必要的.
*  混淆注意:
```
-keepclassmembers class com.jd.YOUR_WebView_ACTIVITY$InJavaScriptLocalObj{
    public *;
}
-keepattributes *JavascriptInterface*
-keepattributes *Annotation*
```

Done!


Welcome To My Website
---

[http://androidso.com](http://androidso.com)
---







