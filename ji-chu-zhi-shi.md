# WebView 的使用 {#webview-的使用}

## 1. 使用场景 {#1-使用场景}

1.1 展示一些需要经常更新的信息，比如用户手册或者免责条款等  
1.2 必须与网络保持连接获取数据的情况，比如邮箱。

## 2. 在应用中添加WebView {#2-在应用中添加webview}

```
<
?xml version="1.0" encoding="utf-8"?
>
<
WebView  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/webview"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
/
>
```

```
WebView myWebView = (WebView) findViewById(R.id.webview);
    myWebView.loadUrl("http://www.example.com");

```

或者

```
String summary = "
<
html
>
<
body
>
You scored 
<
b
>
192
<
/b
>
 points.
<
/body
>
<
/html
>
";
    webview.loadData(summary, "text/html", null);

```

```
<
manifest ... 
>
<
uses-permission android:name="android.permission.INTERNET" /
>

    ...

<
/manifest
>
```

刷新页面

```
    webView.reload(url);

```

## 3. 在 WebView 中使用 JavaScript {#3-在-webview-中使用-javascript}

如果 WebView 加载的页面包含 JavaScript，必须使其可用，之后，还可以在 Java 代码和 JavaScript 代码事件创建接口。

### 3.1 使 JavaScript 在 WebView 中可用 {#31-使-javascript-在-webview-中可用}

```
    WebView myWebView = (WebView) findViewById(R.id.webview);
    WebSettings webSettings = myWebView.getSettings();
    webSettings.setJavaScriptEnabled(true);

```

通过 WebSettings 可以访问到其他非常有用的 settings。比如，设计一个页面专用于自己安卓APP的 webview，可以使用 setUserAgentString\(\) 定义一个自定义的user agent string，就可以知道请求时来自自己的安卓APP。

### 3.2 在 JavaScript 代码中调用 Java 代码 {#32-在-javascript-代码中调用-java-代码}

**此处有安全隐患，使用时需要注意**

在下例中，WebAppInterface 类允许网页调用 showToast\(\) 方法来提示用户。

```
public class WebAppInterface {
    Context mContext;

    /** Instantiate the interface and set the context */
    WebAppInterface(Context c) {
        mContext = c;
    }

    /** Show a toast from the web page */
    @JavascriptInterface
    public void showToast(String toast) {
        Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show();
    }
}

```

* 每个方法前都需要使用注解 @JavascriptInterface。
* 如何使用：

```
    WebView webView = (WebView) findViewById(R.id.webview);
    webView.addJavascriptInterface(new WebAppInterface(this), "android");

```

```
<
input type="button" value="Say hello" onClick="showAndroidToast('Hello Android!')" /
>
<
script type="text/javascript"
>

    function showAndroidToast(toast) {
        android.showToast(toast);
    }

<
/script
>
```

更严谨的写法：

```
    if(typeof androidAppProxy !== "undefined"){
        androidAppProxy.showMessage("Message from JavaScript");
    } else {
        alert("Running outside Android app");
    }

```

* 被绑定到 JavaScript 代码的对象运行在另外的线程，而非构建它的线程。

**另一种写法**

```
mWebView = (WebView) findViewById(R.id.WebView); 
    WebSettings webSettings = mWebView.getSettings(); 
    webSettings.setJavaScriptEnabled(true); 
    mWebView.addJavascriptInterface(new Object() {
      public void clickOnAndroid() {
          mHandler.post(new Runnable() {
              public void run() { 
                  mWebView.loadUrl("javascript:wave()");
              }
          });
      }
    }, "demo"); 
    mWebView.loadUrl("file:///android_asset/demo.html");

```

addJavascriptInterface\(Object obj,String interfaceName\) 方法将一个 Java 对象绑定到一个Javascript对象中, Javascript 对象名就是 interfaceName,作用域是 Global.这样初始化 WebView 后,在 WebView 加载的页面中就可以直接通过 javascript:window.demo 访问到绑定的java对象了.

