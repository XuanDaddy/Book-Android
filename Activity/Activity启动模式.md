# 【四】Activity启动模式

> Tips:
>
> ​    启动模式简单地说就是Activity启动时的策略，在AndroidManifest.xml中的标签的android:launchMode属性设置；
>
> ​    启动模式有4种，分别为standard、singleTop、singleTask、singleInstance；
>
> ​    **任务栈**：每个应用都有一个任务栈，是用来存放Activity的，功能类似于函数调用的栈，先后顺序代表了Activity的出现顺序。

一．**Standard（标准模式）**

> 在这种模式下，谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈中。
>
> 比如Activity A启动了Activity B（B是标准模式），那么B就会进入到A所在栈中。
>
> 当我们用ApplicationContext去启动standard模式的Activity会报错，这是因为standard模式的Activity默认会进入启动它的Activity所属的任务栈中，但是由于非Activity类型的Context（如ApplicationContext）并没有所谓的任务栈，所以这就有问题了。解决这个问题的方法是为待启动Activity指定FLAG_ACTIVITY_NEW_TASK标记位，这样启动的时候就会为它创建一个新的任务栈，这个时候待启动Activity实际上是以singleTask模式启动的。

二．**SingleTop（**栈顶利用模式**）**

> 在这种模式下，如果新的Activity已经位于任务栈的栈顶，那些此Activity不会被重新创建，同时它的onNewIntent方法会被回调，通过此方法的参数我们可以取出当前请求的信息。需要注意的是，这个Activity的onCreate、onStart不会被系统调用，因为它并没有发生改变。如果新Activity的实例已存在但不是位于栈顶，那么新的Activity仍然会重新创建。

三．**SingleTask（**栈内复用模式**）**

> 在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例，系统会调用onNewIntent方法。
>
> 具体一点，当一个具有singleTask模式的Activity请求启动后，比如Activity A，系统首先会寻找是否存在A想要的任务栈，如果不存在，就重新创建一个任务栈，然后创建A的实例后把A放到栈中。如果存在A所需的任务栈，这时要看A是否在栈中有实例存在，如果有实例存在，那么系统就会把A调到栈顶并调用它onNewIntent方法，如果实例不存在，就创建A的实例并把A压入栈中。
>
> 由于singleTask默认具有clearTop的效果，会导致栈内所有在上面的Activity全部出栈。

四．**SingleInstance（**单实例模式**）**

> 这是一种加强的singleTask模式，它除了具有singleTask模式的所有特性外，还加强了一点，那就是具有此种模式的Activity只能单独地位于一个任务栈，换句话说，比如Activity A是singleInstance模式，当A启动后，系统会为它创建一个新的任务栈，然后A独自在这个新任务栈中，由于栈内复用的特性，后续的请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁了。

五．**如何为Activity指定启动模式**

> 通过AndroidManifest为Activity指定启动模式：android:launchMode="singleTask”;
>
> 通过Intent设置标志位来为Activity指定启动模式：intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
>
> 以上两种的区别：
>
> ​    首先，优先级上，第二种方式的优先级要高于第一种，当两种同时存在时，以第二种试为准；
>
> ​    其次，上述两种方式在限定范围上有所不同，比如，第一种方式无法直接为Activity设定FLAG_ACTIVITY_CLEAR_TOP标识，而第二种方式无法为Activity指定singleInstance模式

六．**Intent Flags**

> Activity的Flags有很多，标记位的作用也很多，有的标记位可以设定Activity的启动模式，有的可以影响Activity的运行状态，比较常用的几个如下：
>
> **FLAG_ACTIVITY_NEW_TASK**：这个标记位的作用是为Activity指定"singleTask"启动模式，其效果和在XML中指定该启动模式相同。
>
> **FLAG_ACTIVITY_SINGLE_TOP**：这个标记位的作用是为Activity指定"singleTop"启动模式，其效果和在XML中指定该启动模式相同。
>
> **FLAG_ACTIVITY_CLEAR_TOP**：具有此标记位的Activity，当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。这个标记位一般会和singleTask启动模式一起出现，在这种情况下，被启动Activity的实例如果已经存在，那么系统就会调用它的onNewIntent。如果被启动的Activity采用standard模式启动，那么它连同之上的Activity都要出栈，系统会创建新的Activity实例并放入栈顶。
>
> **FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS**：具有这个标记的Activity不会出现在历史Activity的列表中，当某些情况下我们不希望用户通过历史列表回到我们的Activity的时候这个标记比较有用。它等同于在XML中指定Activity的属性android:excludeFromRecents="true"。