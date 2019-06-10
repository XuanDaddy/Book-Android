# 【三】Service与Thread的关系

> 不少Android初学者都可能会有这样的疑惑，Service和Thread到底有什么关系呢？什么时候应该用Service，什么时候又应该用Thread？答案可能会有点让你吃惊，因为Service和Thread之间没有任何关系！ 
>
> 之所以有不少人会把它们联系起来，主要就是因为Service的后台概念。Thread我们大家都知道，是用于开启一个子线程，在这里去执行一些耗时操作就不会阻塞主线程的运行。而Service我们最初理解的时候，总会觉得它是用来处理一些后台任务的，一些比较耗时的操作也可以放在这里运行，这就会让人产生混淆了。 

#### Service运行线程

> **Service其实是运行在主线程里。** 也就是说如果你在Service里编写了非常耗时的代码，程序必定会出现ANR的。

#### 后台与线程

> 不要把后台和子线程联系在一起就行了，这是**两个完全不同的概念。Android的后台就是指，它的运行是完全不依赖UI的。** 即使Activity被销毁，或者程序被关闭，只要进程还在，Service就可以继续运行。比如说一些应用程序，始终需要与服务器之间始终保持着心跳连接，就可以使用Service来实现。你可能又会问，前面不是刚刚验证过Service是运行在主线程里的么？在这里一直执行着心跳连接，难道就不会阻塞主线程的运行吗？当然会，但是我们可以在Service中再创建一个子线程，然后在这里去处理耗时逻辑就没问题了。

#### 为何在Service创建子线程

> 这是因为Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。而且在一个Activity中创建的子线程，另一个Activity无法对其进行操作。但是Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法，即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。

#### 标准Service示例

```java
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
	new Thread(new Runnable() {
		@Override
		public void run() {
			// 开始执行后台任务
		}
	}).start();
	return super.onStartCommand(intent, flags, startId);
}
 
class MyBinder extends Binder {
 
	public void startDownload() {
		new Thread(new Runnable() {
			@Override
			public void run() {
				// 执行具体的下载任务
			}
		}).start();
	}
}
```

