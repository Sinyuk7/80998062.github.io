---
layout:     post
title:      "Fragments in Dagger2"
subtitle:   "流水账"
date:       2016-03-27
author:     "Sinyuk"
header-img: ""
tags:
    - Android
    - Dagger2

---

#Fragments in Dagger2

> 笔记笔记

##Inject

使用`@Inject`的时候,不能用private修饰符修饰类的成员属性


> 如果是你自己定义的类,你只需要在两边用`@Inject`注解就好了,而不用`@Provides`



##Module 

> 注解类

刚才通过@Inject就可以完成依赖，为什么这里还要用到Module类来提供依赖？

之所以要有Module是为了那些没有构造函数的类的依赖 比如说第三方库,系统类等等



##Qualifier

> 解决依赖注入迷失的


创建类实例有2个维度可以创建：

- 通过用Inject注解标注的构造函数来创建

- 通过工厂模式的Module来创建


这2个维度是有优先级之分的，创建类实例级别Module维度要高于Inject维度


现在有个问题，基于同一个维度条件下，若一个类的实例有多种方法可以创建出来，那注入器（Component）应该选择哪种方法来创建该类的实例呢？

> 就可以用Qualifier了


##Scope

Scope起的更多是一个限制作用.

比如不同层级的Component需要有不同的scope.

注入`@PerActivity`的component后,activity就不能通过@Inject去获得SingleTon的实例，需要从application去暴露接口获得（getAppliationComponent获得component实例然后访问，比如全局的navigator）。


