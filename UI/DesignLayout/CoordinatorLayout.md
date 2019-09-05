# CoordinatorLayout

### 概述

> CoordinatorLayout 是一个 “加强版” FrameLayout,它主要有两个用途：
>
> 1. 用作应用的顶层布局管理器，也就是作为用户界面中所有 UI 控件的容器;
> 2. 用作相互之间具有特定交互行为的 UI 控件的容器，通过为 `CoordinatorLayout`的子 View 指定 Behavior， 就可以实现它们之间的交互行为。 Behavior 可以用来实现一系列的交互行为和布局变化，比如说侧滑菜单、可滑动删除的 UI 元素，以及跟随着其他 UI 控件移动的按钮等。

总结：`CoordinatorLayout` 是一个布局管理器，相当于一个增强版的 `FrameLayout`，但是它**神奇在于可以实现它的子 View 之间的交互行为**。

### 示例

我们的 Button 就是一个被观察者，TextView 作为一个观察者，当 Button 移动的时候通知 TextView， TextView 就跟着移动。

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <TextView android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="观察者"
              app:layout_behavior=".coordinator.FollowBehavior"/>
    <Button
            android:id="@+id/btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="被观察者"/>

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

很简单，一个 TextView， 一个 Button， 外层用 CoordinatorLayout， 然后给我们的 TextView 加一个自定义的 Behavior 文件。

#### Behavior

> 自定义 CoordinatorLayout 的 Behavior， 泛型为观察者 View；
>
> 需要存在构造方法；
>
> 必須须重写两个方法，layoutDependOn和onDependentViewChanged。

```java
public class FollowBehavior extends CoordinatorLayout.Behavior<TextView> {

    /**
     * 构造方法
     */
    public FollowBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
  
  /**
 * 判断child的布局是否依赖 dependency
 *
 * 根据逻辑来判断返回值，返回 false 表示不依赖，返回 true 表示依赖
 *
 * 在一个交互行为中，Dependent View 的变化决定了另一个相关 View 的行为。
 * 在这个例子中， Button 就是 Dependent View，因为 TextView 跟随着它。
 * 实际上 Dependent View 就相当于我们前面介绍的被观察者
 *
 */
    @Override
    public boolean layoutDependsOn(@NonNull CoordinatorLayout parent,
                                   @NonNull TextView child,
                                   @NonNull View dependency) {

        Log.d("FollowBehavior", "layoutDependsOn called...");
        return dependency instanceof Button;
    }

    @Override
    public boolean onDependentViewChanged(@NonNull CoordinatorLayout parent,
                                          @NonNull TextView child,
                                          @NonNull View dependency) {

        Log.d("FollowBehavior", "onDependentViewChanged called...");

        child.setX(dependency.getX() - 100);
        child.setY(dependency.getY() + 300);
        return true;
    }
}
```

在一个交互行为中，Dependent View 的变化决定了另一个相关 View 的行为。

- 拦截 Touch 事件

当我们为一个 `CoordinatorLayout`的直接子 View 设置了 Behavior 时，这个 Behavior 就能拦截发生在这个 View 上的 Touch 事件，那么它是如何做到的呢？实际上， `CoordinatorLayout`重写了 `onInterceptTouchEvent()`方法，并在其中给 Behavior 开了个后门，让它能够先于 View 本身处理 Touch 事件。具体来说， `CoordinatorLayout`的 `onInterceptTouchEvent()`方法中会遍历所有直接子 View ，对于绑定了 Behavior 的直接子 View 调用 Behavior 的 onInterceptTouchEvent() 方法，若这个方法返回 true， 那么后续本该由相应子 View 处理的 Touch 事件都会交由 Behavior 处理，而 View 本身表示懵逼，完全不知道发生了什么。

- 拦截测量及布局

了解了 Behavior 是怎养拦截 Touch 事件的，想必大家已经猜出来了它拦截测量及布局事件的方式 —— CoordinatorLayout 重写了测量及布局相关的方法并为 Behavior 开了个后门。没错，真相就是如此。
CoordinatorLayout 在 `onMeasure()`方法中，会遍历所有直接子 View ，若该子 View 绑定了一个 Behavior ，就会调用相应 Behavior 的 `onMeasureChild()`方法，若此方法返回 true，那么 CoordinatorLayout 对该子 View 的测量就不会进行。这样一来， Behavior 就成功接管了对 View 的测量。
同样，CoordinatorLayout 在 `onLayout()`方法中也做了与 `onMeasure()`方法中相似的事，让 Behavior 能够接管对相关子 View 的布局。

- View 的依赖关系的确定
  现在，我们在探究一下交互行为中的两个 View 之间的依赖关系是怎么确定的。我们称 child 为交互行为中根据另一个 View 的变化做出响应的那个个体，而 Dependent View 为child所依赖的 View。实际上，确立 child 和 Dependent View 的依赖关系有两种方式：

1. 显式依赖：为 child 绑定一个 Behavior，并在 Behavior 类的 `layoutDependsOn()`方法中做手脚。即当传入的 `dependency`为 Dependent View 时返回 true，这样就建立了 child 和 Dependent View 之间的依赖关系。
2. 隐式依赖：通过我们最开始提到的锚（anchor）来确立。具体做法可以这样：在 XML 布局文件中，把 child 的 `layout_anchor`属性设为 Dependent View 的id，然后 child 的 `layout_anchorGravity`属性用来描述为它想对 Dependent View 的变化做出什么样的响应。关于这个我们会在后续篇章给出具体示例

### 原理

> 实际上， `CoordinatorLayout`本身并没有做过多工作，实现交互行为的主要幕后推手是 `CoordinatorLayout`的内部类—— `Behavior`。通过为 `CoordinatorLayout`的**直接子 View **绑定一个 `Behavior`，这个 `Behavior`就会拦截发生在这个 View 上的 Touch 事件、嵌套滚动等。不仅如此，`Behavior`还能拦截对与它绑定的 View 的测量及布局。