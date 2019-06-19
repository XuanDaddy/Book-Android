# 【三】Handler

### 创建Handler

1. #### 无参构造

```java
public Handler() {
    this(null, false);
}

public Handler(Callback callback, boolean async) {
    //匿名类、内部类或本地类都必须申明为static，否则会警告可能出现内存泄露
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }
    //必须先执行Looper.prepare()，才能获取Looper对象，否则为null.
    mLooper = Looper.myLooper();  //从当前线程的TLS中获取Looper对象
    if (mLooper == null) {
        throw new RuntimeException("");
    }
    mQueue = mLooper.mQueue; //消息队列，来自Looper对象
    mCallback = callback;  //回调方法
    mAsynchronous = async; //设置消息是否为异步处理方式
}
```

对于Handler的无参构造方法，默认采用当前线程TLS中的Looper对象，并且callback回调方法为null，且消息为同步处理方式。只要执行的Looper.prepare()方法，那么便可以获取有效的Looper对象。

2. #### 有参构造

```java
public Handler(Looper looper) {
    this(looper, null, false);
}

public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

Handler类在构造方法中，可指定Looper，Callback回调方法以及消息的处理方式(同步或异步)，对于无参的handler，默认是当前线程的Looper。

### 消息分发机制

在Looper.loop()中，当发现有消息时，调用消息的目标handler，执行dispatchMessage()方法来分发消息。

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        //当Message存在回调方法，回调msg.callback.run()方法；
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            //当Handler存在Callback成员变量时，回调方法handleMessage()；
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //Handler自身的回调方法handleMessage()
        handleMessage(msg);
    }
}
```

**分发消息流程：**

1. 当`Message`的回调方法不为空时，则回调方法`msg.callback.run()`，其中callBack数据类型为Runnable,否则进入步骤2；
2. 当`Handler`的`mCallback`成员变量不为空时，则回调方法`mCallback.handleMessage(msg)`,否则进入步骤3；
3. 调用`Handler`自身的回调方法`handleMessage()`，该方法默认为空，Handler子类通过覆写该方法来完成具体的逻辑。

对于很多情况下，消息分发后的处理方法是第3种情况，即Handler.handleMessage()，一般地往往通过覆写该方法从而实现自己的业务逻辑。

### 消息发送

发送消息调用链:

![image](/Users/musikar/WorkSpace/Document/Books/Book-Android/Handler/images/消息发送.png)

从上图，可以发现所有的发消息方式，最终都是调用`MessageQueue.enqueueMessage()`;

1. #### sendEmptyMessage

```java
public final boolean sendEmptyMessage(int what) {
    return sendEmptyMessageDelayed(what, 0);
}
```

2. #### sendEmptyMessageDelayed

```java
public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageDelayed(msg, delayMillis);
}
```

3. #### sendMessageDelayed

```java
public final boolean sendMessageDelayed(Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

4. #### sendMessageAtTime

```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
```

5. #### sendMessageAtFrontOfQueue

```java
public final boolean sendMessageAtFrontOfQueue(Message msg) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        return false;
    }
    return enqueueMessage(queue, msg, 0);
}
```

该方法通过设置消息的触发时间为0，从而使Message加入到消息队列的队头。

6. #### post

```java
public final boolean post(Runnable r) {
   return  sendMessageDelayed(getPostMessage(r), 0);
}

private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```

7. #### postAtFrontOfQueue

```java
public final boolean postAtFrontOfQueue(Runnable r) {
    return sendMessageAtFrontOfQueue(getPostMessage(r));
}
```

8. #### enqueueMessage

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis); 
}
```

### 小节

`Handler.sendEmptyMessage()`等系列方法最终调用`MessageQueue.enqueueMessage(msg, uptimeMillis)`，将消息添加到消息队列中，其中uptimeMillis为系统当前的运行时间，不包括休眠时间。

### Handler其他方法

1. #### obtainMessage

获取消息

```java
public final Message obtainMessage() {
    return Message.obtain(this); 
}
```

`Handler.obtainMessage()`方法，最终调用`Message.obtainMessage(this)`，其中this为当前的Handler对象。

2. #### removeMessages

```java
public final void removeMessages(int what) {
    mQueue.removeMessages(this, what, null); 
}
```

`Handler`是消息机制中非常重要的辅助类，更多的实现都是`MessageQueue`, `Message`中的方法，Handler的目的是为了更加方便的使用消息机制。