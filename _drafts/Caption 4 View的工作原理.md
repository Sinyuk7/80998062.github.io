##View的工作原理

##.1

- ViewRoot:连接WindowManager的纽带

View的三大流程 measure,layout和draw都是通过ViewRoot来完成的

ViewRoot对应于VIewRootImpl类


DecorView被添加到Window中,同时创建ViewRootImpl对象


> View的绘制流程从ViewRoot的`perforTraversals()`方法开始的

大致是:

- performMeasure -> measure -> onMeasure
- performLayout  -> layout  -> onLayout
- performDraw -> draw -> onDraw


performXXX 完成顶级View(**DecorView**)的measure,layout和draw

- measure完成之后: `getMeasuredWidth()`可以获取到**测量后**的width
- layout完成之后: `getWidth()`可以获取到**最终**的width


DecorView其实是包含一个vertical的LinearLayout的FrameLayout

- 上面是标题栏
- 下面是内容栏


我靠原来之所以要叫`setContentView()`就是因为内容栏的id叫做android.R.id.content

> 或者是Window.ID_ANDROID_CONTENT 



##.2

MeasureSpec决定了View的尺寸规格,当然也受到父容器的闲置

> 即LayoutParams

MeasureSpec将SpecMode(高两位)和SpecSize(低30位)打包成一个int值

- SpecMode

 - UNSPECOFIED:父容器对View无限制(一般用于系统内部
 
 - EXACTLY:父容器已经检测出View的精确大小(SpecSize的值);对应于LayoutParams的match_parent和具体的数值
 
 - AT_MOST:父容器指定了一个最大值(SpecSize);对应LayoutParams的wrap_content


- MeasureSpec和LayoutParams

LayoutParams和父容器一起才能决定View的MeasureSpec,从而决定View的宽和高

对于普通的View(DecorView比较简单),这里就提一下:

1. 当View使用固定的宽/高的时候,不管父容器的MeasureSpec是什么,View的MeasureSpec都是精确模式的

2. 当View使用match_parent的时候,如果父容器是精确模式,那么View的大小就是父容器的剩余空间;
   当父容器是最大模式的时候,View也是最大模式且不超过父容器的剩余空间.当View是wrap_content的时候,**View的模式总是最大化且不超过父容器的剩余空间**


> 只要提供父容器的MeasureSpec和子View的LayoutParams 就可以快速确定子View的MeasureSpec了;
> 有了MeasureSpec就可以进一步确定子View****的大小了





  

