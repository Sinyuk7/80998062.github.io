---
layout:     post
title:      "Schedules In Rxjava"
subtitle:   "线程切换"
date:       2016-04-05
author:     "Sinyuk"
header-img: ""
tags:
    - Android
    - Rxjava
---

# Schedules In Rxjava#

`observeOn()`对调用之前的序列默不关心，也不会要求之前的序列运行在指定的线程上

`observeOn()`对之前的序列产生的结果先缓存起来，然后再在指定的线程上,推送给最终的subscriber

`subscribeOn()`的调用，改变了调用前序列所运行的线程. **调用前!**



我们经常多次使用`subscribeOn()`切换线程,那么以后是否可以组合`observeOn()`和`subscribeOn()`达到自由切换的目的呢？

组合是可以的,但是他们的执行顺序是有条件的,如果仔细分析的话,可以知道`observeOn()`调用之后,再调用`subscribeOn()`是无效的,why?

因为`subscribeOn()`改变的是subscribe这句调用所在的线程,大多数情况,产生内容和消费内容是在同一线程的，所以改变了产生内容所在的线程,就改变了消费内容所在的线程.

换句话说,我们知道,`observeOn()`的工作原理是把消费结果先缓存,再切换到新线程上让原始消费者消费，它和生产者是没有一点关系的,就算`subscribeOn()`调用了,也只是改变`observeOn()`这个消费者所在的线程，和OperatorObserveOn中存储的原始消费者一点关系都没有,它还是由`observeOn()`控制.

# 总结 #

如果我们有一段这样的序列:

`Observable`

`.map()                    // 操作1`

`.flatMap()                // 操作2`
`.subscribeOn(io)`

`.map()                    //操作3`

`.flatMap()                //操作4`

`.observeOn(main)`

`.map()                    //操作5`

`.flatMap()                //操作6`
`.subscribeOn(io)        //特别注意!`

`.subscribe(handleData)`

假设这里我们是在主线程上调用这段代码，

那么

- 操作1，操作2是在io线程上，因为之后subscribeOn切换了线程
- 操作3，操作4也是在io线程上，因为在subscribeOn切换了线程之后，并没有发生改变。
- 操作5，操作6是在main线程上，因为在他们之前的`observeOn()`切换了线程。

> 特别注意那一段，对于操作5和操作6是无效的

再简单点总结就是

- `subscribeOn()`的调用**切换之前**的线程。
- `observeOn()`的调用**切换之后**的线程。
- `observeOn()`之后，不可再调用`subscribeOn()` 切换线程

---

最后,下面提到的“操作”包括产生事件、用操作符操作事件以及最终的通过subscriber消费事件

1. 只有第一个`subscribeOn()` 起作用
2. 这个 `subscribeOn()` 控制从流程开始的第一个操作,直到遇到第一个`observeOn()`
3. `observeOn()` 可以使用多次,每个`observeOn()`将导致一次线程切换,这次切换开始于这次`observeOn()`的下一个操作
4. 不论是`subscribeOn()`还是`observeOn()`每次线程切换如果不受到下一个`observeOn()`的干预,线程将不再改变,不会自动切换到其他线程.
