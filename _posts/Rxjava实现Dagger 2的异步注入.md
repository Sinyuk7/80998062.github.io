---
layout:     post
title:      "Rxjava实现Dagger 2的异步注入"
subtitle:   "给大神递🍵"
date:       2017-02-22
author:     "Sinyuk"
header-img: "img/in-post/dagger.png"
tags:
    - Dagger 2
    - Rxjava
---

> Async Injection in Dagger 2 with RxJava

[传送门](http://frogermcs.github.io/async-injection-in-dagger-2-with-rxjava/),稍微翻译一下[**MIROSLAW STANEK**](https://about.me/froger_mcs)的博客,他关于**Dagger 2**的系列文章写得超好啊🌹

这是讲使用**Rxjava**实现异步注入的,有兴趣的可以看看他的另外一篇文章:

[用**Producers**实现**Dagger 2**的异步依赖注入](https://medium.com/@froger_mcs/dependency-injection-with-dagger-2-producers-c424ddc60ba3)

不幸的是`Producers`不是为**Android**设计的,并且有一些缺陷:

- 依赖**Guava**(很快就会达到**64K**方法数和让编译变得更慢)
- 不是特别快(因为注入机制还是会阻塞UI线程)
- 不使用`@Inject`注解(那还是算了吧🙂)

安利的工具:[AndroidDevMetrics](https://github.com/frogermcs/AndroidDevMetrics) 

+ 检测视图加载的时间同时(如果你在用**Dagger 2**) 它也会显示构造每一个依赖对象所花费的时间.



但是我们有**Rxjava**💪

-------

### Async @Singleton injection

比如这里是我们要注入的**重量级**对象:

```java
@Provides
@Singleton
HeavyExternalLibrary provideHeavyExternalLibrary() {
    HeavyExternalLibrary heavyExternalLibrary = new HeavyExternalLibrary();
    heavyExternalLibrary.init(); //This method takes about 500ms
    return heavyExternalLibrary;
}
```

然后我们创建另外一个`provide()`方法,返回一个异步调用的`Observable<HeavyExternalLibrary>`对象:

```java
@Singleton
@Provides
Observable<HeavyExternalLibrary> provideHeavyExternalLibraryObservable(final Lazy<HeavyExternalLibrary> heavyExternalLibraryLazy) {
    return Observable.create(new Observable.OnSubscribe<HeavyExternalLibrary>() {
        @Override
        public void call(Subscriber<? super HeavyExternalLibrary> subscriber) {
            subscriber.onNext(heavyExternalLibraryLazy.get());
        }
    }).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());
}
```

来分析一下:

- 这里提供的不是`HeavyExternalLibrary`对象,而是一个`Observable`
- `Lazy<>`: 懒加载,如果直接初始化就失去了意义
- 通过`heavyExternalLibraryLazy.get()`在`Observable`被订阅的时候初始化,执行在`Schedulers.io()`,实现异步加载

你可以像平时一样注入这个`Observable`对象,真正需要的`heavyExternalLibrary`会在`onNext()`方法里返回:

```java
public class SplashActivity {

	@Inject
	Observable<HeavyExternalLibrary> heavyExternalLibraryObservable;

	//This will be injected asynchronously
	HeavyExternalLibrary heavyExternalLibrary; 

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate();
		//...
		heavyExternalLibraryObservable.subscribe(new SimpleObserver<HeavyExternalLibrary>() {
            @Override
            public void onNext(HeavyExternalLibrary heavyExternalLibrary) {
	            //Our dependency will be available from this moment
	            SplashActivity.this.heavyExternalLibrary = heavyExternalLibrary;
            }
        });
	}
}
```

### Interface Lazy<T>

> 懒注入?(好拗口) 
>
> 在第一次调用`get()`方法的时候才被注入(初始化),之后每一个调用`get()`方法都会返回一个相同的对象.

***Lazy != Singleton*** 

我们说的每次返回相同的对象是指对同一个`@Inject`使用`get()`方法

举个🌰:

```java
 @Module
   final class CounterModule {
     int next = 100;

      @Provides Integer provideInteger() {
       System.out.println("computing...");
       return next++;
     }
   }
```

```java
  final class LazyCounters {
      @Inject LazyCounter counter1;
      @Inject LazyCounter counter2;

     void print() {
       counter1.print();
       counter2.print();
     }
   }
```

输出:

```java
 printing...
   computing...
   100
   100
   100
   printing...
   computing...
   101
   101
   101
```

所以:只有`@Singleton`会注入同一个对象,`Lazy<T>`其实每次都会去初始化一个新的对象

### Async new instance injection

所以,当我们每次都要注入一个新的实例的时候,不仅仅是不能再用`@Singleton`的*scope*,而且`Lazy<>`对象也不再适用了

需要作出这样的改动:

```java
@Provides
HeavyExternalLibrary provideHeavyExternalLibrary() {
    HeavyExternalLibrary heavyExternalLibrary = new HeavyExternalLibrary();
    heavyExternalLibrary.init(); //This method takes about 500ms
    return heavyExternalLibrary;
}
```

`Observable<HeavyExternalLibrary> provide()`方法:

```java
@Singleton
@Provides
Observable<HeavyExternalLibrary> provideHeavyExternalLibraryObservable(final Provider<HeavyExternalLibrary> heavyExternalLibraryProvider) {
    return Observable.create(new Observable.OnSubscribe<HeavyExternalLibrary>() {
        @Override
        public void call(Subscriber<? super HeavyExternalLibrary> subscriber) {
            subscriber.onNext(heavyExternalLibraryProvider.get());
        }
    }).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());
}
```

### Complete async injection

我们可以用`Observable`进一步简化注入的过程:

假如我们这样注入:(参考[GithubClient](https://github.com/frogermcs/GithubClient/)的🌰)

```java
public class SplashActivity extends BaseActivity {

    @Inject
    SplashActivityPresenter presenter;
    @Inject
    AnalyticsManager analyticsManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    //This method is called in super.onCreate() method
    @Override
    protected void setupActivityComponent() {
        final SplashActivityComponent splashActivityComponent = GithubClientApplication.get(SplashActivity.this)
                .getAppComponent()
                .plus(new SplashActivityModule(SplashActivity.this));
        splashActivityComponent.inject(SplashActivity.this);
    }
}
```

使上述过程变成异步,我们只要用`Observable`封装`setupActivityComponent()`方法:

```java
@Override
protected void setupActivityComponent() {
    Observable.create(new Observable.OnSubscribe<Object>() {
        @Override
        public void call(Subscriber<? super Object> subscriber) {
            final SplashActivityComponent splashActivityComponent = GithubClientApplication.get(SplashActivity.this)
                    .getAppComponent()
                    .plus(new SplashActivityModule(SplashActivity.this));
            splashActivityComponent.inject(SplashActivity.this);
            subscriber.onCompleted();
        }
    })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new SimpleObserver<Object>() {
                @Override
                public void onCompleted() {
                    //Here is the moment when injection is done.
                    analyticsManager.logScreenView(getClass().getName());
                    presenter.callAnyMethod();
                }
            });
}
```

这样一来,依赖的初始化就不会阻塞UI线程了😄