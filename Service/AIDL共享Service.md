# 【七】AIDL共享Service

#### Tips

> 跨进程通信的真正意义是为了让一个应用程序去访问另一个应用程序中的Service，以实现共享Service的功能。

#### 同一应用绑定Service的方式

> 如果想要让Activity与Service之间建立关联，需要调用bindService()方法，并将Intent作为参数传递进去，在Intent里指定好要绑定的Service。

```java
Intent bindIntent = new Intent(this, MyService.class);
bindService(bindIntent, connection, BIND_AUTO_CREATE);
```

### 不同应用绑定Service

#### 隐式声明

> 这里在构建Intent的时候是使用MyService.class来指定要绑定哪一个Service的，但是在另一个应用程序中去绑定Service的时候并没有MyService这个类，这时就必须使用到隐式Intent了。现在修改AndroidManifest.xml中的代码，给MyService加上一个action。这样的话，MyService可以响应带有com.example.servicetest.MyAIDLService这个action的Intent。

```java
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.servicetest"
    android:versionCode="1"
    android:versionName="1.0" >
 
    ......
 
    <service
        android:name="com.example.servicetest.MyService"
        android:process=":remote" >
        <intent-filter>
            <action android:name="com.example.servicetest.MyAIDLService"/>
        </intent-filter>
    </service>
 
</manifest>
```

#### 新应用建立Service关联

> 1. 创建一个新的Android项目，起名为ClientTest，我们就尝试在这个程序中远程调用MyService中的方法。
> 2. 将MyAIDLService.aidl文件从ServiceTest项目中拷贝过来，注意要将原有的包路径一起拷贝过来。

#### 远程调用

> 这和在同一应用的代码几乎是完全相同的，只是在让Activity和Service建立关联的时候我们使用了隐式Intent，将Intent的action指定成了com.example.servicetest.MyAIDLService。在当前Activity和MyService建立关联之后，我们仍然是调用了plus()和toUpperCase()这两个方法，远程的MyService会对传入的参数进行处理并返回结果，然后查看结果。

```java
public class MainActivity extends Activity {
 
	private MyAIDLService myAIDLService;
 
	private ServiceConnection connection = new ServiceConnection() {
 
		@Override
		public void onServiceDisconnected(ComponentName name) {
		}
 
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			myAIDLService = MyAIDLService.Stub.asInterface(service);
			try {
				int result = myAIDLService.plus(50, 50);
				String upperStr = myAIDLService.toUpperCase("comes from ClientTest");
				Log.d("TAG", "result is " + result);
				Log.d("TAG", "upperStr is " + upperStr);
			} catch (RemoteException e) {
				e.printStackTrace();
			}
		}
	};
 
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		Button bindService = (Button) findViewById(R.id.bind_service);
		bindService.setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				Intent intent = new Intent("com.example.servicetest.MyAIDLService");
				bindService(intent, connection, BIND_AUTO_CREATE);
			}
		});
	}
 
}
```

#### 注意

> 由于这是在不同的进程之间传递数据，Android对这类数据的格式支持是非常有限的，基本上只能传递Java的基本数据类型、字符串、List或Map等。那么如果我想传递一个自定义的类该怎么办呢？这就必须要让这个类去实现Parcelable接口，并且要给这个类也定义一个同名的AIDL文件。