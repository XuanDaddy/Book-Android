# 【六】AIDL

#### Tips

> Activity既然不能通过bindService与远程Service建立联系，那么如何才能让Activity与一个远程Service建立关联呢？这就要使用AIDL来进行跨进程通信了（IPC）。

#### AIDL是什么

> AIDL（Android Interface Definition Language）是Android接口定义语言的意思，它可以用于让某个Service与多个应用程序组件之间进行跨进程通信，从而可以实现多个应用程序共享同一个Service的功能。

### AIDL使用方式

#### 创建AIDL文件

> 首先需要新建一个AIDL文件，在这个文件中定义好Activity需要与Service进行通信的方法。新建MyAIDLService.aidl文件，代码如下所示：

```java
package com.example.servicetest;
interface MyAIDLService {
	int plus(int a, int b);
	String toUpperCase(String str);
}
```

编译之后，会在build目录下的aidl目录中，会出现MyAIDLService.java文件。

#### 修改Service，使用AIDL

```java
public class MyService extends Service {
 
	......
 
	@Override
	public IBinder onBind(Intent intent) {
		return mBinder;
	}
 
	MyAIDLService.Stub mBinder = new Stub() {
 
		@Override
		public String toUpperCase(String str) throws RemoteException {
			if (str != null) {
				return str.toUpperCase();
			}
			return null;
		}
 
		@Override
		public int plus(int a, int b) throws RemoteException {
			return a + b;
		}
	};
 
}
```

这里先是对MyAIDLService.Stub进行了实现，重写里了toUpperCase()和plus()这两个方法。这两个方法的作用分别是将一个字符串全部转换成大写格式，以及将两个传入的整数进行相加。然后在onBind()方法中将MyAIDLService.Stub的实现返回。这里为什么可以这样写呢？因为Stub其实就是Binder的子类，所以在onBind()方法中可以直接返回Stub的实现。

#### 修改Activity，调用AIDL

```java
public class MainActivity extends Activity implements OnClickListener {
	private Button bindService;
 
	private Button unbindService;
	
	private MyAIDLService myAIDLService;
 
	private ServiceConnection connection = new ServiceConnection() {
 
		@Override
		public void onServiceDisconnected(ComponentName name) {
		}
 
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			myAIDLService = MyAIDLService.Stub.asInterface(service);
			try {
				int result = myAIDLService.plus(3, 5);
				String upperStr = myAIDLService.toUpperCase("hello world");
				Log.d("TAG", "result is " + result);
				Log.d("TAG", "upperStr is " + upperStr);
			} catch (RemoteException e) {
				e.printStackTrace();
			}
		}
	};
 
	......
 
}
```

只是修改了ServiceConnection中的代码。可以看到，这里首先使用了MyAIDLService.Stub.asInterface()方法将传入的IBinder对象传换成了MyAIDLService对象，接下来就可以调用在MyAIDLService.aidl文件中定义的所有接口了。这里我们先是调用了plus()方法，并传入了3和5作为参数，然后又调用了toUpperCase()方法，并传入hello world字符串作为参数，最后将调用方法的查看结果。

#### 注意

> 不过你也可以看出，目前的跨进程通信其实并没有什么实质上的作用，因为这只是在一个Activity里调用了同一个应用程序的Service里的方法。而跨进程通信的真正意义是为了让一个应用程序去访问另一个应用程序中的Service，以实现共享Service的功能。那么后面我们自然要学习一下，如何才能在其它的应用程序中调用到MyService里的方法。