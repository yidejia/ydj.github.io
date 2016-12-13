---
title: iOS 图像处理 - 一种裁剪图片圆角的算法
date: 2016-11-04 16:12:55
tags:
- 曹堃
- 动画
- iOS
categories:
- 移动端

---


# 场景
经常看到各种高效裁剪圆角的文章，正好之前做过一点数字图像处理，就打算用空域处理的办法，写个裁剪圆角的算法，一定要尽可能的快的，不然界面容易卡顿。

裁圆角很简单，对于图像上的一个点(x, y)，判断其在不在圆角矩形内，在的话 alpha 是原值，不在的话 alpha 设为 0 即可。如下图

![15F6A143-2704-402D-88EA-DB80B0266F80.png](http://upload-images.jianshu.io/upload_images/2764502-dd591dcaee665459.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我遍历所有像素，判断每个像素在不在4个圆的圆内就行了，4个角，每个角有一个四分之一的圆。

**一个优化**就是，我不需要遍历全部的像素就能裁出圆角，只需要考虑类似左下角三角形的区域就行了，左下，左上，右上，右下，一共4个三角形区域（另外3个图中没画出），for循环的时候，就循环这个4个三角形区域就行了。

所以对于一幅 w * h 的图像，设圆角大小为 n，n < min(w, h) / 2，其复杂度为 O(n) = 2(n^2)，最坏的情况计算量也不会超过 wh / 2

对于一个像素点(x, y)，判断其在不在圆内的公式
```
如果  (x-cx)^2 + (y-cy)^2 <= r^2  就表示点 (x, y) 在圆内，反之不在。
```

理论说完了，下面看实际的测试数据。

# 测试结果与分析

根据上面的分析，我写了一个裁剪圆角的程序，叫为 **my裁剪**。
还用了苹果 CoreGraphics 库的 CGContext 裁剪圆角，叫为 **CG 裁剪**。
还用了 UIKit 的 UIBezierPath 裁剪圆角，叫为 **贝塞尔裁剪**。

下面来对比三种方法，哪种最快。

---

实验数据：
一张png格式 512 * 512 的 lena 女神的标准实验图像。
圆角大小分别取 10，50，100，250，这4个值。
每次实验裁剪10000张图片数据，获得总耗时。

实验前关闭所有比较耗CPU的软件。实验中不操作电脑，避免影响实验结果准确性。最后得到了以下实验数据

+ **圆角为 10 的情况**

| 裁剪方法 | 用时 ms | 
|:-:|:-:|
| my裁剪 | 2085 |
| CGContext裁剪 | 15270 |
| 贝塞尔裁剪 | 14317 |

![圆角为 10](http://upload-images.jianshu.io/upload_images/2764502-8309f401d0998258.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ **圆角为 50 的情况**

| 裁剪方法 | 用时 ms | 
|:-:|:-:|
| my裁剪 | 2419 |
| CGContext裁剪 | 14676 |
| 贝塞尔裁剪 | 14612 |

![圆角为 50](http://upload-images.jianshu.io/upload_images/2764502-90da5881b333d6e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ **圆角为 100 的情况**

| 裁剪方法 | 用时 ms | 
|:-:|:-:|
| my裁剪 | 2419 |
| CGContext裁剪 | 14676 |
| 贝塞尔裁剪 | 14612 |

![圆角为 100](http://upload-images.jianshu.io/upload_images/2764502-b73e38ee9e93ab7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ **圆角为 250 的情况**

| 裁剪方法 | 用时 ms | 
|:-:|:-:|
| my裁剪 | 12399 |
| CGContext裁剪 | 14692 |
| 贝塞尔裁剪 | 14313 |

![圆角为 250](http://upload-images.jianshu.io/upload_images/2764502-187ddecf716e83d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 结论
从上面数据可以看出：
**时间上**：不管圆角大小 n 是多少，CGContext 和 UIBezierPath，耗时都在 14.6 秒左右。而 my裁剪在圆角小的时候，性能较好，耗时在 3 秒左右，随着圆角增到250，耗时也去到了 12 秒，但最坏不会超过 w * h / 2，在 n < min(w, h) / 2 时，具有较高的性能，比CGContext, UIBezierPath要快。

**空间上**：内存使用上，没精确测量，大致看了一下，裁剪1万张 512 * 512的图片，3种算法的内存使用都在 10MB 左右，还可以接受，但 UIBezierPath 裁剪时居然会写磁盘。

另外，在图像编码/解码中，用了 CGDataProviderRef，CGImageRef，这两个对象，它们的速度应该是很快的了，如果自己写的话，预测可以更快。

最后口说无凭，我发个代码可以自己下载运行看看结果（[下载地址](https://github.com/hehe520/ClipCorner)）
使用方法都在代码的 ViewController.m 文件里，就这么一个文件，很好找，没有其他了。请在 iphone6，6 plus 的屏幕下运行。本实验重点在于3个算法，无关紧要的东西我没做太多处理。


