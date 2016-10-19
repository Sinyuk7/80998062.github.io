---
layout:     post
title:      "Retrofit and Okhttp"
subtitle:   "随便写写"
date:       2016-05-27
author:     "Sinyuk"
header-img: "img/2016-05-27.webp"
tags:
    - Android
    - Retrofit
    - Okhttp

---

#Retrofit and Okhttp

之前都没用过Retrofit,发现是好方便啊...

> 想到我第一次用的是Volley,上传个东西是在是太麻烦了

水一点东西

##OAuth

关于授权的处理,基本就是用Interceptor了...

比如在项目里我定义了这么一个interceptor

> 其实好像okhttp里面本来就有一个authenticator 之后去看看

	@Singleton
	public final class OauthInterceptor implements Interceptor {
	    private Preference<String> mAccessToken;

	    public OauthInterceptor(@Token Preference<String> accessToken) {
	        Timber.tag("OauthInterceptor");
	        this.mAccessToken = accessToken;
	    }

	    @Override
	    public Response intercept(Chain chain) throws IOException {
	        Request.Builder builder = chain.request().newBuilder();
	        if (mAccessToken.isSet()) {
	            Timber.d("Add access token : %s", mAccessToken.get());
	            builder.header("Authorization", DribbleApi.ACCESS_TYPE + " " + mAccessToken.get());
	        } else {
	            Timber.d("Default access token ");
	            builder.header("Authorization", DribbleApi.ACCESS_TYPE + " " + BuildConfig.DRIBBBLE_CLIENT_ACCESS_TOKEN);
	        }

	        return chain.proceed(builder.build());
	    }
	}

这里就是当登录的时候看看sp里面有没有保存access token 没有的话就用游客的token

> 当然虽然我是这么写把access token保存在sp里面,不知道在实际开发里面有更好的方法没


##Service

一般的话我们都是要写两个service的

- 授权: 比如叫OauthService,
- 资源: 比如叫ApiService.

> 主要还是因为它们的end point不一样,一个可能是/aouth 一个可能是/api

其实就算一样了我也会写两个...因为这里用了Dagger,然后我得用两个okhttp client的单例.

因为一个要有缓存,一个没有

> 感觉授权的请求有了缓存总觉得会有问题


##缓存

这里直接上代码...免得暴露出我没什么web相关的知识

比如像缓存策略,是看了这篇文章才弄懂的

> [你应该了解的 一些web缓存相关的概念.](http://www.cnblogs.com/_franky/archive/2011/11/23/2260109.html)


        final Interceptor REWRITE_RESPONSE_INTERCEPTOR = chain -> {
            Response originalResponse = chain.proceed(chain.request());
            String cacheControl = originalResponse.header("Cache-Control");
            if (cacheControl == null || cacheControl.contains("no-store") || cacheControl.contains("no-cache") ||
                    cacheControl.contains("must-revalidate") || cacheControl.contains("max-age=0")) {
                return originalResponse.newBuilder()
                        .header("Cache-Control", "public, max-age=" + 0)
                        .build();
            } else {
                return originalResponse;
            }
        };

        final Interceptor OFFLINE_INTERCEPTOR = chain -> {
            Request request = chain.request();

            if (!NetWorkUtils.isNetworkConnection(application)) {
                Timber.d("Offline Rewriting Request");
                int maxStale = 60 * 60 * 24 * 3;
                request = request.newBuilder()
                        .header("Cache-Control", "public, only-if-cached, max-stale=" + maxStale)
                        .build();
            }
            return chain.proceed(request);
        };

        builder.cache(cache)
                .addNetworkInterceptor(REWRITE_RESPONSE_INTERCEPTOR)
                .addInterceptor(OFFLINE_INTERCEPTOR);

上面要注意的就是`addNetworkInterceptor()`和`addInterceptor()`的区别了

> = =我也不记得了,反正它github的wiki里面有的...

##TODO

然后想起一个东西,一直不知道Retrofit是怎么取消请求的,本来感觉取消一个请求是挺容易的,一般这个请求会有个TAG

然后取消就是了.但是加上Rxjava好像就不是那么一回事了....

之前好像看过一篇文章...找到了在补上
