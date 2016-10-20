---
layout:     post
title:      "Subcomponents和Component Dependencies"
subtitle:   "Dagger 2 subcomponents vs component dependencies"
date:       2016-04-11
author:     "Sinyuk"
header-img: ""
tags:
    - Android
    - Dagger2
    - Component

---


Component Dependencies:

- 让两个components相互独立
- 需要显示的暴露出它们之间的依赖

Subcomponents

- 关联两个components
- **不**需要显示的暴露出它们之间的依赖


---

> 举个栗子

ClassB1依赖于ClassA1,从ModuleB的`provideSomeClassB1()`方法里面可以看出来:

```java

@Module
public class ModuleA {
    @Provides
    public SomeClassA1 provideSomeClassA1() {
        return new SomeClassA1();
    }
}
@Module
public class ModuleB {
    @Provides
    public SomeClassB1 provideSomeClassB1(SomeClassA1 someClassA1) {
        return new SomeClassB1(someClassA1);
    }
}
public class SomeClassA1 {
    public SomeClassA1() {}
}
public class SomeClassB1 {
    private SomeClassA1 someClassA1;
public SomeClassB1(SomeClassA1 someClassA1) {
        this.someClassA1 = someClassA1;
    }
}

```


> 先是 Component Dependency


关键点:

- SomeClassB1依赖于SomeClassA1. ComponentA要显示的暴露出这个.
- 但是ComponentA中不用声明ModuleB.2个Components保持独立


```java

public class ComponentDependency {
    @Component(modules = ModuleA.class)
    public interface ComponentA {
        SomeClassA1 someClassA1();
    }

@Component(modules = ModuleB.class, dependencies = ComponentA.class)
    public interface ComponentB {
        SomeClassB1 someClassB1();
    }

public static void main(String[] args) {

        ModuleA moduleA = new ModuleA();

        ComponentA componentA =
			DaggerComponentDependency_ComponentA.builder()
                .moduleA(moduleA)
                .build();

		ModuleB moduleB = new ModuleB();
        ComponentB componentB =
			DaggerComponentDependency_ComponentB.builder()
                .moduleB(moduleB)
                .componentA(componentA)
                .build();
    }
}

```


> 然后的SubComponent的栗子

关键点:

- SomeClassB1依赖于SomeClassA1.但是ComponentB不用显示的暴露出这个.
- ComponentA中必须声明ModuleB. 从而绑定了2个components.

```java

public class SubComponent {
    @Component(modules = {ModuleA.class, ModuleB.class})
    public interface ComponentA {
        ComponentB componentB(ModuleB moduleB);
    }

    @Subcomponent(modules = ModuleB.class)
    public interface ComponentB {
        SomeClassB1 someClassB1();
    }

    public static void main(String[] args) {
        ModuleA moduleA = new ModuleA();
        ModuleB moduleB = new ModuleB();
        ComponentA componentA = DaggerSubComponent_ComponentA.builder()
                .moduleA(moduleA)
                .moduleB(moduleB)
                .build();

        ComponentB componentB = componentA.componentB(moduleB);
    }
}

```


 > 注意ComponentB用了A的依赖,然后在builder()那里:
 > `.componentA(componentA).build()` 就可以这样写


---


##这两种方法的区别？



上面讨论了依赖可视性的区别。

- @Subcomponent从它的父类访问所有依赖
- @Component只能访问在基类@Component接口暴露的公共性的依赖



其实没什么区别啊....

> 但是我基本上只用@SubComponent 因为可以减少方法数

传送门:[Dagger 2 on production — reducing methods count](https://medium.com/azimolabs/dagger-2-on-production-reducing-methods-count-5a13ff671e30#.8u97jl3ev)