```
<
html
>
<
script language="javascript"
>

  function wave() {
    document.getElementById("droid").src="android_waving.png";
  }

<
/script
>
<
body
>
<
a onClick="window.demo.clickOnAndroid()"
>
<
img id="droid" src="android_normal.png" mce_src="android_normal.png"/
>
<
br
>
 Click me! 
<
/a
>
<
/body
>
<
/html
>
```

这样在 javascript 中就可以调用 java 对象的 clickOnAndroid\(\)方法了,同样我们可以在此对象中定义很多方法\(比如发短信,调用联系人列表等手机系统功能\),这里 wave\(\)方法是 java 中调用 javascript 的例子.

需要说明一点:addJavascriptInterface方法中要绑定的Java对象及方法要运行另外的线程中,不能运行在构造他的线程中,这也是使用 Handler 的目的.

#### 3.2.1 检查域名，降低安全风险 {#321-检查域名，降低安全风险}

```
public class AppJavaScriptProxy {

    private Activity activity = null;
    private WebView  webView  = null;

    public AppJavaScriptProxy(Activity activity, WebView webview) {

        this.activity = activity;
        this.webView  = webview;
    }

    @JavascriptInterface
    public void showMessage(final String message) {

        final Activity theActivity = this.activity;
        final WebView theWebView = this.webView;

        this.activity.runOnUiThread(new Runnable() {

            @Override
            public void run() {
                if(!theWebView.getUrl().startsWith("http://tutorials.jenkov.com")){
                    return ;
                }

                Toast toast = Toast.makeText(
                        theActivity.getApplicationContext(),
                        message,
                        Toast.LENGTH_SHORT);

                toast.show();
            }
        });
    }
}

```

### 3.3 在 Java 代码中调用 JavaScript 代码 {#33-在-java-代码中调用-javascript-代码}

#### 3.3.1 API 19 之前使用 WebView 的 loadUrl\(\) 方法 {#331-api-19-之前使用-webview-的-loadurl-方法}

API 19 之前可以使用以下代码，直接操作当前加载的页面的 JavaScript 方法。但是这种方法只有结合回调函数才能从 JavaScript 代码中获取返回值。

```
    webView.loadUrl("javascript:theFunction('text')");

```

