---
layout: post
title: "Android Architecture Componets"
subtitle: ""
date: 2017-02-20 17:00:00
author: "steven"
catalog: true
tags:
    - Android
---

2017 google I/O 开发者大会发布了android o,并确认了支持Kotlin。后来看官方网站，发现发布了一个新的App架构：Android Architecture Componets。

看了一下官方文档，这个架构组件其实就是将mvvm模式继续细化。架构图：

![image]({{ site.baseurl }}/img/post/android_architecture.png)

View层：还是acitivty/fragment

ViewModel:用来和View和model层交互，与之前的Android databinding不同的是，这次改为使用LiveData来存放和监控数据的变化

Repository：通过网络，db或缓存获取数据，并将数据提供给ViewMode,并负责数据的更新和同步。

model:获取，更新本地数据，包括缓存和db

Remote Data Source:从网络API获取的数据


该组件融合了android databinding和dagger，继续将组件分离。并主张模型驱动UI，并优先持久化模型。

该组件添加了一些新的工具：

### Lifecycle：

Lifecycle的目的是来监控acitivty/fragment的声明周期事件，可以将acitivty/fragment声明周期事件从acitivty/fragment中剥离出来，方便数据和UI同步。

LifecycleOwner:声明周期的拥有着，也就是Activity或Fragment，通过注册到Activity或Fragment获取到生命周期事件，并提供给其它组件


### LiveData：

LiveData是数据持用着，并且在给定的声明周期只内是可监控的。所有它和Lifecycle是绑定的

### ViewModel：

这个组件本次提供了一个ViewModel的基类，并用工厂模式将生成的ViewModel提供给View层。ViewModel负责向 Activity or a Fragment提供数据，并管理数据。也与Lifecycle绑定。

### Room：

本次组件中推出的一个sqlite组件，将数据库记录映射为数据模型。Room的特点是使用注解来进行数据操作，比较方便


ps:目前Android Architecture Componets还在开发期间，api可能会有变动。

补充：

架构思路就是上面的了，如果对其中的组件不满意或不符合需求，可以自己开发或选择其它替代组件，如LiveData可以是使用Rxjava或agera代替。Room可以使用Realm，greenDao或其它sqlite组件代替。Room的实现和编写风格很像Retrofit。所以网络请求也被推荐使用Retrofit。而各层之间解耦官方推荐使用Service Locator或依赖注入（DI）。官方推荐使用了Dagger 2。


## 测试：

View层：

使用Android UI Instrumentation test，即使用Espresso来测试UI，因为View层只和ViewModel交互，所有通过Mock ViewModel可以方便的来测试UI和各种场景。


ViewModel：

使用JUnit来测试，ViewModel只和Repository交互，所以可以通过Mock Repository来测试View model

Repository:

使用JUnit来测试即可，需要Mock Webservice或DAO.

Dao/Model:

推荐的测试方法是instrumentation，因为instrumentation测试不需要任何UI.Room提供了JUnit 测试实现。

Webservice：

通过Mock方法来测试，如[MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver)


## 网络状态

从网络获取数据，然后缓存数据显示数据，或着直接从缓存取出数据. 这是app获取数据的常用逻辑。

![image]({{ site.baseurl }}/img/post/network-bound-resource.png)

从网络获取数据时就需要处理网络的各种状态，还要决定是否使用缓存数据，所以官方推荐使用一个工具类来处理这部分逻辑，也方便复用。

官方的例子：

```java
public abstract class NetworkBoundResource<ResultType, RequestType> {
    private final MediatorLiveData<Resource<ResultType>> result = new MediatorLiveData<>();

    @MainThread
    NetworkBoundResource() {
        result.setValue(Resource.loading(null));
        LiveData<ResultType> dbSource = loadFromDb();
        result.addSource(dbSource, data -> {
            result.removeSource(dbSource);
            if (shouldFetch(data)) {
                fetchFromNetwork(dbSource);
            } else {
                result.addSource(dbSource,
                        newData -> result.setValue(Resource.success(newData)));
            }
        });
    }

    private void fetchFromNetwork(final LiveData<ResultType> dbSource) {
        LiveData<ApiResponse<RequestType>> apiResponse = createCall();
        // we re-attach dbSource as a new source,
        // it will dispatch its latest value quickly
        result.addSource(dbSource,
                newData -> result.setValue(Resource.loading(newData)));
        result.addSource(apiResponse, response -> {
            result.removeSource(apiResponse);
            result.removeSource(dbSource);
            //noinspection ConstantConditions
            if (response.isSuccessful()) {
                saveResultAndReInit(response);
            } else {
                onFetchFailed();
                result.addSource(dbSource,
                        newData -> result.setValue(
                                Resource.error(response.errorMessage, newData)));
            }
        });
    }

    @MainThread
    private void saveResultAndReInit(ApiResponse<RequestType> response) {
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... voids) {
                saveCallResult(response.body);
                return null;
            }

            @Override
            protected void onPostExecute(Void aVoid) {
                // we specially request a new live data,
                // otherwise we will get immediately last cached value,
                // which may not be updated with latest results received from network.
                result.addSource(loadFromDb(),
                        newData -> result.setValue(Resource.success(newData)));
            }
        }.execute();
    }
}
```
使用：
```java

class UserRepository {
    Webservice webservice;
    UserDao userDao;

    public LiveData<Resource<User>> loadUser(final String userId) {
        return new NetworkBoundResource<User,User>() {
            @Override
            protected void saveCallResult(@NonNull User item) {
                userDao.insert(item);
            }

            @Override
            protected boolean shouldFetch(@Nullable User data) {
                return rateLimiter.canFetch(userId) && (data == null || !isFresh(data));
            }

            @NonNull @Override
            protected LiveData<User> loadFromDb() {
                return userDao.load(userId);
            }

            @NonNull @Override
            protected LiveData<ApiResponse<User>> createCall() {
                return webservice.getUser(userId);
            }
        }.getAsLiveData();
    }
}
```

## 最后

Android Architecture Componets还是在开发阶段的一个组件，还极不完善,API随时可能有改动.不建议在实际项目中使用。不过这个组件的思想可以借鉴，也可以使用其它的第三方类库来实现该架构的思想。

官方文档：https://developer.android.com/topic/libraries/architecture/guide.html

官方demo:https://github.com/googlesamples/android-architecture-components
