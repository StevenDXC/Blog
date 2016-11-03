---
layout:     post
title:      "Android DataBinding"
subtitle:   ""
date:       2016-08-12 13:00:00
author:     "steven"
catalog: true
tags:
    - IOS
---

Android DataBinding 是google在android上对MVVM设计模式的一个实现框架，目前还在测试阶段，稳定性，性能和兼容性还存在一些问题，所以目前在稳定版本的App上应用的还很少。

Android DataBinding主要实现了View和ViewModel的双向绑定，包括用户的响应。并且实现了自动更新。

Android DataBinding是一个support library,可以兼容android2.1(API Level 7)及以上的版本，开发需要使用Gradle1.5.0-alphal或更高版本的插件


开发环境
---

在你的module中的build.gradle文件中添加dataBinding配置：


```java
android {  
    ....  
    dataBinding {  
        enabled = true  
    }  
}  
```

添加完成之后，使用gradle同步项目，会自动下载支持类库。


布局
---

```xml
<?xml version="1.0" encoding="utf-8"?>  
<layout xmlns:android="http://schemas.android.com/apk/res/android">  
   <data>  
       <variable name="user" type="com.example.User"/>  
   </data>  
   <LinearLayout  
       android:orientation="vertical"  
       android:layout_width="match_parent"  
       android:layout_height="match_parent">  
       <TextView android:layout_width="wrap_content"  
           android:layout_height="wrap_content"  
           android:text="@{user.firstName}"/>  
       <TextView android:layout_width="wrap_content"  
           android:layout_height="wrap_content"  
           android:text="@{user.lastName}"/>  
   </LinearLayout>  
</layout>  
```

Android DataBinding的布局文件必须与<layout>标签为起点。<data>标签内就是这个布局要绑定的数据。
variable表示一个可能会在这个布局中作用的属性。
一个<data>标签内可以包含多个variable标签，即一个布局可以绑定多个数据对象类
variable的name就是这个布局中引用该数据对象的名称，type为该数据对象对应的类，通常为包名+类名
布局属性的设置使用“@{ }”语法


数据对象
---
POJO：

```java
public class User {  
   public final String firstName;  
   public final String lastName;  
   public User(String firstName, String lastName) {  
       this.firstName = firstName;  
       this.lastName = lastName;  
   }  
}  
```

常在应用程序的数据读取一次，不会改变.还可以改为javabean对象:

```java
public class User {  
   private final String firstName;  
   private final String lastName;  
   public User(String firstName, String lastName) {  
       this.firstName = firstName;  
       this.lastName = lastName;  
   }  
   public String getFirstName() {  
       return this.firstName;  
   }  
   public String getLastName() {  
       return this.lastName;  
   }  
}  
```

从数据绑定的角度来看,这两个类是没什么区别的。用于设置TextView的android:文本的表达式@{user.firstName}将访问第一个类中的firstName字段和后一个类的getFirstName()方法
另外,如果对象中firstName()方法存在，它还将访问firstName()方法。



数据绑定
---


```java
@Override  
protected void onCreate(Bundle savedInstanceState) {  
   super.onCreate(savedInstanceState);  
   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);  
   User user = new User("Test", "User");  
   binding.setUser(user);  
}  
```
默认情况下,绑定类的名称是基于布局文件的名称生成的,它是将布局文件名开头大写并加上“Binding”而成。这个类拥有所有从属性(例如用户变量)到布局的绑定关系并知道如何赋值绑定表达式。最简单的方法创建绑定的方法就是通过反射。



事件处理
---

事件出来主要是处理用户交互的事件响应，如点击，长按，控件的拖动等，有两种方式来实现事件处理：

Method References: 和android的现在的方式类似，在layout文件中绑定事件对应的方法。
如：

```java
public class MyHandlers {
    public void onClickFriend(View view) { ... }
}
```


```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.Handlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```

Listener Bindings: 和Method References很相似，不过可以使用lambda表达式传递任意的数据。不过只有Gradle 2.0及以上才支持。

如：

```java
public class Presenter {
    public void onSaveClick(Task task){}
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
 <layout xmlns:android="http://schemas.android.com/apk/res/android">
     <data>
         <variable name="task" type="com.android.example.Task" />
         <variable name="presenter" type="com.android.example.Presenter" />
     </data>
     <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
         <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
         android:onClick="@{() -> presenter.onSaveClick(task)}" />
     </LinearLayout>
 </layout>
 ```

 如果点击事件执行的方法中要使用当前的View，也可以通过参数传过去：

```xml
 android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```
在lambda表达式中可以传递多个参数。如果表达是中参数无NULL，Data Binding会返回默认个的值。表达式中还可以使用void的关键字

