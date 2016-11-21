---
layout:     post
title:      "Android Espresso"
subtitle:   ""
date:       2015-05-10 14:00:00
author:     "steven"
catalog:    true
tags:
    - Android
---

Espresso 是Android官方提供的一个UI测试框架，适合应用中的功能性 UI 测试。用于测试应用中的用户流，适合用来白盒自动化测试。支持android2.3.4（API 10）及以上版本。

Espresso 是基于instrumentation api,在AndroidJUnitRunner上运行。


配置环境
---

在module的build.gradle文件中添加依赖

```java
dependencies {
    // Other dependencies ...
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
}
```
使用android studio生成的项目中已经自动添加了改依赖

```java
androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
      exclude group: 'com.android.support', module: 'support-annotations'
  })
```

排斥support-annotations是因为espresso的support-annotations类库与android的support-annotations类库版本冲突


然后关闭模拟器或测试机的动画效果，因为测试用例执行的速度比较快，动画慢的话会影响测试用例的执行(比如消失动画会遮住后面View的点击事件)。
关闭动画步骤：打开设置-开发者选项,关闭以下动画缩放：

>窗口动画缩放

>过渡动画缩放

>动画程序时长缩放



创建测试类
---

Espresso测试逻辑：

1.通过Espresso.onView(view)来找到要测试的UI组件在Activity中
2.通过ViewInteraction.perform(action)或DataInteraction.perform(action)来模拟用户的操作事件，如文本输入，点击，手势等。可以通过逗号分隔一次传入多个操作事件。
3.如果要测试用例要通过多个activity，重复以上的步骤
4.使用ViewAssertions来检查预期的结果与测试用例结束后的结果是否一致

例如：

```java
onView(withId(R.id.my_view))            // withId(R.id.my_view) is a ViewMatcher
        .perform(click())               // click() is a ViewAction
        .check(matches(isDisplayed())); // matches(isDisplayed()) is a ViewAssertion
```


登录测试：

```java
@RunWith(AndroidJUnit4.class)
@LargeTest
public class LoginTest{

    @Rule
    public ActivityTestRule<LoginActivity> mActivityRule = new ActivityTestRule<>(
            LoginActivity.class);

    @Before
    public void setup() {
         ...
    }

    @After
    public void tearDown() {
         ...
    }

    @Test
    public void testLogin(){
        onView(withId(R.id.edtName)).perform(typeText("android"), closeSoftKeyboard());
        onView(withId(R.id.edtPwd)).perform(typeText("123456"),closeSoftKeyboard());
        onView(withId(R.id.btnLogin)).perform(click());
        onView(withId(R.id.tvLoginResult)).check(matches(withText("登录成功")));

    }
}
```

ActivityTestRule 提供单个Activity的方法测试。和JUnit的测试逻辑一致，@Test注解的方法就是会运行的测试方法。@Before即测试方法执行之前会被调用的方法。@After为测试方法执行完成之后会被调用的方法。在所有@Test标注的方法执行时，该Activity会被启动。

要找到对应的View，需要在OnView()方法中传入对应的view matcher。onView方法返回一个ViewInteraction对象，该对象允许测试方法与view交互。但是若onView的对象是RecyclerView，可能不会到达期望的交互结果，需要特殊处理。

若 OnView方法传入的view matcher找不到对应的view.则会抛出NoMatchingViewException，中断测试方法的执行

view matcher
---

view matcher就是根据一个条件在布局中找到对应的View组件。

如，通过显示的文本内容找到对应的View:


```java
onView(withText("text"));
```

通过Id:

```java
onView(withId(R.id.edtName))
```

还可以使用allOf来组合多个条件：

```java
onView(allOf(withId(R.id.button_signin), withText("text")));
```
allOf可以使用not关键字：

```java
onView(allOf(withId(R.id.button_signin), not(withText("text"))));
```

为提高测试方法执行的效率，应该尽量使用最少，最快的匹配规则。如通过的唯一的Id能找到对应的View,就不要通过text去匹配。


