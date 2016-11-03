

#View的工作原理

> 自定义View和布局

##Tips

- 支持Padding

直接继承View的控件,要在`onDraw()`方法中处理**padding**,不然padding属性是无法起作用的

> 举个栗子

```java

protected void onDraw(Canvas canvas){
	super.onDraw(canvas);
	final int paddingLeft = getPaddingLeft()
	...
	int width = getWidth() - paddingLeft - paddingRight;
	int height = getHeight() - paddingTop - paddingBottom;
	canvas.drawRect(...);
}

```

同样的,直接继承自ViewGroup的控件要在`onMeasure()`和`onLayout()`里面处理padding和子元素的margin

> margin属性是由父容器控制的

自定义的布局比自定义View要复杂一点



- 线程和动画

记得在View#onDetachFromWindow方法中停止动画,避免内存泄露

- 滑动冲突

