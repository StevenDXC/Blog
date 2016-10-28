---
layout:     post
title:      "LoadingView"
subtitle:   ""
date:       2016-09-15 11:00:00
author:     "steven"
catalog: true
tags:
    - Android
---

android Loading view animation

demo:
---

![image]({{ site.baseurl }}/img/post/loading_view.gif)

Usage:
---
layout:


```xml
<com.dx.dxloadingview.DxLoadingView
       android:id="@+id/loading_view"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:layout_gravity="center"
       app:loadingColor="@color/colorPrimary"
       app:animationDuration="2400"
       app:loadingViewSize="large"
       />
```
animationDuration:单次动画的时间长度

开始动画：

```java
DxLoadingView loadingView = (DxLoadingView) findViewById(R.id.loading_view);
loadingView.startAnimation();
```

取消加载动画:

```java
loadingView.cancelAnimation();
```



参考设计:https://www.uplabs.com/posts/liquid-motion-marketing



>[source](https://github.com/StevenDXC/DxLoadingView)
---
