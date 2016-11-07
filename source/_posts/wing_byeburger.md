---
title: 一句代码实现标题栏、导航栏滑动隐藏，ByeBurger库的使用和实现
date: 2016-11-07 11:13:23
tags:
- wing
- ByeBurger
categories:
- 移动端

---
ByeBuger是一个能够让你快速实现标题栏以及底部导航栏在滑动时隐藏的库。本文将介绍ByeBuger的使用和实现。

现在，ByeBuger可以轻易地将**任何view**在滑动的时候隐藏或者显示。同时支持头部(标题栏)和底部(导航栏)效果。

[ByeBurger项目地址](https://github.com/githubwing/ByeBurger)

先看一下全新的效果：

![](http://ac-mhke0kuv.clouddn.com/56781ae4da8c044aceec.gif)

![](http://ac-mhke0kuv.clouddn.com/fff0d5a7d0a1b1dfcbb1.gif)


还不错吧。然而，实现这么炫酷的效果，**仅仅需要一句代码！**

# 使用

1.在gradle 编译库文件

```gralde

allprojects {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
    }
}


dependencies {
   compile 'com.github.githubwing:ByeBurger:1.1.0'
  compile 'com.android.support:design:25.0.0'
  }
```
2.**你仅仅需要一句代码，对，没错，只需要在你的View上加入一句。**
先使用CoordinatorLayout作为根布局,然后向你的**任何**View中插入一句app:layout_behavior属性，即可实现滑动的隐藏和显示。
你的标题栏可以是Toolbar或者LinearLayout或者什么鬼。
同样你的底部导航栏可以是最新的BottomNavigationView亦或者TabLayout在古老一点的RadioButton都可以！

```xml
<android.support.design.widget.CoordinatorLayout>
 
  <Viewpager />
   <Toolbar
  	app:layout_behavior="@string/bye_burger_title_behavior"
  />
  <BottomTab 
   android:layout_gravity="bottom"
   app:layout_behavior="@string/bye_burger_bottom_behavior"
  />      
</android.support.design.widget.CoordinatorLayout>

```

具体的一句话是:
```
 对于标题栏
 app:layout_behavior="@string/bye_burger_title_behavior"
```

```
 对于底部导航栏
 app:layout_behavior="@string/bye_burger_bottom_behavior"
```
之后你就可以让你的app享受更多的阅读空间啦，相信这给用户带来了极大的便利。

# 注意
CoordinatorLayout类似于FrameLayout，所以注意xml层次，TitleBar和BottomTab要在xml下方。
只有实现NestScorll接口View的才可以实现监听，例如RecyclerView、NestScrollView.
在ListView下，是不生效的。


[ByeBurger项目地址](https://github.com/githubwing/ByeBurger)

以上就是ByeBurger的使用方式，接下来将会介绍ByeBurger的实现方式。如果你只是使用或者不感兴趣就不用往下看啦~~不过我还是建议你看一看，因为知道一个轮子的原理，百利无一害。


# 实现

### 0.改名原因
在ByeBurger 1.0版本的时候，其实ByeBurger不叫ByeBurger的，而叫做ByeBurgerNavigationView, 由名字可以看出，他是扩展了系统的BottomNavigationView，可是这样做有许多弊端，比如强制用户使用了某个控件，这样通用性不强。第二，为了用户有更多的选择，加入了头部隐藏效果。所以说，这个项目已经不能被称为NavigationView。于是我将名称进行了修改。

### 1.历史实现
在1.0版本，我继承了BottomNavigationView。提供了两个方法，一个是show,另一个是hide.
```java
 public void show() {

    setY(mStartY);
    TranslateAnimation ta = new TranslateAnimation(0f, 0f,getMeasuredHeight(),0);
    ta.setDuration(300);
    ta.setAnimationListener(this);
    startAnimation(ta);

  }
public void hide() {
    setY(mStartY + getMeasuredHeight());

    TranslateAnimation ta = new TranslateAnimation(0f, 0f, -getMeasuredHeight(), getMeasuredHeight());
    ta.setDuration(300);
    ta.setAnimationListener(this);
    startAnimation(ta);
  }
```

实际上就是对Y坐标进行了一些处理。给个动画再让他改变他的Y坐标。来达到隐藏效果。(总之隐藏就是让用户看不到)

然后利用Behavior，对NestScroll相关滑动进行监听，来改变ByeBurgerNavigationView的状态。
```java
public class ByeBurgerBehavior extends CoordinatorLayout.Behavior<ByeBurgerNavigationView> {
```

当然，这是1.0的实现，实用性比较低。所以有了1.1.0版本。

### 2.更新实现

整体思路就是利用自定义behavior去监听nestScroll的滑动，来让对应的View改变。之前在[CoordinatorLayout 自定义Behavior并不难，由简到难手把手带你撸三款！](http://androidwing.net/index.php/70) 中介绍过一种Behavior的使用方式，不熟悉的可以先过去看看。这里用到的是第二种，主要是针对NestScroll进行监听。

首先，要自定义一个Behavior，他的泛型，也就是child view为View。 这也保证了它的通用性。
```java
public class ByeBurgerBottomBehavior extends CoordinatorLayout.Behavior<View> {}
```

其次来处理一下onStartNestedScroll()方法，他提供一个返回值，用于过滤掉滑动事件。这里我们只关心上下滑动。所以应该这样。
```java
@Override public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child,
      View directTargetChild, View target, int nestedScrollAxes) {

    return (nestedScrollAxes & ViewCompat.SCROLL_AXIS_VERTICAL) != 0;
  }
```

最后，在onNestedPreScroll()进行child的对应操作。首先要根据参数dy来判断上下滑动,然后再根据view当前的状态来显示或者隐藏目标View。


```java
public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, View child, View target,
      int dx, int dy, int[] consumed) {
	//初始化一些参数
    if (isFirstMove) {
      isFirstMove = false;
      mAnimateHelper = AnimateHelper.get(child);
      mAnimateHelper.setStartY(child.getY());
      mAnimateHelper.setMode(AnimateHelper.MODE_BOTTOM);
    }
    if (Math.abs(dy) > mTouchSlop) {
      if (dy < 0) {

        if (mAnimateHelper.getState() == AnimateHelper.STATE_HIDE) {
          mAnimateHelper.show();
        }
      } else if (dy > 0) {
        if (mAnimateHelper.getState() == AnimateHelper.STATE_SHOW) {
          mAnimateHelper.hide();
        }
      }
    }
  }
}
```

在上面函数里，view的隐藏和显示委托给了AnimateHelper类。这是一个Helper，用来管理View的状态，也就是托付View隐藏或者显示。他的代码很简单，如下：

```java
public class AnimateHelper {
  public View mTarget;
  public float mStartY;
  public static int STATE_SHOW = 1;
  public static int STATE_HIDE = 0;
  public int mCurrentState = STATE_SHOW;
  public int mMode = MODE_TITLE;
  public static int MODE_TITLE = 233;
  public static int MODE_BOTTOM = 2333;
}
```
提供一些成员，用于保存 状态，模式，起始Y坐标等等。

提供一个工厂方法，用于获得处理目标view的helper:
```java
 public static AnimateHelper get(View target) {
    return new AnimateHelper(target);
  }
```

之后，提供show方法和hide方法，用于执行view的隐藏或者显示：
```java
public void show() {
    if (mMode == MODE_TITLE) {
      showTitle();
    } else if (mMode == MODE_BOTTOM) {
      showBottom();
    }
  }

 public void hide() {
    if (mMode == MODE_TITLE) {
      hideTitle();
    } else if (mMode == MODE_BOTTOM) {
      hideBottom();
    }
  }
```

而 showTitle()和 showBottom()之类的如1.0版一样，是改变了一下y坐标，然后执行动画。

## 如何让用户使用

这里参考了系统预留behavior的使用方式，由于Behavior实例是系统反射出来的，所以需要完整的包名。于是我就将Beavior的包名写在了string.xml里，如下：
```xml
  <string name="bye_burger_bottom_behavior">com.wingsofts.byeburgernavigationview.ByeBurgerBottomBehavior</string>
  <string name="bye_burger_title_behavior">com.wingsofts.byeburgernavigationview.ByeBurgerTitleBehavior</string>
```

所以用户使用起来及其简便，只需要在xml中给view加入一行代码app:layoutbehavior="@string/byeburgerbottom_behavior"即可。

以上，ByeBurger库的使用和实现就写完了。

如果你觉得还不错 欢迎star, 更欢迎贡献代码。

本库下载地址：https://github.com/githubwing/ByeBurger
