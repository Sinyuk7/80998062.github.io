---
layout:     post
title:      "Retrofit with Rxjava"
subtitle:   "Retrofit和Rxjava的组合拳"
date:       2016-06-02
author:     "Sinyuk"
header-img: "img/2016-06-12.webp"
tags:
    - Android
    - Rxjava
    - Retrofit

---

#Retrofit with Rxjava

> Retrofit和Rxjava的组合拳

发现Rxjava真的蛮有意思的...可以不断发现各种操作符新的应用场景

有很多刚get的东西,这里先随便水一点,之后在补上


- Response处理

这里我们假装服务器返回的是常见的{status,data,code}类型的数据

然后我们根据这个写一个HttpResult<T>的类,里面有我们关心的`T data`

> 当然后来感觉好像不用这么做,被okhttp搞过的Response类已经有code,body之类的东西


然后我们统一处理Http的resultCode,并将HttpResult的Data部分剥离出来返回给subscriber

	public class HttpResultFunc<T> implements Func1<HttpResult<T>, T> {
	@Override
    public T call(HttpResult<T> httpResult) {
        if (httpResult.getCode() != 2001) {
            throw new ApiException(httpResult.getCode());
        	}
        	return httpResult.getData();
    	}
	}

这里我们自己定义了一个ApiException,构造函数里面传进去一个code,然后根据这个code你可以用switch语句返回你喜欢的localized message;

> 当然你也可以定义一个Transformer来转换Response和Data的Observable,在里面处理错误信息

不过...Transformer实际上就是一个Func1<Observable<T>, Observable<R>>,一毛一样的吧


- 线程处理

因为基本到最后都是一个io线程和一个主线程,所以我们可以封装在一个transformer里面

	<T> Transformer<T, T> applySchedulers() {  
		return observable -> observable.subscribeOn(Schedulers.io())
    	.observeOn(AndroidSchedulers.mainThread());
	}

> 当然要用`compose()`

- 错误过滤

其实上面已经讲了Response的错误处理了,但是如果code是200,而body部分有错误,这个时候我想跳过这个出错的东西(可能是解析出错上面的),而正常显示其他的东西...

> 那么,或许我可以

定义一个Result(Rxjava里面的)的集合

	public final class Results {
	private static final Func1<Result<?>, Boolean> SUCCESSFUL =
          result -> !result.isError() && result.response().isSuccessful();

		public static Func1<Result<?>, Boolean> isSuccessful() {
    		return SUCCESSFUL;
		}
		private Results() {
    	throw new AssertionError("No instances.");
		}
	}

最后我们在显示数据的时候就可以先过滤一下:

       resultObservable.filter(Results.isSuccessful()).subscribe(adpter) //

> 这里订阅了一个adapter

> adapter如果只关心`onNext()`发射的数据,继承一个Action1< T >就好了

> 然后就不用写什么`setData()`方法了,直接在里面实现`onNext(T data)`
