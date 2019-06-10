# 【五】Activity任务栈

* ### 默认任务栈

>   默认情况下，启动Activity的时候，系统会创建它的实例并把它们一一放入任务栈中，当我们单击back键，会发现这些Activity会一一回退。
>
> ​    (动作设置为“android.intent.action.MAIN”，类别设置为“android.intent.category.LAUNCHER”， 可以使这个ACT(activity)实例称为一个任务栈的入口)
>
> ​    任务栈是一种“后进先出”的栈结构，每按一下back键就会有一个Activity出栈，直到栈空为止，当栈中无任何Activity的时候，系统就会回收这个任务栈。
>
> ​    默认情况下，所有Activity所需的任务栈的名字为应用的包名。

* ### 所需任务栈

> 什么是Activity所需要的任务栈呢？这要从一个参数说起：
>
> ​    TaskAffinity，可以翻译为任务相关性。这个参数标识了一个Activity所需要的任务栈的名字，默认情况下，所有Activity所需的任务栈的名字为应用的包名。
>
> ​    当然，我们可以为每个Activity都单独指定TaskAffinity属性，这个属性值必须不能和包名相同，否则就相当于没有指定。
>
> ​    TaskAffinity属性主要和singleTask启动模式或者allowTaskReparenting属性配对使用，在其他情况下没有意义。
>
> **1、TaskAffiniity与singleTask配合使用：**
>
>    当TaskAffinity和singleTask启动模式配对使用的时候，它是具有该模式的Activity的目前任务栈的名字，待启动的Activity会运行在名字和TaskAffinity相同的任务栈中。
>
> 2、**TaskAffinity和allowTaskReparenting结合使用**
>
> ​    allowTaskReparenting属性，它的主要作用是activity的迁移，即从一个task迁移到另一个task，这个迁移跟activity的taskAffinity有关。当allowTaskReparenting的值为“true”时，则表示Activity能从启动的Task移动到有着affinity的Task（当这个Task进入到前台时），当allowTaskReparenting的值为“false”，表示它必须呆在启动时呆在的那个Task里。如果这个特性没有被设定，元素(当然也可以作用在每次activity元素上)上的allowTaskReparenting属性的值会应用到Activity上。默认值为“false”。这样说可能还比较难理解，我们举个例子，比如现在有两个应用A和B，A启动了B的一个ActivityC，然后按Home键回到桌面，再单击B应用时，如果此时，allowTaskReparenting的值为“true”，那么这个时候并不会启动B的主Activity，而是直接显示已被应用A启动的ActivityC，我们也可以认为ActivityC从A的任务栈转移到了B的任务栈中。
>
> ​    如果allowTaskReparenting值为false时，ActivityC并不会直接从A应用的任务栈迁移到B应用的任务栈，而是B应用直接重新创建了ActivityC的实例。到此我们对于allowTaskReparenting和taskAffinity属性的了解就已经相当深入了，不过有点需要说明的是allowTaskReparenting仅限于singleTop和standard模式，这是因为一个activity的affinity属性由它的taskAffinity属性定义（代表栈名），而一个task的affinity由它的root activity定义。所以，一个task的root activity总是拥有和它所在task相同的affinity。由于以singleTask和singleInstance启动的activity只能是一个task的root activity，因此allowTaskReparenting仅限于以standard 和singleTop启动的activity

* ### 前、后台任务栈

> 任务栈分为前台任务栈和后台任务栈，后台任务栈中的Activity位于暂停状态，用户可以通过切换将后台任务栈再次调到前台。