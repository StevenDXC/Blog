---
layout:     post
title:      "Android插件化实现原理"
subtitle:   ""
date:       2016-04-18 15:00:00
author:     "steven"
catalog:    true
tags:
    - Android
---


最近android插件化比较火热，陆陆续续出了好多插件化方案，大有百花齐放之势。


其实android实现方式就两种，要么hook AMS(ActivityManagerService),要么hook Instrumentation。想要详细了解hook的原理，先要弄清楚Activity的启动流程


## Activity启动流程

启动一个activity：

```java
Intent intent = new Intent(ActivityA.this,ActivityB.class);
startActivity(intent);
```

通过调用Activity的startActivity方法来启动另一个Activity. 而startActivity继承自ContextThemeWrapper。看Wrapper后缀就知道这只是一个封装类。真正的逻辑在ContextImpl中

```java
@Override
public void startActivity(Intent intent, Bundle options) {
    warnIfCallingFromSystemProcess();
    if ((intent.getFlags()&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
        throw new AndroidRuntimeException(
                "Calling startActivity() from outside of an Activity "
                + " context requires the FLAG_ACTIVITY_NEW_TASK flag."
                + " Is this really what you want?");
    }
    mMainThread.getInstrumentation().execStartActivity(
        getOuterContext(), mMainThread.getApplicationThread(), null,
        (Activity)null, intent, -1, options);
}
```
通过MainThread获取到Instrumentation对象，Instrumentation对象类似Activiy的管家，所有对Activity的操作都通过它去执行。然后调用它的execStartActivity方法来执行启动activity

```java
public ActivityResult execStartActivity(Context who, IBinder contextThread, IBinder token, Activity target,
        Intent intent, int requestCode, Bundle options) {
    IApplicationThread whoThread = (IApplicationThread) contextThread;
    ...
    try {
        intent.migrateExtraStreamToClipData();
        intent.prepareToLeaveProcess();
        int result = ActivityManagerNative.getDefault()
            .startActivity(whoThread, who.getBasePackageName(), intent,
                    intent.resolveTypeIfNeeded(who.getContentResolver()),
                    token, target != null ? target.mEmbeddedID : null,
                    requestCode, 0, null, options);
        checkStartActivityResult(result, intent);
    } catch (RemoteException e) {
    }
    return null;
}
```
调用ActivityManagerNative.getDefault()获取一个ActivityManagerNative对象，然后调用来该对象的startActivity方法。

而ActivityManagerNative对象继承自Binder，所以ActivityManagerNative是可以跨进程通信的。
ActivityManagerNative跨入到了ActivityManagerService，简称AMS。是ActivityManagerNative的实现，运行在Android内核进程。

AMS的startActivity：

```java
@Override
public final int startActivity(IApplicationThread caller, String callingPackage,
        Intent intent, String resolvedType, IBinder resultTo,
        String resultWho, int requestCode, int startFlags,
        String profileFile, ParcelFileDescriptor profileFd, Bundle options) {
    return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
            resultWho, requestCode,
            startFlags, profileFile, profileFd, options, UserHandle.getCallingUserId());
}
```

调用了startActivityAsUser方法，startActivityAsUser又调用了ActivityStackSupervisor.startActivityMayWait方法：

```java
final int startActivityMayWait(IApplicationThread caller, int callingUid,String callingPackage, Intent intent, String resolvedType, IBinder resultTo,String resultWho, int requestCode, int startFlags, String profileFile,
ParcelFileDescriptor profileFd, WaitResult outResult, Configuration config,Bundle options, int userId) {

     ..........//权限判断

     int res = startActivityLocked(caller, intent, resolvedType,aInfo, resultTo, resultWho, requestCode, callingPid, callingUid,callingPackage, startFlags, options, componentSpecified, null);

    .........
  }
}
```
调用startActivityLocked之后，然后是和ActivityStack的调用，主要处理activity栈的逻辑。然后调用到realStartActivityLocked方法


