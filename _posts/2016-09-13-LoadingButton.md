---
layout:     post
title:      "LoadingButton"
subtitle:   ""
date:       2016-09-13 09:00:00
author:     "steven"
catalog:    true
tags:
    - Android
---

android button视图动态转换为加载动画，并显示动画显示请求的结果（成功或失败）。

demo:
---

![image]({{ site.baseurl }}/img/post/loading_button.gif)

Usage:
---
layout:


```xml
<com.dx.dxloadingbutton.widget.LoadingButton
        android:id="@+id/loading_btn"
        android:layout_gravity="center"
        android:layout_width="228dp"
        android:layout_height="wrap_content"
        app:resetAfterFailed="true"
        app:rippleColor="#000000"
        app:text="@string/button_text"
        app:resetAfterFailed="true"
        />
```
resetAfterFailed:请求失败后重置视图显示，还原为Button视图

code:

```java
LoadingButton lb = (LoadingButton)findViewById(R.id.loading_btn);
lb.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                lb.startLoading(); //start loading
            }
});
```

请求成功之后，显示成功的动画：


```java
 lb.loadingSuccessful();
```

请求失败之后，显示失败的动画：

```java
 lb.loadingFiled();
```

重置界面的显示为Button状态

```java
 lb.reset();
```

取消loading动画，还原为Btton view

```java
 lb.cancelLoading();
```


>[source](https://github.com/StevenDXC/DxLoadingButton)
---
