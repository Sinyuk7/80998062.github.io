# EventBus官方文档

## Configuration

`EventBusBuilder`这个类从各个方面来设置**EventBus**.

比如:

```java
EventBus eventBus = EventBus.builder()
    .logNoSubscriberMessages(false)
    .sendNoSubscriberEvent(false)
    .build();
```

或者:

```java
EventBus eventBus = EventBus.builder().throwSubscriberException(true).build();
```

默认情况下,EventBus能捕获`subscriber`方法中的异常然后发送一个`SubscriberExceptionEvent`(但是你可以不用去处理它)

> 查看EventBusBuilder的JavaDoc来获取所有的设置选项

### Configure the default EventBus instance

一般我们都会简单地使用`EventBus.getDefault()`来获取一个全局的实例.所以`EventBusBuilder`也可以改变这个默认实例的设置,通过		`installDefaultEventBus()`方法

比如我们只想在DEBUG版本里的`subscriber`方法中抛出异常,

```java
EventBus.builder().throwSubscriberException(BuildConfig.DEBUG).installDefaultEventBus();
```

>  在EventBus被调用之后再`installDefaultEventBus()`就会报错,所以在`Application`类中初始化EventBus是一个不错的选择.



## Priorities and Event Cancellation

大多数情况下我们都不会用到**EventBus**的优先级或者事件的取消发送,但是有些时候它还是能派上用场.

比如一个事件在前台或者后台的时候需要有不同的UI处理逻辑

### Subscriber Priorities

你可以在注册事件的时候设置订阅者的优先级.

举个🌰

```java
@Subscribe(priority = 1);
public void onEvent(MessageEvent event) {
    ...
}
```

在**同一个**发送事件的线程下,设置优先级才会生效(默认为0)

### Cancelling event delivery

在事件发送过程中,你可以在订阅者的处理事件方法中调用`cancelEventDelivery(Object event)`来取消它.

```java
// Called in the same thread (default)
@Subscribe
public void onEvent(MessageEvent event){
    // Process the event
    ...
    // Prevent delivery to other subscribers
    EventBus.getDefault().cancelEventDelivery(event) ;
}
```

取消事件传递的通常的高优先级的订阅者

取消事件只能在发送事件的同一个线程(`ThreadMode.PostThread`)

### Subscriber Index

Index是EventBus 3 的新功能.

你可以**选择**使用它来优化订阅者注册的速度.

#### Index Preconditions

只有被`@Subscriber`标注的方法(`subscriber`和`event`只能是**public**)才能被indexed

并且`@Subscriber`注解在匿名类中不起作用

### How to generate the index

> 如果你正在使用的Gradle Plugin版本在2.2以上,如下设置android-apt

开启index生成功能,你需要在`build.gradle`中添加下面这个snippet:

```groovy
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [ eventBusIndex : 'com.example.myapp.MyEventBusIndex' ]
            }
        }
    }
}
 
dependencies {
    compile 'org.greenrobot:eventbus:3.0.0'
    annotationProcessor 'org.greenrobot:eventbus-annotation-processor:3.0.1'
}
```

这样就加入了EventBus的注解处理器,同时设置`eventBusIndex`参数来添加所有你想要生成index的`event`类



如果上述方法不起作用,你可以使用android-apt Gradle插件来添加EventBus注解处理器;

在build.gradle文件中添加以下snippet:

```groovy
buildscript {
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
 
apply plugin: 'com.neenbedankt.android-apt'
 
dependencies {
    compile 'org.greenrobot:eventbus:3.0.0'
    apt 'org.greenrobot:eventbus-annotation-processor:3.0.1'
}
 
apt {
    arguments {
        eventBusIndex "com.example.myapp.MyEventBusIndex"
    }
}
```

这样当你Rebuild你的工程之后,会生成所有带有`eventBusIndex`的类.然后你可以像这样设置EventBus:

```java
EventBus eventBus = EventBus.builder().addIndex(new MyEventBusIndex()).build();
```

或者你想用默认的实例:

```java
EventBus.builder().addIndex(new MyEventBusIndex()).installDefaultEventBus();
// Now the default instance uses the given index. Use it like this:
EventBus eventBus = EventBus.getDefault();
```

## ProGuard

**ProGuard**会混淆方法名和移除unused的方法(dead code removal)

因为`subscriber`方法不是直接被调用的,所以会被ProGuard移除.

所以如果你使用了ProGuard,那么就需要保留`subscriber`方法



添加下面的snippet到你的proguard.cfg文件:

```groovy
-keepattributes *Annotation*
-keepclassmembers class ** {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }
 
# Only required if you use AsyncExecutor
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
```

>  无论你有没有使用subscriber index,你都需要这样设置

## AsyncExecutor

**AsyncExecutor**就像一个线程池,但是能够处理异常.

**AsyncExecutor**能够wrap捕获的异常然后放入一个`event`里面发送出去.

> *Disclaimer: AsyncExecutor is a non-core utility class. It might save you some code with error handling in background threads, but it’s not a core EventBus class.*

通常,你可以用`AsyncExecutor.create()`方法创建一个跟`Application`相同生命周期的**AsyncExecutor**实例.

**AsyncExecutor**执行的不是`Runnable`而是`RunnableEx`,

能够抛出异常.

如果`RunnableEx`抛出了一个异常,它会被捕获并wrapped到一个

[ThrowableFailureEvent](http://greenrobot.org/files/eventbus/javadoc/3.0/org/greenrobot/eventbus/util/ThrowableFailureEvent.html)然后发送出去.



**AsyncExecutor**如何执行:

```java
AsyncExecutor.create().execute(
    new AsyncExecutor.RunnableEx() {
        @Override
        public void run() throws LoginException {
            // No need to catch any Exception (here: LoginException)
            remote.login();
            EventBus.getDefault().postSticky(new LoggedInEvent());
        }
    }
);
```

`subscriber`方法:

```java
@Subscribe(threadMode = ThreadMode.MAIN)
public void handleLoginEvent(LoggedInEvent event) {
    // do something
}
 
@Subscribe(threadMode = ThreadMode.MAIN)
public void handleFailureEvent(ThrowableFailureEvent event) {
    // do something
}
```

### AsyncExecutor Builder

使用`AsyncExecutor.builder()`方法来自定义**AsyncExecutor**

你可以定义:

- EventBus实例
- 线程池
- 失败的Event

你也可以自定义**AsyncExecutor**执行的生命周期,失败的`event`会带有`Context`的信息.

> 比如一个失败的`event`可能只对应某个特定的`context`

如果你定义的失败的`event`实现了[HasExecutionScope](http://greenrobot.org/files/eventbus/javadoc/3.0/org/greenrobot/eventbus/util/HasExecutionScope.html)接口,那么**AsyncExecutor**会自动设置`scope`.这样,你在失败的`event`的处理方法中可以根据不同的`scope`来做不同的响应;