![](https://frankyzj.gitbooks.io/android-1/content/assets/Java%E5%92%8CJavaScript%E7%9A%84%E4%BA%A4%E4%BA%92.png)

#### 3.3.2 API 19 之后使用 WebView 的 evaluateJavascript\(\) 方法 {#332-api-19-之后使用-webview-的-evaluatejavascript-方法}

这个方法可以像在 JavaScript 中一样执行 JavaScript 代码。

```
function getGreetings() {
      return 1;
}

```

```
webView.evaluateJavascript("getGreetings()", new ValueCallback
<
String
>
() {
    @Override
    public void onReceiveValue(String value) {
        //store / process result received from executing Javascript.
    }
});

```

## 4 WebView 的页面导航 {#4-webview-的页面导航}

在 WebView 上点击一个链接，Android的默认动作是打开浏览器。可以如3.3.1重写这个动作，使之跳转到另一个 WebView，从而用户可以根据历史记录向前或者向后导航。

### 4.1 为 WebView 提供 WebViewClient 对象 {#41-为-webview-提供-webviewclient-对象}

```
    WebView myWebView = (WebView) findViewById(R.id.webview);
    myWebView.setWebViewClient(new WebViewClient());

```

好了，这样 WebView 就会执行上述的动作。

### 4.2 自定义 WebViewClient {#42-自定义-webviewclient}

重写 shouldOverrideUrlLoading\(\) 方法，你可以对页面导航获得更多的控制。

```
private class MyWebViewClient extends WebViewClient {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        // view.loadUrl(url);
    // return true;

        if (Uri.parse(url).getHost().equals("www.example.com")) {
            // This is my web site, so do not override; let my WebView load the page
            return false;
        }
        // Otherwise, the link is not for a page on my site, so launch another Activity that handles URLs
        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
        startActivity(intent);
        return true;
    }
}

```

```
    WebView myWebView = (WebView) findViewById(R.id.webview);
    myWebView.setWebViewClient(new MyWebViewClient());

```

### 4.3 历史记录导航 {#43-历史记录导航}

WebView 重写 Urlloading 之后, 会自动保存历史记录。如果不做任何处理,浏览网页,点击系统“Back”键,整个 Browser 会调用 finish\(\)而结束自身。可以调用 goBack\(\) 和 goForward\(\) 进行导航。

```
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    // Check if the key event was the Back button and if there's history
    if ((keyCode == KeyEvent.KEYCODE_BACK) 
&
&
 myWebView.canGoBack()) {
        myWebView.goBack();
        return true;
    }
    // If it wasn't the Back key or there's no web page history, bubble up to the default
    // system behavior (probably exit the activity)
    return super.onKeyDown(keyCode, event);
}

```

如果还有历史记录，canGoBack\(\) 方法返回 true，canGoForward\(\)同理。

### 4.4 放大页面 {#44-放大页面}

调用以下方法即可激活内建的放大器：

```
webView.getSettings().setSupportZoom(true);
    webView.getSettings().setBuiltInZoomControls(true);
    webView.getSettings().setDisplayZoomControls(true);

```

* 注：此时，要避免 height 或 width 设为 WRAP\_CONTENT。

## 5. 深入WebView {#5-深入webview}

### 5.1 WebView、WebViewClient、WebChromeClient 之间的区别 {#51--webview、webviewclient、webchromeclient-之间的区别}

WebView 专心干好 自己的解析、渲染工作就行了.

WebViewClient 就是帮助 WebView 处理各种通知、请求事件等 。

WebChromeClient 是辅助 WebView 处理 Javascript 的对话框,网站图标,网站 title.

### 5.2 拦截 WebView 的 HTTP 请求 {#52-拦截-webview-的-http-请求}

可以在 WebView 加载页面或者页面中的资源（图片，JS文件，CSS文件等）时进行拦截。这时候，你就可以用新版本的资源替换原来的。 重写WebViewClient的实现类中的 shouldInterceptRequest\(\) 方法即可实现拦截，如下例：

```
public class WebViewClientImpl extends WebViewClient {

    private Activity activity = null;

    public WebViewClientImpl(Activity activity) {
        this.activity = activity;
    }

    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if(url.indexOf("jenkov.com") 
>
 -1 ) return false;

        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
        activity.startActivity(intent);
        return true;
    }

    @Override
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
        if(url.startsWith("http://tutorials.jenkov.com/images/logo.png")){
            String mimeType = "image/png";
            String encoding = "";
            InputStream input = ...;

            WebResourceResponse response =
                    new WebResourceResponse(mimeType, encoding, input);

            return response;
        }

        return null;
    }
}

```

WebResourceResponse 的构造器中要求的参数 InputStream 是用来下载资源文件的。在后边从 assets 中加载资源的例子中会有相应的实现。如果 shouldInterceptRequest\(\) 方法返回null，则自动加载默认内容。

### 5.3 从 App 的 APK Assets 中加载资源文件 {#53-从-app-的-apk-assets-中加载资源文件}

可以把比较稳定的数据放到 assets 中，比如 logo 图片、JavaScript 文件、CSS 文件等。

```
public class WebViewClientImpl extends WebViewClient {

    private Activity activity = null;

    public WebViewClientImpl(Activity activity) {
        this.activity = activity;
    }

    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if(url.indexOf("jenkov.com") 
>
 -1 ) return false;

        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
        activity.startActivity(intent);
        return true;
    }


    @Override
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {

        if(url.startsWith("http://tutorials.jenkov.com/images/logo.png")){
            return loadFromAssets(url, "images/logo.png", "image/png", "");
        }

        return null;
    }

    private WebResourceResponse loadFromAssets( String url,
        String assetPath, String mimeType, String encoding){

        AssetManager assetManager = this.activity.getAssets();
        InputStream input = null;
        try {
            Log.d(Constants.LOG_TAG, "Loading from assets: " + assetPath);


            input = assetManager.open("/images/logo.png");
            WebResourceResponse response =
                    new WebResourceResponse(mimeType, encoding, input);

            return response;
        } catch (IOException e) {
            Log.e("WEB-APP", "Error loading " + assetPath + " from assets: " +
                e.getMessage(), e);
        }
        return null;
    }
}

```

### 5.4 WebView缓存 {#54-webview缓存}

在项目中如果使用到 WebView 控件,当加载 html 页面时,会在/data/data/包名目录下生成 database 与 cache 两个文件夹。 请求的 url 记录是保存在 WebViewCache.db,而 url 的内容是保存在 WebViewCache 文件夹下。定义一个html文件,在里面显示一张图片,用WebView加载出来,然后就可以从缓存里把这张图片读取出来并显示。

以下是一个自定义缓存的例子：[http://tutorials.jenkov.com/android/android-web-apps-using-android-webview.html\#calling-from-javascript-to-the-android-web-app](http://tutorials.jenkov.com/android/android-web-apps-using-android-webview.html#calling-from-javascript-to-the-android-web-app)

```
public class WebViewClientImpl extends WebViewClient {

    private Activity activity = null;
    private UrlCache urlCache = null;

    public WebViewClientImpl(Activity activity) {
        this.activity = activity;
        this.urlCache = new UrlCache(activity);

        this.urlCache.register("http://tutorials.jenkov.com/", "tutorials-jenkov-com.html",
                "text/html", "UTF-8", 5 * UrlCache.ONE_MINUTE);

    }

    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if(url.indexOf("jenkov.com") 
>
 -1 ) return false;

        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
        activity.startActivity(intent);
        return true;
    }

    @Override
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {

        return this.urlCache.load(url);
    }
}

```

```
public class UrlCache {

  public static final long ONE_SECOND = 1000L;
  public static final long ONE_MINUTE = 60L * ONE_SECOND;
  public static final long ONE_HOUR   = 60L * ONE_MINUTE;
  public static final long ONE_DAY    = 24 * ONE_HOUR;

  private static class CacheEntry {
    public String url;
    public String fileName;
    public String mimeType;
    public String encoding;
    public long   maxAgeMillis;

    private CacheEntry(String url, String fileName,
        String mimeType, String encoding, long maxAgeMillis) {

        this.url = url;
        this.fileName = fileName;
        this.mimeType = mimeType;
        this.encoding = encoding;
        this.maxAgeMillis = maxAgeMillis;
    }
  }


  protected Map
<
String, CacheEntry
>
 cacheEntries = new HashMap
<
String, CacheEntry
>
();
  protected Activity activity = null;
  protected File rootDir = null;


  public UrlCache(Activity activity) {
    this.activity = activity;
    this.rootDir  = this.activity.getFilesDir();
  }

  public UrlCache(Activity activity, File rootDir) {
    this.activity = activity;
    this.rootDir  = rootDir;
  }



  public void register(String url, String cacheFileName,
                       String mimeType, String encoding,
                       long maxAgeMillis) {

    CacheEntry entry = new CacheEntry(url, cacheFileName, mimeType, encoding, maxAgeMillis);

    this.cacheEntries.put(url, entry);
  }



  public WebResourceResponse load(String url){
    CacheEntry cacheEntry = this.cacheEntries.get(url);

    if(cacheEntry == null) return null;

    File cachedFile = new File(this.rootDir.getPath() + File.separator + cacheEntry.fileName);

    if(cachedFile.exists()){
      long cacheEntryAge = System.currentTimeMillis() - cachedFile.lastModified();
      if(cacheEntryAge 
>
 cacheEntry.maxAgeMillis){
        cachedFile.delete();

        //cached file deleted, call load() again.
        Log.d(Constants.LOG_TAG, "Deleting from cache: " + url);
        return load(url);
      }

      //cached file exists and is not too old. Return file.
      Log.d(Constants.LOG_TAG, "Loading from cache: " + url);
      try {
        return new WebResourceResponse(
                cacheEntry.mimeType, cacheEntry.encoding, new FileInputStream(cachedFile));
      } catch (FileNotFoundException e) {
        Log.d(Constants.LOG_TAG, "Error loading cached file: " + cachedFile.getPath() + " : "
                + e.getMessage(), e);
      }

    } else {
      try{
        downloadAndStore(url, cacheEntry, cachedFile);

        //now the file exists in the cache, so we can just call this method again to read it.
        return load(url);
      } catch(Exception e){
        Log.d(Constants.LOG_TAG, "Error reading file over network: " + cachedFile.getPath(), e);
      }
    }

    return null;
  }

  private void downloadAndStore(String url, CacheEntry cacheEntry, File cachedFile)
    throws IOException {

    URL urlObj = new URL(url);
    URLConnection urlConnection = urlObj.openConnection();
    InputStream urlInput = urlConnection.getInputStream();

    FileOutputStream fileOutputStream =
            this.activity.openFileOutput(cacheEntry.fileName, Context.MODE_PRIVATE);

    int data = urlInput.read();
    while( data != -1 ){
      fileOutputStream.write(data);

      data = urlInput.read();
    }

    urlInput.close();
    fileOutputStream.close();
    Log.d(Constants.LOG_TAG, "Cache file: " + cacheEntry.fileName + " stored. ");
  }
}

```

#### 5.4.1 WebView删除缓存 {#541-webview删除缓存}

```
    webview.clearCache(true);
    webview.clearHistory();

```

或者

```
private int clearCacheFolder(File dir,long numDays) { 
  int deletedFiles = 0;
  if (dir!= null 
&
&
 dir.isDirectory()){
    try {
      for (File child:dir.listFiles()){
          if (child.isDirectory()) {
            deletedFiles += clearCacheFolder(child, numDays);
        }
        if (child.lastModified() 
<
 numDays) {
          if (child.delete()) {
           deletedFiles++; 
          }
        }
      }
    } catch(Exception e) {
      e.printStackTrace(); 
    }
  }
  return deletedFiles; 
}

```

* 缓存的控制

```
    //优先使用缓存: 
    WebView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); 
    //不使用缓存: 
    WebView.getSettings().setCacheMode(WebSettings.LOAD_NO_CACHE);

```

* 退出应用时清空缓存

```
    File file = CacheManager.getCacheFileBaseDir();
    if (file != null 
&
&
 file.exists() 
&
&
 file.isDirectory()) {
        for (File item : file.listFiles()) {
            item.delete();
        }
        file.delete();
    }
    context.deleteDatabase("WebView.db"); 
    context.deleteDatabase("WebViewCache.db");

```

### 5.5 WebView处理404 {#55-webview处理404}

```
public class WebView_404 extends Activity {    
    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
          if(msg.what==404) {//主页不存在
            //载入本地 assets 文件夹下面的错误提示页面 404.html 
            web.loadUrl("file:///android_asset/404.html");
          }else{
            web.loadUrl(HOMEPAGE);
          }
       }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      web.setWebViewClient(new WebViewClient() {
        public boolean shouldOverrideUrl(WebView view,String url) { 
        if(url.startsWith("http://") 
&
&
 getRespStatus(url)==404) {
          view.stopLoading();
          //载入本地 assets 文件夹下面的错误提示页面 404.html 
          view.loadUrl("file:///android_asset/404.html");
        }else{
          view.loadUrl(url);
        }
          return true;
        }
      });
      new Thread(new Runnable() {
    public void run() {
          Message msg = new Message();
      //此处判断主页是否存在,因为主页是通过 loadUrl 加载的,
      //此时不会执行 shouldOverrideUrlLoading 进行页面是否存在的判断 
      //进入主页后,点主页里面的链接,链接到其他页面就一定会执行shouldOverrideUrlLoading 方法了 
         if(getRespStatus(HOMEPAGE)==404) {
          msg.what = 404;
          }
          handler.sendMessage(msg);
      }).start();
  }
}

```

或者

```
webview.setWebViewClient(new WebViewClient(){

            @Override
            public void onReceivedError(WebView view, int errorCode,
                    String description, String failingUrl) {
                switch(errorCode)
                {
                case HttpStatus.SC_NOT_FOUND:
                    view.loadUrl("file:///android_assets/error_handle.html");
                    break;
                }
            }
        });

```

当出错的时候，我们还可以选择隐藏掉webview，而显示native的错误处理控件，这个时候只需要在onReceivedError里面显示出错误处理的native控件同时隐藏掉webview即可。

### 5.6 WebView 是否滚动到页面底端 {#56-webview-是否滚动到页面底端}

在View中有一个getScrollY\(\)方法，可以返回当前可见区域的顶端距整个页面顶端的距离,也就是当前内容滚动的距离。 还有getHeight\(\)或者 getBottom\(\)方法都返回当前 View 这个容器的高度 在ViewView中还有getContentHeight\(\) 方法可以返回整个 html 页面的高度,但并不等同于当前整个页面的高度 ,因为 WebView 有缩放功能。你可以通过如下代码来启动或关闭webview的缩放功能。

### 5.7 WebView获取服务器中的 session 问题 {#57-webview获取服务器中的-session-问题}

1、Android 中的 WebView 如何获取服务器页面的 jsessionid 的值

2、Android 的 WebView 又是如何把得到的 jsessionid 的值再 set 到服务器中,同一个 jsessionid 的会话中.

```
    CookieManager cm = CookieManager.getInstance(); 
    cm.removeAllCookie();
    cm.getCookie(url);
    cm.setCookie(url, cookie);

```

* 现在WebView已经是自动的异步缓存

### 5.8 WebView清除本地cookies {#58-webview清除本地cookies}

```
    /***
     * 如果用户已经登录，则同步本地的cookie到webview中
     */
    public void synCookies() {
        if (!CacheUtils.isLogin(this)) return;
        CookieSyncManager.createInstance(this);
        CookieManager cookieManager = CookieManager.getInstance();
        cookieManager.setAcceptCookie(true);
        cookieManager.removeSessionCookie();//移除
        String cookies = PreferenceHelper.readString(this, AppConfig.COOKIE_KEY, AppConfig.COOKIE_KEY);
        KJLoger.debug(cookies);
        cookieManager.setCookie(url, cookies);
        CookieSyncManager.getInstance().sync();
    }

```

在使用网页版淘宝或百度登录时,WebView会自动登录上次的帐号!\(因为WebView 记录了帐号和密码的cookies\) 所以,需要清除 SessionCookie也是有必要的。 那么CookieManager同样也为我们提供了清除cookie的方法 CookieManager.getInstance\(\).removeSessionCookie\(\);

**一个自定义页面的思路**

以字符串的形式去加载一个已经获取到了的html源代码。这样做的好处在于显示的页面可以完全的根据自己喜好来定义，比如我想在末尾添加一张图片，那么简单，在这个html字符串的末尾插入一个img标签就可了。至于使用方法，其实我们在上一篇博客的时候有提到过：

myWebView.loadData\(htmlText, “text/html”, “utf-8”\);

其中htmltext就是我们需要加载的html字符串，使用这个方法可以直接将这个字符串作为网页来显示。

