---
layout:     post
title:      "Android&IOS自动打包上传到蒲公英"
subtitle:   ""
date:       2015-05-08 14:00:00
author:     "steven"
catalog:    true
tags:
    - Android
---

使用python调用系统的打包命令，打包完成之后将应用包通过curl上传到蒲公英。

python调用系统命令：
---

```java
import os
import sys

project_path = "..."
//查看版本状态
os.system('cd %s;git status' % project_path)
//拉取代码
os.system('cd %s;git pull' % project_path)
```

android studio项目打包命令：
---

```java
//clean
./gradlew clean
//debug 版本
./gradlew assembleDebug
//realse版本
./gradlew assembleRelease
//可以和clean 合起来用
./gradlew clean assembleRelease
```

默认会打包所有的渠道版本，可在productFlavors中配置，如：


```java
productFlavors {
    baidu {
      manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu"]
    }
    wandoujia {
      manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
    }
    yingyongbao {
      manifestPlaceholders = [UMENG_CHANNEL_VALUE: "yingyongbao"]
    }
  }
```

UMENG_CHANNEL_VALUE 为umeng统计的渠道的Placeholder

配置打包APK的存放路径：

```java
def releaseTime() {
  return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
}

applicationVariants.all { variant ->
       variant.outputs.each { output ->
         if (outputFile != null && outputFile.name.endsWith('.apk')) {
          def fileName = "app_v${defaultConfig.versionName}_${releaseTime()}_${variant.productFlavors[0].name}.apk"
          output.outputFile = new File(outputFile.parent, fileName)
        }
      }
   }
```
配置之后也可以打包指定的渠道：

```java
./gradlew assembleWandoujiaRelease
```

IOS打包：
-----

clean:

```java
xcodebuild clean
```

若没有使用cocoapods管理第三方类库:

```java
xcodebuild build   //默认build realse
```

若使用了cocoaPods管理第三方类库，则编译时需指定xcworkspace文件和scheme

```java
xcodebuild -workspace xxx.xcworkspace -scheme xxxx -configuration debug -derivedDataPath xxx(app文件存放的路径) ONLY_ACTIVE_ARCH=NO
```

执行后会生成app文件

打包ipa：

```java
xcrun -sdk iphoneos PackageApplication -v app路径 -o 存放ipa文件路径/文件名.ipa
```

上传到蒲公英
------

打包完成之后可以使用curl上传到蒲公英，然后通知测试人员开始进行测试
蒲公英开放了上传应用api

```java
curl -F "file=@上传文件路径" -F "uKey=用户key" -F "_api_key=API Key" http://www.pgyer.com/apiv1/app/upload
```

执行改命令之后可以自动完成上传发布，然后就可以测试了
