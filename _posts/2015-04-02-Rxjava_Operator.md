---
layout:     post
title:      "RxJava操作符"
subtitle:   ""
date:       2015-04-02 13:00:00
author:     "steven"
catalog:    true
tags:
    - Android
---

RxJava 组合操作符

本文所用的到Observable对象和subscriber：

```java
Observable<Integer> o1 = Observable.just(1,2,3).delay(5, TimeUnit.SECONDS);
Observable<Integer> o2 = Observable.just(11,12,13).delay(3,TimeUnit.SECONDS);
Observable<Integer> o3 = Observable.just(21,22,23);
Observable<Integer> o4 = Observable.error(new Throwable("--->error<---"));

Subscriber subscriber = new Subscriber<Integer>() {
    public void onCompleted() {
        System.out.println("onCompleted");
    }

    public void onError(Throwable throwable) {
        System.out.println("onError:"+throwable);
    }

    public void onNext(Integer integer) {
        System.out.println("onNext:"+integer);
    }
};

```

amb
---

传入多个Observable时，只发射其中首先发射数据或通知的那个Observable的所有数据，其它的被丢弃。

```java
Observable.amb(o1,o2,o3).subscribe(subscriber);
```
输出日志：

onNext:21
onNext:22
onNext:23
onCompleted

concat
---
Concat操作符将多个Observable结合成一个Observable并发射数据，并且严格按照先后顺序发射数据，前一个Observable的数据没有发射完，后面Observable不发射数据。


```java
Observable.concat(o1,o2,o3).subscribe(subscriber);
```
输出：

```java
onNext:1
onNext:2
onNext:3
onNext:11
onNext:12
onNext:13
onNext:21
onNext:22
onNext:23
onCompleted
```

若中间发生错误，则调用onError结束执行，不再发射后面的数据

```java
Observable.concat(o1,o4,o2).subscribe(subscriber);
```

输出：

```java
onNext:1
onNext:2
onNext:3
onError:java.lang.Throwable: --->error<---
```j

concatDelayError:若其中的某个Observable发生错误，则继续执行完后面的Observable，将发生错误的Observable放在最后。


```java
Observable.concatDelayError(o4,o1,o2).subscribe(subscriber);
```

输出日志：

```java
onNext:1
onNext:2
onNext:3
onNext:11
onNext:12
onNext:13
onError:java.lang.Throwable: --->error<---
```

concatEager:与concat基本逻辑一致，不同的是一旦合并后的Observable被订阅，operator订阅所有的原始Observable，operator缓存每个Observable发射出来的值，所有Observable发射完成之后，operator将所有缓存的值按顺序一次排放出来。

```java
Observable.concatEager(o1,o2,o3).subscribe(subscriber);
```

```java
onNext:1
onNext:2
onNext:3
onNext:11
onNext:12
onNext:13
onNext:21
onNext:22
onNext:23
onCompleted
```


merge
---

merge操作符是按照多个Observable提交结果的时间顺序，对Observable进行合并。合并为一个Observable后并发执行所有原始Observable。

```java
Observable.merge(o1,o2,o3).subscribe(subscriber);
```

输出：

```java
onNext:21
onNext:22
onNext:23
onNext:11
onNext:12
onNext:13
onNext:1
onNext:2
onNext:3
onCompleted
```

mergeDelayError:与concatDelayError的DelayError逻辑一致

```java
Observable.mergeDelayError(o4,o2,o3).subscribe(subscriber);
```

输出：

```java
onNext:21
onNext:22
onNext:23
onNext:11
onNext:12
onNext:13
onError:java.lang.Throwable: --->error<---
```

zip
---

zip操作符是把多个observable提交的结果，严格按照顺序进行合并，每个数据只能组合一次。所有原Observable并发执行。最终组合的数据的数量由发射数据最少的Observable来决定，其它observable多出来的值被丢弃。

```java
Observable o1 = Observable.just(1,2).delay(5, TimeUnit.SECONDS);
Observable.zip(o1, o2, o3, new Func3<Integer, Integer, Integer, Integer>() {
           public Integer call(Integer integer, Integer integer2, Integer integer3) {
               System.out.println("call:"+integer+"---"+integer2+"---"+integer3);
               return integer+integer2+integer3;
           }
       }).subscribe(subscriber);
```

输出：

```java
call:1---11---21
onNext:33
call:2---12---22
onNext:36
onCompleted
```

combineLatest
---
当两个Observables中的任何一个发射了数据时，使用一个函数结合每个Observable发射的最近数据项，并且基于这个函数的结果发射数据。
CombineLatest操作符行为类似于zip，但是只有当原始的Observable中的每一个都发射了一条数据时zip才发射数据。CombineLatest则在原始的Observable中任意一个发射了数据时发射一条数据。当原始Observables的任何一个发射了一条数据时，CombineLatest使用一个函数结合它们最近发射的数据，然后发射这个函数的返回值。

```java
Observable o1 = Observable.just(1,2).delay(5, TimeUnit.SECONDS);
Observable.combineLatest(o1, o2, o3, new Func3<Integer, Integer, Integer, Integer>() {
           public Integer call(Integer integer, Integer integer2, Integer integer3) {
               System.out.println("call:"+integer+"---"+integer2+"--"+integer3);
               return integer+integer2+integer3;
           }
       }).subscribe(subscriber);
```

输出：

```java
call:1---13--23
onNext:37
call:2---13--23
onNext:38
onCompleted
```
