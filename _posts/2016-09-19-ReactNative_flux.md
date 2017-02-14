---
layout: post
title: "ReactNative flux"
subtitle: ""
date: 2016-09-19 15:00:00
author: "steven"
catalog: true
tags:
    - ReactNative
---


react-native-router-flux 是React Native的一个基于Navigation API 的一个路由组件。

功能：

* 集中定义routers和scene
* 集成navigator,无需再在页面之间传递navigator
* 可以在任何地方简单方便地调用场景切换，如跳转到登录页面：Actions.login({username, password}

特点：

* 可以高度自定义Navigation bar
* 支持tab bar,如[react-native-tabs](https://github.com/aksonov/react-native-tabs)
* 支持嵌套的Navigators，如每个tab都可以有自己的Navigator
* 自定义Scene Renderers
* 动态路由，可以通用应用的不同状态动态选择要切换的场景
* 自己特有的Reducer
* 可以重置已经切换过的场景
* 强大的state控制机制

### 安装


```javascript
npm i react-native-router-flux --save
```

### 使用

在index.js中定义应用的场景，例如：


```javascript
import {Scene, Router} from 'react-native-router-flux';

class App extends React.Component {
  render() {
    return <Router>
      <Scene key="root">
        <Scene key="login" component={Login} title="Login"/>
        <Scene key="register" component={Register} title="Register"/>
        <Scene key="home" component={Home}/>
      </Scene>
    </Router>
  }
}
```

### Actions

定义完Router之后就可以使用Actions来跳转场景了

添加引用：

```javascript
import {Actions} from 'react-native-router-flux'
```

跳转到某个场景

```javascript
Actions.ACTION_NAME(PARAMS) //如跳转到login，Actions.login
```

返回上一个场景

```javascript
Actions.pop() //pop当前场景
   {popNum: [number]} //可以弹出多个场景
   {refresh: {...propsToSetOnPreviousScene}} //刷新并跳转某个场景
```

刷新当前场景：

```javascript
Actions.refresh(PARAMS)
```   


<br/>
<br/>

[#API文档](https://github.com/aksonov/react-native-router-flux/blob/master/docs/API_CONFIGURATION.md)
---
