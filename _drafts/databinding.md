#Fragment on DataBinding

> DataBinding笔记

##Observable

- DataBinding会给<data>标签内的variable生成对应的BR常量

- 或者说是在model的getter方法前加上@Bindable注解

> 难道说只要在`<data></data>`里面声明的variable


> DataBinding会自动注解它的getter方法吗?

##BR

> 生成的BR类 举个栗子:

```java

public class BR {        
    public static final int _all = 0;    
    public static final int avatar = 1;      
    public static final int user = 2;
}


```

- 每一个有ID的View 都是public final
> 所以我们可以直接引用

- 没有id的View 但是在处理中添加了Tag的View 会是private final的(比如mBoundView0,mBoundView1)

##@Bindable注解

> 为什么我们要把它放在getter方法前面

```java

public String getAlias(){
    return "Alias";
}

```

即使你没有声明一个String类型的alias field,你也可以在binding expression中这样写@{user.alias}


---

#Advanced

##Expression

> 现在支持的表达式有

- 算术 +-/*%
- 字符串合并 +
- 逻辑 && ||
- 二元 &|^
- 三元 ?:
- 一元 +-!~
- 移位 >> << >>>
- 比较 == > < >= <=
- Instanceof
- Grouping() 这是干嘛的?
- 文字 character String numeric null
- Cast
- 方法调用
- Field访问
- Array访问

> 尚不支持this,super,new已经显式的泛型调用

> 空合并运算符(取第一个非空值作为结果

```xml

android:text="@{user.displayName ?? user.lastName}"

```

本来我们肯定需要自己计算一个dimension来存放另外两个dimension的和 现在:

```xml

android:marginLeft="@{@dimen/margin + @dimen/avatar_size}"

```

> 我靠碉堡了

然后还有`String.format()`可以这样用了:

```xml

android:text="@{@string/nameFormat(firstName, lastName)}"

<string name="nameFormat">%s, %s</string>

```

##Null-safety
DataBinding会帮我们避免空指针

> 但是一定要注意数组的越界...xml不能判断这个东西

##include

include的布局要在外面绑定一次,在里面再绑定一次

> include的布局必须是一个ViewGroup

##ViewStub

ViewStub在DataBinding里面用了代理:ViewStubProxy,它监听了ViewStub#OnInflateListener:


##Binding

- BindingMethods

View里面有setter的话 就可以直接在xml引用

> 但是如果View本身就支持这种属性的set,只是方法名字不一样,我们难道就要重新写一个??

这种情况我们就可以用BindingMethods注释

> 举个栗子 `setImageTintList()`


```java
@BindingMethods({
       @BindingMethod(type = “android.widget.ImageView”,
                      attribute = “android:tint”,
                      method = “setImageTintList”),
})

```

BindingMethods注解可以为setter重命名

> 但是这是在方法签名相同的情况下

- BindingAdapter

> 当没有对应的setter或者方法签名不同的时候


比如说android:paddingLeft和android:layout_margin,是要通过`setpadding(l,t,r,b)`和LayoutParams来修改的

> 这个时候

```java
@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int padding) {
    view.setPadding(padding,
                    view.getPaddingTop(),
                    view.getPaddingRight(),
                    view.getPaddingBottom());
}

```
> 虽然这个DataBinding已经帮我们实现好了

可以在这里看到所有的BindingAdapter和BindingMethod

[传送门](https://android.googlesource.com/platform/frameworks/data-binding/+/android-6.0.0_r7/extensions/baseAdapters/src/main/java/android/databinding/adapters)

你还可以传入旧的值

> 举个栗子

```java

@BindingAdapter("android:onLayoutChange")
 public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue,
                                              View.OnLayoutChangeListener newValue) {
     if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
         if (oldValue != null) {
             view.removeOnLayoutChangeListener(oldValue);
         }
         if (newValue != null) {
             view.addOnLayoutChangeListener(newValue);
         }
     }
 }

```


当然也可以同时绑定多个属性

> 举个栗子

```java

@BindingAdapter({"android:onClick", "android:clickable"})
 public static void setOnClick(View view, View.OnClickListener clickListener,
                               boolean clickable) {
     view.setOnClickListener(clickListener);
     view.setClickable(clickable);
 }

```

只有当所有的属性都关联上的时候,方法才会被调用

> 我靠这个好给力

**后面参数的顺序必须和注解里面的一样**

当一个Listener里不止一个方法的时候

我们必须要用到多个BindingAdapter,每个属性一个,然后全部的也要有一个

- BindingConversion

有时候在xml里面绑定的值不一定是setter需要的

> 举个栗子

```java

@BindingAdapter("android:onViewAttachedToWindow")
public static void setListener(View view, OnViewAttachedToWindow attached) {
    setListener(view, null, attached);
}

@BindingAdapter("android:onViewDetachedFromWindow")
public static void setListener(View view, OnViewDetachedFromWindow detached) {
    setListener(view, detached, null);
}

@BindingAdapter({"android:onViewDetachedFromWindow", "android:onViewAttachedToWindow"})
public static void setListener(View view, final OnViewDetachedFromWindow detach,
        final OnViewAttachedToWindow attach) {
 
}

```

> 比如

```xml
<View
    android:background=“@{isError ? @color/red : @color/white}”
    android:layout_width=“wrap_content”
    android:layout_height=“wrap_content”/>

```

```java

@BindingConversion
    public static ColorDrawable convertColorToDrawable(int color) {
        return new ColorDrawable(color);
}

```

##双向绑定

使用**@=**来进行双向绑定

```xml

<EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:inputType="textNoSuggestions"
    android:text="@={model.name}"/>

```
这样，我们对这个EditText的输入，就会自动set到对应model的name字段上

> 卧槽碉堡了啊

原理: InverseBindingListener(事件触发时的监听器


##OnPropertyChangedCallback

BaseObservable会有这么一个回调,我们可以用它在某个属性改变的时候搞点事情

> 举个栗子

```java

mModel.addOnPropertyChangedCallback(new Observable.OnPropertyChangedCallback() {
    @Override
    public void onPropertyChanged(Observable observable, int i) {
        if (i == BR.name) {
            Toast.makeText(TwoWayActivity.this, "name changed",
                    Toast.LENGTH_SHORT).show();
        } else if (i == BR.password) {
            Toast.makeText(TwoWayActivity.this, "password changed",
                    Toast.LENGTH_SHORT).show();
        }
    }
});

```

##表达式

- 重复的表达式

```xml
<ImageView android:id=“@+id/avatar”
 android:visibility=“@{user.isAdult ? View.VISIBLE : View.GONE}”/>
<TextView android:visibility=“@{avatar.visibility}”/>
<CheckBox android:visibility="@{avatar.visibility}"/>

```

- 隐式更新

```xml

<CheckBox android:id=”@+id/seeAds“/>
<ImageView android:visibility=“@{seeAds.checked ?
  View.VISIBLE : View.GONE}”/>

```

> checkBox的状态变更后ImageView的Visibility也会变化


##Lambda表达式

除了直接使用方法引用，在Presenter中写和OnClickListener一样参数的方法，我们还能使用Lambda表达式:

> 就是指`@{presenter::onClick}`吗?

```xml
android:onClick=“@{(view)->presenter.save(view, item)}”
android:onClick=“@{()->presenter.save(item)}”
android:onFocusChange=“@{(v, fcs)->presenter.refresh(item)}”

```

##ReBindCallback

我们可以用OnReBindCallback搞一点事情

> 举个栗子

```java

binding.addOnRebindCallback(new OnRebindCallback() {
    @Override
    public boolean onPreBind(ViewDataBinding binding) {
        ViewGroup sceneRoot = (ViewGroup) binding.getRoot();
        TransitionManager.beginDelayedTransition(sceneRoot);
        return true;
    }
});

```

> 自动的去做Transition动画

> 我们还可以在lambda表达式引用view id（像上面表达式链那样），以及context


##Component


