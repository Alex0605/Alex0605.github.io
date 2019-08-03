---
title: Android知识-五-layout_weight和weightSum属性的详解及使用
date: 2018-04-28 14:34:02
tags: [Android]
---


由于Android设备的尺寸大小不一，种类繁多，当我们在开发应用的时候就要考虑屏幕的适配型了，尽可能让我们的应用适用于主流机型的尺寸，这样我们的应用不会因为尺寸不同而不美观，解决屏幕适配问题的方法有很多，在这里我所讲的是其中的一种解决方案---巧妙的使用layout_weight属性。

​     在布局中Layout_weight属性的作用：它是用来分配属于空间的一个属性，你可以设置它所占据屏幕的权重。

### 1.layout_weight属性  

   首先我们先看下效果，然后再给出计算公式。

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:orientation="horizontal" >  
  
    <Button  
        android:layout_width="0dp"  
        android:layout_height="wrap_content"  
        android:layout_weight="1"  
        android:text="buttton1" />  
  
    <Button  
        android:layout_width="0dp"  
        android:layout_height="wrap_content"  
        android:layout_weight="2"  
        android:text="button2" />  
  
</LinearLayout>
```

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:orientation="horizontal" >  
  
    <Button  
        android:layout_width="0dp"  
        android:layout_height="wrap_content"  
        android:layout_weight="1"  
        android:text="buttton1" />  
  
    <Button  
        android:layout_width="0dp"  
        android:layout_height="wrap_content"  
        android:layout_weight="2"  
        android:text="button2" />  
  
</LinearLayout>
```

   计算公式如下：

​     首先我们假设屏幕的宽度为L

​     实际宽度 = 控件原来的长度 + 剩余空间所占百分比的宽度

​     比如第一图片上的计算式子如下

​      button1实际宽度 = 0 + 1/(1+2)L= 1/3L     button2实际宽度 = 0 +2/(1+2)L = 2/3L

​     第二张图片上的计算式子如下 

​      button1实际宽度 = L +1[L-(L+L)]/(1+2) L = 2/3 L    button2实际宽度 = L + 2[L-(L+L)]/(1+2) L = 1/3 L

​     好了，看到这里weight属性是不是很好理解，最主要的是对剩余空间的理解 。

### 2.weightSum属性

android:weightSum属性在官方文档中的解释包含下面的内容：“定义weight综合的最大值，如果未指定该值，以所有子师徒的layout_weight属性的累加值作为总和的最大值。典型案例是：通过指定子视图的layout_weight属性为0.5.并设置LinearLayout的weightSum属性为1.0，实现子视图占据可用宽高的50%”。

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="wrap_content"  
    android:gravity="center"  
    android:orientation="horizontal"  
    android:weightSum="1" >  
  
    <Button  
        android:layout_width="0dp"  
        android:layout_height="wrap_content"  
        android:layout_weight="0.5"  
        android:text="test" />  
  
</LinearLayout>
```

