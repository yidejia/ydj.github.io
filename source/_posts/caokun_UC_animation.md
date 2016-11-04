---
title: 仿UC浏览器首页下拉动画及实现分析
date: 2016-11-04 16:12:55
tags:
- 曹堃
- 动画
categories:
- 移动端

---

# 动画效果

![图1](http://upload-images.jianshu.io/upload_images/2764502-f79c05d1173662a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

经常用UC看到首页有这么一个动画，就仿造写了一下。

# 实现分析
### 1.画曲线的动画
这个一眼看去就想到用贝塞尔曲线画，来看贝塞尔曲线方法，给出两个定点，和一个控制点就可以画。
```
CGContextAddQuadCurveToPoint(context, 控制点x, 控制点y, 目标点x, 目标点y);
```
于是按照下图，两个黄色的点是定点，绿色的是控制点，于是画出了这样的图。
![图2](http://upload-images.jianshu.io/upload_images/2764502-6c714da4d784ba32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看左边的图，中间有大片空白，看起来很浪费屏幕空间，用户体验不太好，于是想着怎么让贝塞尔曲线过某个定点，比如让曲线过绿色的定点，而不是把控制点设在绿色的位置。

重诉一下，现有的方法是给出两个定点和一个控制点，能画一条曲线。
现在是要，**已知两个定点，和过另外一个定点D，画一条曲线。**

我现在想让这条曲线过绿色的点，就像下图那样，求控制点坐标是多少？

![图3](http://upload-images.jianshu.io/upload_images/2764502-05b3633524b49863.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看下图，求出控制点坐标的过程
![图4](http://upload-images.jianshu.io/upload_images/2764502-33f2c618ab9c3e46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由上图就得出了控制点的坐标，然后就可以画出“图3”的样子了，实际中我觉得图3贴太紧了，也不美观，于是 yc 乘了个0.6的系数，即 yc = 0.6 * yc，就看起来比较顺眼了。

### 2.页面结构

页面结构大概是这样，底下的 tableView 铺满整个 view，然后蓝色的headerView 加在 tableView 的上面，不是加 tableView.tableHeaderView 上面哦，至于为什么你加加看就知道了。会跟着 tableview 动

![图5](http://upload-images.jianshu.io/upload_images/2764502-dd0d035e4b25f762.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.不规则事件点击，事件穿透
headerView 上有一个头像，是可以点击的，其他地方的点击事件要传给底下的 tableView 也叫事件穿透，通过修改 hitTest 可以实现
```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;
```
hitTest 主要用来做事件分发的，可以实现不规则点击，它在整个 view 结构上是递归的，深度优先的，今天不讲算法，因为 hitTest 太厉害，不规则点击用 pointInside 函数就够了。
```
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;
```
这个函数会被 hitTest 调用，返回 false 表示点击的不是自己，返回 true 表示点击的是自己。
那么，我只要判断点击的 point 在不在头像的那个圆圈里面就可以了，就是判断点在不在圆内，高中讲过了，point 到圆心的距离小于半径就表示在了，那么返回 true 就行。具体的还是看代码吧。
如果有多个控件，需要自己确定每个控件的点击区域。

最后还是上个代码 [下载地址](https://github.com/hehe520/PullAnimation)，下载慢慢看吧。

### 4.后期改进

写完这个博客突然想到一个还要改的地方，就是当用户手指松开的时候，scrollViewWillEndDragging，这个方法内判断一下，contentOffset.y 值，如果超过多少值，那么自动回调一个 block，可实现下拉刷新。