```xml
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

避免复杂的listener表达式，会使代码难以阅读和维护，表达式应该只传递有效的参数。


布局细节
---

1.import

import和Java的import的功能类似，在layout文件中使用该标签导入对应的类就可以使用该类的属性和方法

```xml
<data>
    <import type="android.view.View"/>
</data>
```
在布局中就可以使用view的属性和方法

```xml
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```


```xml
<data>  
    <import type="com.example.User"/>  
    <import type="java.util.List"/>  
    <variable name="user" type="User"/>  
    <variable name="userList" type="List<User>"/>  
</data>  
```

导入类型时也可以使用类型的静态方法和属性：

```xml
<data>  
    <import type="com.example.MyStringUtils"/>  
    <variable name="user" type="com.example.User"/>  
</data>  
…  
<TextView  
   android:text="@{MyStringUtils.capitalize(user.lastName)}"  
   android:layout_width="wrap_content"  
   android:layout_height="wrap_content"/>  
```

变量：

在<data>标签中可以声明多个不同类型的变量

```xml
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```

在编译时会检查引入的变量，如果一个变量继承了Observable或Observable集合对象，那会通过反射得到，如没有实现Observable接口，则数据的变化不会被观察。


自定义绑定类型名称

绑定类可以被重命名或放在不同的包中通过class标签

```xml
<data class="ContactItem">
    ...
</data>
```

生成绑定类ContactItem将会存在module包下的databinding包中。如果想让生成的类放在module中的另一个包下，可以加入前缀：


```xml
<data class=".ContactItem">  
    ...  
</data>
```

也可以使用包名的全称：

```xml
<data class="com.example.ContactItem">  
    ...  
</data>  
```

includes

和layout的include标签一致，可以在一个布局中引入另一个布局,并绑定数据：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```
merge标签不支持include数据绑定


表达式语法

表达式语法跟java语法很像，下面的一样的语法：

· 数学计算 + - / * %

· 字符串连接 +

· 逻辑运算符&& ||

· 位运算& | ^

· 一元运算 + - ! ~

· 位移>> >>> <<

· 比较== > < >= <=

· instanceof

· Grouping ()

· 文字 - character, String, numeric, null

· Cast

· 方法调用

· 字段访问

· 数组访问 [ ]

· 三元运算符?

如：

```xml
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```

不支持的：

· this
· super
· new
· Explicit generic invocation

NULL 判断：

```xml
android:text="@{user.displayName ?? user.lastName}"
```
与下面的一致：

```xml
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

集合：

```xml
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String&gt;"/>
    <variable name="sparse" type="SparseArray&lt;String&gt;"/>
    <variable name="map" type="Map&lt;String, String&gt;"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
```

Map:


由于Map的key为字符串，在XML中引用map的子项也是双引号，所以可以使用单引号和`：

```xml
android:text='@{map["firstName"]}'  
//or
android:text="@{map[`firstName`}"  
android:text="@{map["firstName"]}"  
```

Resources：

与布局一致，可以使用表达式：

```xml
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```

字符串格式化：

```xml
android:text="@{@string/nameFormat(firstName, lastName)}"  
```
复数：

```xml
android:text="@{@plurals/banana(bananaCount)}"
```

如果复数有多个参数，那所有参数都应该传入：

```xml
Have an orange
Have %d oranges
android:text="@{@plurals/orange(orangeCount, orangeCount)}"
```

数据对象

要使数据对象可监控，必须实现Observable接口，有三种不同的可观察的数据机制：Observable objects, observable fields, and observable collections.


Observable objects：

一个可以监控对象的接口实现，可以把监听器和对象绑定，可以监听所有属性的变化

要实现一个可监控数据对象，该对象(POJO)需继承BaseObservable对象：

```java
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   @Bindable
   public String getLastName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
```
@Bindable是一个注解，来标注该属性是可以监控的，在编译时会生成BR类，放在module的包下
数据发生改变时，应该在setter方法用调用notifyPropertyChanged方法来通知监听器。


 ObservableFields：

 如果一个类中只有几个变量需要监控，那可以不集成BaseObservable类，使用ObservableFields来监控。
 一些基本类型的字段监控类： ObservableBoolean, ObservableByte, ObservableChar, ObservableShort, ObservableInt, ObservableLong, ObservableFloat, ObservableDouble, 和ObservableParcelable.

 ObservableFields是一个独立的可监控对象，只能监控一个属性。原始的版本使用时为了避免装箱和拆箱，使用public final 修饰


```java
 private static class User {
   public final ObservableField<String> firstName =
       new ObservableField<>();
   public final ObservableField<String> lastName =
       new ObservableField<>();
   public final ObservableInt age = new ObservableInt();
}
```


赋值和获取：


```java
user.firstName.set("Google");
int age = user.age.get();

```

