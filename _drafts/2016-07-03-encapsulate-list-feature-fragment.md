---
layout:     post
title:      "封装一下显示列表的Fragment"
subtitle:   "记记流水账"
date:       2016-07-03
author:     "Sinyuk"
header-img: "img/2016-07-03.jpg"
tags:
    - Android
    - RecyclerView

---


# 封装一下显示列表的Fragment#

> 我的滑板鞋 时尚时尚最时尚


## 列表状态的切换##

> 其实已经有很多开源的库封装了页面状态切换的代码,基本上都是通过`addView()`和`removeView()`

> 好像没有更加高效的办法了 有吗?


我基本也差不多,用了ViewAnimator...

> 一开始也没听过这个类 = =

其实就是可以切换两个View 然后还能带一点动画的东西.

类似TextSwitcher,ImageSwitcher之类的


这里自己改了一下,为了方便使用

```java

public class BetterViewAnimator extends ViewAnimator {
  public BetterViewAnimator(Context context, AttributeSet attrs) {
    super(context, attrs);
  }

  public void setDisplayedChildId(int id) {
    if (getDisplayedChildId() == id) {
      return;
    }
    for (int i = 0, count = getChildCount(); i < count; i++) {
      if (getChildAt(i).getId() == id) {
        setDisplayedChild(i);
        return;
      }
    }
    String name = getResources().getResourceEntryName(id);
    throw new IllegalArgumentException("No view with ID " + name);
  }

  public int getDisplayedChildId() {
    return getChildAt(getDisplayedChild()).getId();
  }
}

```

## 布局

然后你就可以像这样写一个通用的布局

```xml

<com.sinyuk.yukdaily.widgets.BetterViewAnimator
android:layout_width="match_parent"
android:inAnimator="@anim/fade_in"
android:outAnimator="@anim/fade_out"
android:layout_height="match_parent">

<include layout="@layout/layout_list"/>

<include layout="@layout/layout_error"/>

<include layout="@layout/layout_empty"/>
</com.sinyuk.yukdaily.widgets.BetterViewAnimator>

```

感觉这样能满足大部分业务需求吧,layout_list里面有个SwipeRefreshlayout+RecyclerView就够了吧...

因为你还可以加header和footer....

> 不过header的话最好不要写死,让它也可以被recyle最好吧....

## BaseListFragment


然后就用一个(Fragment轻量点吧),比如叫BaseListFragment

里面可以先写好类似这些方法

```java

@CallSuper
protected void assertError(String message) {
    binding.errorLayout.setErrorMessage(message);
    binding.viewAnimator.setDisplayedChildId(R.id.errorLayout);
}

public void onClickEmpty() {
    startRefresh();
}

public void onClickError() {
    startRefresh();
}

@CallSuper
protected void assertEmpty(String message) {
    binding.emptyLayout.setEmptyMessage(message);
    binding.viewAnimator.setDisplayedChildId(R.id.emptyLayout);
}

public void startRefresh() {
    binding.viewAnimator.setDisplayedChildId(R.id.listLayout);
	//...skip
}

public void stopRefresh() {
   //...skip
}


protected abstract void refreshData();

protected abstract void fetchData();

```

主要就是三个状态的切换可以写在父类里面,这个基本不会有什么改变了...

然后子类想自己处理的时候加上super.xxx()就好了


## 再比如##

当然在这个类里面你还可以写好用来加载更多的ScrollListener

> 举个栗子

```java

protected RecyclerView.OnScrollListener getLoadMoreListener() {
    return new RecyclerView.OnScrollListener() {
        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            if (isLoading) {
                return;
            }
            final LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();
            boolean isBottom =
                    layoutManager.findLastCompletelyVisibleItemPosition() >= recyclerView.getAdapter().getItemCount() - PRELOAD_THRESHOLD;
            if (isBottom) {
                fetchData();
            }
        }
    };
}

```

这样一来写一个列表的Fragment就会很轻松了



## 有个坑##

判断是否要显示一个EmptyView的时候,我们可以像这样:

```java

adapter.registerDataSetObserver(new DataSetObserver() {
        @Override
        public void onChanged() {
            if (adapter.getCount() == 0)
            {
                // show empty view
            }
        }
    });

```

一开始感觉自己好机智啊,后来....

我靠原来这个`onChange()`只会在你调用`notifiDataSetChange()`的时候才会触发...

而很多时候我们为了更高效的更新列表,或者是为了动画,都是会去用一些局部更新的方法

> 比如`notifyDataRangeInsert( start , count )`之类的

当然后来我都会用stable id,然后就算用`notifiDataSetChange()`也会有动画...


言归正传,其实也可以不用DataSetObserver,后来我是在刷新之后的Observer(或者叫subscriber吧)的onComplete()方法
里面做的判断...

因为刷新之后item count什么的为零,肯定就是空列表了吧....

像这样:

```java

@Override
public void onCompleted() {
    if (binding.listLayout.recyclerView.getAdapter().getItemCount() <= 0) {
        assertEmpty(getString(R.string.no_news));
    }
}

```

好了好了,记点流水账....