从AdapterView中定位View
---

在 AdapterView中，所有子view都是动态生成的在运行中，如果通过onView方法去匹配对应的view,可能会找不到。所以Espresso提供了一个onData方法，该方法会返回一个DataInteraction对象去交互对应的View.Espresso加载目标View到当前的view hierarchy.Espresso同时也处理滚动到目标View,并获取焦点。

例如：

```java
onData(allOf(is(instanceOf(Map.class)),hasEntry(equalTo(LongListActivity.ROW_TEXT), is("test input"))));
```


执行操作
---

调用 ViewInteraction.perform() 或 DataInteraction.perform() 方法来模拟用户在View上的交互事件。当传入多个事件时，Espresso会按顺序执行对应的操作。

 ViewActions包含多种常用的操作方式：

 ViewActions.click() 点击事件

 ViewActions.typeText() 点击输入框并输入对应的文本

 ViewActions.scrollTo() 滚动到某个View. 对应的View必须是 ScrollView的子类，并且 View的android:visibility属性必须是VISIBLE. 若是 AdapterView 的子类则使需用 onData() 方法来处理

 ViewActions.pressKey() 点击按键

 ViewActions.clearText() 清除文本框的文本


Espresso-Intents
---

Espresso Intents 用来验证和模拟应用发出的Intent

添加依赖：

```java
 dependencies {
  // Other dependencies ...
  androidTestCompile 'com.android.support.test.espresso:espresso-intents:2.2.2'
}
```

为了测试Intent，你先要创建一个 IntentsTestRule类的实例，和ActivityTestRule作用一致。


```java
@Large
@RunWith(AndroidJUnit4.class)
public class SimpleIntentTest {

    private static final String MESSAGE = "This is a test";
    private static final String PACKAGE_NAME = "com.example.myfirstapp";

    /* Instantiate an IntentsTestRule object. */
    @Rule
    public IntentsTestRule≶MainActivity> mIntentsRule =
      new IntentsTestRule≶>(MainActivity.class);

    @Test
    public void verifyMessageSentToMessageActivity() {

        // Types a message into a EditText element.
        onView(withId(R.id.edit_message))
                .perform(typeText(MESSAGE), closeSoftKeyboard());

        // Clicks a button to send the message to another
        // activity through an explicit intent.
        onView(withId(R.id.send_message)).perform(click());

        // Verifies that the DisplayMessageActivity received an intent
        // with the correct package name and message.
        intended(allOf(
                hasComponent(hasShortClassName(".DisplayMessageActivity")),
                toPackage(PACKAGE_NAME),
                hasExtra(MainActivity.EXTRA_MESSAGE, MESSAGE)));

    }
}
```



测试WebView
---

Espresso Web是用来测试WebView的一个组件，它使用 WebDriver API来检查和控制WebView的行为

添加依赖：

dependencies {
  // Other dependencies ...
  androidTestCompile 'com.android.support.test.espresso:espresso-web:2.2.2'
}

要测试WebView，需要在WebView中允许运行JavaScript。

例：


```java
@LargeTest
@RunWith(AndroidJUnit4.class)
public class WebViewActivityTest {

    private static final String MACCHIATO = "Macchiato";
    private static final String DOPPIO = "Doppio";

    @Rule
    public ActivityTestRule mActivityRule =
        new ActivityTestRule(WebViewActivity.class,
            false /* Initial touch mode */, false /*  launch activity */) {

        @Override
        protected void afterActivityLaunched() {
            // Enable JavaScript.
            onWebView().forceJavascriptEnabled();
        }
    }

    @Test
    public void typeTextInInput_clickButton_SubmitsForm() {
       // Lazily launch the Activity with a custom start Intent per test
       mActivityRule.launchActivity(withWebFormIntent());

       // Selects the WebView in your layout.
       // If you have multiple WebViews you can also use a
       // matcher to select a given WebView, onWebView(withId(R.id.web_view)).
       onWebView()
           // Find the input element by ID
           .withElement(findElement(Locator.ID, "text_input"))
           // Clear previous input
           .perform(clearElement())
           // Enter text into the input element
           .perform(DriverAtoms.webKeys(MACCHIATO))
           // Find the submit button
           .withElement(findElement(Locator.ID, "submitBtn"))
           // Simulate a click via JavaScript
           .perform(webClick())
           // Find the response element by ID
           .withElement(findElement(Locator.ID, "response"))
           // Verify that the response page contains the entered text
           .check(webMatches(getText(), containsString(MACCHIATO)));
    }
}
```

