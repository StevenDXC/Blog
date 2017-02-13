---
layout:     post
title:      "ReactNative Navigation"
subtitle:   ""
date:       2016-09-18 15:00:00
author:     "steven"
catalog: true
tags:
    - ReactNative
---


## NavigatorIOS


NavigatorIOS是IOS中UINavigationController的包装，功能基本和UINavigationController一致。

### 使用  


```javascript
<NavigatorIOS
    initialRoute={{
       component: MyScene,
       title: 'My Initial Scene',
       passProps:{ myProp: 'genius' },
   }}
   style={{flex: 1}}
/>
```

routes是用来传递每一个nav页面的，第一个路由是由NavigatorIOS中的initialRoute提供的.
title 页面的标题，即UINavigationBar的标题
component 即为当前页面要显示的组件
passProps 为要传递的属性（值），也可以在push的时候再放入，通过{this.props.myProp}获取值

### 压栈

```javascript
this.props.navigator.push()；
```

例如：

```javascript
this.props.navigator.push({
     component:PageSecond,
     title: 'Scene 2',
   });
```

### 推栈

```javascript
this.props.navigator.popN(N);
this.props.navigator.pop();//返回上一个页面
```

N 要返回的页面数量

其他方法：

```javascript
//替换当前页面，并立即刷新
this.props.navigator.replace(route);
//替换前一个页面
this.props.navigator.replacePrevious(route);
//返回到栈顶
this.props.navigator.popToTop();
//返回到某个页面
this.props.navigator.popToRoute(route);
//替换并返回到某个页面
this.props.navigator.replacePreviousAndPop(route);
//替换并pop到栈顶的页面
this.props.navigator.resetTo(route);
```

### Props：

```javascript

barTintColor: navigationbar的背景颜色
interactivePopGestureEnabled: 是否激活手势返回上一页
navigationBarHidden：是否隐藏navigationBar
shadowHidden：是否影藏navigationBar下方的1个像素高度的引用。
tintColor：navigationBar上button的颜色
titleTextColor：navigationBar标题的颜色
translucent：是否半透明
backButtonTitle: 返回按钮的标题
backButtonIcon: 返回按钮的图标
leftButtonTitle: 左边按钮的标题
rightButtonTitle: 右边边按钮的标题
leftButtonIcon: 左边按钮的图标
rightButtonIcon: 右边按钮的图标
onRightButtonPress/onLeftButtonPress:
左边或右边按钮的点击事件
```

## Navigator

Navigator是用来处理两个场景之间的切换，同时兼容IOS和Android.

### 初始化

```javascript
<Navigator
   style={{flex:1}}
    initialRoute={{ title: 'Awesome Scene', index: 0 }}
    renderScene={(route, navigator) =>
        <Text>Hello {route.title}!</Text>
   }
   style={{padding: 100}}
);
```

renderScene：渲染对应route的页面，将会被route和navigator调用

### 配置多个场景

```javascript
const routes = [
    {title: 'First Scene', index: 0},
    {title: 'Second Scene', index: 1},
  ];
...

<Navigator
    initialRoute={routes[0]}
    initialRouteStack={routes}
/>
```

### 自定义Navigation Bar 样式

```javascript
 navigationBar={
     <Navigator.NavigationBar
       routeMapper={{
         LeftButton: (route, navigator, index, navState) =>
          { return (<Text>Cancel</Text>); },
         RightButton: (route, navigator, index, navState) =>
           { return (<Text>Done</Text>); },
         Title: (route, navigator, index, navState) =>
           { return (<Text>Awesome Nav Bar</Text>); },
       }}
       style={{backgroundColor: 'gray'}}
     />
  }
```


### 转场动画和手势

```javascript
configureScene={(route, routeStack) =>
    Navigator.SceneConfigs.FloatFromBottom}
```

支持的所有动画及手势：

* Navigator.SceneConfigs.PushFromRight (default)
* Navigator.SceneConfigs.FloatFromRight
* Navigator.SceneConfigs.FloatFromLeft
* Navigator.SceneConfigs.FloatFromBottom
* Navigator.SceneConfigs.FloatFromBottomAndroid
* Navigator.SceneConfigs.FadeAndroid
* Navigator.SceneConfigs.SwipeFromLeft
* Navigator.SceneConfigs.HorizontalSwipeJump
* Navigator.SceneConfigs.HorizontalSwipeJumpFromRight
* Navigator.SceneConfigs.HorizontalSwipeJumpFromLeft
* Navigator.SceneConfigs.VerticalUpSwipeJump
* Navigator.SceneConfigs.VerticalDownSwipeJump

### Props

```javascript
configureScene：配置转场动画及手势
initialRouteStack：初始化已经声明的场景集合
onDidFocus：完成转场之后新场景中将被调用
onWillFocus：转场动画开始之前在新场景中被调用
```

### 方法


```javascript
//重置所有场景，已经存在的所有场景将被回收，
immediatelyResetRouteStack(nextRouteStack)
//跳转到某个场景
jumpTo(route)
//跳转到栈中的下个场景
jumpForward()
//跳转回上个场景
jumpBack()
//将下个场景压入栈中，并跳转过去
push(route)
//一次返回N个场景。N无效时无反应
popN(n)
//返回上个场景并卸载当前的场景
pop()
//替换index对应的场景
replaceAtIndex(route, index, callback)
//替换当前场景
replace(route)
//返回到栈中的第一个场景，并卸载所有场景
popToTop()
//跳转到某个场景，该场景之后的所有场景将被卸载
popToRoute(route)
//替换并pop到上一个场景
replacePreviousAndPop(route)
//跳转到一个新的场景，并重置route栈
resetTo(route)
//获取当前route的列表
getCurrentRoutes
```
