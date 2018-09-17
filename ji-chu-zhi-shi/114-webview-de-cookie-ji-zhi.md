[http://www.jianshu.com/p/24827940b21a](http://www.jianshu.com/p/24827940b21a)

#### 基于session的登录验证 {#基于session的登录验证}

在程序请求接口的时候判断服务器端是否有当前会话的session，如果没有则被认为没有登录。客户端没有session这一概念，但有cookie与其对应。每一个session都有一个session id作为唯一标识。在登录成功后服务器会在请求头中返回cookie，cookie包含着这次登录会话的session id，在接下来的请求中只需要将登陆返回的cookie设置到请求头中便可以通过验证。

### WebView的cookie机制 {#webview的cookie机制}

WebView是基于webkit内核的UI控件，相当于一个浏览器客户端。它会在本地维护每次会话的cookie\(保存在data/data/package\_name/app\_WebView/Cookies.db\)。

当WebView加载URL的时候,WebView会从本地读取该URL对应的cookie，并携带该cookie与服务器进行通信。 WebView通过android.webkit.CookieManager类来维护cookie。CookieManager是WebView的cookie管理类。

### 将cookie同步到WebView {#将cookie同步到webview}

##### 1. 登录时从服务器的返回头中取出cookie {#1-登录时从服务器的返回头中取出cookie}

使用不同连接类，取 cookie 的方法不同。以 HttpURLconnection 为例：

```
    String cookieStr = conn.getHeaderField("Set-Cookie");

```

##### 2. 将cookie同步到WebView中 {#2-将cookie同步到webview中}

```
 /** 
  * @param url WebView要加载的url
  * @param cookie 要同步的cookie
  * @return true 同步cookie成功，false同步cookie失败
  */
 public static boolean syncCookie(String url, String cookie) {
      if (Build.VERSION.SDK_INT 
<
 Build.VERSION_CODES.LOLLIPOP) {   
          CookieSyncManager.createInstance(context);
      }  
      CookieManager cookieManager = CookieManager.getInstance();
      cookieManager.setCookie(url, cookie);//如果没有特殊需求，这里只需要将session id以"key=value"形式作为cookie即可
      String newCookie = cookieManager.getCookie(url);
      return TextUtils.isEmpty(newCookie)?false:true;
}

```

* 注：
  1. 同步cookie要在WebView加载url之前，否则WebView无法获得相应的cookie，也就无法通过验证。
  2. 每次登录成功后都需要调用"syncCookie"方法将cookie同步到WebView中，同时也达到了更新WebView的cookie。如果登录后没有及时将cookie同步到WebView可能导致WebView拿的是旧的session id和服务器进行通信。