Observable Collections：

 ObservableArrayMap：

 ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);

layout：

<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap&lt;String, Object&gt;"/>
</data>
…
<TextView
   android:text='@{user["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user["age"])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>


ObservableArrayList：

ObservableArrayList<Object> user = new ObservableArrayList<>();
user.add("Google");
user.add("Inc.");
user.add(17);

<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList&lt;Object&gt;"/>
</data>
…
<TextView
   android:text='@{user[Fields.LAST_NAME]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>


Views With IDs

当一个Layout中的view声明了ID时，会自动生成对应的public final变量。数据绑定通过View的结构提取出有ID的view。这种机制比调用findViewById更高效

<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
   android:id="@+id/firstName"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"
  android:id="@+id/lastName"/>
   </LinearLayout>
</layout>

数据绑定在上面的布局下将会自动生成下面的变量：

public final TextView firstName;
public final TextView lastName;

Variables：


每个layout中声明的Variables对象，都应该声明对应的操作方法：

<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>


public abstract com.example.User getUser();
public abstract void setUser(com.example.User user);
public abstract Drawable getImage();
public abstract void setImage(Drawable image);
public abstract String getNote();
public abstract void setNote(String note);


ViewStubs

ViewStub和常规的View不太一样，它开始的时候是不可见的，可以延迟到运行时填充布局资源。

由于ViewStub会在视图层消失，为了正常的连接，对应的绑定对象也要随之消失。由于视图层是final类型的,ViewStubProxy对象替代ViewStub 后,开发者可以访问 ViewStub，并且当 ViewStub 在视图层中被加载时，开发者也可以访问加载的视图。

当ViewStub填充其他布局时，新的布局中要建立绑定关系。因此，ViewStubProxy 对象要监听ViewStub的OnInflateListener并建立绑定。开发者可以在ViewStubProxy 对象上建立一个OnInflateListener，绑定建立后，便可调用OnInflateListener。


高级绑定
---

1，动态变量

布局有时候要绑定的数据对象是不确定的，例如：RecyclerView.Adapter，必须在onBindViewHolder中指定绑定值，才可以识别出对应的绑定类。

public void onBindViewHolder(BindingHolder holder, int position) {
   final T item = mItems.get(position);
   holder.getBinding().setVariable(BR.item, item);
   holder.getBinding().executePendingBindings();
}

上面这个例子中，RecyclerView绑定的所有layout有一个item变量，BindingHolder有一个getBinding方法来获取ViewDataBinding对象


2.立即绑定

当一个变量或Observable对象发生变化时，绑定应该在下一帧之前按序进行改变，但是有时候需要立即执行绑定，可以调用 executePendingBindings()方法强制执行绑定

3.后台线程：
可以改变你的数据对象在后台线程，除了集合。数据绑定将会本地化保存每一个变量/字段，防止并发错误

Attribute Setters：
---

当一个绑定的值发生变化时，自动生成的数据绑定对象必须调用对应的setter方法在界面上赋值，数据绑定框架有几种方法去自定义绑定的方法

1.自动Setters

数据绑定会自动寻找参数对应的setter方法，跟命名空间无关，只跟属性的名称相关。
例如：textview 在不居中通过android:text赋值之后，会自动寻找setText(String)方法，如果表达式返回的是Int类型，熟悉绑定会自动寻找setText(Int)方法。所以要确保表达式返回的类型正确。若需要可以转换。即使没有给定的属性，数据绑定也会执行。可以通过数据绑定调用setter方法创建属性。例如，DrawerLayout没有任何属性，但有很多setter方法，可以调用setter方法自动创建属性。

android.support.v4.widget.DrawerLayout  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    app:scrimColor="@{@color/scrim}"  
    app:drawerListener="@{fragment.drawerListener}"/>  

2.重命名Setters

一些属性的setter方法与名称不一致，可以通过BindingMethods注解关联名称和方法。
例如：可以将android:tint 属性与setImageTintList(ColorStateList)方法相关联

@BindingMethods({  
       @BindingMethod(type = "android.widget.ImageView",  
                      attribute = "android:tint",  
                      method = "setImageTintList"),  
})  

开发者不需要重命名setter方法，数据绑定框架已经实现。


3.自定义setter

有一些参数需要自定义绑定逻辑，如：android:paddingLeft属性没有setter方法，但是有一个setPadding(left,top,right,bottom)方法，@BindingAdapter可以自定义怎么绑定该属性

@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int padding) {
   view.setPadding(padding,
                   view.getPaddingTop(),
                   view.getPaddingRight(),
                   view.getPaddingBottom());
}

BindingAdapter是很有用的去自定义别的属性，例如，自定的加载器可以利用空线程加载图片。
当适配器冲突的时候，开发者定义的BindingAdapter将覆盖默认的适配器。
一个适配器也可以绑定多个属性

