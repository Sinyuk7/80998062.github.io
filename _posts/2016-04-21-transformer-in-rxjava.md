---
layout:     post
title:      "Transformer in Rxjava"
subtitle:   "注意 我要变身了"
date:       2016-05-21
author:     "Sinyuk"
header-img: "img/2016-05-21.webp"
tags:
    - Android
    - Rxjava

---

#Transformer in Rxjava

看了[小鄧子](http://www.jianshu.com/users/df40282480b4/latest_articles)的博客以及他引述的其他文章

好多干货

> 还是应该找个时间去看看**RxJava Essentials**


> 之前都没有发现有Transformer这个东西,好菜啊



##Opertor

说到Transformer,先提一下Operator,implement你自己的Opertor很简单,用处也更transformer差不多:

```java
fooObservable = barObservable.ofType(Integer).map({it*2}).lift(new myOperator<T>()).map({"transformed by myOperator: " + it});
```

只是要用`lift()`来把它插入整个chain

##Transformer

然后Transformer的话,其实文档都叫它Transformational Operator了

```java
fooObservable = barObservable.ofType(Integer).map({it*2}).compose(new myTransformer<Integer,String>()).map({"transformed by myOperator: " + it});

```

可以看到Transformer需要和`compose()`一起使用

> compose()这个操作符用来处理表单的逻辑的时候超方便


> 然后大家基本上都是看这篇东西的:

> - [“Don’t break the chain: use RxJava’s compose() operator”](http://blog.danlew.net/2015/03/02/dont-break-the-chain/) by Dan Lew


##Tips

-  你自定义的Operator应该检查`Subscriber's isUnsubscribed()`,接触绑定之后就不该xxx了

- 遵守这几个规则


 - 一个Subscriber的`onComplete()`或者`onError()`方法总会被调用一个,当然也只是一个.但是`onNext()`方法就不一定了


 - `onNext()`方法可以被调用很多次,但是不要让它重叠 *这什么意思啊,背压吗?*


 - 当然你也可以用`serialize()`操作符来保证不出错


- Operator里面不要进行耗时操作


- 你最好组合现成的operator而不是自己写,比如说在Rxjava里面


 - `first()` 就是 `take(1).single()`


 - `ignoreElements()` 就是 `filter(alwaysFalse())`


 - `reduce(a)` 就是 `scan(a).last()`


- 不要对**FATAL** Exception 用`onError()`,因为这个时候你弹个对话框或者toast已经没用了.所以应该先判断是不是Fatal Excetion,可以用`Exceptions.throwIfFatal(throwable)`这个


- 注意`null`是会被Observable发射的.然后基本上很多错误的源头(自己踩过的坑...)
 都是`null`引起的..比如发射了一个`null`而你本以为会是一个empty list


- 然后开始进阶:[ Advanced RxJava](http://akarnokd.blogspot.hu/)