```java
final boolean realStartActivityLocked(ActivityRecord r,ProcessRecord app, boolean andResume, boolean checkConfig)
    throws RemoteException {

r.startFreezingScreenLocked(app, 0);
if (false) Slog.d(TAG, "realStartActivity: setting app visibility true");
mWindowManager.setAppVisibility(r.appToken, true);

r.startLaunchTickingLocked();

if (checkConfig) {
    Configuration config = mWindowManager.updateOrientationFromAppTokens(
            mService.mConfiguration,
            r.mayFreezeScreenLocked(app) ? r.appToken : null);
    mService.updateConfigurationLocked(config, r, false, false);
}

r.app = app;
app.waitingToKill = null;
r.launchCount++;
r.lastLaunchTime = SystemClock.uptimeMillis();

if (localLOGV) Slog.v(TAG, "Launching: " + r);

int idx = app.activities.indexOf(r);
if (idx < 0) {
    app.activities.add(r);
}
mService.updateLruProcessLocked(app, true, true);

final ActivityStack stack = r.task.stack;
try {
    if (app.thread == null) {
        throw new RemoteException();
    }
    List<ResultInfo> results = null;
    List<Intent> newIntents = null;
    if (andResume) {
        results = r.results;
        newIntents = r.newIntents;
    }
    if (DEBUG_SWITCH) Slog.v(TAG, "Launching: " + r
            + " icicle=" + r.icicle
            + " with results=" + results + " newIntents=" + newIntents
            + " andResume=" + andResume);
    if (andResume) {
        EventLog.writeEvent(EventLogTags.AM_RESTART_ACTIVITY,
                r.userId, System.identityHashCode(r),
                r.task.taskId, r.shortComponentName);
    }
    if (r.isHomeActivity() && r.isNotResolverActivity()) {
        // Home process is the root process of the task.
        mService.mHomeProcess = r.task.mActivities.get(0).app;
    }
    mService.ensurePackageDexOpt(r.intent.getComponent().getPackageName());
    r.sleeping = false;
    r.forceNewConfig = false;
    mService.showAskCompatModeDialogLocked(r);
    r.compat = mService.compatibilityInfoForPackageLocked(r.info.applicationInfo);
    String profileFile = null;
    ParcelFileDescriptor profileFd = null;
    boolean profileAutoStop = false;
    if (mService.mProfileApp != null && mService.mProfileApp.equals(app.processName)) {
        if (mService.mProfileProc == null || mService.mProfileProc == app) {
            mService.mProfileProc = app;
            profileFile = mService.mProfileFile;
            profileFd = mService.mProfileFd;
            profileAutoStop = mService.mAutoStopProfiler;
        }
    }
    app.hasShownUi = true;
    app.pendingUiClean = true;
    if (profileFd != null) {
        try {
            profileFd = profileFd.dup();
        } catch (IOException e) {
            if (profileFd != null) {
                try {
                    profileFd.close();
                } catch (IOException o) {
                }
                profileFd = null;
            }
        }
    }
    app.forceProcessStateUpTo(ActivityManager.PROCESS_STATE_TOP);
    app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
            System.identityHashCode(r), r.info,
            new Configuration(mService.mConfiguration), r.compat,
            app.repProcState, r.icicle, results, newIntents, !andResume,
            mService.isNextTransitionForward(), profileFile, profileFd,
            profileAutoStop);

    if ((app.info.flags&ApplicationInfo.FLAG_CANT_SAVE_STATE) != 0) {
        if (app.processName.equals(app.info.packageName)) {
            if (mService.mHeavyWeightProcess != null
                    && mService.mHeavyWeightProcess != app) {
                Slog.w(TAG, "Starting new heavy weight process " + app
                        + " when already running "
                        + mService.mHeavyWeightProcess);
            }
            mService.mHeavyWeightProcess = app;
            Message msg = mService.mHandler.obtainMessage(
                    ActivityManagerService.POST_HEAVY_NOTIFICATION_MSG);
            msg.obj = r;
            mService.mHandler.sendMessage(msg);
        }
    }

} catch (RemoteException e) {
    if (r.launchFailed) {
        // This is the second time we failed -- finish activity
        // and give up.
        Slog.e(TAG, "Second failure launching "
              + r.intent.getComponent().flattenToShortString()
              + ", giving up", e);
        mService.appDiedLocked(app, app.pid, app.thread);
        stack.requestFinishActivityLocked(r.appToken, Activity.RESULT_CANCELED, null,
                "2nd-crash", false);
        return false;
    }

    // This is the first time we failed -- restart process and
    // retry.
    app.activities.remove(r);
    throw e;
}

r.launchFailed = false;
if (stack.updateLRUListLocked(r)) {
    Slog.w(TAG, "Activity " + r
          + " being launched, but already in LRU list");
}

if (andResume) {
    stack.minimalResumeActivityLocked(r);
} else {
    if (DEBUG_STATES) Slog.v(TAG, "Moving to STOPPED: " + r
            + " (starting in stopped state)");
    r.state = ActivityState.STOPPED;
    r.stopped = true;
}

if (isFrontStack(stack)) {
    mService.startSetupActivityLocked();
}

return true;
}

```

方法比较长，重点的部分是

