---
layout:     post
title:      "Android Small"
subtitle:   ""
date:       2016-05-05 14:00:00
author:     "steven"
catalog:    true
tags:
    - Android
---


关于Small
---

Small是2015年底新出来的一个插件化App开发方案，适用于将一个APK拆分为多个公共库插件、业务模块插件的场景。

Small的实现方式：

. Small的gradle插件生成的是.so包，在初始化的时候会通过.so文件生成zip文件，在由zip文件生成Dex,反射添加到宿主类加载器的dexPathList里面

. 资源文件处理：Small插件的资源是可以共享的，所有插件可以通过共用一个AssetManager互相访问资源文件，而且通过修改aapt解决了资源Id冲突的问题。

. 之前的插件化方案都是到宿主工程里先注册Activity占位，然后通过占位Activity将生命周期回调传递给插件的Activity.而Small则是通过替换Activity里    的mInstrumentation对象，在Instrumentation的newActivty实现中实例化了插件的Activity.

Small的优点：

1. 所有插件可内置于宿主包中

2. 插件可以独立开发与调试

3. 资源彻底分离，并且可以互相访问

4. 通过URI，宿主可以启动本地化插件，web组件，现在网页和任何自定义的插件，并传递参数

5. 跨平台，目前已经支持android,IOS及Html5

使用：

1.使用Android studio创建一个项目


2.添加small,在Project的build.gradle中添加


```java
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.0'
        classpath 'net.wequick.tools.build:gradle-small:1.1.0-beta3'
    }
}

apply plugin: 'net.wequick.small'
```

3.创建Module:
  因为 Small会根据包名对插件进行归类，所以要求：
    
  >模块命名：”app.\*” ,”lib.\*” 或 “web.\*”
  
  >包名包含：“.app.”, “.lib.” 或 “.web.”

  >lib.* 为Android Library模块，app.*为Phone & Tablet模块

4.配置签名
 在app模块的build.gradle中增加签名配置

```java
signingConfigs {
    release {
        storeFile file('../sign/release.keystrore’)
        storePassword “…..”
        keyAlias “…”
        keyPassword “…”
    }
}
buildTypes {
    release {
        signingConfig signingConfigs.release
    }
}
```

5.在app模块增加共享的依赖库,例如support类库
 6.在app模块的Application中配置：

 ```java
@Override
public void onCreate() {
   super.onCreate();
   Small.preSetUp(this);
}
```    

7.在app模块的assert/bundle.json中声明插件，格式：

```xml
{
  "version": "1.0.0",
  "bundles": [
    {
      "uri": "lib.utils",
      "pkg": "net.wequick.example.small.lib.utils"
    },
    {
      "uri": "app.module1”,
      "pkg": "com.example.mysmall.lib.style"
    }
    {
      "uri": "home",
      "pkg": "net.wequick.example.small.app.home"，
      "rules"{
          "page1",".MyPage1",
          "page2","net.wequick.example.small.app.home.MyPage2"
       }
    }
}
```

Small 跳转插件页面是通过uri来指定的，一个uri唯一对应一个插件. pkg为包名,若一个插件有多个页面供其他插件或宿主调用，则用rules区分。

例如：

启动home插件界面：

```java
Small.openUri("home", context);
```

启动home插件中的page1:

```java
Small.openUri("home/page1", context);
```

调用的时候也可以传参数：

```java
Small.openUri("home？from=main", context);
```


获取参数：

```java
Uri uri = Small.getUri(this);
if (uri != null) {
    String from = uri.getQueryParameter("from");
}
```

调用Fragment:

```java
Fragment fragment = Small.createObject("fragment-v4", uri, context);
```

获取Intent:

不是直接打开界面,而是从别的地方进入页面，如点击通知

```java
Intent intent = Small.getIntentOfUri("main",context)
```

8.编译

  编译插件：

```java
./gradlew Build libraries  -q
```
  -q 为安静模式，可以忽略

   打包所有组件：

```java
./gradlew buildBundle
```

  编译单个组件：

```java
./gradlew :app.main:assembleRelease
```

 然后就可以运行App了
