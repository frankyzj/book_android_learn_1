### 常用的WebSetting API {#常用的websetting-api}

```
setAllowFileAccess //启用或禁用WebView访问文件数据

setBlockNetworkImage //是否显示网络图像

setBuiltInZoomControls //设置是否支持缩放

setCacheMode //设置缓冲的模式

setDefaultFontSize //设置默认的字体大小

setDefaultTextEncodingName //设置在解码时时候用的默认编码

setFixedFontFamily  //设置固定使用的字体

setJavaScriptEnabled //设置是否支持Javascript

setLayoutAlgorithm //设置布局方式

setSupportZoom //设置是否支持变焦

void setWebViewClient(WebViewClient client)  //设置将接收各种通知和请求的WebViewClient。

```

### WebViewClient 辅助 WebView 处理各种通知、请求事件，常用方法包括： {#webviewclient-辅助-webview-处理各种通知、请求事件，常用方法包括：}

```
doUpdateVisitedHistory(WebView view, String url ,boolean isReload)//更新历史记录

onFormResubmission(WebView view, Message dontResend, Message resend)//应用程序重新请求页面数据

onLoadResource(WebView view, String url)//在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次

onPageStarted(WebView view, String url, Bitmap favicon)//这个事件就是开始载入页面调用的，通常我们可以在这个设定一个loading的页面，告诉用户程序在等待网络相应。

onPageFinished(WebView view, String url)//在页面加载结束时调用，同样道理，我们知道一个页面载入完成，于是我们可以关闭loading条，切换程序动作。

onReceivedError(WebView view, int errorCode, String description, String failingUrl)//报告错误信息

onReceivedHttpAuthRequest(WebView view, HttpAuthHandler handler, String host,Stirng realm)//获取返回信息授权请求

onScaleChanged(WebView view, float oldScale, float newScale)//WebView 发生改变时调用

onUnhandledKeyEvent(WebView view, KeyEvent event)//key事件未被加载时调用

shouldOverrideUrlLoading//并不是每次都在onPageStarted之前开始调用的，就是说一个新的URL不是每次都经过shouldOverrideUrlLoading的，只有在调用webview.loadURL的时候才会调用。

```

### 一个例子 {#一个例子}

```
    private class webViewClient extends WebViewClient {

        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            //拦截网页跳转
            if (url != null 
&
&
 url.contains("/user/setting")) {
                Intent intent = new Intent(MainActivity.this, SettingActivity.class);
                startActivity(intent);
                return true;
            //拦截网页拨打电话逻辑，调用本地拨打电话
            }else if (url.startsWith("tel:")) {
                Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
                startActivity(intent);
                return true;
            //拦截网页版支付
            }else if (url.contains("/payment/pay")){
                int currentapiVersion=android.os.Build.VERSION.SDK_INT;
                if (currentapiVersion
<
19) {
                    String call = "javascript:sumAndOidJava()";
                    webView.loadUrl(call);
                }else {
                    getPriceEvaluateJavascript(webView);
                }
                return true;

            }else {
                //给WebView添加标志位
                currentURL = url;
                if (currentURL.contains("?")) {
                    app_url = "
&
aa=" + appId + "
&
bb=" + token;
                } else {
                    app_url = "?aa=" + appId + "
&
bb=" + token;
                }
                view.loadUrl(currentURL + app_url);
            }
            return true;
        }

        //调用Js的getSumAndOid()方法
        @TargetApi(Build.VERSION_CODES.LOLLIPOP)
        private void getPriceEvaluateJavascript(WebView webView) {
            webView.evaluateJavascript("getSumAndOid()", new ValueCallback
<
String
>
() {
                @Override
                public void onReceiveValue(String value) {
                    String total=value.replace("\"","");
                    if (total.contains("*")) {
                        String price = total.substring(0, total.indexOf("*"));
                        String orderId = total.substring(total.indexOf("*") + 1, total.length());
                        pay(orderId, price);
                    }else {
                        Toast.makeText(MainActivity.this, "网络不给力,请稍后再试", Toast.LENGTH_SHORT).show();
                    }
                   }});

        }


        //拦截网页里面的控件
        @Override
        public void onLoadResource(WebView view, String url) {

            view.loadUrl("javascript:function app_setTop(){document.getElementsByClassName('banner-Bottom')[0].style.display = 'none';}app_setTop();");
            view.loadUrl("javascript:function app_setTop(){document.getElementsByClassName('detial-serve-content')[0].style.display = 'none';}app_setTop();");

                super.onLoadResource(view, url);

        }

        @Override
        public void onReceivedError(WebView view, int errorCode,String description, String failingUrl) {
            super.onReceivedError(view, errorCode, description, failingUrl);
            //这里进行无网络或错误处理，具体可以根据errorCode的值进行判断，做跟详细的处理。

            errorUrl = failingUrl;
            if (loadingDialog.isShowing()) {
                loadingDialog.dismiss();
            }

        }

        @Override
        public void onPageFinished(WebView view, String url) {
        //在这里拦截网页控件不如onLoadResource效果快好
            //view.loadUrl("javascript:function app_setTop(){document.getElementsByClassName('banner-Bottom')[0].style.display = 'none';}app_setTop();");
            //view.loadUrl("javascript:function app_setTop(){document.getElementsByClassName('detial-serve-content')[0].style.display = 'none';}app_setTop();");
            //给网页里面的参数abc和def赋值
            view.loadUrl("javascript:function app_setAa(){abc='" + appId + "';}app_setAa();");
            view.loadUrl("javascript:function app_setBb(){def='" + token + "';}app_setBb();");
        }
    }

```

