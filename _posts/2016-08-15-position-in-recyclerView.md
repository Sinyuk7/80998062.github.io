---
layout:     post
title:      "Position in RecyclerView"
subtitle:   "一个Exception"
date:       2016-08-15
author:     "Sinyuk"
header-img: ""
tags:
    - Android
    - RecyclerView
---

## Position in RecyclerView

We know, there are two types of position related methods inRecyclerView:

- layout position: Position of an item in the latest layout calculation. This is the position from the LayoutManager's perspective.
- adapter position: Position of an item in the adapter. This is the position from theAdapter's perspective.

These two positions are the same except the time between dispatching adapter.notify* events and calculating the updated layout.



**不同点:**

- layout position 是在最近一次layout update或者是calculation之后才更新的 这里位置指的就是能在屏幕上看到的位置;

- adapter position更新的话 肯定是在notify**的方法之后.

  **那么问题来了:这之后可能不会更新layout 因为位置不会发生变化**

 **文档里面说:**

> Beware that these methods may not be able to calculate adapter positions if [notifyDataSetChanged()](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html#notifyDataSetChanged()) has been called and new layout has not yet been calculated. For this reasons, you should carefully handle [NO_POSITION](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html#NO_POSITION) or null results from these methods.

而这样一来,不可见的item就不会初始化,那么根据Adapter Position 返回的itemView就有可能是空的!
