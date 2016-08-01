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

success:
```swift
DxLoadingHUD.sharedInstance.showSuccessAnimation();
```

failed:
```swift
DxLoadingHUD.sharedInstance.showErrorAnimation();
```

empty:
```swift
DxLoadingHUD.sharedInstance.showEmptyAnimation();
```

hide:
```swift
DxLoadingHUD.sharedInstance.hide(animated:true);
```


* source:https://github.com/StevenDXC/DxLoadingHUD