```java
app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
        System.identityHashCode(r), r.info,
        new Configuration(mService.mConfiguration), r.compat,
        app.repProcState, r.icicle, results, newIntents, !andResume,
        mService.isNextTransitionForward(), profileFile, profileFd,
        profileAutoStop);
```

调用了ApplicationThread的scheduleLaunchActivity方法，ApplicationThread也是binder,实现了IApplicationThread接口。

在Instrumentation调用了AMS的startActivity方法之后的逻辑都是在系统进程中进行的，启动完Activity之后自然需要回归到当初的app进程。这也是跨进程操作，自然需要binder。

scheduleLaunchActivity方法：

```java
public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,ActivityInfo info, Configuration curConfig, CompatibilityInfo compatInfo,int procState, Bundle state, List<ResultInfo> pendingResults,List<Intent> pendingNewIntents, boolean notResumed, boolean isForward,String profileName, ParcelFileDescriptor profileFd, boolean autoStopProfiler) {

        updateProcessState(procState, false);

        ActivityClientRecord r = new ActivityClientRecord();

        r.token = token;
        r.ident = ident;
        r.intent = intent;
        r.activityInfo = info;
        r.compatInfo = compatInfo;
        r.state = state;

        r.pendingResults = pendingResults;
        r.pendingIntents = pendingNewIntents;

        r.startsNotResumed = notResumed;
        r.isForward = isForward;

        r.profileFile = profileName;
        r.profileFd = profileFd;
        r.autoStopProfiler = autoStopProfiler;

        updatePendingConfiguration(curConfig);

        queueOrSendMessage(H.LAUNCH_ACTIVITY, r);
  }
```

可以看到，最后调用了queueOrSendMessage(H.LAUNCH_ACTIVITY, r)。

H是一个handler,queueOrSendMessage方法其实就是通过hander去发送一个message。

handle message:

```java
public void handleMessage(Message msg) {    
    if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
        switch (msg.what) {
            case LAUNCH_ACTIVITY: {
                Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                ActivityClientRecord r = (ActivityClientRecord)msg.obj;

                r.packageInfo = getPackageInfoNoCheck(
                        r.activityInfo.applicationInfo, r.compatInfo);
                handleLaunchActivity(r, null);
                Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
            } break;

     ..........
}
```

通过message调用了handleLaunchActivity方法，接着在handleLaunchActivity方法中调用了performLaunchActivity来启动一个activity


```java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {

    ....

    Activity activity = null;
    try {
        java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
        activity = mInstrumentation.newActivity(
                cl, component.getClassName(), r.intent);
        StrictMode.incrementExpectedActivityCount(activity.getClass());
        r.intent.setExtrasClassLoader(cl);
        if (r.state != null) {
            r.state.setClassLoader(cl);
        }
    } catch (Exception e) {
        if (!mInstrumentation.onException(activity, e)) {
            throw new RuntimeException(
                "Unable to instantiate activity " + component
                + ": " + e.toString(), e);
        }
    }

    ......

    return activity;
}
```

又调用了Instrumentation的newActivity方法去创建一个Activity.

总结下：

1. 通过activity去调用startActivity方法，其实是调用的ContextImpl的同名方法
2. 在ContextImpl获取到Instrumentation，然后通过Instrumentation的execStartActivity继续启动
3. Instrumentation跨进程通信，调用到AMS的startActivity
4. AMS处理权限和ActivityStack的逻辑
5. 通过ApplicationThread跨进程，回到app进程
6. 通过ActivityThread中的handler传递启动activity的消息
7. 最终调用Instrumentation的newActivity方法创建一个Activity对象


其实其它组件如Service 调用过程和Activity的大同小异，都是跨进程和AMS，WMS或者PMS进行通信。然后通过ApplicationThread回到app进程，最后通过handler传递消息。

如果要启动一个未事先声明的Activity，在AMS的权限校验的时候会验证不过。

startActivityMayWait方法

```java
if (err == ActivityManager.START_SUCCESS && aInfo == null) {   
		// We couldn't find the specific class specified in the Intent.
        // Also the end of the line.
        err = ActivityManager.START_CLASS_NOT_FOUND;
}

if (err != ActivityManager.START_SUCCESS) {
            if (resultRecord != null) {
                resultStack.sendActivityResultLocked(-1,
                    resultRecord, resultWho, requestCode,
                    Activity.RESULT_CANCELED, null);
            }
            setDismissKeyguard(false);
            ActivityOptions.abort(options);
            return err;
}
```
找不到对应的类就会直接抛出错误。所以要解决插件化的核心为题就是让没有在manifst中注册的activity顺利启动起来。

