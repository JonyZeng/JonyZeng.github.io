###WebView
&emsp;&emsp;在Android应用中，混编需要使用webView控件对web进行解析，这个控件实际上是使用WebKit内核的内嵌浏览器。webView可以使得网页轻松的内嵌到APP里。使用webView控件解析web程序的步骤如下：
1. 创建webview对象
如果使用webview控件用于显示网页，通常在布局文件里面需要声明WebView控件，并通过id获取对象。如果webView控件只用于解析JavaScript，只需要通过Java代码创建webView对象就可以
> WebView webView = new WebView();
2. 允许解析JavaScript
webView控件默认不会解析JavaScript，需要使用下面的代码是webView可以解析JavaScript
> WebSetting webSetting = webView.getSettings();
webSetting.setJavaScriptEnable(true);
3. 将Java对象映射到JavaScript
>webView.addJavascriptInterface(new Object(){
     //在JavaScript中可以访问的方法
      public void process(int value){   
            }
        },"demo");

其中Object参数表示要在JavaScript中调用的Java对象，可以是任何Java对象。
> // 在JavaScript中调用process方法
webView.demo.process(200);
4. 加载web程序
如果直接加载Url指向的Web程序,可以使用WebView.loadUrl方法。
> public void loadUrl(String url);
public void loadUrl(String url , Map<String , String> additionalHttpHeaders);

webView有两个方法：
> setWebClient: 主要处理解析，渲染网页等浏览器做的事情。
setWebChromeClient:辅助WebView处理JavaScript的对话框，网站图标，网站title，加载进度等。webViewClient就是帮助webView处理各种通知，请求事件等。

在AndroidMainifest.xml设置访问网络权限：
> <uses-permission android:name="android.permission.INTERNET" />

用途
```
/**
 * 研究下那个WebView的用法
 */
public class MainActivity extends AppCompatActivity {
    WebView mWebView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main3);
        mWebView= (WebView) findViewById(R.id.mWebView);
        WebSettings settings = mWebView.getSettings();
        settings.setJavaScriptEnabled(true);   //能够运行简单的JS代码
        settings.setAppCacheEnabled(true);   //给这个页面设置缓存
        settings.setAllowFileAccess(true);   //是否能够接受一个文件
        mWebView.setWebChromeClient(new WebChromeClient());  //可以运行复杂的JS代码
        mWebView.setWebViewClient(new WebViewClient());
//        mWebView.addJavascriptInterface(new MyNsativeToJS(),"demo");  //这个是JS调用JAVA
        mWebView.addJavascriptInterface(this,"demo1");   //JAVA调用JS并返回值
}
    /**
     * 事件的处理
     * @param view
     */
    public void myClick(View view){
       switch (view.getId()){
           case R.id.mBtnLoadWeb:
               mWebView.loadUrl("file:///android_asset/index.html");
//               mWebView.loadUrl("http://www.baidu.com");
               break;
           case R.id.mBtnJavaToJS:   //演示那个JAVA调用JS
               mWebView.loadUrl("javascript:sum(1,1)");
               break;
       }
    }
    @JavascriptInterface
    public void onResult(int result){
      Log.e("两个数之和是:",result+"");
    }
    /**
     * JS调用JAVA，显示一个对话框
     */
    public class MyNsativeToJS{
        @JavascriptInterface
        public void print(){
            AlertDialog.Builder mBuilder=new AlertDialog.Builder(Main3Activity.this);
            mBuilder.setTitle("小标题111");
            mBuilder.setMessage("我是小王子And you?");
            mBuilder.setNegativeButton("取消",null);
            mBuilder.setPositiveButton("确定",null);
            mBuilder.show();
        }
    }
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if(keyCode==KeyEvent.KEYCODE_BACK){
            if(mWebView.canGoBack()){
                 mWebView.goBack();
                return false;
            }
        }
        return super.onKeyDown(keyCode, event);
    }
}
```
