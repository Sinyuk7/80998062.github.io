---
layout:     post
title:      "Something about View"
subtitle:   "笔记笔记"
date:       2016-03-21
author:     "Sinyuk"
header-img: ""
tags:
    - Android
    - View

---

## Something about View

- View的绘制从在ViewrRootde performTraversals()方法之后开始的.

- measure() 接受两个参数:widthMeasureSpec和heightMeasureSpec

  > 即父布局对View的限制条件 而最外层的视图,是从getRootMeasureSpec()得到widthMeasureSpec和heightMeasureSpec

 

- The actual measurement work of a view is performed in {@link #onMeasure(int, int)}, called bythis method.

  > Therefore,only onMeasure(int, int) can and must beoverridden by subclasses

 

- EXACTLY 相当于match_parent
- AT_MOST 相当于wrap_content

 

- measure()这个方法是final的,所以Android系统不希望我们去重写这个方法

 

## onMeasure()

 必须调用`setMeasuredDimension()`之后 才能用`getMeasuredWidth()`和`getMeasuredHeight()`来获取视图测量出的宽高,以此之前得到的值都是0

> 由此可见，视图大小的控制是由父视图、布局文件、以及视图本身共同完成的.
>
> 父视图会提供给子视图参考的大小,而开发人员可以在XML文件中指定视图的大小,然后视图本身会对最终的大小进行拍板

- 在measure过程之后ViewRoot会再一次performTraversals(),然后调用View的layout()方法

 

- `onLayout()`方法是ViewGroup才应该有的

  > 在内部按照各自的规则对子视图进行布局 比如最简单的一个例子:

```java
@Override
protected void onLayout(boolean changed,int l, int t, int r, int b) {

	if (getChildCount() > 0) { 
		View childView =getChildAt(0);
			childView.layout(0, 0,childView.getMeasuredWidth(), childView.getMeasuredHeight());
        }
}
```


> `getWidth()`方法和`getMeasureWidth()`方法到底有什么区别呢?

看起来它们的值是相同的,那只是因为开发者的编程习惯非常好.`getMeasureWidth()`方法在`measure()`过程结束后就可以获取到了 `getWidth()`方法要在`layout()`过程结束后才能获取到`getMeasureWidth()`方法中的值是通过`setMeasuredDimension()`方法来进行设置的



- `getWidth()`方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的