由于系统进程无法hook，所有只能通过欺骗的方法，现在manifst中声明几个Activity来占坑，在校验权限之前将实际的activity改为占坑的activity，通过校验之后再替换回来。

要想实现上面欺骗的逻辑，有两种选择：hook Instrumentation 或 hook AMS

## hook Instrumentation

这部分代码来自[smail](https://github.com/wequick/Small)

```java
@Override
public void setUp(Context context) {
    super.setUp(context);
    // Inject instrumentation
    if (sHostInstrumentation == null) {
        try {
            final Class<?> activityThreadClass = Class.forName("android.app.ActivityThread");
            final Method method = activityThreadClass.getMethod("currentActivityThread");
            Object thread = method.invoke(null, (Object[]) null);
            Field field = activityThreadClass.getDeclaredField("mInstrumentation");
            field.setAccessible(true);
            sHostInstrumentation = (Instrumentation) field.get(thread);
            Instrumentation wrapper = new InstrumentationWrapper();
            field.set(thread, wrapper);

            if (context instanceof Activity) {
                field = Activity.class.getDeclaredField("mInstrumentation");
                field.setAccessible(true);
                field.set(context, wrapper);
            }
        } catch (Exception ignored) {
            ignored.printStackTrace();
            // Usually, cannot reach here
        }
    }
}
```
通过反射ActivityThread来获取到Instrumentation field。然后将系统的Instrumentation替换成为自己的InstrumentationWrapper。

然后在nstrumentationWrapper中重写execStartActivity和newActivty这两个方法

```java
/** @Override V21+
 * Wrap activity from REAL to STUB */
public ActivityResult execStartActivity(
        Context who, IBinder contextThread, IBinder token, Activity target,
        Intent intent, int requestCode, android.os.Bundle options) {
    wrapIntent(intent);
    return ReflectAccelerator.execStartActivityV21(sHostInstrumentation,
            who, contextThread, token, target, intent, requestCode, options);
}

/** @Override V20-
 * Wrap activity from REAL to STUB */
public ActivityResult execStartActivity(
        Context who, IBinder contextThread, IBinder token, Activity target,
        Intent intent, int requestCode) {
    wrapIntent(intent);
    return ReflectAccelerator.execStartActivityV20(sHostInstrumentation,
```    


wrapIntent方法：

```java
private void wrapIntent(Intent intent) {
    ComponentName component = intent.getComponent();
    if (component == null) return; // ignore system intent

    String realClazz = intent.getComponent().getClassName();
    if (sLoadedActivities == null) return;

    ActivityInfo ai = sLoadedActivities.get(realClazz);
    if (ai == null) return;

    // Carry the real(plugin) class for incoming `newActivity' method.
    intent.addCategory(REDIRECT_FLAG + realClazz);
    String stubClazz = dequeueStubActivity(ai, realClazz);
    intent.setComponent(new ComponentName(Small.getContext(), stubClazz));
}
```
这个方法先获取了真正的Activity的信息，并且保存起来。然后通过dequeueStubActivity方法生成一个占坑的activity

```java
private String dequeueStubActivity(ActivityInfo ai, String realActivityClazz) {
    if (ai.launchMode == ActivityInfo.LAUNCH_MULTIPLE) {
        // In standard mode, the stub activity is reusable.
        return STUB_ACTIVITY_PREFIX;
    }

    int availableId = -1;
    int stubId = -1;
    int countForMode = STUB_ACTIVITIES_COUNT;
    int countForAll = countForMode * 3; // 3=[singleTop, singleTask, singleInstance]
    if (mStubQueue == null) {
        // Lazy init
        mStubQueue = new String[countForAll];
    }
    int offset = (ai.launchMode - 1) * countForMode;
    for (int i = 0; i < countForMode; i++) {
        String usedActivityClazz = mStubQueue[i + offset];
        if (usedActivityClazz == null) {
            if (availableId == -1) availableId = i;
        } else if (usedActivityClazz.equals(realActivityClazz)) {
            stubId = i;
        }
    }
    if (stubId != -1) {
        availableId = stubId;
    } else if (availableId != -1) {
        mStubQueue[availableId + offset] = realActivityClazz;
    } else {
        // TODO:
        Log.e(TAG, "Launch mode " + ai.launchMode + " is full");
    }
    return STUB_ACTIVITY_PREFIX + ai.launchMode + availableId;
}
```

据真正的Activity的launchMode等信息生成一个占坑Activity，而这些占坑Activity都声明在manifest中

```java
<application>
    <!-- Stub Activities -->
    <!-- 1 standard mode -->
    <activity android:name=".A" android:launchMode="standard"/>
    <!-- 4 singleTask mode -->
    <activity android:name=".A10" android:launchMode="singleTask"/>
    <activity android:name=".A11" android:launchMode="singleTask"/>
    <activity android:name=".A12" android:launchMode="singleTask"/>
    <activity android:name=".A13" android:launchMode="singleTask"/>
    <!-- 4 singleTop mode -->
    <activity android:name=".A20" android:launchMode="singleTop"/>
    <activity android:name=".A21" android:launchMode="singleTop"/>
    <activity android:name=".A22" android:launchMode="singleTop"/>
    <activity android:name=".A23" android:launchMode="singleTop"/>
    <!-- 4 singleInstance mode -->
    <activity android:name=".A30" android:launchMode="singleInstance"/>
    <activity android:name=".A31" android:launchMode="singleInstance"/>
    <activity android:name=".A32" android:launchMode="singleInstance"/>
    <activity android:name=".A33" android:launchMode="singleInstance"/>

    <!-- Web Activity -->
    <activity android:name=".webkit.WebActivity"
        android:screenOrientation="portrait"
        android:windowSoftInputMode="stateHidden|adjustPan"/>
    <!--<service android:name="net.wequick.small.service.UpgradeService"-->
        <!--android:exported="false"/>-->
