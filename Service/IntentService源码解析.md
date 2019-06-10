# 【九】IntentService源码解析

### 源代码

```java
public abstract class IntentService extends Service {
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;
    private String mName;
    private boolean mRedelivery;

    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }

    public IntentService(String name) {
        super();
        mName = name;
    }

    public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        mServiceLooper.quit();
    }

    @Override
    @Nullable
    public IBinder onBind(Intent intent) {
        return null;
    }

    @WorkerThread
    protected abstract void onHandleIntent(@Nullable Intent intent);
}
```

### 解析

#### 构造方法

> 构造方法需要传入一个字符串name,会在onCreate中拼装字符串name，传递给HandlerThread，最终传递给Thread(name)，其实就是线程名称。

```java
public IntentService(String name) {
    super();
    mName = name;
}
```

#### onCreate方法

```java
public void onCreate() {
   super.onCreate();
   HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
   thread.start();

   mServiceLooper = thread.getLooper();
   mServiceHandler = new ServiceHandler(mServiceLooper);
}
```

>1. 创建HandlerThread，并启动该线程; 
>
>2. 创建ServiceHandler,这是一个继承自Handler的内部类，并且绑定子线程的Looper; 
>3.  ServiceHandler会在handleMessage()方法中，传递intent对象给onHandleIntent()调用，完成后结束当前service; 

```java
private final class ServiceHandler extends Handler {
    public ServiceHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        onHandleIntent((Intent)msg.obj);
        stopSelf(msg.arg1);
    }
}
```

#### onStart方法

> 1. 接收外部传入的intent对象，通过ServiceHandler发送一个消息，传递给handleMessage()进行处理;
> 2. 回调完成后回调用 stopSelf(msg.arg1)，注意这个msg.arg1是个int值，相当于一个请求的唯一标识。每发送一个请求，会生成一个唯一的标识，然后将请求放入队列，当全部执行完成(最后一个请求也就相当于getLastStartId == startId)，或者当前发送的标识是最近发出的那一个（getLastStartId == startId），则会销毁我们的Service。如果传入的是-1则直接销毁。

```java
public void onStart(@Nullable Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}
```

#### onStartCommand方法

> 每次调用onStartCommand的时候，通过mServiceHandler发送一个消息，消息中包含我们的intent。然后在该mServiceHandler的handleMessage中去回调onHandleIntent(intent)就可以了。
>
> onStartCommand返回值：
>
> * START_STICKY：sticky的意思是“粘性的”。使用这个返回值时，我们启动的服务跟应用程序"粘"在一起，如果在执行完onStartCommand后，服务被异常kill掉，系统会自动重启该服务。当再次启动服务时，传入的第一个参数将为null;
> * START_NOT_STICKY：“非粘性的”。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统不会自动重启该服务。
> * START_REDELIVER_INTENT：重传Intent。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统会自动重启该服务，并将Intent的值传入。
> * START_STICKY_COMPATIBILITY：START_STICKY的兼容版本，但不保证服务被kill后一定能重启。

```java
public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
   onStart(intent, startId);
   return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
}
```

#### onDestrory方法

> 当任务完成销毁Service回调onDestory，可以看到在onDestroy中释放了我们的Looper:mServiceLooper.quit()。

```java
public void onDestroy() {
   mServiceLooper.quit();
}
```

