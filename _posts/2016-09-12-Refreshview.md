---
layout:     post
title:      "模仿锤子阅读的下拉刷新"
subtitle:   ""
date:       2016-09-12 14:00:00
author:     "steven"
catalog:    true
tags:
    - IOS
---

模仿锤子阅读的下拉刷新效果

Demo:
---

![image]({{ site.baseurl }}/img/post/refresh_header.gif)

Usage:
---

```Swift
//oc
_scrollView = [[UIScrollView alloc] initWithFrame:self.view.bounds];
DxRefreshView *refreshHeader = [[DxRefreshView alloc] init];
refreshHeader.color = [UIColor blueColor];
refreshHeader.actionHandler = ^{
     //刷新数据
};
_scrollView.refreshHeader = refreshHeader;

//swift
let refreshHeader:DxRefreshView = DxRefreshView();
refreshHeader.color = UIColor.blue;
refreshHeader.actionHandler = {
    //刷新数据
};
tableView.setRefreshHeader(refreshHeader: refreshHeader);
```

刷新完成之后：

```Swift
//oc
[_scrollView.refreshHeader endRefreshing];
//swift
scrollView.endRefreshing();
```

不通过下拉触发，直接开始刷新:

```Swift
//oc
[_scrollView.refreshHeader beginRefreshing];
//swift
scrollView.beginRefreshing();
```



Source
---

>[Swift 3](https://github.com/StevenDXC/DxRefreshView)

>[Objective-c](https://github.com/StevenDXC/DxRefreshView_OC)
