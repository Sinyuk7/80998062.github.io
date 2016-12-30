# 我只是小安卓😢

> 入坑一年 记一点流水账      



学Android有一年了吧,这一年下来,不知道踩了多少坑,年关了,记记流水账.



> 插图 github

前前后后想写过一些东西....结果都放弃了...

一些因为外力因素 一些是因为没什么意思...

写app大部分还是累死累活的在写轮子和 感觉提升不是很大

但是多看看别人的代码和文章有用



前后待了两个公司,都是小规模的创业公司

至今都还是几个人(现在是一个人)开发...

没体验过大规模项目的开发模式(插件化什么的

没怎么写过规范点的文档

没有什么code review

没有人来测试



以前我都不怎么喜欢用fragment,现在我开始在项目里面大量的使用fragment

在尝试过封装一些基类来复用之后

> 比如一个显示列表的`Fragment`基本就是传入一个ViewModel和一个Item的布局文件就可以了

发现虽然这样的做法虽然复用了大量代码,但是耦合度还是很高

每次产品经理一开口,我都感觉自己的代码是一摊🐶屎



之前自己写东西的时候还感觉不到MVP的好处 

所以现在在公司的项目里面没有用,现在想想真是后悔



>  感觉android开发真是难,相比iOS
>
> 每次我改死改活公司的iOS都只要随便搞搞



可能我能通过很好的封装来解决Activity/Fragment过重的问题,但是项目还是很难维护

尤其是当需求一直在改变,新功能不断在迭代

项目的历史负担越来越大(KPI压力倒是没用…感觉我们的用户都是iOS人群👅)

每隔一段时间看自己的代码,我都感觉是在屎山里面翻滚...

所以V/P分离貌似成为了一种必需...

尽管各种架构前期总有一些额外的工作,但是之后的开发会变得很轻松

在App体积不断增大的时候,让项目细分下来的codebase尽量小



然后想分享最近有关clean架构的一些尝试 然后提点问题(大家一起探讨下

> 给大佬递🍵



## 前言

最近了解并尝试了一下clean架构 发现它真的很给力

好后悔自己没有早点熟悉它并投入项目

Clean架构可以使你的代码有如下特性:

- 独立于架构
- 易于测试
- 独立于UI
- 独立于数据库
- 独立于任何外部类库


如果你不了解clean架构 可以先看一下篇文章:

https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html

关于MV

可以看看大帅的文章

> 马克宅卧槽

我讲的肯定也没有它好...

> 给大佬地插



## 开始

算了,少一点套路,我们直接来就这代码看看:

试着写了一个demo:

> 本来想做一个tubmlr样子的weibo, 写着写着发现渣浪对第三放开发者的限制太多了.
>
> 发现授个权还要认证,还要申请一个蓝V的微博…我感觉很难受

虽然只写了一点点,但是clean架构的样子感觉有了(我是这么觉得

> 害羞

所以就针对自己的项目来讲讲

这张图你肯定见过...

> 上图 



 其实更直观的,我们可以看一下3个module之间的依赖:

- data(数据层

```groovy
apply plugin: 'java'
def cfg = rootProject.ext;
dependencies {
    // ReactiveX
    compile cfg.dependencies["rxjava"]
    // Square
    compile cfg.dependencies["retrofit"]
    compile cfg.dependencies["converter-gson"]
    compile cfg.dependencies["adapter-rxjava"]
    compile cfg.dependencies["logging-interceptor"]
    compile cfg.dependencies["dagger"]
}
```



- usecase(用例层 当然你叫domain也行

```groovy
apply plugin: 'java'
dependencies {
    compile project(':data')
}
```

> data和usecase层都应该是java代码,跟android framework无关

- app(就是我们平时写的app module

```groovy
  dependencies {
        compile fileTree(include: ['*.jar'], dir: 'libs')
        // skip
        compile project(':usecase')
  }
```

你们感受一下:

然后就基本能理解用例层从数据层获取数据,然后在这里进行处理

> 当然我发现一般的项目根本没有什么业务逻辑可以在这里写的...

视图层从用例层获取数据然后显示在界面上



然后讲一下各种乱七八糟的东西

---

## Standing

**data层**

![http://west.cdn.gui-quan.com/media/upload/315e592927c8e8f62822e9e7cfcde5912d0793e5](data)



**entities:**

 实体类(最原始的数据,不该随着业务逻辑而改变



**wrapper:**

对实体类的一层封装,在这里你可能要自己加一些东西:比如:

- 处理timestamp,转换输出本地时间
- 一些额外的参数,比如数据边界的验证等等

> 当然这些你也可以放在View层的ViewModel里面
>
> 但是感觉放这里比较清晰

**remote:**

顾名思义嘛,这就是RemoteDataSource,在这里配置和封装了(对于我来说)OkHttp和Retrofit的接口

**utils:**

- 自定义了一些`transformer`,比如切换线程和全局的错误处理
- 网络请求的Interceptor
- 其他乱七八糟的







当时这样写了我就感觉特别酷炫(说出来别笑我...

因为可以在这里直接用一个java类来测试api,而不是编译整个项目...

每次叫我们python过来看logcat的时候 = = 他都会在旁边发牢骚: android编译怎么会这么慢....

>  卧槽我也不想啊

当然你会说用postman不就好了...

但是配置了OkHttp和Retrofit之后,就能有一个App运行时的环境嘛...



**usecase层**

> 用例层就比较简单了...因为一般的应用都没有什么业务逻辑

举个🌰

我要从一个从server端获取微博(status)显示到timeline列表上

首先我有一个`Usecase`的接口要继承(`<T>`是网络请求返回的数据类型

```java
public abstract class Usecase<T> {
    protected abstract Observable<T> buildObservable();
}
```

具体代码:

```java
public class GetTimelineUsecase extends Usecase<Timeline> {
    private final RemoteRepository mRepository;
    private final SchedulerTransformer<Timeline> mSchedulerTransformer;
    private int page = 1;
    @Named("uid")
    private long uid;
    @Named("timeline_type")
    private String timeline_type;

    @Inject
    GetTimelineUsecase(RemoteRepository repository,
                       @Named("io_main") SchedulerTransformer schedulerTransformer,
                       @Named("timeline_type") String timeline_type,
                       @Named("uid") long uid) {
        mRepository = repository;
        mSchedulerTransformer = schedulerTransformer;
        this.uid = uid;
        this.timeline_type = timeline_type;
    }
}
```

在构造函数里面我们注入了:

- 根据页面逻辑相关的参数(`timeline_type`,`uid`)

- 跟android framework相关的东西(比如` AndroidSchedulers.mainThread()`

  > 因为这一层是`apply plugin: 'java'`嘛~

  ​

然后我需要有:

- RemoteRepository(源,返回数据

- SchedulerTransformer(切换线程,比如IO和UI线程之间

  > 这里我直接注入一个transformer...当然你也可以注入不同的`thread`

  ​


- timeline_type,uid(接口的参数

  > 针对返回相同数据类型的接口 我们可以封装在一个用例里面




我在`buildObservable()`中根据`timeline_type`和`uid`请求不同的接口:

```java
   @Override
    protected Observable<Timeline> buildObservable() {
        switch (timeline_type) {
            case "home_timeline":
                return mRepository.home_timeline(page, feature)
                        .compose(mSchedulerTransformer);
            case "friends_timeline":
         	    skip;
            default:
                return Observable.error(new Throwable("Timeline type can't be null!"));
        }
```

然后在`GetTimelineUsecase`中会有一个`excute()`方法,我在这里处理业务逻辑:

```java
public Observable<Timeline> excute(final boolean refresh) {
    return buildObservable()
            .doOnSubscribe(new Action0() {
                @Override
                public void call() {
                    if (refresh) {
                        page = 1;
                    }
                }
            })
            .doOnNext(new Action1<Timeline>() {
                @Override
                public void call(Timeline timeline) {
                    if (0 != timeline.getNextCursor()) {
                        ++page;
                    }
                }
            });
}
```

然后你会问业务逻辑在哪里...

> 黑人问号???

我就说嘛,一般的项目没什么业务逻辑....

这里不是有一点点处理分页的逻辑(就这么多了

所以View层(或者说是你的Presenter)不用再关心分页,不用有什么page或者cursor

只要在那边一直调用`excute()`就好了

> 想到一个View层的设计原则:让View尽可能的笨拙和被动

虽然会传过来一个`refresh`的`Boolean`值哈,但是可以认为这只是用户操作(页面逻辑),和业务逻辑无关.



**PS:**

在这里还自定义了一个注解

```java
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD})
public @interface Nullable {
}
```

>  因为这个module没有`support-annotation`可以用

Dagger会要求如果你注入的参数是`null`,就必须用`@Nullable`注解,不然就会报错.

在一些地方我需要用到它,来传入一些默认值(比如null和0,因为可能这个请求参数用不到

另外,在注入`String` `Long`等类型的参数的时候,需要用`@Name`注解,不然编译器肯定不知道你需要的是什么



**View层**

View层其实没什么好讲的,就是Databinding+Dagger+MVP

**Dagger:**

**MVP:**

基本上是照着goggle写出来的 

有很多分析这几个项目的不错的文章,我也是看了好久才搞清楚的

这里就不展开了…具体的可以看看我的代码

> 链接呢



**DataBinding**

其实用了之后感觉没有一开始那么酷炫啦

双向绑定什么的,实际使用场景很少...

而且clean结构决定了View和真正的Model只会是单向的绑定



如果你还没用过DataBinding,赶紧尝试吧

其实google的文档写的不是很全面,最受用的还是大帅的博客:

> 机智



**ViewModel:**

既然是单向绑定,ViewModel扮演的角色就只是

- 暴露出需要的properties
- 让View实现它的接口

当然关于ViewModel的写法,讲道理应该是对data层的model的一次映射,

但是感觉这样要用手打好多代码...如果AS有插件(类似GsonFormat之类的)我可能会这样写...

所以一般我们直接`extends`或者`wrap`某个model就好了…再调用`getter`方法获得properties



**Tips**

其实用ViewModel的一个好处是可以很方便的封装一些方法.

比如图片加载库...你可能会觉得图片加载库还要封装,不是只要一句话:

```java
load(url).into(imageView);
```

对啊,但是可能你因为Glide的方法数太多,要换成Picasso,这下就麻烦了

但是当你把所有图片加载的场景(简单的...)都写在`@BindingAdapter`里面

举个🌰:

```java
    @BindingAdapter("avatar")
    public static void loadAvatar(ImageView view, String url) {
        Glide.with(view.getContext()).load(url).into(view);
    }

    @BindingAdapter("photo")
    public static void loadPhoto(ImageView view, String uri) {
        Glide.with(view.getContext()).load(Uri.parse(uri)).into(view);
    }
```

这样到时候更换一个库就很方便了



**Rxjava**

用`Subscriber`来代替`Callback`,在Presenter里面就全部都是从上往下的Rx的stream

比如一个列表加载数据

举个🌰

```java
	public void loadMore() {
        if (dataInTransit || reachBottom) return;
        mSubscriptions.add(mUsecase.execute(false) // boolean: needRefresh
                .doOnSubscribe(() -> dataInTransit = true)
                .doOnTerminate(() -> dataInTransit = false)
                .doOnSubscribe(mView::startLoading)
                .doOnTerminate(mView::stopLoading)
                .doOnError(mView::showError)
                .subscribe(timeline -> {
                    if (timeline.getNextCursor() == 0) {
                        reachBottom = true;
                        mView.showNoMore();
                    } else {
                        mView.setData(timeline.getStatuses(), false);
                    }
                }));
    }
```

写起来特别舒服~























关于其他的东西 我感觉真的没什么好讲的...

分享一些今年,说今年感觉太远了....反正就是陆陆续续看过的文章吧





关于EventBus

之前在用过一段时间EventBus之后一直很克制地使用它

因为感觉代码量一上去,各种Event会让逻辑变得很混乱

现在发现了适合发送消息的应该是一些全局事件,其他一些耦合度很高的可以直接用普通的Observable模式来实现.



比如说你删除一个评论,发送一个PostCommentEvent 那么就应该让

所有和comment有关的页面都订阅这个事件

我开始在一个Event类里面写注释:

备注下到底这个事件发送之后应该或者或者影响到哪些页面





我们太穷了 买不起服务器...

我开始在每一步要进行网络请求的时候都开始解决app的开销

我不再刷新界面

所见不一定是所得








