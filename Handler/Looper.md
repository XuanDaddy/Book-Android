# 【二】Looper

### prepare()

对于无参的情况，默认调用`prepare(true)`，表示的是这个Looper允许退出，而对于false的情况则表示当前Looper不允许退出。

```java
private static void prepare(boolean quitAllowed) {
    //每个线程只允许执行一次该方法，第二次执行时线程的TLS已有数据，则会抛出异常。
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    //创建Looper对象，并保存到当前线程的TLS区域
    sThreadLocal.set(new Looper(quitAllowed));
}
```

这里的`sThreadLocal`是ThreadLocal类型，下面，先说说ThreadLocal。

**ThreadLocal**： 线程本地存储区（Thread Local Storage，简称为TLS），每个线程都有自己的私有的本地存储区域，不同线程之间彼此不能访问对方的TLS区域。TLS常用的操作方法：

- `ThreadLocal.set(T value)`：将value存储到当前线程的TLS区域，源码如下：

  ```java
  public void set(T value) {
      Thread currentThread = Thread.currentThread(); //获取当前线程
      Values values = values(currentThread); //查找当前线程的本地储存区
      if (values == null) {
          //当线程本地存储区，尚未存储该线程相关信息时，则创建Values对象
          values = initializeValues(currentThread);
      }
      //保存数据value到当前线程this
      values.put(this, value);
  }
  ```

- `ThreadLocal.get()`：获取当前线程TLS区域的数据，源码如下：

  ```java
  public T get() {
      Thread currentThread = Thread.currentThread(); //获取当前线程
      Values values = values(currentThread); //查找当前线程的本地储存区
      if (values != null) {
          Object[] table = values.table;
          int index = hash & values.mask;
          if (this.reference == table[index]) {
              return (T) table[index + 1]; //返回当前线程储存区中的数据
          }
      } else {
          //创建Values对象
          values = initializeValues(currentThread);
      }
      return (T) values.getAfterMiss(this); //从目标线程存储区没有查询是则返回null
  }
  ```

ThreadLocal的get()和set()方法操作的类型都是泛型，接着回到前面提到的`sThreadLocal`变量，其定义如下：

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>()
```

可见`sThreadLocal`的get()和set()操作的类型都是`Looper`类型。

**Looper.prepare()**

Looper.prepare()在每个线程只允许执行一次，该方法会创建Looper对象，Looper的构造方法中会创建一个MessageQueue对象，再将Looper对象保存到当前线程TLS。

对于Looper类型的构造方法如下

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);  //创建MessageQueue对象. 
    mThread = Thread.currentThread();  //记录当前线程.
}
```

另外，与prepare()相近功能的，还有一个`prepareMainLooper()`方法，该方法主要在ActivityThread类中使用。

```java
public static void prepareMainLooper() {
    prepare(false); //设置不允许退出的Looper
    synchronized (Looper.class) {
        //将当前的Looper保存为主Looper，每个线程只允许执行一次。
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```

### loop()

```java
public static void loop() {
    final Looper me = myLooper();  //获取TLS存储的Looper对象
    final MessageQueue queue = me.mQueue;  //获取Looper对象中的消息队列

    Binder.clearCallingIdentity();
    //确保在权限检查时基于本地进程，而不是调用进程。
    final long ident = Binder.clearCallingIdentity();

    for (;;) { //进入loop的主循环方法
        Message msg = queue.next(); //可能会阻塞 
        if (msg == null) { //没有消息，则退出循环
            return;
        }

        //默认为null，可通过setMessageLogging()方法来指定输出，用于debug功能
        Printer logging = me.mLogging;  
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }
        msg.target.dispatchMessage(msg); //用于分发Message 
        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        //恢复调用者信息
        final long newIdent = Binder.clearCallingIdentity();
        msg.recycleUnchecked();  //将Message放入消息池 
    }
}
```

loop()进入循环模式，不断重复下面的操作，直到没有消息时退出循环

- 读取MessageQueue的下一条Message；
- 把Message分发给相应的target；
- 再把分发后的Message回收到消息池，以便重复利用。

这是这个消息处理的核心部分。另外，上面代码中可以看到有logging方法，这是用于debug的，默认情况下`logging == null`，通过设置setMessageLogging()用来开启debug工作。

### quit()

```java
public void quit() {
    mQueue.quit(false); //消息移除
}

public void quitSafely() {
    mQueue.quit(true); //安全地消息移除
}
```

Looper.quit()方法的实现最终调用的是MessageQueue.quit()方法

**MessageQueue.quit()**

```java
void quit(boolean safe) {
        // 当mQuitAllowed为false，表示不运行退出，强行调用quit()会抛出异常
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }
        synchronized (this) {
            if (mQuitting) { //防止多次执行退出操作
                return;
            }
            mQuitting = true;
            if (safe) {
                removeAllFutureMessagesLocked(); //移除尚未触发的所有消息
            } else {
                removeAllMessagesLocked(); //移除所有的消息
            }
            //mQuitting=false，那么认定为 mPtr != 0
            nativeWake(mPtr);
        }
    }
```

消息退出的方式：

- 当safe =true时，只移除尚未触发的所有消息，对于正在触发的消息并不移除；
- 当safe =flase时，移除所有的消息

### myLooper()

用于获取TLS存储的Looper对象

```java
public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
```

