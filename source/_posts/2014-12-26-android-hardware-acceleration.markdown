---
layout: post
title: "Android--Hardware Acceleration"
date: 2014-12-26 09:55
comments: true
categories: android
---
Android中的硬件加速
本文的主要内容来自SDK文章的"Hardware Acceleration”.

从Android 3.0开始，Android的2D渲染管线可以更好的支持硬件加速。硬件加速使用GPU进行View上的绘制操作。

硬件加速可以在一下四个级别开启或关闭：

Application
Activity
Window
View
Application级别

往您的应用程序AndroidManifest.xml文件为application标签添加如下的属性即可为整个应用程序开启硬件加速：
```
<application android:hardwareAccelerated="true" .../>
```
Activity级别

您还可以控制每个activity是否开启硬件加速，只需在activity元素中添加android:hardwareAccelerated属性即可办到。比如下面的例子，在application级别开启硬件加速，但在某个activity上关闭硬件加速。
```
<application android:hardwareAccelerated="true">
    <activity ... />
    <activity android:hardwareAccelerated="false" />
</application>
```
Window级别

如果您需要更小粒度的控制，可以使用如下代码开启某个window的硬件加速：
```java
getWindow().setFlags(
    WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED,
    WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED);
```
注：目前还不能在window级别关闭硬件加速。

View级别

您可以在运行时用以下的代码关闭单个view的硬件加速：

myView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
注：您不能在view级别开启硬件加速

为什么需要这么多级别的控制？

很明显，硬件加速能够带来性能提升，android为什么要弄出这么多级别的控制，而不是默认就是全部硬件加速呢？原因是并非所有的2D绘图操作支持硬件加速，如果您的程序中使用了自定义视图或者绘图调用，程序可能会工作不正常。如果您的程序中只是用了标准的视图和Drawable，放心大胆的开启硬件加速吧！具体是哪些绘图操作不支持硬件加速呢?以下是已知不支持硬件加速的绘图操作：

Canvas
clipPath()
clipRegion()
drawPicture()
drawPosText()
drawTextOnPath()
drawVertices()
Paint
setLinearText()
setMaskFilter()
setRasterizer()
另外还有一些绘图操作，开启和不开启硬件加速，效果不一样：

Canvas
clipRect()： XOR, Difference和ReverseDifference裁剪模式被忽略，3D变换将不会应用在裁剪的矩形上。
drawBitmapMesh()：colors数组被忽略
drawLines()：反锯齿不支持
setDrawFilter()：可以设置，但无效果
Paint
setDither()： 忽略
setFilterBitmap()：过滤永远开启
setShadowLayer()：只能用在文本上
ComposeShader
ComposeShader只能包含不同类型的shader (比如一个BitmapShader和一个LinearGradient，但不能是两个BitmapShader实例)
ComposeShader不能包含ComposeShader
如果应用程序受到这些影响，您可以在受影响的部分调用setLayerType(View.LAYER_TYPE_SOFTWARE, null)，这样在其它地方仍然可以享受硬件加速带来的好处

Android的绘制模型

开启硬件加速后，Android框架将采用新的绘制模型。基于软件的绘制模型和基于硬件的绘制模型有和不同呢？

基于软件的绘制模型

在软件绘制模型下，视图按照如下两个步骤绘制：

1. Invalidate the hierarchy（注：hierarchy怎么翻译？）

2. Draw the hierarchy

应用程序调用invalidate()更新UI的某一部分，失效(invalidation)消息将会在整个视图层中传递，计算每个需要重绘的区域（即脏区域）。然后Android系统将会重绘所有和脏区域有交集的view。很明显，这种绘图模式存在缺点：

1. 每个绘制操作中会执行不必要的代码。比如如果应用程序调用invalidate()重绘button，而button又位于另一个view之上，即使该view没有变化，也会进行重绘。

2. 可能会掩盖一些应用程序的bug。因为android系统会重绘与脏区域有交集的view，所以view的内容可能会在没有调用invalidate()的情况下重绘。这可能会导致一个view依赖于其它view的失效才得到正确的行为。

基于硬件的绘制模型

Android系统仍然使用invalidate()和draw()来绘制view，但在处理绘制上有所不同。Android系统记录绘制命令到显示列表，而不是立即执行绘制命令。另一个优化就是Android系统只需记录和更新标记为脏（通过invalidate()）的view。新的绘制模型包含三个步骤：

1. Invalidate the hierarchy

2. 记录和更新显示列表

3. 绘制显示列表