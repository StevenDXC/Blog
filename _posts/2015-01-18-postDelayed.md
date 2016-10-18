---
layout:     post
title:      "android postDelayed实现"
subtitle:   ""
date:       2015-01-18 12:00:00
author:     "steven"
catalog: true
tags:
    - Android
---

在android中做延时处理一般用handler.postDelayed()和view.postDelayed(action,delay)来实现，view.postDelayed也是通过handlder.postDelayed来实现的，不过有一些不同的地方。

handler.postDelayed
----

handler处理延时逻辑是通过发送延时消息来处理的


```java
//source
public final boolean postDelayed(Runnable r, long delayMillis){
     return sendMessageDelayed(getPostMessage(r), delayMillis);
}
```

sendMessageDelayed:


```java
public final boolean sendMessageDelayed(Message msg, long delayMillis)
  {
      if (delayMillis < 0) {
          delayMillis = 0;
      }
      return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
  }
```

sendMessageAtTime:

```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```
将延时消息放入到消息队列中。SystemClock.uptimeMillis()是开机到现在的时间（毫秒），不包括睡眠的时间。不用System.currentTimeMillis()的原因是System.currentTimeMillis()的时间是可以被System.setCurrentTimeMillis修改的，如果被修改了则发生难以想象的后果。

enqueueMessage调用MessageQueue.enqueueMessage(msg,uptimeMillis)

```java
boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```
此处只是把应该执行的时间when设置到msg里面,并没有处理执行

looper执行MessageQueue中的消息：
```java
for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            try {
                msg.target.dispatchMessage(msg);
            } finally {
                ...
            }

            msg.recycleUnchecked();
        }
    }
```
