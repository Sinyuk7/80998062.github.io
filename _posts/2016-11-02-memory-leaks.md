---
layout:     post
title:      "Memory Leaks"
subtitle:   "内存泄露"
date:       2016-11-02
author:     "Sinyuk"
header-img: "img/22.jpg"
tags:
    - Android
    - Performance
    
---


#Memory Leaks

> 内存泄露

##静态变量

```java

private static Context sContext

```

> 静态变量无法被释放


##单例模式

因为单例模式的特点是生命周期和Application保持一致,所以当Activity对象被销毁的时候,它还会持有context的引用


- 属性动画

> 这个坑我踩过好多次....

如果你在Activity的`onDestory()`中没有停止这个动画,那么这个View就会被动画持有,而View有持有Activity的引用,所以发生了内存泄露


##Handler

当Android应用首次启动的时候,framework会在UI thread创建一个Looper对象.

Looper就是一个简单的消息队列,依次来处理所有的message

> 比如Activity Lifecycle callback,events

UI线程的Looper存在于整个application的lifecycle里


> In Java, non-static inner and anonymous classes hold an implicit reference to their 
outer class. Static inner classes, on the other hand, do not.


> 在Java中，非静态的内部类或者匿名类会隐式的持有其外部类的引用，而静态的内部类则不会。

可能Activity被销毁的时候,延迟发送的message还存在消息队列里面.

- 这个消息可能会持有activity的引用
- handler又隐式的持有外部类的引用(也就是Activity
- handler里面的Runnable也会导致内存泄露:**静态的匿名类会隐式的持有外部类的引用**


##解决方式

> 使用弱引用

```java

private static class MyHandler extends Handler {
	private final WeakReference<SampleActivity> mActivity;

	public MyHandler(SampleActivity activity) {
	  mActivity = new WeakReference<SampleActivity>(activity);
	}

	@Override
	public void handleMessage(Message msg) {
	  SampleActivity activity = mActivity.get();
	  if (activity != null) {
	    // ...
	  }
	}
}

```

> 使用static修饰内部类

```java

private static final Runnable sRunnable = new Runnable() {
  @Override
  public void run() { /* ... */ }
};

```

> 当然还可以在activity的`onDestroy()`里面通过handler的removeCallbacksAndMessages(null)来防止泄露

---

所以为什么不干脆用EventBus(黑人问号???