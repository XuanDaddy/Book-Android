# 【六】HandlerThread

### 简介

HandlerThread 类继承至 Thread 类，你可以把它看做是一个普通的线程类；当然，既然我们今天要说它，就不能在把它看做是一个普通的线程类了类处理了。HandlerThread 类与普通的线程类的主要区别就是：重写 run() 方法，并且创建了一个属于自己线程包含消息队列 Looper 对象；同时提供了 getLooper() 方法给外界获取 Looper 对象。

对于Handler的用法，往往是在一个线程中运行Looper，其他线程通过Handler来发送消息到Looper所在线程，这里涉及线程间的通信。

### 优点

* 开发中如果多次使用类似 new Thread(){...}.start() 这种方式开启一个子线程，会创建多个匿名线程，难于管理且使得程序运行起来越来越慢，而 HandlerThread 自带 Looper 包含一个独立的队列使它可以通过消息来多次重复使用当前线程，节省开支；
* 既然 HandlerThread 自带包含队列的 Looper ，那么它就可以分担主线程的 Looper 队列的压力；
* Android系统提供的 Handler 类内部的 Looper 默认绑定的是UI线程的消息队列，对于非UI线程又想使用消息机制，那么 HandlerThread 内部的 Looper 是最合适的，它不会干扰或阻塞UI线程。

### 利用HandlerThread创建

既然涉及多个线程的通信，会有同步的问题，Android为了简化Handler的创建过程，提供了HandlerThread类， 很多时候，在HandlerThread线程中运行Loop()方法，在其他线程中通过Handler发送消息到HandlerThread线程。通过wait/notifyAll的方式，有效地解决了多线程的同步问题。

示例代码：

```java
// Step 1: 创建并启动HandlerThread线程，内部包含Looper
HandlerThread handlerThread = new HandlerThread("gityuan.com");
handlerThread.start();

// Step 2: 创建Handler
Handler handler = new Handler(handlerThread.getLooper());

// Step 3: 发送消息
handler.post(new Runnable() {

        @Override
        public void run() {
            System.out.println("thread id="+Thread.currentThread().getId());
        }
    });
```

### 直接创建线程

```java
class LooperThread extends Thread {
    public Handler mHandler;

    public void run() {
        Looper.prepare();
        // Step 1: 创建Handler
        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                //处理即将发送过来的消息
                System.out.println("thread id="+Thread.currentThread().getId());
            }
        };

        Looper.loop();
    }
}

// Step 2: 创建并启动LooperThread线程，内部包含Looper
LooperThread looperThread = new LooperThread("gityuan.com");
looperThread.start();

// Step 3: 发送消息
LooperThread.mHandler.sendEmptyMessage(10);
```

### HandlerThread源码

* 创建HandlerThread对象

  HandlerThread 继承于Thread类

  ```java
  public HandlerThread(String name) {
      super(name);
      mPriority = Process.THREAD_PRIORITY_DEFAULT; //默认优先级
  }
  
  public HandlerThread(String name, int priority) {
      super(name);
      mPriority = priority;
  }
  ```

* #### 获取Looper对象

  获取HandlerThread线程中的Looper对象

  ```java
  public Looper getLooper() {
      // 当线程没有启动或者已经结束时，则返回null
      if (!isAlive()) {
          return null;
      }
  
      //当线程已经启动，则等待直到looper创建完成
      synchronized (this) {
          while (isAlive() && mLooper == null) {
              try {
                  wait(); //休眠等待
              } catch (InterruptedException e) {
              }
          }
      }
      return mLooper;
  }
  ```

* #### 执行HandlerThread的run()

  ```java
  public void run() {
      mTid = Process.myTid();  //获取线程的tid
      Looper.prepare();   // 创建Looper对象
      synchronized (this) {
          mLooper = Looper.myLooper(); //获取looper对象
          notifyAll(); //唤醒等待线程
      }
      Process.setThreadPriority(mPriority);
      onLooperPrepared();  // 该方法可通过覆写，实现自己的逻辑
      Looper.loop();   //进入循环模式
      mTid = -1;
  }
  ```

* #### Looper退出

  ```java
  public boolean quit() {
      Looper looper = getLooper();
      if (looper != null) {
          looper.quit(); //普通退出
          return true;
      }
      return false;
  }
  
  public boolean quitSafely() {
      Looper looper = getLooper();
      if (looper != null) {
          looper.quitSafely(); //安全退出
          return true;
      }
      return false;
  }
  ```

  quit()与quitSafely()的区别，仅仅在于是否移除当前正在处理的消息。移除当前正在处理的消息可能会出现不安全的行为。

### 总结

*  HandlerThread 将 Looper 转到子线程中处理，分担了 MainLooper 的工作量，降低了主线程的压力，使主界面更流畅；
* 开启一个线程起到多个线程的作用。处理任务是串行执行，按消息发送顺序进行处理。HandlerThread 本质是一个线程，在线程内部，代码是串行处理的；
* 由于每一个任务都将以队列的方式逐个被执行到，一旦队列中有某个任务执行时间过长，那么就会导致后续的任务都会被延迟处理；
* HandlerThread 拥有自己的消息队列，它不会干扰或阻塞UI线程；
* 对于网络操作，HandlerThread 并不适合，因为它只有一个线程，延时可能比较严重。