</application>
```
这样就将真的activity转化成来一个占坑的activity。可以通过权限校验了。

newActivity：

```java
@Override
/** Unwrap activity from STUB to REAL */
public Activity newActivity(ClassLoader cl, String className, Intent intent)
        throws InstantiationException, IllegalAccessException, ClassNotFoundException {
    // Stub -> Real
    if (!className.startsWith(STUB_ACTIVITY_PREFIX)) {
        return super.newActivity(cl, className, intent);
    }
    className = unwrapIntent(intent, className);
    Activity activity = super.newActivity(cl, className, intent);
    return activity;
}

private String unwrapIntent(Intent intent, String className) {
            Set<String> categories = intent.getCategories();
            if (categories == null) return className;

            // Get plugin activity class name from categories
            Iterator<String> it = categories.iterator();
            String realClazz = null;
            while (it.hasNext()) {
                String category = it.next();
                if (category.charAt(0) == REDIRECT_FLAG) {
                    realClazz = category.substring(1);
                    break;
                }
            }
            if (realClazz == null) return className;
            return realClazz;
}
```
通过unwrapIntent获取真正的Activity并且调用父类（Instrumentation）的newActivity方法来生成真正的activity。这样就成功地完成来欺骗过程。


## hook ActivityManagerNative

下面的代码来自[DroidPlugin](https://github.com/DroidPluginTeam/DroidPlugin)

主要的是IActivityManagerHook类，它继承自ProxyHook

```java
public abstract class ProxyHook extends Hook implements InvocationHandler {

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        try {
            if (!isEnable()) {
                return method.invoke(mOldObj, args);
            }
            HookedMethodHandler hookedMethodHandler = mHookHandles.getHookedMethodHandler(method);
            if (hookedMethodHandler != null) {
                return hookedMethodHandler.doHookInner(mOldObj, method, args);
            }
            return method.invoke(mOldObj, args);
        }

        ...........
}
```
ProxyHook是一个动态代理实现类，通过动态代理获取对应的hookedMethodHandler。

mHookHandles是IActivityManagerHookHandle类的实例，IActivityManagerHookHandle：

```java
public class IActivityManagerHookHandle extends BaseHookHandle {

