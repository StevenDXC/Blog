---
layout:     post
title:      "List Index View"
subtitle:   ""
date:       2015-04-15 9:00:00
author:     "steven"
catalog:    true
tags:
    - Android
---

#### android 列表索引View，点击或移动有放大效果

 ![image]({{ site.baseurl }}/img/post/list_index_view.gif)



布局:

```xml
<com.dxc.listindexview.widget.ListIndexView
        android:id="@+id/index_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="right|center_vertical" />
```

*索引view的宽度是match_parent*

代码:

```java
indexView.setIndexLetters(String[]{"A","B",....});
indexView.setOnTouchIndexListener(new ListIndexView.OnTouchIndexListener() {
            @Override
            public void onTouchIndex(int index) {
                
            }
        });
```

*在Listener中实现触摸某个索引之后的逻辑，例如列表滚动到索引对应的Item*

索引数量较少时，索引会垂直居中，并且限制了字体大小和mei'ge每个suo'yin高度


如：


 ![image]({{ site.baseurl }}/img/post/list_index_view2.gif)
 
 
* ![source]: https://github.com/StevenDXC/ListIndexView
 

