# 【一】Service直接运行(startService)

### Service是什么

> Service是四大组件之一，主要用于在后台处理一些耗时的逻辑或者去执行某些需要长期运行的任务。必要的时候我们甚至可以在程序退出的情况下，让Service在后台继续保持运行状态。

### Service基本使用

#### 新建Service

> 自定义类继承自Service，重写了onCreate、onStartCommand、onDestrory方法

```java
public class MyService extends Service {
	public static final String TAG = "MyService";
 
	@Override
	public void onCreate() {
		super.onCreate();
		Log.d(TAG, "onCreate() executed");
	}
 
	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		Log.d(TAG, "onStartCommand() executed");
		return super.onStartCommand(intent, flags, startId);
	}
	
	@Override
	public void onDestroy() {
		super.onDestroy();
		Log.d(TAG, "onDestroy() executed");
	}
 
	@Override
	public IBinder onBind(Intent intent) {
		return null;
	}
 
}
```

#### 注册Service

> 同Activity一样，在androidManifest中注册service

```java
<service android:name="com.example.servicetest.MyService" />
```

#### 运行Service

> 使用startService方法启动一个Service

```java
Intent startIntent = new Intent(this, MyService.class);
startService(startIntent);
```

#### 停止Service

> 使用stopService方法停止Service,会执行onDestrory()方法，销毁Service。

```java
Intent stopIntent = new Intent(this, MyService.class);
stopService(stopIntent);
```

#### 查看Service

> 可以到手机的应用程序管理界面查看一下Service是否正在运行。

#### 生命周期

> 直接启动Service，会依次执行onCreate、onStartCommand方法。

#### 再次启动Service

> 如果Service已经创建过了，不会再执行onCreate（这是由于onCreate()方法只会在Service第一次被创建的时候调用），只会执行onStartCommand。多次启动同一个Service，也是一样的效果。