---
layout: post
title: "IOS组件化的思考"
subtitle: ""
date: 2016-06-23 14:00:00
author: "steven"
catalog: true
tags:
    - IOS
---


如果app的业务模块比较多，开发人员再多的话，开发一个中大量的IOS App确实比较麻烦和繁琐，也不利于测试。

### 组件化实现要解决的问题：

中间件：

1.组件之间要相互独立

2.中间件要能传递普通和复杂(无法通过URL传递)数据

3.能够通过url启动相应的controller，并能定制跳转方式

4.url的注册须用代码或静态配置完成，须去中性化

组件服务(service）:

1.能在组件之间通信，传递复杂参数（尽量不要使用customModel）传递

3.只通过接口文件的统一基础库进行依赖

4.接口和实现类的对应注册用代码完成，由中间件去控制服务实现类的生成

IOS的组件化实现方案目前有两种：

1. 通过注册url Scheme -> open url来调用指定的模块

2. 通过注册URL或组件到中间路由控制器，然后由中间件根据协议生成对应target的来调用组件


## URL Scheme

通过URL scheme来组件化比较方便，只需在应用中注册URL，然后open对应的URL的来启动对应的模块。

方便是方便，但是也有缺点：

1.如果注册组件的实例，则需要实例常驻内存，占用资源

2.无法传递复杂的参数

3.无法动态注册新的组件

4.只注册class无法直接响应模块返回的事件或数据


## Router -> Target -> Action 或 protocol - class

### [CTMediator](https://github.com/casatwy/CTMediator)

建立一个中间组件，然后各个模块在该组件中组册class和URL，中间件通过URL解析到对应的target和action，然后调用对应的target，触发对应的action。

特点

* 注册约束通过category来实现，每个模块对应一个category，category里定义了模块下所有可能调用的场景和事件。
* 解析完url之后Runtime来触发对应target或对应的事件
* 拆分了远程调用和应用内调用

### [BeeHive](https://github.com/alibaba/BeeHive)


BeeHive是阿里开源的IOS模块化编程的框架实现方案,吸收了Spring框架Service的理念来实现模块间的API耦合

![image]({{ site.baseurl }}/img/post/beehive.jpg)

通过BHCore创建，注册模块，实现Service逻辑。BHModuleProtocol定义了Module的生命周期回调接口和系统事件，BHServiceManager收到事件之后会分给所有实现BHModuleProtocol相应接口的Module

特点：

1.支持静态，动态注册组件

2.各个模块相互独立，并具有自己的生命周期

2.支持异步加载模块

3.扩展了系统事件，更加细化方便设置，初始化

4.支持自定义事件

BeeHive实现方式比较优雅，但也比较复杂。上手难度比较大，模块多的app维护成本也比较高。而且还在完善之中



## 关于去Model化

customModel方便了开发，但是在不同组件传递却成了问题。若将model转化为字典去传递，开发者又无法很直观的看到字典的内容。
若customModel仅存在一个组件内部，不对其它组件产生依赖，则无需去model化。
但是现实中有一些customModel是多模块必须依赖的，这时候就必须得通过组件服务层去处理对应数据类型的传递


## 事件传递

一般的中间件只负责组件之间的相互调用，并不负责组件之间的相互的事件传递。
常用事件传递的实现方式有：

1.通过action或block传递

2.通过Notification传递

3.通过注册protocol来实现

使用action传递相对简单一些，通过Notification传递则需要考虑传递数据的复杂性，要么添加约束，要么事先添加声明。
