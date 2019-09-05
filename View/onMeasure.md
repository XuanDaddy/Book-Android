# 【一】onMeasure

### onMeasure调用时机

* 直接创建View（new）的时候不会调用onMeasure；
* 只有将这个View放入一个容器（父控件）中的时候才需要测量（xml中或动态addView时触发）；
* onMeasure的调用是由父控件唤起的，当父控件需要放置该控件时，父控件会调用子控件的onMeasure方法，然后会传入两个参数（widthMeasureSpec和heightMeasureSpec），这两个参数就是父控件告诉子控件可获得的空间以及关于这个空间的约束条件；

### onMeasure执行流程

测量的时候父控件的onMeasure方法会遍历他所有的子控件，挨个调用子控件的measure方法，measure方法会调用onMeasure，然后会调用setMeasureDimension方法保存测量的大小，一次遍历下来，第一个子控件以及这个子控件中的所有子控件都会完成测量工作；然后开始测量第二个子控件…；最后父控件所有的子控件都完成测量以后会调用setMeasureDimension方法保存自己的测量大小。值得注意的是，这个过程不只执行一次，也就是说有可能重复执行，因为有的时候，一轮测量下来，父控件发现某一个子控件的尺寸不符合要求，就会重新测量一遍。

**1. ViewRootImpl.performTraversals()->performMeasure():**

这里面会调getRootMeasureSpec（）根据手机屏幕的宽高和DecorView的LayoutParams生成DecorView的MeasureSpec,然后调用DecorView的measure()开始DecorView的测量

**2.DecorView.measure()->onMeasure():**

DecorView继承自FrameLayout，所以会走到FrameLayout的onMeasure(),onMeasure()里调measureChild()来根据上面说的规则为ViewGroupA生成MeasureSpec，并通过ViewGroupA.measure（）开始ViewGroupA的测量

**3.ViewGroupA.measure()->onMeasure():**

这是我们自定义的一个ViewGroup(继承自ViewGroup) 假如我们没有重写onMeasure()的话，则默认调的是View.onMeasure()，则不会发起对子View的measure,它里面的子View也就不会被测量(0),而这个ViewGroup如果没有设置具体宽高的话，（wrap_content）则ViewGroup展示的就是父容器的宽高（根据上面说的MeasureSpec生成规则)。

### MeasureSpec

> MeasureSpec约束是由父控件传递给子控件的。
>
> MeasureSpec中包含两部分，一个是SpecMode(测量模式)，一个是SpecSize(某种测量模式下的规格大小)。它是用一个32位的int值来表示的，高2位代码SpecMode,低30位代表SpecSize。

* **specMode主要分为三种：**

| 模式        | 说明                                                         |
| :---------- | :----------------------------------------------------------- |
| EXACTLY     | 设置了精确的宽高。如width、height设置了具体值或者设置为 match_parent，都属于这种模式 |
| AT_MOST     | width、height设置为wrap_content则属于这种模式                |
| UNSPECIFIED | 以上两种模式是我们布局里常见的，最大也不会大过父布局，而这种模式一般用于系统， 父容器不对View有任何限制 |

* **MeasureSpec如何生成**

在不重写onMeasure()的情况下，一个View/ViewGroup的MeasureSpec就决定了这个View/ViewGroup的宽高，显然这个MeasureSpec是这个View/ViewGroup的父容器在调用子View的measure()方法时传进来的，也就是说一个View/ViewGroup的MeasureSpec是由其父容器生成的，那么是怎么生成的呢？里面的SpecSize和SpecMode是由什么决定的呢？

​    这里由于代码比较多就不贴了，父容器通过调ViewGroup中的getChildMeasureSpec()来生成子View的MeasureSpec。getChildMeasureSpec()中主要是通过父容器的MeasureSpec以及子Views设置的宽高来共同决定子View的MeasureSpec中的SpecMode和SpecSize。

* **getChildMeasureSpec()代码里的生成规则：**

  | 父控件的约束规则                              | 子控件的宽高属性                 | **子控件的约束规则** | **说明**                                                     |
  | --------------------------------------------- | -------------------------------- | -------------------- | ------------------------------------------------------------ |
  | EXACTLY（父控件是填充父窗体，或者具体size值） | 具体的size（20dip）/MATCH_PARENT | EXACTLY              | 子控件如果是具体值，约束尺寸就是这个值，模式为确定的；子控件为填充父窗体，约束尺寸是父控件剩余大小，模式为确定的。 |
  |                                               | WRAP-CONTENT                     | AT_MOST              | 子控件如果是包裹内容，约束尺寸值为父控件剩余大小 ，模式为至多 |
  | AT_MOST（父控件是包裹内容）                   | 具体的size（20dip）              | EXACTLY              | 子控件如果是具体值，约束尺寸就是这个值，模式为确定的；       |
  |                                               | MATCH_PARENT/WRAP_CONTENT        | AT_MOST              | 子控件为填充父窗体或者包裹内容 ，约束尺寸是父控件剩余大小 ，模式为至多 |
  | UNSPECIFIED（父控件未指定）                   | 具体的size（20dip）              | EXACTLY              | 子控件如果是具体值，约束尺寸就是这个值，模式为确定的；       |
  |                                               | MATCH_PARENT/WRAP_CONTENT        | UNSPECIFIED          | 子控件为填充父窗体或者包裹内容 ，约束尺寸0，模式为未指定View的onMeasure |

### View的onMeasure

```java
protected void onMeasure( int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension( getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
/**
 * 为宽度获取一个建议最小值
 */
protected int getSuggestedMinimumWidth () {
    return (mBackground == null) ? mMinWidth : max(mMinWidth , mBackground.getMinimumWidth());
}
/**
 * 获取默认的宽高值
 */
public static int getDefaultSize (int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec. getMode(measureSpec);
    int specSize = MeasureSpec. getSize(measureSpec);
    switch (specMode) {
    case MeasureSpec. UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec. AT_MOST:
    case MeasureSpec. EXACTLY:
        result = specSize;
        break;
    }
    return result;
}
```

从源码我们了解到：

如果View的宽高模式为未指定，他的宽高将设置为android:minWidth/Height =""值与背景宽高值中较大的一个；
如果View的宽高 模式为 EXACTLY （具体的size ），最终宽高就是这个size值；
如果View的宽高模式为EXACTLY （填充父控件 ），最终宽高将为填充父控件；
如果View的宽高模式为AT_MOST （包裹内容），最终宽高也是填充父控件。

**也就是说如果我们的自定义控件在布局文件中，只需要设置指定的具体宽高，或者MATCH_PARENT 的情况，我们可以不用重写onMeasure方法。但如果自定义控件需要设置包裹内容WRAP_CONTENT ，我们需要重写onMeasure方法，为控件设置需要的尺寸；默认情况下WRAP_CONTENT 的处理也将填充整个父控件。**
