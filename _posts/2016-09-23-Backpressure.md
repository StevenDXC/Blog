---
layout: post
title: "BackPressure"
subtitle: ""
date: 2016-09-23 14:00:00
author: "steven"
catalog: true
tags:
    - Develop
---


最近更新了RxJava2，看到了新增的Flowable支持背压。什么是背压呢？

背压（BackPressure）是一个比较抽象的概念，背压是“流控”的一种实现方案。

## 流控

流控就是控制流量，例如当服务端能处理的请求数量少于当时接受的请求数量的时候，就需要流控。

常用的流控实现方案有4种：

* 背压（BackPressure）
* 节流（Throttling）
* 打包
* 阻塞


## 背压

Reactive Pull,即服务端告诉客户端能处理多少请求，客户端就送多少请求。这种方案只适用于允许降低发送速率的发送源，例如文件传输，可以把发送速度控制的很小。但是想直播类就不行，速率低到某个程度之后就没办法看了。

RxJava支持的背压的方式：

* BackpressureStrategy.BUFFER
* BackpressureStrategy.DROP
* BackpressureStrategy.LATEST

BUFFER 就是不丢弃请求，把收到所有请求都缓存起来，当来请求处理时再发出去。
DROP 和 LATEST 都是丢弃请求，服务端通过请求产生配额，将配额告诉客户端，客户端收到多少配额就发送多少请求。配额消耗完之后，客户端丢弃之后的请求。
DROP即配额消耗完之后直接丢弃请求。而LATEST则缓存总后（最新）的一条请求，当服务端又发来新的配额时，将缓存的最新的请求发送给服务端。


## 节流

节流就是服务端直接丢弃处理不了的数据。主要有三种策略：

* throttleLast
* throttleFirst
* throttleWithTimeout

throttleLast 和 throttleFirst 类似，就是接受到多个处理不了的请求时，选择处理那个，剩下的丢弃。throttleLast就是处理最新的请求。
throttleFirst 则为处理最先发来的请求，剩下都丢弃掉。
throttleWithTimeout 每次收到一个请求之后就开始计时，超时之后的所有过来的请求就会被丢弃。


## 打包

就是把请求的数据都打包成一个大的包裹，服务端处理的请求数量就少了。

## 阻塞

Callstack blocking,阻塞住整个调用栈。这种方式要求所有的请求都在一个线程之内。相对于服务端只能同时处理一个请求，后面来的请求只能在后面等待。
而背压是服务端告诉客户端能处理请求了，客户端再发送请求。
