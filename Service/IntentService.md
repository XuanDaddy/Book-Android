# 【八】IntentService

#### Tips

> IntentService是一个基于Service的一个类，用来处理异步的请求。你可以通过startService(Intent)来提交请求，该Service会在需要的时候创建，当完成所有的任务以后自己关闭，且请求是在工作线程处理的。
>
> 这么说，我们使用了IntentService最起码有两个好处：
>
> 1. 一方面不需要自己去new Thread了；
> 2. 另一方面不需要考虑在什么时候关闭该Service了。

#### IntentService使用

> * 同Service一样，继承自IntentService，自定义一个MyIntentService；
> * 需要调用父类的有参构造方法；
> * 覆写onHandleIntent()方法；

```java
public class MyIntentService extends IntentService {

    public MyIntentService() {
        super("MyIntentService");
    }

    @Override
    public void onCreate() {
        super.onCreate();
        System.out.println("IntentService创建");
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {

        System.out.println("IntentService线程ID:" + Thread.currentThread().getId());
        try {
            Thread.sleep(3000);
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("IntentService执行结束，线程ID："+ Thread.currentThread().getId());
    }
}
```

通过对比主线程和IntentService中的线程ID，可以判断为不同线程。 

并且，当IntentService执行完成后就销毁了，重新开启IntentService，会重新执行onCreate()方法。 