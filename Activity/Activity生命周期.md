# 【一】Activity生命周期

> Tips:
>
> ​    1、当新Activity启动时，旧的Activity必须执行完onPause，新的Activity才会执行onResume。
>
> ​    2、如果新的Activity采用了透明主题，那么当前Activity不会回调onStop.

* ### 经典生命周期图

![image](./images/Activity生命周期.png)

* ### 生命周期详解

  * **onCreate**

  该方法是在Activity被创建时回调，它是生命周期第一个调用的方法，我们在创建Activity时一般都需要重写该方法，然后在该方法中做一些初始化的操作，如通过setContentView设置界面布局的资源，初始化所需要的组件信息等。

  * **onStart**

  此方法被回调时表示Activity正在启动，此时Activity已处于可见状态，只是还没有在前台显示，因此无法与用户进行交互。可以简单理解为Activity已显示而我们无法看见摆了。

  * **onResume**

  当此方法回调时，则说明Activity已在前台可见，可与用户交互了（处于前面所说的Active/Running形态），onResume方法与onStart的相同点是两者都表示Activity可见，只不过onStart回调时Activity还是后台无法与用户交互，而onResume则已显示在前台，可与用户交互。当然从流程图，我们也可以看出当Activity停止后（onPause方法和onStop方法被调用），重新回到前台时也会调用onResume方法，因此我们也可以在onResume方法中初始化一些资源，比如重新初始化在onPause或者onStop方法中释放的资源。

  * **onPause**

  此方法被回调时则表示Activity正在停止（Paused形态），一般情况下onStop方法会紧接着被回调。但通过流程图我们还可以看到一种情况是onPause方法执行后直接执行了onResume方法，这属于比较极端的现象了，这可能是用户操作使当前Activity退居后台后又迅速地再回到到当前的Activity，此时onResume方法就会被回调。当然，在onPause方法中我们可以做一些数据存储或者动画停止或者资源回收的操作，但是不能太耗时，因为这可能会影响到新的Activity的显示——onPause方法执行完成后，新Activity的onResume方法才会被执行。

  * **onStop**

  一般在onPause方法执行完成直接执行，表示Activity即将停止或者完全被覆盖（Stopped形态），此时Activity不可见，仅在后台运行。同样地，在onStop方法可以做一些资源释放的操作（不能太耗时）。

  * **onRestart**

  表示Activity正在重新启动，当Activity由不可见变为可见状态时，该方法被回调。这种情况一般是用户打开了一个新的Activity时，当前的Activity就会被暂停（onPause和onStop被执行了），接着又回到当前Activity页面时，onRestart方法就会被回调。

  * **onDestroy**

  此时Activity正在被销毁，也是生命周期最后一个执行的方法，一般我们可以在此方法中做一些回收工作和最终的资源释放。

* ### 小结

当Activity启动时，依次会调用onCreate(),onStart(),onResume()，而当Activity退居后台时（不可见，点击Home或者被新的Activity完全覆盖），onPause()和onStop()会依次被调用。当Activity重新回到前台（从桌面回到原Activity或者被覆盖后又回到原Activity）时，onRestart()，onStart()，onResume()会依次被调用。当Activity退出销毁时（点击back键），onPause()，onStop()，onDestroy()会依次被调用，到此Activity的整个生命周期方法回调完成。