### WebChromeClient解析 {#webchromeclient解析}

WebChromeClient是辅助WebView处理JavaScript的对话框，网站图标，网站title，加载进度等：

```
    onCloseWindow //关闭WebView

    onCreateWindow //创建WebView

    onJsAlert //处理Javascript中的Alert对话框

    onJsConfirm//处理Javascript中的Confirm对话框

    onJsPrompt //处理Javascript中的Prompt对话框

    onProgressChanged //加载进度条改变

    onReceivedlcon //网页图标更改

    onReceivedTitle //网页Title更改

    onRequestFocus WebView //显示焦点

```

### 又一个例子 {#又一个例子}

场景：当H5在调用本地相册的时候，没有反应，重写了打开本地文件的openFileChooser方法并做了多方兼容，最后调用本地的对话框提示用户进行选择拍照还是相册。

```
    private class SelfChromeClient extends WebChromeClient {
        // file upload callback (Android 2.2 (API level 8) -- Android 2.3 (API level 10)) (hidden method)
        public void openFileChooser(ValueCallback
<
Uri
>
 filePathCallback) {
            handle(filePathCallback);
        }

        // file upload callback (Android 3.0 (API level 11) -- Android 4.0 (API level 15)) (hidden method)
        public void openFileChooser(ValueCallback filePathCallback, String acceptType) {
            handle(filePathCallback);
        }

        // file upload callback (Android 4.1 (API level 16) -- Android 4.3 (API level 18)) (hidden method)
        public void openFileChooser(ValueCallback
<
Uri
>
 filePathCallback, String acceptType, String capture) {
            handle(filePathCallback);
        }

        // for Lollipop
        @Override
        public boolean onShowFileChooser(WebView webView, ValueCallback
<
Uri[]
>
 filePathCallback, android.webkit.WebChromeClient.FileChooserParams fileChooserParams) {
            // Double check that we don't have any existing callbacks
            if (mFilePathCallbackArray != null) {
                mFilePathCallbackArray.onReceiveValue(null);
            }
            mFilePathCallbackArray = filePathCallback;
            showDialog();
            return true;
        }

        /**
         * 处理5.0以下系统回调
         *
         * @param filePathCallback
         */
        private void handle(ValueCallback
<
Uri
>
 filePathCallback) {
            if (mFilePathCallback != null) {
                mFilePathCallback.onReceiveValue(null);
            }
            mFilePathCallback = filePathCallback;
            showDialog();
        }


        /**
         * 显示照片选取Dialog
         */
        public void showDialog() {
            if (ip == null) {
                ip = new ImagePick(MainActivity.this);
            }
            ip.setCancel(new ImagePick.MyDismiss() {
                @Override
                public void dismiss() {
                    handleCallback(null);
                }
            });
            ip.show();
        }

    }
```