IdlingResource
---

Espresso在测试操作中是线程安全的。Espresso会等待当前进程的消息队列中的UI事件，并且在任何一个测试操作中会等待其中的AsyncTask结束才会执行下一个测试。
但是有许多的应用后台的操作都是使用自定义的服务去创建以及管理线程的，这时Espresso就无法同步了。Espresso等待app处于idle状态，才会执行下个动作或检查下个断言。

Idle状态是指应用的当前消息队列中没有UI事件或默认的AsyncTask线程池没有任务。app以其他方式执行长时间异步操作，Espresso无法知道何时这些操作已经完成。所以引入了IdlingResource来告诉Espresso是否可以继续执行测试。

IdlingResource是一个简单的接口，包含三个方法：

getName()：必须返回代表idling resource的非空字符串；
isIdleNow()：返回当前idlingresource的idle状态。如果返回true，onTransitionToIdle()上注册的ResourceCallback必须必须在之前已经调用；
registerIdleTransitionCallback：通常此方法用于存储对回调的引用来通知idle状态的变化。


例：等待登录接口返回结果


```java
private class LoginIdlingResource implements IdlingResource {

        LoginActivity activity;
        ResourceCallback resourceCallback;

        long startTime = -1;
        long maxTime = 30 * 1000;  //最大等待时间

        public LoginIdlingResource(LoginActivity activity){
           this.activity = activity;
        }

        @Override
        public String getName() {
            return "LoginIdlingResource";
        }


        @Override
        public boolean isIdleNow() {
            Fragment fragment = activity.getFragmentManager().findFragmentByTag(LoadingDialogFragment.TAG);
            long lastTime = startTime == -1 ? maxTime : System.currentTimeMillis() - startTime; //持续时间
            /**
             * 超过最大等待时间
             */
            if(lastTime > maxTime){
                return true;
            }

            if(fragment != null){
                if(startTime == -1){
                    startTime = System.currentTimeMillis();
                }
                resourceCallback.onTransitionToIdle();
                return false;
            }
            return true;
        }

        @Override
        public void registerIdleTransitionCallback(ResourceCallback callback) {
            this.resourceCallback = callback;
        }
    }

```
调用登录接口时会先弹出LoadingDialogFragment，若LoadingDialogFragment已经能从FragmentManager中找到时说明已经开始调用接口了，调用resourceCallback.onTransitionToIdle()，isIdleNow返回false，告诉Espresso等待。若接口调用完成会关闭LoadingDialogFragment，此时isIdleNow返回true,表示app进入Idle状态，可以继续执行后面的测试。若接口调用失败，LoadingDialogFragment会显示出对应的错误信息，并不会自动关闭，此时还是非Idle状态。所以加入了一个等待时间的最大值，接口等待时间超过这个时长则直接执行后面的检查。

注册IdlingResource

只有注册了的IdlingResource才能被执行，大部分情况下是在@Before方法内注册，在@After内取消注册。

```java

@Before
public void registerIntentServiceIdlingResource() {
    idlingResource = new LoginIdlingResource(mActivityRule.getActivity());
    Espresso.registerIdlingResources(idlingResource);
}

@After
public void unregisterIntentServiceIdlingResource() {
    Espresso.unregisterIdlingResources(idlingResource);
}
```
