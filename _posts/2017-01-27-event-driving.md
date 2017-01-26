---
layout:     post
title:      "我可能上了一个假的EventBus"
subtitle:   "事件驱动编程"
date:       2017-01-27
author:     "Sinyuk"
header-img: "img/in-post/event-driving/bus.jpg"
tags:
    - Android
    - EventBus
---

# Android事件驱动编程 — 我可能上了一个假的EventBus

肯定每个人都用过**EventBus**,因为被设计的简单易用,看下文档就可以很快上手

之前看关于它的东西基本都是介绍+分析源码+*hello world*的**demo**

- 比如在`Fragment`和`Activity`之前传递事件(不过是减少一点样板代码,as Callback or Lisenter)

- 比如在一些难以控制UI的地方发送事件,然后在View层更新(往往是**Controller**,**Presenter** ,**Adapter**里面)

  ​

🤔感觉很少有人在这之前先告诉我什么是*事件总线,事件驱动*?

之前在看**[Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture)**的时候就发现里面有**[ThreadPoolExecutor](https://github.com/android10/Android-CleanArchitecture/blob/master/data/src/main/java/com/fernandocejas/android10/sample/data/executor/JobExecutor.java)**

当时一脸懵逼,再看了一遍**EventBus**的文档,然后简单了解了一下事件驱动编程

🙄发现一直以来,我可能上了一个假的**EventBus**🚍

---

关于解耦关于模块化的话题,老生常谈了.

主要还是因为代码是写给人看的...自己都不想碰了就tm尴尬了

- 为什么我要把这段代码在不同的地方写两遍?
- 为什么我10w行的*code*,会有3w行的*refactor*和*rewritting*? 当然还有6w行是*.xml*,😄

然后事件驱动编程范式其实很常见…随手:

![mindly](https://github.com/80998062/80998062.github.io/raw/master/img/in-post/event-driving/mindly.jpg)

- 外面是视图层

  用户的不同操作(比如*上上下下左左右右ABAB*)可能会代表同一个事件(背后的业务逻辑),发送然后等待回调作出响应.

- 中间是事件处理器.

  许多情况下,事件处理器可以自己处理事件,因此也可能发送一个事件(回调)

  - 它可能是一个`main loop`,不停地检测和处理事件
  - 也可能会是一个`ThreadPoolExecutor`,注入不同的Thread,处理和发送事件 

不知道这样说对不对,不同于*软件架构*(为了解耦和模块化),*事件驱动程序设计*着重于弹性和异步化,并且尽可能的***modeless***:不用强制等待回调然后响应

不过既然用**EventBus**,就不用关心一些底层实现了

写一个类同时作为`Observer`和`Subscriber`,需要的时候注入一下,在那里处理和发送事件就好了

> *So for example*,一个发送评论的🌰 

#### Event Handler

```java
public class CommentBus implements CommentContact.Bus {
    @NotNull
    private final RemoteDataSource remoteDataSource;
    private final CompositeSubscription mCompositeSubscription;

    @Inject
    public CommentBus(@NotNull RemoteDataSource remoteDataSource) {
        this.remoteDataSource = remoteDataSource;
        mCompositeSubscription = new CompositeSubscription();
    }

    @Inject
    @Override
    public void register() {
        // do register
    }

    @Override
    public void unregister() {
        // do unregister and unsubscribe
    }

    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onPostComment(final CommentPostEvent event) {
        postComment(event.getExecutionScope());
    }

    @Override
    public void postComment(Object scope) {
        mCompositeSubscription.add(remoteDataSource.postComment()
                .subscribe(new Action<Comment>() {
                    @Override
                    public void onNext(Comment comment) {
                        CommentPostCallback callback = new CommentPostCallback(comment);
                        callback.setExecutionScope(scope);
                        EventBus.getDefault().post(callback);
                    }
                }));
    }
}
```

`CommentBus`在被需要的时候注入,它的`Scope`一般会和某个`Activity`/`Fragment`相同,因为我们并不是任何时候都需要监听此类事件.

*PS:用了**Dagger2**之后 感觉能整理每个实例的`Scope`特别好,因为我们的`Context`也都是有自己的生命周期*

比如你可能需要在**Event**中加入一个`Scope`(举个🌰

#### Event

```java
public class CommentPostEvent implements HasExecutionScope {
  	private Object executionScope;
  
    @Override
    public Object getExecutionScope() {
        return executionScope;
    }

    @Override
    public void setExecutionScope(Object executionScope) {
        this.executionScope = executionScope;
    }
}
```

### HasExecutionScope

炒鸡有用的一个接口,因为订阅和注册有时候变得很混乱和难以处理.

比如页面ABCD都订阅了事件E,页面ABCD的实例都存在于栈中,然后你只想页面A响应它自己发出事件的回调.

之前不知道这个接口的时候,我会自己傻乎乎地给**Event**一个个加上TAG😅

现在只要在发送事件的时候设置一个`Scope`

```java
CommentPostEvent event = new CommentPostEvent();
event.setExecutionScope(PostCommentActivity.class);
EventBus.getDefault().post(event);
```

然后在`Subscriber`里判断是否是同一个`Scope`

```java
@Subscribe
public void onPostCommentCallback(CommentPostCallback callback) {
    if (PostCommentActivity.class.equals(callback.getExecutionScope())) {
       // do something 
    }
}
```
### Priorities

当然订阅还可以有优先级

```java
@Subscribe(priority = 1);
public void onEvent(CommentPostCallback callback) {
    ...
}
```

大多数情况下我们都不会用到**Event**的优先级或者事件的取消发送,但是有些时候它还是能派上用场.

- 比如一个事件在前台或者后台的时候需要有不同的UI处理逻辑

- 比如在高优先级的`Subscriber`处理事件的时候取消向下发送,通过调用`cancelEventDelivery(Object event)`

  ​

> 注意: 在**同一个**发送事件的线程下,设置优先级才会生效(默认为0)



---

### FAQs

水了这么多字,其实我还有好多为什么…谁带我上车🙏

**Q1**:

因为**EventBus**只是一个组件之间解耦的工具,我们在开发中还是要选择合适的软件架构(比如**MVP**).那么问题来了,在View,Usecase,Data三个层次,应该用一个事件总线呢还是多个?

如果是一个,复杂度明显会增加很多;如果是多个,可能我们在不同的层级之间要重复发送很多相同的事件.

**Q1.1**:

然后还要面临另外一个问题,当项目里面都是各种**Event**的时候,怎么更好地管理混乱的代码?

>  *If any one give me a working example much appreciated*,*thanks in advance*

**Q2**:

既然有了事件驱动,那么是不是就不用**Presenter**了.因为**Presenter**有时候还是会难以复用,避免不了一堆样板代码或者空接口

当然你可以把**Presenter**的功能再细分,在一个页面注入多个Presenter.但是感觉没有用`post(Event event)`的方式来的酷炫,而且更方便协作开发上层的代码

是这样吗...

