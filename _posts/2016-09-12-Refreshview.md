---
layout:     post
title:      "Refresh Header View"
subtitle:   ""
date:       2016-09-12 14:00:00
author:     "steven"
catalog: true
tags:
    - IOS
---

模仿锤子手机的下拉刷新效果.基于swift 3.0编写

demo:
---

![image]({{ site.baseurl }}/img/post/refresh_header.gif)

Usage:
---

```Swift
scrollView = UIScrollView(frame:self.view.bounds)
scrollView.addRefreshHeader(color: UIColor.blue) {
         //刷新数据
      }
```

刷新完成之后：

```Swift
scrollView.endRefreshing();
```

不通过下拉触发，直接开始刷新:

```Swift
scrollView.beginRefreshing();
```

应用或界面退出时，移除observer:

```Swift
scrollView.removeScrollObserver();
```

实现原理:
---

1.用聊条竖线和两条斜线实现两个箭头，然后根据下拉的距离移动两个箭头的位置，使两个箭头向相反的方向移动。并根据下拉的距离改变header view的高度

2.将两个直线箭头转换为两个圆弧，根据下拉距离画出对应长度的上下两个圆弧，并根据圆弧的长度移动箭头（斜线）的位置，并逐渐减少直线的长度，动起来看上去像是直线变成了圆弧

3.圆弧完成，直接长度为0，scrollView滚动到顶端时，开始播放旋转动画

4.刷新完成，向上移动隐藏refresh header.并重置状态


>[Source](https://github.com/StevenDXC/DxRefreshView)
