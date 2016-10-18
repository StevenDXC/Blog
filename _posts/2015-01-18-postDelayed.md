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

在android中做延时处理一般用handler.postDelayed()和view.postDelayed(action,delay)来实现，view.postDelayed也是通过handlder.postDelayed来实现的，不过有一些特殊处理的地方。

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
此处会将传入的消息对象根据触发时间（when）插入到message queue中。然后判断是否要唤醒等待中的队列。
. 如果插在队列中间。说明该消息不需要马上处理，不需要由这个消息来唤醒队列。
. 如果插在队列头部（或者when=0），则表明要马上处理这个消息。如果当前队列正在堵塞，则需要唤醒它进行处理。
通过nativeWake方法唤醒队列。


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
只是调用了MessageQueue.next()方法。可能会阻塞。

```java
Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```


该方法会先调用nativePollOnce阻塞，然后进入死循环。nativePollOnce()的作用类似与object.wait()，使用了nvtive方法来对线程进行精确时间的唤醒，
如果head Message是有延迟而且延迟时间没到的（now < msg.when），计算下时间间隔（nextPollTimeoutMillis），设置timeout为两者之差，进入下一次循环。
如果Message无延时或已到达执行时间，则直接返回该Message.

如果当前的Message阻塞住了MessageQueue（pendingIdleHandlerCount <= 0），把全局变量mBlocked改为true，在下一个Message放入队时会判断这个message的位置。如果在queue的头部，则用nativeWake唤醒线程。


总结：[msg loop] -->调用MessageQueue.next(),若头部的消息需要被执行在返回该消息，send该消息之后继续调用next方法获取下一个消息，若头部的不需要被执行则阻塞住队列，若有等待执行的Message,计算一下剩余时间,继续调用nativePollOnce()阻塞，无限循环到阻塞时间到或者下一次有Message进队。下个消息进队时若不需要延时在放在Queue的头部，否则在放在Queue的中部。


view.postDelayed
---

view的延时也是调用了handler的postDelayed.

```java
public boolean postDelayed(Runnable action, long delayMillis) {
       final AttachInfo attachInfo = mAttachInfo;
       if (attachInfo != null) {
           return attachInfo.mHandler.postDelayed(action, delayMillis);
       }

       // Postpone the runnable until we know on which thread it needs to run.
       // Assume that the runnable will be successfully placed after attach.
       getRunQueue().postDelayed(action, delayMillis);
       return true;
   }
```
若view的AttachInfo不为空，则调用AttachInfo中的handler的postDelayed。若AttachInfo为空，则先将action放入RunQueue中。RunQueue为HandlerActionQueue，用来存放view没有handler时的action。

执行action：


```java
public void executeActions(Handler handler) {
    synchronized (this) {
        final HandlerAction[] actions = mActions;
        for (int i = 0, count = mCount; i < count; i++) {
            final HandlerAction handlerAction = actions[i];
            handler.postDelayed(handlerAction.action, handlerAction.delay);
        }

        mActions = null;
        mCount = 0;
    }
}
```

还是需要传入一个Handler，利用handler来执行action。当view被关联到window时，会执行该队列中的Action.   

AttachInfo是View的一个附加信息存储类，当view被关联到window时会被赋值。
