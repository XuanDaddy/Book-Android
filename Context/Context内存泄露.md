# 【五】Context内存泄露

### Activity Context内存泄露的几种情况

> Android 中的 Activity Context 内存泄露，简单说就是 Activity 调用 onDestroy() 方法销毁后，此 Activity 还被其他对象强引用，导致此 Activity 不能被 GC（JAVA 垃圾回收器） 回收，最后出现内存泄露。

* #### 静态Activity

```java
static Activity activity;
这里使用了 static 来修饰 Activity，静态变量持有 Activity 对象很容易造成内存泄漏，因为静态变量是和应用存活时间相同的，所以当 Activity 生命周期结束时，引用仍被持有。
解决办法：
    1.去掉 static 关键字，使用别的方法来实现想要的功能。任何时候不建议 static 修饰 Activity，如果这样做了，Android Studio 也会给出警告提示。
    2.在 onDestroy 方法中置空 Activity 静态引用。    
    @Override
    public void onDestroy() { 
        super.onDestroy(); 
        if (activity != null) { 
            activity = null;     
        }
     }
    3.也可以使用到弱引用解决，确保在 Activity 销毁时，垃圾回收机制可以将其回收。
    private static WeakReference<MainActivity> activityReference;
    private void setStaticActivity() {
        activityReference = new WeakReference<MainActivity>(this);
    }
    // 注意在使用时，必须判空
    private void useActivityReference(){
        MainActivity activity = activityReference.get();
        if (activity != null) {
            // ...
        }
    }

```

* #### static间接修饰Activity Context

```java
错误代码：
public class LoadingDialog extends Dialog {
    private static LoadingDialog mDialog;
    private TextView mText;
}
这里 static 虽然没有直接修饰 TextView（拥有 Context 引用），但是修饰了 mDialog 成员变量，mDialog 是 一个 LoadingDialog 对象， LoadingDialog 对象 包含一个 TextView 类型的成员变量，所以 mText 变量的生命周期也是全局的，和应用一样。这样，mText 持有的 Context 对象销毁时，没有 GC 回收，导致内存泄露。
解决办法：
    1.不使用 static 修饰；
    2.在适当的地方如，dismissLoadingDialog() 方法中置空 mDialog，这样虽然可以解决，但是也存在风险，如果 dismissLoadingDialog() 没有或忘记被调用，同样也会导致内存泄漏。
```

* #### 单例引用Activity Context

```java
问题代码：
public class AppManager {
    private static AppManager instance;
    private Context context;
    private AppManager(Context context) {
        this.context = context;
    }
    public static AppManager getInstance(Context context) {
        if (instance == null) {
            instance = new AppManager(context);
        }
        return instance;
    }
}
这是一个非线程安全的单例模式，instance作为静态对象，其生命周期要长于普通的对象，其中也包含Activity，假如Activity A去getInstance获得instance对象，传入this，常驻内存的Singleton保存了你传入的Activity A对象，并一直持有，即使Activity被销毁掉，但因为它的引用还存在于一个Singleton中，就不可能被GC掉，这样就导致了内存泄漏。
解决办法：
    1.使用 Applicaion Context 代替 Activity Context (推荐)
    private AppManager(Context context) {
        this.context = context.getAppcalition();
    }
    或者在 App 中写一个获取 Applicaion Context 的方法。
    private AppManager() {
        this.context = App.getAppcalitionContext();
    }
    2.在调用的地方使用弱引用
    WeakReference<MainActivity> activityReference = new WeakReference<MainActivity>(this);
    Context context = activityReference.get();
    if(context != null){
        AppManager.getInstance(context);
        // ...
    }
```

* #### 非静态或匿名内部类执行耗时任务

```java
 public class MainActivity extends Activity {  
    @Override
    protected void onCreate(Bundle savedInstanceState) {    
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        test();
    }  
    public void test() {    
        new Thread(new Runnable() {     
            @Override
            public void run() {        
                while (true) {          
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }
}
由于非静态内部类或匿名内部类都会拥有所在外部类的引用，上边的代码，由于 new Thread 是匿名内部类，并且执行了长时间（一直）的任务，当 Activity 销毁后，该匿名内部类还在执行任务，导致外部的 Activity 不能被回收，导致内存泄露。
解决方法：
// static 修饰方法
public static void test() {
    new Thread(new Runnable() {     
            @Override
            public void run() {        
                while (true) {          
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
}
如果是内部类，这样写:
static class MyThread extends Thread {
    @Override
    public void run() {
        // ...
    }
}
```

* #### 静态内部类持有外部类静态成员变量

```java
虽然静态内部类的生命周期和外部类无关，但是如果在内部类中想要引入外部成员变量的话，这个成员变量必须是静态的了，这也可能导致内存泄露。
问题代码：
public class MainActivity extends Activity {  
    private static MainActivity mMainActivity;
    @Override
    protected void onCreate(Bundle savedInstanceState) {    
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mMainActivity = this;
    }  
    private static class MyThread extends Thread {
        @Override
        public void run() {
           // 耗时操作
           mMainActivity...
        }
    }
}
解决方法：
使用弱引用:
public class MainActivity extends Activity {  
    private static WeakReference<MainActivity> activityReference;
    @Override
    protected void onCreate(Bundle savedInstanceState) {    
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        activityReference = new WeakReference<MainActivity>(this);;
    }  
    private static class MyThread extends Thread {
        @Override
        public void run() {
           // 耗时操作
           MainActivity activity = activityReference.get();
           if(activity != null){
               activity...
           }
        }
    }
}
```