    @Override
    protected void init() {
        sHookedMethodHandlers.put("startActivity", new startActivity(mHostContext));
        sHookedMethodHandlers.put("startActivityAsUser", new startActivityAsUser(mHostContext));
        sHookedMethodHandlers.put("startActivityAsCaller", new startActivityAsCaller(mHostContext));
        sHookedMethodHandlers.put("startActivityAndWait", new startActivityAndWait(mHostContext));
        sHookedMethodHandlers.put("startActivityWithConfig", new startActivityWithConfig(mHostContext));
        sHookedMethodHandlers.put("startActivityIntentSender", new startActivityIntentSender(mHostContext));
        sHookedMethodHandlers.put("startVoiceActivity", new startVoiceActivity(mHostContext));
        sHookedMethodHandlers.put("startNextMatchingActivity", new startNextMatchingActivity(mHostContext));
        sHookedMethodHandlers.put("startActivityFromRecents", new startActivityFromRecents(mHostContext));
        sHookedMethodHandlers.put("finishActivity", new finishActivity(mHostContext));
        sHookedMethodHandlers.put("registerReceiver", new registerReceiver(mHostContext));
        sHookedMethodHandlers.put("broadcastIntent", new broadcastIntent(mHostContext));
        sHookedMethodHandlers.put("unbroadcastIntent", new unbroadcastIntent(mHostContext));
        sHookedMethodHandlers.put("getCallingPackage", new getCallingPackage(mHostContext));
        sHookedMethodHandlers.put("getCallingActivity", new getCallingActivity(mHostContext));
        sHookedMethodHandlers.put("getAppTasks", new getAppTasks(mHostContext));
        sHookedMethodHandlers.put("addAppTask", new addAppTask(mHostContext));
        sHookedMethodHandlers.put("getTasks", new getTasks(mHostContext));
        sHookedMethodHandlers.put("getServices", new getServices(mHostContext));
        sHookedMethodHandlers.put("getProcessesInErrorState", new getProcessesInErrorState(mHostContext));
        sHookedMethodHandlers.put("getContentProvider", new getContentProvider(mHostContext));
        sHookedMethodHandlers.put("getContentProviderExternal", new getContentProviderExternal(mHostContext));
        sHookedMethodHandlers.put("removeContentProviderExternal", new removeContentProviderExternal(mHostContext));
        sHookedMethodHandlers.put("publishContentProviders", new publishContentProviders(mHostContext));
        sHookedMethodHandlers.put("getRunningServiceControlPanel", new getRunningServiceControlPanel(mHostContext));
        sHookedMethodHandlers.put("startService", new startService(mHostContext));
        sHookedMethodHandlers.put("stopService", new stopService(mHostContext));
        sHookedMethodHandlers.put("stopServiceToken", new stopServiceToken(mHostContext));
        sHookedMethodHandlers.put("setServiceForeground", new setServiceForeground(mHostContext));
        sHookedMethodHandlers.put("bindService", new bindService(mHostContext));
        sHookedMethodHandlers.put("publishService", new publishService(mHostContext));
        sHookedMethodHandlers.put("unbindFinished", new unbindFinished(mHostContext));
        sHookedMethodHandlers.put("peekService", new peekService(mHostContext));
        sHookedMethodHandlers.put("bindBackupAgent", new bindBackupAgent(mHostContext));
        sHookedMethodHandlers.put("backupAgentCreated", new backupAgentCreated(mHostContext));
        sHookedMethodHandlers.put("unbindBackupAgent", new unbindBackupAgent(mHostContext));
        sHookedMethodHandlers.put("killApplicationProcess", new killApplicationProcess(mHostContext));
        sHookedMethodHandlers.put("startInstrumentation", new startInstrumentation(mHostContext));
        sHookedMethodHandlers.put("getActivityClassForToken", new getActivityClassForToken(mHostContext));
        sHookedMethodHandlers.put("getPackageForToken", new getPackageForToken(mHostContext));
        sHookedMethodHandlers.put("getIntentSender", new getIntentSender(mHostContext));
        sHookedMethodHandlers.put("clearApplicationUserData", new clearApplicationUserData(mHostContext));
        sHookedMethodHandlers.put("handleIncomingUser", new handleIncomingUser(mHostContext));
        sHookedMethodHandlers.put("grantUriPermission", new grantUriPermission(mHostContext));
        sHookedMethodHandlers.put("getPersistedUriPermissions", new getPersistedUriPermissions(mHostContext));
        sHookedMethodHandlers.put("killBackgroundProcesses", new killBackgroundProcesses(mHostContext));
        sHookedMethodHandlers.put("forceStopPackage", new forceStopPackage(mHostContext));
        sHookedMethodHandlers.put("getRunningAppProcesses", new getRunningAppProcesses(mHostContext));
        sHookedMethodHandlers.put("getRunningExternalApplications", new getRunningExternalApplications(mHostContext));
        sHookedMethodHandlers.put("getMyMemoryState", new getMyMemoryState(mHostContext));
        sHookedMethodHandlers.put("crashApplication", new crashApplication(mHostContext));
        sHookedMethodHandlers.put("grantUriPermissionFromOwner", new grantUriPermissionFromOwner(mHostContext));
        sHookedMethodHandlers.put("checkGrantUriPermission", new checkGrantUriPermission(mHostContext));
        sHookedMethodHandlers.put("startActivities", new startActivities(mHostContext));
        sHookedMethodHandlers.put("getPackageScreenCompatMode", new getPackageScreenCompatMode(mHostContext));
        sHookedMethodHandlers.put("setPackageScreenCompatMode", new setPackageScreenCompatMode(mHostContext));
        sHookedMethodHandlers.put("getPackageAskScreenCompat", new getPackageAskScreenCompat(mHostContext));
        sHookedMethodHandlers.put("setPackageAskScreenCompat", new setPackageAskScreenCompat(mHostContext));
        sHookedMethodHandlers.put("navigateUpTo", new navigateUpTo(mHostContext));
        sHookedMethodHandlers.put("serviceDoneExecuting", new serviceDoneExecuting(mHostContext));
    }
}
```
在init方法中生成很多类，去hook不同的方法。

拿到HookedMethodHandler之后，执行它的doInnerHook方法：

```java
public synchronized Object doHookInner(Object receiver, Method method, Object[] args) throws Throwable {
    long b = System.currentTimeMillis();
    try {
        mUseFakedResult = false;
        mFakedResult = null;
        boolean suc = beforeInvoke(receiver, method, args);
        Object invokeResult = null;
        if (!suc) {
            invokeResult = method.invoke(receiver, args);
        }
        afterInvoke(receiver, method, args, invokeResult);
        if (mUseFakedResult) {
            return mFakedResult;
        } else {
            return invokeResult;
        }
    } finally {
        long time = System.currentTimeMillis() - b;
        if (time > 5) {
            Log.i(TAG, "doHookInner method(%s.%s) cost %s ms", method.getDeclaringClass().getName(), method.getName(), time);
        }
    }
}
```

beforeInvoke

```java
@Override
protected boolean beforeInvoke(Object receiver, Method method, Object[] args) throws Throwable {

    RunningActivities.beforeStartActivity();
    boolean bRet;
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR2) {
        bRet = doReplaceIntentForStartActivityAPILow(args);
    } else {
        bRet = doReplaceIntentForStartActivityAPIHigh(args);
    }
    if (!bRet) {
        setFakedResult(Activity.RESULT_CANCELED);
        return true;
    }

    return super.beforeInvoke(receiver, method, args);
}
```
doReplaceIntentForStartActivityAPIHigh：

```java
protected boolean doReplaceIntentForStartActivityAPIHigh(Object[] args) throws RemoteException {
            int intentOfArgIndex = findFirstIntentIndexInArgs(args);
            if (args != null && args.length > 1 && intentOfArgIndex >= 0) {
                Intent intent = (Intent) args[intentOfArgIndex];
                //XXX String callingPackage = (String) args[1];
                if (!PluginPatchManager.getInstance().canStartPluginActivity(intent)) {
                    PluginPatchManager.getInstance().startPluginActivity(intent);
                    return false;
                }
                ActivityInfo activityInfo = resolveActivity(intent);
                if (activityInfo != null && isPackagePlugin(activityInfo.packageName)) {
                    ComponentName component = selectProxyActivity(intent);
                    if (component != null) {
                        Intent newIntent = new Intent();
                        try {
                            ClassLoader pluginClassLoader = PluginProcessManager.getPluginClassLoader(component.getPackageName());
                            setIntentClassLoader(newIntent, pluginClassLoader);
                        } catch (Exception e) {
                            Log.w(TAG, "Set Class Loader to new Intent fail", e);
                        }
                        newIntent.setComponent(component);
                        newIntent.putExtra(Env.EXTRA_TARGET_INTENT, intent);
                        newIntent.setFlags(intent.getFlags());


                        String callingPackage = (String) args[1];
                        if (TextUtils.equals(mHostContext.getPackageName(), callingPackage)) {
                        newIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                        args[intentOfArgIndex] = newIntent;
                        args[1] = mHostContext.getPackageName();
                    } else {
                        Log.w(TAG, "startActivity,replace selectProxyActivity fail");
                    }
                }
            }

            return true;
}
```
该方法就是生成一个占坑的Intent，把真正的activity当作参数放在Intent中。然后在hook掉AMS通过权限验证

DroidPlugin是将activity替换回来过程

在ActivityThread中最后是通过handler来发送一个消息来通过Instrumentation创建一个Activity的

Handler dispatchMessage：

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```

在handler内有一个变量mCallback，若mCallback不为null,则使用它去处理message.DroidPlugin自己生成了一个CallBack，并通过反射注入到了ActivityThread的H类中。

然后通过自己的handleLaunchActivity方法来生成一个新的activity：

```java
private boolean handleLaunchActivity(Message msg) {
    try {
        Object obj = msg.obj;
        Intent stubIntent = (Intent) FieldUtils.readField(obj, "intent");
        //ActivityInfo activityInfo = (ActivityInfo) FieldUtils.readField(obj, "activityInfo", true);
        stubIntent.setExtrasClassLoader(mHostContext.getClassLoader());
        Intent targetIntent = stubIntent.getParcelableExtra(Env.EXTRA_TARGET_INTENT);
        // 这里多加一个isNotShortcutProxyActivity的判断，因为ShortcutProxyActivity的很特殊，启动它的时候，
        // 也会带上一个EXTRA_TARGET_INTENT的数据，就会导致这里误以为是启动插件Activity，所以这里要先做一个判断。
        // 之前ShortcutProxyActivity错误复用了key，但是为了兼容，所以这里就先这么判断吧。
        if (targetIntent != null && !isShortcutProxyActivity(stubIntent)) {
            IPackageManagerHook.fixContextPackageManager(mHostContext);
            ComponentName targetComponentName = targetIntent.resolveActivity(mHostContext.getPackageManager());
            ActivityInfo targetActivityInfo = PluginManager.getInstance().getActivityInfo(targetComponentName, 0);
            if (targetActivityInfo != null) {

                if (targetComponentName != null && targetComponentName.getClassName().startsWith(".")) {
                    targetIntent.setClassName(targetComponentName.getPackageName(), targetComponentName.getPackageName() + targetComponentName.getClassName());
                }

                ResolveInfo resolveInfo = mHostContext.getPackageManager().resolveActivity(stubIntent, 0);
                ActivityInfo stubActivityInfo = resolveInfo != null ? resolveInfo.activityInfo : null;
                if (stubActivityInfo != null) {
                    PluginManager.getInstance().reportMyProcessName(stubActivityInfo.processName, targetActivityInfo.processName, targetActivityInfo.packageName);
                }
                PluginProcessManager.preLoadApk(mHostContext, targetActivityInfo);
                ClassLoader pluginClassLoader = PluginProcessManager.getPluginClassLoader(targetComponentName.getPackageName());
                setIntentClassLoader(targetIntent, pluginClassLoader);
                setIntentClassLoader(stubIntent, pluginClassLoader);

                boolean success = false;
                try {
                    targetIntent.putExtra(Env.EXTRA_TARGET_INFO, targetActivityInfo);
                    if (stubActivityInfo != null) {
                        targetIntent.putExtra(Env.EXTRA_STUB_INFO, stubActivityInfo);
                    }
                    success = true;
                } catch (Exception e) {
                    Log.e(TAG, "putExtra 1 fail", e);
                }

                if (!success && Build.VERSION.SDK_INT <= Build.VERSION_CODES.KITKAT) {
                    try {
                        ClassLoader oldParent = fixedClassLoader(pluginClassLoader);
                        targetIntent.putExtras(targetIntent.getExtras());

                        targetIntent.putExtra(Env.EXTRA_TARGET_INFO, targetActivityInfo);
                        if (stubActivityInfo != null) {
                            targetIntent.putExtra(Env.EXTRA_STUB_INFO, stubActivityInfo);
                        }
                        fixedClassLoader(oldParent);
                        success = true;
                    } catch (Exception e) {
                        Log.e(TAG, "putExtra 2 fail", e);
                    }
                }

                if (!success) {
                    Intent newTargetIntent = new Intent();
                    newTargetIntent.setComponent(targetIntent.getComponent());
                    newTargetIntent.putExtra(Env.EXTRA_TARGET_INFO, targetActivityInfo);
                    if (stubActivityInfo != null) {
                        newTargetIntent.putExtra(Env.EXTRA_STUB_INFO, stubActivityInfo);
                    }
                    FieldUtils.writeDeclaredField(msg.obj, "intent", newTargetIntent);
                } else {
                    FieldUtils.writeDeclaredField(msg.obj, "intent", targetIntent);
                }
                FieldUtils.writeDeclaredField(msg.obj, "activityInfo", targetActivityInfo);

                Log.i(TAG, "handleLaunchActivity OK");
            } else {
                Log.e(TAG, "handleLaunchActivity oldInfo==null");
            }
        } else {
            Log.e(TAG, "handleLaunchActivity targetIntent==null");
        }
    } catch (Exception e) {
        Log.e(TAG, "handleLaunchActivity FAIL", e);
    }

    if (mCallback != null) {
        return mCallback.handleMessage(msg);
    } else {
        return false;
    }
}
```



## 两种方式的比较


两种方式个有优劣， DroidPlugin的方法更加复杂，但是也更加高级点。还可以hook service,broadcast等操作，而smail只hook了Instrumentation，只能影响activity。
然而在现实开发app中，几乎也不需要动态注册service等组件，所以hook Instrumentation可以应付绝大部分需求了。
