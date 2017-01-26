

**Q1**:

因为**EventBus**只是一个组件之间解耦的工具,我们在开发中还是要选择合适的软件架构(比如**Clean Architecture**).

那么问题来了,在视图层,用例层,数据层三个层面,我们应该用一个事件总线呢还是多个?

如果是一个,复杂度明显会增加很多;如果是多个,可能我们在不同的层级之间要重复发送很多相同的事件.



当你发送一个事件,你的`subscriber`方法可能会被调用多次

- 记得调用`unregister()`



**Why am I getting the warning “The following options were not recognized by any processor: ‘[eventBusIndex]'”?**

This happens when you set up the build script, and you did not add any annotations to your code yet. Without any @Subscribe annotations, annotation processor for indexing will not be triggered to consume the ‘eventBusIndex’ option. So, just add some @Subscribe annotations and you should be fine

你在onEvent()做某种初始化的话

当acitivity重建的时候 不能retain state