### Android 原生调用 JS 中的方法  
Android 调用 JS 有两种方式，都是通过 WebView 的方法：  
1. webview.loadUrl()
2. webview.evaluateJavascript()  
> 二者区别：1. loadUrl() 会刷新页面，evaluateJavascript() 则不会使页面刷新，所以 evaluateJavascript() 的效率更高2. loadUrl() 得不到 js 的返回值，evaluateJavascript() 可以获取返回值。3. evaluateJavascript() 在 Android 4.4 之后才可以使用  


要实现目标：在安卓原生中改变js中一个文本并接收js回传的值。  
Vue中的代码：  
```
mounted() {
    //将要给原生调用的方法挂载到 window 上面
    window.callJsFunction = this.callJsFunction
},
data() {
    return {
    	msg: "哈哈"
	}
},
methods: {
    callJsFunction(str) {
        this.msg = "我通过原生方法改变了文字" + str
        return "js调用成功"
	}
}
```  
在 methods 中定义一个供 Android 调用的方法 callJsFunction(str) , 并可接收一个参数 str。  
如果只是在 methods 中定义方法，原生调用会找不到这个方法。所以要在页面加载的时候将方法挂载在 window 上，这样 WebView 就可以拿到此方法了。注意，这步很重要一定要写！  
注意一个细节，this.callJsFunction 后面不要加括号 ()，加括号相当于直接调用了。  
总结起来 Vue 中要做的事情就两步：

1. 在 methods 中定义方法
2. 在 mounted 中将方法挂载在 window 上  

 Android 中代码：  
 loadUrl实现：  
 ```
 tbsWebView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                view.loadUrl(url, headerMap);
                return true;
            }

            @Override
            public void onPageFinished(WebView webView, String s) {
                super.onPageFinished(webView, s);
                //安卓调用js方法。注意需要在 onPageFinished 回调里调用
                tbsWebView.post(new Runnable() {
                    @Override
                    public void run() {
                        tbsWebView.loadUrl("javascript:callJsFunction('soloname')");
                    }
                });
            }
        });
    }
});

 ```  
 
 如果不需要传参数，把参数去掉即可 tbsWebView.loadUrl("javascript:callJsFunction()");  
 
 evaluateJavascript()实现  
 其他地方跟loadUrl()一样，只是把 tbsWebView.loadUrl("javascript:callJsFunction('soloname')"); 替换掉  
 ```
 @Override
public void onPageFinished(WebView webView, String s) {
    super.onPageFinished(webView, s);
    //安卓调用js方法。注意需要在 onPageFinished 回调里调用
    tbsWebView.post(new Runnable() {
        @Override
        public void run() {
            tbsWebView.evaluateJavascript("javascript:callJsFunction('soloname')", new ValueCallback<String>() {
                @Override
                public void onReceiveValue(String s) {
                    Logger.d("js返回的结果： " + s);
                }
            });
        }
    });
}
 ```  
 
 ### js调用安卓原生的方法  
 对于JS调用Android代码的方法有3种：  
 1. 通过 WebView 的 addJavascriptInterface() 进行对象映射
2. 通过 WebViewClient 的 shouldOverrideUrlLoading()方法回调拦截 url
3. 通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt()方法回调拦截JS对话框alert()、confirm()、prompt() 消息  
4. 
由于目前的设备系统版本基本都在 4.2 以上，所以用第一种就可以了，简单快捷。  

实现目标：  
点击js的控件调用安卓中的Toast  

Vue代码:  
```
methods: {
  showAndroidToast() {
    $App.showToast("哈哈，我是js调用的")
  }
}
```  
Android代码：  
新建类 JsJavaBridge  
```
public class JsJavaBridge {

    private Activity activity;
    private WebView webView;

    public JsJavaBridge(Activity activity, WebView webView) {
        this.activity = activity;
        this.webView = webView;
    }

    @JavascriptInterface
    public void onFinishActivity() {
        activity.finish();
    }

    @JavascriptInterface
    public void showToast(String msg) {
        ToastUtils.show(msg);
    }
}
```  
然后通过 WebView 设置 Android 类与 JS 代码的映射  
```
tbsWebView.addJavascriptInterface(new JsJavaBridge(this, tbsWebView), "$App");
```  
这里将类 JsJavaBridge 在 JS 中映射为了 $App，所以在 Vue 中可以这样调用 $App.showToast("哈哈，我是js调用的")。  
#### 注入风险  
4.2版本以下，js利用android获取到的一些危险权限，解决方法，通过js与java交互的promp等方法传递信息然后通过反射调用具体方法。
#### 内存泄漏  
js连续几个页面申请native的堆内存的内存过大导致保存页面元素，并且为了打开和返回页面较快前面占用的内存可能也不会释放。可以将处理js的页面activity放到单独进程中，监测这个进程，一旦内存超出预定值，退出这个进程。  

#### webview缓存
可设置缓存策略