@BindingAdapter({"bind:imageUrl", "bind:error"})
public static void loadImage(ImageView view, String url, Drawable error) {
   Picasso.with(view.getContext()).load(url).error(error).into(view);
}

<ImageView app:imageUrl="@{venue.imageUrl}"
app:error="@{@drawable/venueError}"/>

绑定适配器的方法可能需要持有旧的值，采用旧值和新值的方法应该首先具有属性的所有旧值，然后是新值，

@BindingAdapter("android:paddingLeft")  
public static void setPaddingLeft(View view, int oldPadding, int newPadding) {  
   if (oldPadding != newPadding) {  
       view.setPadding(newPadding,  
                       view.getPaddingTop(),  
                       view.getPaddingRight(),  
                       view.getPaddingBottom());  
   }  
}  

事件处理只能使用接口或有抽象方法的抽象类

@BindingAdapter("android:onLayoutChange")
public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue,
       View.OnLayoutChangeListener newValue) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        if (oldValue != null) {
            view.removeOnLayoutChangeListener(oldValue);
        }
        if (newValue != null) {
            view.addOnLayoutChangeListener(newValue);
        }
    }
}


当一个监听器有多个方法的时候，必须拆分进多个监听器。例如 View.OnAttachStateChangeListener有两个方法：

@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewDetachedFromWindow {
    void onViewDetachedFromWindow(View v);
}

@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewAttachedToWindow {
    void onViewAttachedToWindow(View v);
}

因为改变一个监听器方法的时候会影响到另一个，所以要提供三个绑定适配器，添加一个适配器来同时处理两个事件：

@BindingAdapter("android:onViewAttachedToWindow")
public static void setListener(View view, OnViewAttachedToWindow attached) {
    setListener(view, null, attached);
}

@BindingAdapter("android:onViewDetachedFromWindow")
public static void setListener(View view, OnViewDetachedFromWindow detached) {
    setListener(view, detached, null);
}

@BindingAdapter({"android:onViewDetachedFromWindow", "android:onViewAttachedToWindow"})
public static void setListener(View view, final OnViewDetachedFromWindow detach,
        final OnViewAttachedToWindow attach) {
    if (VERSION.SDK_INT >= VERSION_CODES.HONEYCOMB_MR1) {
        final OnAttachStateChangeListener newListener;
        if (detach == null && attach == null) {
            newListener = null;
        } else {
            newListener = new OnAttachStateChangeListener() {
                @Override
                public void onViewAttachedToWindow(View v) {
                    if (attach != null) {
                        attach.onViewAttachedToWindow(v);
                    }
                }

                @Override
                public void onViewDetachedFromWindow(View v) {
                    if (detach != null) {
                        detach.onViewDetachedFromWindow(v);
                    }
                }
            };
        }
        final OnAttachStateChangeListener oldListener = ListenerUtil.trackListener(view,
                newListener, R.id.onAttachStateChangeListener);
        if (oldListener != null) {
            view.removeOnAttachStateChangeListener(oldListener);
        }
        if (newListener != null) {
            view.addOnAttachStateChangeListener(newListener);
        }
    }
}

 android.databinding.adapters.ListenerUtil类用来帮助跟踪之前的监听器，保证之前的监听器被移除


3.转换器

当一个对象从绑定表达式中被返回，对象的setter方法将会从自动，重命名和自定义setter中选择，对象会被转化为setter指定的类型
例如：
<TextView
   android:text='@{userMap["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>

userMap 是一个ObservableMaps，userMap返回一个对象，对象将会被转化为字段对应的类型在setter方法中setText(CharSequence)。当参数类型不明确的时候，开发者应该在表达式中转化

自定义转化器

有时候转换器应该自动转化两个特殊类型的对象，如：
<View
   android:background="@{isError ? @color/red : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>

background参数类型是Drawable,而color是Int，无论预期的是Drawable还是int类型，int类型都应该被转换为colorDrawable.

@BindingConversion
public static ColorDrawable convertColorToDrawable(int color) {
   return new ColorDrawable(color);
}

注意：转化只能发生在setter级别，所以不允许在表达式中使用混合类型：

<View
   android:background="@{isError ? @drawable/error : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>



Android studio对DataBinding的支持

Android studio支持DataBinding的多种代码边距功能，例如：

语法高亮显示，在编辑器中显示语法错误，xml代码自动补全，引用，代码源码导航。

布局预览也支持在表达式中设置默认值，例如：

<TextView android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:text="@{user.firstName, default=PLACEHOLDER}"/>

在预览页面中该TextView的文本将会显示为PLACEHOLDER。如果在设计时需要显示不同的值，也可以使用tools参数代替表达式中的default值。
