---
layout:     post
title:      "自定义View的onMeasure()"
subtitle:   "All abount View"
date:       2015-11-05
author:     "Sinyuk"
header-img: "img/2015-11-05.jpg"
tags:
    - Android
    - View

---

##自定义View的onMeasure()

> Custom View's onMeasure() 

首先,我们来看一下View的onMeasure方法:

```java

protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}

```

`setMeasuredDimension()`方法会设置View宽高的测量值


> `onMeasure(int,int)`里面必须调用`setMeasuredDimension()`来设置宽高
> 否则会报错

然后看`getDefaultSize()`这个方法:

```java

public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        result = specSize;
        break;
    }
    return result;
}

```

- AT_MOST和EXACTLY的情况: 返回measureSec中的specSize
 - specSize就是View测量过后的大小

- UNSPECIFIED的情况,View的大小为参数size,即`getSuggestedMinimumWidth()`

```java

protected int getSuggestedMinimumHeight() {
    return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());

}

```

从这里可以看出,

- 如果View没有设置背景,那么最小值就是android:minWidth

- 如果View设置了背景,那么宽度就是`max(mMinHeight, mBackground.getMinimumHeight())`
 - 那么`mBackground.getMinimumHeight()`的值是什么呢

我们来看一下Drawable的`getMinimumHeight()`方法:

```java

public int getMinimumWidth() {
    final int intrinsicWidth = getIntrinsicWidth();
    return intrinsicWidth > 0 ? intrinsicWidth : 0;
}

```

>  return The minimum width suggested by this Drawable. If this Drawable doesn't have a suggested minimum width, 0 is returned.

比如说一个ShapeDrawable就是无原始宽/高,而BitmapDrawable就有



##所以

直接继承View的控件要重写onMeasure方法并设置wrap_content时候的大小,否则在布局中使用wrap_content的时候就相当于使用了match_parent;

> why

如果是wrap_content,那么View的specMode就是AT_MOST,那么它的宽/高等于specSize.

而这种情况下的specSize等于parentSize 即父容器的剩余空间

解决方法:

```java

@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

    if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST){
        setMeasuredDimension(mWidth,mHeight);
    }else if (widthSpecMode == MeasureSpec.AT_MOST ){
        setMeasuredDimension(mWidth,heightSpecSize);
    }else if (heightSpecMode == MeasureSpec.AT_MOST){
        setMeasuredDimension(widthSpecSize,mHeight);
    }
}

```

- 在wrap_conent的时候我们给View定义一个默认的内部宽/高

- 非wrap_content的时候我们沿用系统的测量值即可
