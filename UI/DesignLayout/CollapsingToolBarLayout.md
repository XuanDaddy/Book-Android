# CollapsingToolBarLayout

### 概述

> CollapsingToolbarLayout 出现的目的只是为了增强 Toolbar，以实现具有“折叠效果“”的顶部栏。它需要是 AppBarLayout 的直接子 View，这样才能发挥出效果。

CollapsingToolbarLayout包含以下特性：

- **Collapsing Title 可折叠的标题**
- **Content Scrim 内容纱布**
- **Status bar scrim 状态栏纱布**
- **Parallax scrolling children 子 View 的视差滚动行为**
- **Pinned position children 子类的位置固定行为**

CollapsingToolbarLayout和ScrollView一起使用会有滑动bug，注意要使用NestedScrollView来替代ScrollView。

###Collapsing title 可折叠的标题

> * Collapsing title 的概念其实就是 Title 在大小与位置会变化。
>
> * CollapsingToolbarLayout 与 Toolbar 都定义了 title，CollapsingToolbarLayout 中的 title 会覆盖Toolbar 中的 title。需要注意的是 Collapsing title 有两种状态，分别是 **展开(Expanded)** 和 **折叠(Collapsed)**。

###Content scrim 内容纱布

> * 指定 Contetn scrim 后，CollapsingToolbarLayout 会在折叠状态显示指定的颜色或者是图片，它就像是一块纱布一样遮住 title 下面的内容，所以被称为内容纱布。
> * 如果一个 CollapsingToolbarLayout 中只有 Toolbar 的话，那么它就不起作用。CollapsingToolbarLayout 本质上是一个 FrameLayout，所以需要在 Toolbar 的前面位置加入其它的 View 作为内容，Content scrim 才会起作用。

###Status bar scrim 状态栏纱布

> 这个与 Content scrim 作用类似，不过作用的对象却是在系统的 Statusbar 的位置。
>
> 它还有一个特别的地方就是，它只作用在 SDK 5.0 版本之后，并且需要正确配置。
>
> 所谓的正确配置其实是说它需要一个 system inset 的矩形，什么意思呢？说的是包裹 CollapsingToolbarLayout 的 AppbarLayout 需要设置 fitsSystemWindows 为 true。

### Parallax scrolling children 子 View 的视差滚动行为

CollapsingToolbarLayout 可以控制的子 View 滚动模式有 3 种：

- none 默认，无任何效果
- Parallax 视差滚动
- pin 固定某个 View

需要注意的是，这个属性作用对象是 CollapsingToolbarLayout 中的子 View 并不是 CollapsingToolbarLayout。

如何理解视差？就是滚动的速度不同，造成的视觉差异效果。也就是说 CollapsingToolbarLayout 中有的 view 滚动的快一些，其它的滚动的慢一些。 它滚动的快慢受 Parallax multiplier 这个因子的影响，默认值为 DEFAULT_PARALLAX_MULTIPLIER。也就是 0.5f。也就是正常速度的一半。

### Pinned position children 子类的位置固定行为

这个很好理解，将 CollapsingToolbarLayout 中某个子 View 固定，无论是否存在滚动事件，只要设置 app:layout_collapseMode=”pin”。

```xml
<android.support.design.widget.CollapsingToolbarLayout
android:layout_width="match_parent"
android:layout_height="match_parent"
app:title="collapsing title"
android:minHeight="100dp"
app:layout_scrollFlags="scroll|exitUntilCollapsed|snap">
<ImageView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:src="@drawable/wowan"
    android:scaleType="centerCrop"/>
<android.support.v7.widget.Toolbar
    android:id="@+id/toolbar"
    app:title="title"
    app:layout_collapseMode="pin"
    android:layout_width="match_parent"
    android:layout_height="120dp"
    android:background="#a0ffff00"
     />
</android.support.design.widget.CollapsingToolbarLayout>
```

可以看到，不管怎么滚动，Toolbar 固定不动。

到这里为止，利用 AppBarLayout 就能实现可折叠的 Toolbar 了。

### OnOffsetChangedListener

这是 AppBarLayout 定义的监听器。

```java
void onOffsetChanged (AppBarLayout appBarLayout, int verticalOffset)
```

verticalOffset 是 AppBarLayout 相对于完全展开时没有滑动的距离。它在初始位置为 0，其它时候都为负数。它绝对值的最大值为 AppBarLayout 的 TotalScollRange。

我们可以根据实际的业务需求，通过监听 AppBarLayout 的位移做一些相应的处理。

```xml
<android.support.design.widget.AppBarLayout
    android:id="@+id/app_bar"
    android:layout_width="match_parent"
    android:layout_height="@dimen/app_bar_height"
    android:fitsSystemWindows="true"
    android:theme="@style/AppTheme.AppBarOverlay">

    <android.support.design.widget.CollapsingToolbarLayout
        android:id="@+id/toolbar_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true"
        app:expandedTitleMarginStart="64dp"
        app:contentScrim="?attr/colorPrimary"
        app:layout_scrollFlags="scroll|exitUntilCollapsed">
        <ImageView
            android:id="@+id/circle_icon"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:layout_margin="@dimen/fab_margin"
            android:layout_gravity="bottom"
            app:srcCompat="@android:drawable/ic_dialog_info" />
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:layout_collapseMode="pin"
            app:popupTheme="@style/AppTheme.PopupOverlay" />

    </android.support.design.widget.CollapsingToolbarLayout>
</android.support.design.widget.AppBarLayout>
```

```java
final ImageView circleIcon = (ImageView) findViewById(R.id.circle_icon);

AppBarLayout appBarLayout = (AppBarLayout) findViewById(R.id.app_bar);
appBarLayout.addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {
    @Override
    public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
        int newalpha = 255 + verticalOffset;
        newalpha = newalpha < 0 ? 0 : newalpha;
        circleIcon.setAlpha(newalpha);
    }
});
```

在布局文件中添加一个图标，然后监听 AppBarLayout 的滑动来改变自身的透明度。