# 【二】Service与Activity通信(bindService)

> 在【一】节中，直接运行了Service，可以在onCreate或onStartCommand方法里执行一些具体的逻辑了。不过这样的话，Service与Activity的关系不大，只是Activity通知了Service：”你可以启动了。“

### Service如何与Activity通信

#### 修改Service类

> 在之前的Service中，有一个onBind()方法始终没有使用过，这个方法就是用于和Activity建立关联的。

> 新增一个MyBinder类继承自Binder类，并添加一个startDownload方法执行任务。

```java
public class MyService extends Service {
 
	public static final String TAG = "MyService";
 
	private MyBinder mBinder = new MyBinder();
 
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
		return mBinder;
	}
 
	class MyBinder extends Binder {
 
		public void startDownload() {
			Log.d("TAG", "startDownload() executed");
			// 执行具体的下载任务
		}
 
	}
}
```

#### 修改Activity

> 首先创建了一个ServiceConnection的匿名类，在里面重写了onServiceConnected()方法和onServiceDisconnected()方法，这两个方法分别会在Activity与Service建立关联和解除关联的时候调用。在onServiceConnected()方法中，我们又通过向下转型得到了MyBinder的实例，有了这个实例，Activity和Service之间的关系就变得非常紧密了。现在我们可以在Activity中根据具体的场景来调用MyBinder中的任何public方法，即实现了Activity指挥Service干什么Service就去干什么的功能。

```java
public class MainActivity extends Activity {
 
	private MyService.MyBinder myBinder;
 
	private ServiceConnection connection = new ServiceConnection() {
 
		@Override
		public void onServiceDisconnected(ComponentName name) {
		}
 
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			myBinder = (MyService.MyBinder) service;
			myBinder.startDownload();
		}
	};
    
   ....
```

#### 绑定Service

> 这里我们仍然是构建出了一个Intent对象，然后调用bindService()方法将Activity和Service进行绑定。bindService()方法接收三个参数，第一个参数就是刚刚构建出的Intent对象，第二个参数是前面创建出的ServiceConnection的实例，第三个参数是一个标志位，这里传入BIND_AUTO_CREATE表示在Activity和Service建立关联后自动创建Service，这会使得MyService中的onCreate()方法得到执行，***但onStartCommand()方法不会执行***。

```java
Intent bindIntent = new Intent(this, MyService.class);
bindService(bindIntent, connection, BIND_AUTO_CREATE);
```

#### 解除绑定

> 如何我们想解除Activity和Service之间的关联怎么办呢？调用一下unbindService()方法就可以了，会调用onDestrory()方法，销毁Service。

```java
unbindService(connection);
```

#### 生命周期

> 使用bindService，只会执行onCreate()方法，不会执行onStartCommand()方法。

#### 注意

> 1. 任何一个Service在整个应用程序范围内都是通用的，即MyService不仅可以和MainActivity建立关联，还可以和任何一个Activity建立关联，而且在建立关联时它们都可以获取到**相同的MyBinder实例**。
> 2. 如果既调用了startService()，又执行了bindService(),这个时候你会发现，不管你是单独执行stopService还是unbindService，Service都不会被销毁，必要将两个方法都执行，Service才会被销毁。也就是说，使用stopService只会让Service停止，执行unbindService只会让Service和Activity解除关联，一个Service必须要在既没有和任何Activity关联又处理停止状态的时候才会被销毁。
> 3. 我们应该始终记得在Service的onDestroy()方法里去清理掉那些不再使用的资源，防止在Service被销毁后还会有一些不再使用的对象仍占用着内存