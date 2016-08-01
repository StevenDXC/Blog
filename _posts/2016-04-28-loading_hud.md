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
