# 关于布局的小知识点 {#关于布局的小知识点}

## 1. 使用标签减少视图层次 {#1-使用标签减少视图层次}

[https://developer.android.com/training/improving-layouts/reusing-layouts.html](https://developer.android.com/training/improving-layouts/reusing-layouts.html)如下：可以通过引用文件名重复利用该布局，而不会增加View的层级

```
<
merge xmlns:android="http://schemas.android.com/apk/res/android"
>
<
Button
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/add"/
>
<
Button
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/delete"/
>
<
/merge
>
```



