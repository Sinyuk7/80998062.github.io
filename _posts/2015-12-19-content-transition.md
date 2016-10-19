---
layout:     post
title:      "Content Transitions"
subtitle:   "内容变换"
date:       2015-12-19
author:     "Sinyuk"
header-img: ""
tags:
    - Android
    - Transition

---

# Content Transitions 

首先有几个最关键的问题

1. content Transition 是怎么触发的

2. 什么类型的Transition可以作为content transition

3. 一个ViewGroup和它的子View可以一起变换在content transition之中吗

---

Content Transition和Transition一样,在动画被创建之前,framework必须提供目标视图的状态在视图的visibility变化的时候.比如:当Activity A 启动Activity B的时候:

- A.startActivity()

 - framework检索整个视图层次找到要变化的那些视图
 - transition获取这些视图的开始状态
 - framework把这些视图都设置为INVISIBLE
 - 在下一帧,transition获取视图的结束状态
 - transition比较开始和结束状态,创建Animators.

由此可见,只有当目标视图的visibility变化的时候,framew才能获取视图的状态传递给对应的transitions.

> 所以!!!!

有一个抽象类叫做**Visibility**.

Transition只需要继承这个类(比如默认的Fade,Slide和Explode就是),

在onAppear()和onDisappear()方法里面创建和返回Animator就可以了


---

Views和ViewGroups在transition开始之前,framework会检索整个视图层次,通过调用root view的
ViewGroup#captureTransitioningViews

	/** @hide */
	@Override
	public void captureTransitioningViews(List<View> transitioningViews) {
	    if (getVisibility() != View.VISIBLE) {
	        return;
	    }
	    if (isTransitionGroup()) {
	        transitioningViews.add(this);
	    } else {
	        int count = getChildCount();
	        for (int i = 0; i < count; i++) {
	            View child = getChildAt(i);
	            child.captureTransitioningViews(transitioningViews);
	        }
	    }
	}


向下遍历直到找到VISIBLE的子view或者transition group;(Transition groups允许整个viewgroup作为一个整体变换,吐过一个ViewGroup的isTransitionGroup()返回rue,那么它和它的子View就能一起变换);

> 比如像WebView 你有时候就可能要显式地吊桶setTransitionGroup(true)在transitions开始之前
> 
> isTransitionGroup()会返回true如果ViewGroup带有background drawable和transition name