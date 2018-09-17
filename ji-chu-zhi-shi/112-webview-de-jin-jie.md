## 1. 资源预加载 {#1-资源预加载}

### 1.1 加载已知的资源 {#11-加载已知的资源}

可以在 APP 启动的时候就加载那些已知需要加载的资源。如果是使用前边提到的缓存类，直接注册相应的 URL，然后调用 load\(\) 方法即可。预加载的时机和位置后文再做介绍。

### 1.2 加载未知资源 {#12-加载未知资源}

以为视频或者大图才需要。

### 1.3 何时开始预加载 {#13-何时开始预加载}

先加载首页要展示的内容。

#### 1.3.1 如果是预加载已知的页面，重写 WebViewClient 子类的 onPageFinished\(\)方法即可。 {#131-如果是预加载已知的页面，重写-webviewclient-子类的-onpagefinished方法即可。}

```
@Override
public void onPageFinished(WebView view, String url) {
    super.onPageFinished(view, url);

    if("http://tutorials.jenkov.com/".equals(url)){
        this.urlCache.load("http://tutorials.jenkov.com/java/index.html");
    }
}

```

* URL
  [http://tutorials.jenkov.com/java/index.html](http://tutorials.jenkov.com/java/index.html)
  必须先在 UrlCache 类中注册。

#### 1.3.2 如果是加载未知页面，可参看下例 {#132-如果是加载未知页面，可参看下例}

* 注意是在 shouldInterceptRequest\(\) 方法中。

  ```
  @Override
  public WebResourceResponse shouldInterceptRequest(WebView view, String url) {

    if(url.startsWith("http://mydomain.com/article/") {
        String cacheFileName = url.substring(url.lastIndexOf("/"), url.length());
        this.urlCache.register(url, cacheFileName,
                "text/html", "UTF-8", 60 * UrlCache.ONE_MINUTE);

    }

    return this.urlCache.load(url);
  }

  ```

  上述方法只适用于可以从 URL 得知是否需要预加载的情况。对于不能从 URL 得知的情况，要么让网页调用 Java 方法，要么用 Java 代码调用 JavaScript 代码获取需要预加载的资源列表。

## 2. 在 H5 本地存储中保存缓存数据 {#2-在-h5-本地存储中保存缓存数据}

在 HTML5 中，可以在 JavaScript 对象 sessionStorage 或 localStorage 中缓存数据。其中，sessionStorage 中保存的数据是会话级别的，只在 WebView 打开期间有效，而 localStorage 是把数据持久化到本地。

```
WebSettings webSettings = webView.getSettings();

webSettings.setJavaScriptEnabled(true);
webSettings.setDomStorageEnabled(true);

```

上边三行代码应当放在 Activity 的 onCreate\(\) 方法中。

## 3. 通过webview控件下载文件 {#3-通过webview控件下载文件}

通常webview渲染的界面中含有可以下载文件的链接，点击该链接后，应该开始执行下载的操作并保存文件到本地中。webview来下载页面中的文件通常有两种方式：

### 3.1 自己通过一个线程写java io的代码来下载和保存文件（可控性好） {#31-自己通过一个线程写java-io的代码来下载和保存文件（可控性好）}

首先要写一个下载并保存文件的线程类

```
public class HttpThread extends Thread {


    private String mUrl;

    public HttpThread(String mUrl) {
        this.mUrl = mUrl;
    }

    @Override
    public void run() {
        URL url;
        try {
            url = new URL(mUrl);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setDoInput(true);
            conn.setDoOutput(true);
            InputStream in = conn.getInputStream();

            File downloadFile;
            File sdFile;
            FileOutputStream out = null;
            if(Environment.getExternalStorageState().equals(Environment.MEDIA_UNMOUNTED)){
                downloadFile = Environment.getExternalStorageDirectory();
                sdFile = new File(downloadFile, "test.file");
                out = new FileOutputStream(sdFile);
            }

            //buffer 4k
            byte[] buffer = new byte[1024 * 4];
            int len = 0;
            while((len = in.read(buffer)) != -1){
                if(out != null)
                    out.write(buffer, 0, len);
            }

            //close resource
            if(out != null)
                out.close();

            if(in != null){
                in.close();
            }



        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

```

随后要实现一个 DownloadListener 接口，这个接口实现方法 OnDownloadStart\(\)，当用户点击一个可以下载的链接时，该回调方法被调用同时传进来该链接的 URL，随后即可以将该 URL 传入 HttpThread 进行下载操作：

```
//创建DownloadListener (webkit包)
class MyDownloadListenter implements DownloadListener{

        @Override
        public void onDownloadStart(String url, String userAgent,
                String contentDisposition, String mimetype, long contentLength) {
            System.out.println("url ==== 
>
" + url);
            new HttpThread(url).start();
        }

    }

//给webview加入监听
webview.setDownloadListener(new MyDownloadListenter());

```

### 3.2 调用系统download的模块（代码简单） {#32--调用系统download的模块（代码简单）}

```
class MyDownloadListenter implements DownloadListener{

        @Override
        public void onDownloadStart(String url, String userAgent,
            String contentDisposition, String mimetype, long contentLength) {
            System.out.println("url ==== 
>
" + url);
            //new HttpThread(url).start();

            Uri uri = Uri.parse(url);
            Intent intent = new Intent(Intent.ACTION_VIEW, uri);
            startActivity(intent);
        }
    }

```

## 4. WebView 同步 cookie 的例子 {#4-webview-同步-cookie-的例子}

1.首先，假设登陆界面需要提供两个参数，一个是name，一个是pwd，那么要是对这个页面进行登陆，那么必须给与这两个信息。我们假设服务器已经注册了name为jason，pwd为123456这个账号。

2.Thread 用来将 name 和 pwd 自动登入，在服务器返回的 response 中获得 cookie 信息，稍后对这个 cookie进行保存。

```
public class HttpCookie extends Thread {

    private Handler mHandler;

    public HttpCookie(Handler mHandler) {
        this.mHandler = mHandler;
    }

    @Override
    public void run() {
        HttpClient client = new DefaultHttpClient();
        HttpPost post = new HttpPost("");//this place should add the login address

        List
<
NameValuePair
>
 list = new ArrayList
<
NameValuePair
>
();
        list.add(new BasicNameValuePair("name", "jason"));
        list.add(new BasicNameValuePair("pwd", "123456"));

        try {
            post.setEntity(new UrlEncodedFormEntity(list));
            HttpResponse reponse = client.execute(post);
            if(reponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK){
                AbstractHttpClient absClient = (AbstractHttpClient) client;
                List
<
Cookie
>
 cookies = absClient.getCookieStore().getCookies();

                for(Cookie cookie:cookies){
                    if(cookie != null){
                        //TODO
                        //this place would get the cookies
                        Message msg = new Message();
                        msg.obj = cookie;
                        if(mHandler != null){
                        mHandler.sendMessage(msg);
                        return;
                        }
                    }
                }
            }

        } catch (UnsupportedEncodingException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (ClientProtocolException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

```

由于这是一个子线程，所以需要在主线程中创建并执行。

同时，因为其实子线程，那么里面必须含有一个handler的元素，用来当成功获取cookie后通知主线程进行同步和保存。初始化这个子线程的时候需要将主线程上的handler给传过来，随后在以上代码的TODO中发送消息，让主线程记录cookie，发送的这个消息需要将cookie信息包含进去。

随后在主线程中（webview加载登陆界面前），在handler中将会获取到cookie信息，下面将对该cookie进行保存和同步：

```
private Handler mHandler = new Handler(){
        public void handleMessage(android.os.Message msg) 
        {

            CookieSyncManager.createInstance(MainActivity.this);
            CookieManager cookieMgr = CookieManager.getInstance();
            cookieMgr.setAcceptCookie(true);
            cookieMgr.setCookie("", msg.obj.toString());// this place should add the login host address(not the login index address)
            CookieSyncManager.getInstance().sync();

            webview.loadUrl("");// login index address
        };
    };

```

这个时候发现webview加载的login index页面中可以自动的登陆了并显示登陆后的界面。

* 另一个例子

如果是webview加載網頁登錄的話，在登錄成功後cookie就保存在了webview中，你訪問需要cookie驗證的玩也就不會有問題，如果你退出的話，webview中的cookie也會被清除掉，你下次訪問的時候需要驗證的cookie頁面就會出問題的！如果希望退出後在用戶沒有註銷用戶登錄的時候能夠順利訪問需要驗證的cookie的頁面，就需要往webview中放入登錄的cookie。我的建議是在用戶登錄後並未註銷登錄的情況下，程序通過httpClient進行登錄，並獲取登錄後的cookie，然後將cookie放入到webview中！

```

        public static Cookie getCookie() {
                Cookie cookie = null;
                DefaultHttpClient mHttpClient = (DefaultHttpClient) getHttpClient()（這個方法獲取的是HttpClient實例！）;
                List
<
Cookie
>
 cookies = mHttpClient.getCookieStore().getCookies();
                if (!cookies.isEmpty()) {
                        for (int i = 0; i 
<
 cookies.size(); i++) {
                                cookie = cookies.get(i);
                        }
                        return cookie;
                } else {
                        return cookie;
                }
        }

```

設個是獲取cookie的方法；

下面的代碼是在加載webView的Activity中寫的，this就是這個Activity的實例咯！

```

private boolean setWebCookie() {

                Cookie sessionCookie = JlxCustomerHttpClient.getCookie();
                CookieSyncManager.createInstance(this);
                CookieManager cookieManager = CookieManager.getInstance();

                if (sessionCookie != null) {
                        if (JlxApp.IS_LOGIN) {
                                String cookieString = sessionCookie.getName() + "="
                                                + sessionCookie.getValue() + "; domain="
                                                + sessionCookie.getDomain();
                                cookieManager.setCookie(Constant.UTILS_WEB（這個就是你需要訪問的網站首頁地址）, cookieString);
                                return true;
                        } else {
                                cookieManager.removeSessionCookie();
                        }
                        CookieSyncManager.getInstance().sync();
                }
                return false;
        }

```

這是設置cookie的代碼

* 又一个例子

```
new Thread(){
            @Override
            public void run() {
                try {
                    SharedPreferences spf = getSharedPreferences("Cookie", Context.MODE_PRIVATE);

                    HttpClient client = new DefaultHttpClient();
                    HttpGet get = new HttpGet(Gloable.DOLOAD+"code.gif");
                    HttpResponse response = client.execute(get);
                    Cookie cookie = ((AbstractHttpClient) client).getCookieStore().getCookies().get(0);
                    String sessionId = cookie.getValue();
                    SharedPreferences.Editor editor = spf.edit();
                    editor.putString("sessionId", sessionId);
                    String cookieString = cookie.getName()+"="+cookie.getValue()+
                            ";domain="+cookie.getDomain();
                    Log.e("test", "cookieString:"+cookieString);
                    editor.putString("cookieString", cookieString);
                    editor.commit();
                    Log.i("info", "b--JSESSIONID=" + sessionId);
                    if (response.getStatusLine().getStatusCode() == 200) {
                        byte[] bytes = EntityUtils.toByteArray(response.getEntity());
                        final Bitmap bitmap = BitmapFactory.decodeByteArray(bytes, 0, bytes.length);
                        runOnUiThread(new Runnable() {
                            public void run() {
                                Drawable drawable = new BitmapDrawable(bitmap);
                                iv_showCode.setBackgroundDrawable(drawable);
                            }
                        });
                    }
                } catch (ClientProtocolException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                } catch (IOException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }.start();

```

```
SharedPreferences spf = getSharedPreferences("Cookie", Context.MODE_PRIVATE);
        CookieSyncManager.createInstance(this);
        CookieManager cookieManager = CookieManager.getInstance();
        //cookieManager.setAcceptCookie(true);
        String cookieString = spf.getString("cookieString", "");
        cookieManager.setCookie(url, cookieString); //Sets a cookie for the given URL.
        // Any existing cookie with the same host, path and name will be replaced 
        //with the new cookie. The cookie being set must not have expired and must 
        //not be a session cookie, otherwise it will be ignored.
        CookieSyncManager.getInstance().sync();

        webview.loadUrl(url);

```

## 5.WebView与JavaScript相互调用混淆问题 {#5webview与javascript相互调用混淆问题}

混淆的时候已经把本地的代码的引用给打乱了，导致js中的代码找不到本地的方法的地址。

解决这个问题很简单，即在 proguard.cfg 文件中加上一些代码，声明本地中被 js 调用的代码不被混淆。下面举例说明：

被 js 调用的类 DemoJavaScriptInterface 的包名为 com.test.webview，那么就要在 proguard.cfg 文件中加入：

-keep public class com.test.webview.DemoJavaScriptInterface{ public; } 若是内部类，则大致写成如下形式：

-keep public class com.test.webview.DemoJavaScriptInterface$InnerClass{ public; } 若android版本比较新，可能还需要添加上下列代码：

-keepattributes_Annotation_  
-keepattributes_JavascriptInterface_

