---
layout:     post
title:      "Android DataBinding Demo"
subtitle:   ""
date:       2016-08-13 17:00:00
author:     "steven"
catalog:    true
tags:
    - Android
---

早就看到Google出了一个MVVM的框架，找个机会上手了下，发现用起来没那么复杂，所以就写了一个简单的Demo。

![image]({{ site.baseurl }}/img/post/databinding.gif)

这个Demo的内容很简单，就是获取github的repo列表，点击repo列表某项的时候跳转到对应的项目页面（HTML）。

使用到的第三方类库有：

· [RxJava](https://github.com/ReactiveX/RxJava)

· [RxAndroid](https://github.com/ReactiveX/RxAndroid)

· [Retrofit](https://github.com/square/retrofit)

· [RxBus](https://github.com/AndroidKnife/RxBus)


ViewModel
---
MainViewModel
· 用来控制加载页面的显示和隐藏，使用ObservableField来控制需要变化显示状态的view的状态，以及获取数据出错后的重试逻辑。加载数据完成之后自动隐藏加载布局，显示出repo列表
· 获取repo列表，使用Retrofit调用接口，通过GsonConverter和RxJavaCallAdapter将接口转换为RxJava的Observable对象。然后使用RxBus将数据和事件传递到activity。activity刷新RecyclerView的数据

RepoListViewModel

repo列表数据对应的ViewModel,继承BaseObservable，用来处理具体要显示的字段，以及和界面显示的绑定，更新。还有就是列表项的点击事件，点击之后跳转进入详情页面

Layout
---

layout 比较简单，只使用了基本的字段绑定和点击事件绑定，详细的请看源码




[Source](https://github.com/StevenDXC/AndroidDataBindingDemo)
---
