---
layout:     post
title:      "Floating label textfield"
subtitle:   ""
date:       2015-04-03 13:00:00
author:     "steven"
catalog:    true
tags:
    - IOS
---

Material design风格的输入框的IOS版本，swift语言实现

预览:

![image](https://github.com/StevenDXC/DxMDTextField/blob/master/img/demo.gif)

使用:


初始化：

```swift
let textField = MDTextField.init(frame: CGRectMake(x, y, width, height))

```

设置高亮颜色

```swift
textField.highLightColor = UIColor.blueColor()
```

显示校验失败的错误提示：

```swift
textField.setErrorMsg("error message")
```

隐藏错误提示

```swift
textField.setErrorMsg(nil)
```

显示已输入的字符数和最大能输入的字符数量：

```swift
textField.showMaxInputLength(true)
textField.maxInputLength = 20;
```

#source:https://github.com/StevenDXC/DxMDTextField
