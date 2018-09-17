[http://www.jianshu.com/p/e3965d3636e7](http://www.jianshu.com/p/e3965d3636e7)

```
public class WebViewFragment extends AbsTitleFragment {
  ......

        R.layout.fragment_web_view;
  ......

    @Override
    protected void initViewsAndEvents(View rootView) {
        mWebView.setVisibility(View.INVISIBLE);
        toggleShowLoading(true, "loading");

        setWebViewOption(mWebView);
        if (getFragmentUrl() != null) {
            mWebView.loadUrl(getFragmentUrl());
        }
    }

    private void setWebViewOption(WebView webview) {
     ......
        //设置是否支持运行JavaScript，仅在需要时打开
        webview.getSettings().setJavaScriptEnabled(true);

        ......
        //设置WebViewClient
        webview.setWebViewClient(new MyWebViewClient());
        webview.setWebChromeClient(new MyWebChromeClient());
    }

    private class MyWebViewClient extends WebViewClient {

        //
        @Override
        public boolean shouldOverrideUrlLoading(WebView view, String url) {
//            view.loadUrl(url);
            Intent intent = new Intent(mContext, WebViewActivity.class);
            intent.putExtra(WebViewActivity.WEB_VIEW_URL, url);
            startActivity(intent);
            return true;
        }
  ......

        @Override
        public void onPageFinished(WebView view, String url) {

            super.onPageFinished(view, url);
            toggleShowLoading(false, "");
            if (view.getVisibility() == View.INVISIBLE) {
                view.setVisibility(View.VISIBLE);
            }
        }

   ......
    }


    private class MyWebChromeClient extends WebChromeClient {
    ......
        @Override
        public void onProgressChanged(WebView view, int newProgress) {
            if (newProgress 
>
 25) {
                injectJS(view);
            }

            super.onProgressChanged(view, newProgress);
        }
    }


    private void injectJS(WebView webview) {
        webview.loadUrl("javascript:(function() " +
                "{ " +
                "document.getElementsByClassName('m-top-bar')[0].style.display='none'; " +
                "document.getElementsByClassName('m-footer')[0].style.display='none';" +
                "document.getElementsByClassName('m-page')[0].style.display='none';" +
                "})()");
    }
}
```



