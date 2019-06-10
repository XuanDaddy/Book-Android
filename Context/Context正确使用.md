# 【六】Context正确使用

**一般Context造成的内存泄漏，几乎都是当Context销毁的时候，却因为被引用导致销毁失败，而Application的Context对象可以理解为随着进程存在的，所以我们总结出使用Context的正确姿势：**

* 当Application的Context能搞定的情况下，并且生命周期长的对象，优先使用Application的Context。

* 不要让生命周期长于Activity的对象持有到Activity的引用。

* 尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用，如果使用静态内部类，将外部实例引用作为弱引用持有。