# Animation

Android的动画主要包括三大类：逐帧动画Frame、补间动画Tween、属性动画，其中属性动画功能非常强大，也是最常用的动画方法。

```java
//帧动画，需资源文件frame_animation.xml
ImageView img = (ImageView)findViewById(R.id.wheel_image);
img.setBackgroundResource(R.drawable.frame_animation);
AnimationDrawable frameAnimation = (AnimationDrawable) img.getBackground();
frameAnimation.start();

//补间动画，需资源文件tween_animation.xml
ImageView img = (ImageView)findViewById(R.id.wheel_image);
Animation tweenAnimation = AnimationUtils.loadAnimation(this, R.anim.tween_animation);
img.startAnimation(tweenAnimation);

// 属性动画，不需资源文件
ObjectAnimator anim = ObjectAnimator.ofFloat(targetObject, "alpha", 0f, 1f);
anim.setDuration(1000);
anim.start();
```

