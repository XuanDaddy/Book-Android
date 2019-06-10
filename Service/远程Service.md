# 【五】远程Service

#### Tips

> Service其实是运行在主线程里的，如果直接在Service中处理一些耗时的逻辑，就会导致程序ANR。 
>
>  所以，应该在Service中开启线程去执行耗时任务，这样就可以有效地避免ANR的出现。 

#### 什么是远程Service(Remote Service)

> Service运行于主线程，如果直接睡眠60秒，会出现ANR。
>
> 如果把Service作为远程线程启动，不会出现ANR,这是由于，使用了远程Service后，Service已经在另外一个进程当中运行了，所以只会阻塞该进程中的主线程，并不会影响到当前的应用程序。

可以通过查看进程ID，进一步确认是否为同一进程:

```java
Log.d("TAG", "process id is " + Process.myPid());
```

#### 如何创建远程Service

> 将一个普通的Service转换成远程Service其实非常简单，只需要在注册Service的时候将它的android:process属性指定成:remote就可以了，代码如下所示：

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
	</service>
 
</manifest>
```

#### 启动远程Service

> 使用startService直接启动远程Service,不再有ANR。(与Activity无联系)
>
> 如果使用bindService启动远程Service，**程序崩溃了**。这是由于在bindService时，我们会让Activity和Service建立关联，但是目前Service已经是一个远程Service了，Activity和Service运行在两个不同的进程当中，这时就不能再使用传统的建立关联的方式，程序也就崩溃了。

#### 注意

> 远程Service非但不好用，甚至可以称得上是较为难用。一般情况下如果可以不使用远程Service，就尽量不要使用它。