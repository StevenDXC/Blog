---
layout:     post
title:      "LoadingStateView"
subtitle:   ""
date:       2015-04-18 13:00:00
author:     "steven"
catalog:    true
tags:
    - android
---

####android LoadingStateView，显示加载，加载成功，失败和无数据的状态

Demo:

![image]({{ site.baseurl }}/img/post/loading_state_view.gif)


layout:

```xml
 <com.dxc.loadingstateview.widget.LoadingStateView
        android:id="@+id/loading_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        app:view_size="large"
        app:color_empty="@color/empty_color"
        app:color_normal="@color/loading_color"
        app:color_success="@color/success_color"
        app:color_failed="@color/failed_color"/>
```


可以自定义：

```xml
大小：
view_size:large/small
状态颜色：
app:color_empty="@color/empty_color"
app:color_normal="@color/loading_color"
app:color_success="@color/success_color"
app:color_failed="@color/failed_color"
app:color_failed="@color/failed_color"
```

加载成功:

```java
mLoadingView.setViewState(LoadingStateView.STATE_SUCCESS);
```

加载失败:
```java
mLoadingView.setViewState(LoadingStateView.STATE_FAILED);
```

无数据时:

```java
mLoadingView.setViewState(LoadingStateView.STATE_EMPTY_RESULT);
```

loading:

```java
mLoadingView.setViewState(LoadingStateView.STATE_LOADING);
```





* source:https://github.com/StevenDXC/LoadingStateView


* swift版本:https://github.com/StevenDXC/DxLoadingHUD
