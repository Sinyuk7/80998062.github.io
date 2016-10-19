---
layout:     post
title:      "Fragments on Activity"
subtitle:   "笔记笔记"
date:       2015-10-18
author:     "Sinyuk"
header-img: ""
tags:
    - Android
    - Activity

---
#Fragments on Activity

##LauchMode

	1. Standard
	2. SingleTop
	3. SingleTask
	4. SIngleInstance

- singleTop
如果发现有对应的Activity实例正位于栈顶，则重复利用，不再生成新的实例。

- singleTask
将FirstActivity之上的Activity实例统统出栈，将FirstActivity变为栈顶对象，显示到幕前

- singleInstance

## isFinishing()
if the activity is finishing, returns true; else returns false.
  > Check to see whether this activity is in the process of finishing, either because you called finish on it or someone else has requested that it finished. This is often used in onPause to determine whether the activity is simply pausing or completely finishing.

> 判断Activity是否finish，自己调用或者某种因素请求finish，经常用在onPause函数里面

##onPause()

一般建议在onPause()中停止一切动画效果，或者其他消耗cpu的行为

在离开之前提交未被保存改动，你应该只保存那些用户认为应该保存的数据，比如在邮箱应用中，用户正在向文本输入框中写入文字，那么这种改动 被保存下来才显得合理。

释放系统资源，比如广播接受者，传感处理器（gps等），或者其他影响电池消耗的东西，因为在pause状态下用户并不需要他们。

##finish()

一个Activity在Finish的时候还在接受Broadcast Handle还在sendMessage或者还在执行transition Animation,
所以最好调用removeAllCallbackAndMessage(this)


##overridePendingTransition()

实现两个 Activity 切换时的动画。在Activity中使用

- 第一个activity退出时的动画；
- 第二个activity进入时的动画；

> 注意
> 1、必须在`startActivity()`或`finish()`之后立即调用。
> 2、而且在 2.1 以上版本有效
> 3、手机设置-显示-动画，要开启状态

##onPostCreate()

> Called when activity start-up is complete (after onStart() and onRestoreInstanceState(Bundle) have been called). Applications will generally not implement this method; it is intended for system classes to do final initialization after application code has run.

> Derived classes must call through to the super class's implementation of this method. If they do not, an exception will be thrown.