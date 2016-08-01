---
layout:     post
title:      "Loading HUD"
subtitle:   ""
date:       2015-04-03 13:00:00
author:     "steven"
catalog:    true
tags:
    - IOS
---

####IOS Loading HUD，可以显示加载成功，失败，或无数据的状态

基于https://github.com/pkluz/PKHUD和https://github.com/iamim2/OneLoadingAnimation做了部分修改，去除了无用的代码和部分动画效果，并添加了无结果时的动画


Demo:

![image]({{ site.baseurl }}/img/post/loading_hud.gif)


使用:


show:

```swift
DxLoadingHUD.sharedInstance.show();
```

加载成功：

```swift
DxLoadingHUD.sharedInstance.showSuccessAnimation();
```

加载失败：


```swift
DxLoadingHUD.sharedInstance.showErrorAnimation();
```

无结果：

```swift
DxLoadingHUD.sharedInstance.showEmptyAnimation();
```

隐藏:

```swift
DxLoadingHUD.sharedInstance.hide(animated:true);
```




* source:https://github.com/StevenDXC/DxLoadingHUD
