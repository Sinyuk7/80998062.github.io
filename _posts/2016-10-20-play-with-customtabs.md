---
layout:     post
title:      "用一下Customtabs"
subtitle:   "Play with custom tabs"
date:       2016-10-20
author:     "Sinyuk"
header-img: "img/2016-10-20.jpg"
tags:
    - Android
    - Customtabs
    - WebView

---

## About

其实在国内customtabs的作用真的不大...因为没几个人在手机上用chrome的

一个是因为系统都是定制的不是原生的,一开始就内嵌了qq浏览器和UC等等,另一个就是Google都被墙了...

但是有时候还是要考虑那么一部分chrome用户的....

> 知乎就用了customtabs 当时第一次看到感觉知乎也开始注重用户体验了....之前感觉它是内容营销...客户端改的再烂也不该吐槽....


哦哦,插一句,还有就是Chrome早就可以定制Overview App Bar的颜色,超简单啊,就是在网站<head>中加一句

```html
<meta name="theme-color" content="#262a30">
```

然而没几个网站这么有心的....


![image](https://github.com/80998062/80998062.github.io/raw/master/img/in-post/post-play-with-customtabs/sample.jpg)


> 最一开始是在浏览V2EX的时候发现加了这个特性,后来知乎也有了,反正就是...挺喜欢这些细节的

## Features

- UI定制

- toolbar颜色
- Action button
- 菜单项
- 进出动画

- 导航通知:提供回调让app知道导航栏被点击

实现上面这些功能你这只要发送一个ACTION_VIEW的intent给浏览器就好了

- 性能优化:
- 后台预加载
- URL联想功能

实现这两个功能你的app需要绑定一个chrome的服务

## Usecase

改颜色和设置动画就不说了....

其他的我自己也说不清楚...分析下官方给的[Demo](https://github.com/GoogleChrome/custom-tabs-client) = = 哈哈哈


> 我们来看这个类好了:**[CustomTabActivityHelper](https://github.com/GoogleChrome/custom-tabs-client/blob/master/demos/src/main/java/org/chromium/customtabsdemos/CustomTabActivityHelper.java)**


首先我们看它implement了这个ServiceConnectionCallback,所以就需要override这两个方法

```java

@Override
public void onServiceConnected(CustomTabsClient client) {
    mClient = client;
    mClient.warmup(0L);
    if (mConnectionCallback != null) mConnectionCallback.onCustomTabsConnected();
}

@Override
public void onServiceDisconnected() {
    mClient = null;
    mCustomTabsSession = null;
    if (mConnectionCallback != null) mConnectionCallback.onCustomTabsDisconnected();
}

```

> 这个是在什么时候回调的呢?

当你通过`CustomTabsClient#bindCustomTabsService`绑定一个service的时候.

在CustomTabActivityHelper这个类里的话就是这个方法

```java

public void bindCustomTabsService(Activity activity) {
    if (mClient != null) return;

    String packageName = CustomTabsHelper.getPackageNameToUse(activity);
    if (packageName == null) return;

    mConnection = new ServiceConnection(this);
    CustomTabsClient.bindCustomTabsService(activity, packageName, mConnection);
}
```

这里注意这一句

```java
String packageName = CustomTabsHelper.getPackageNameToUse(activity);
```

因为可能你的手机里面不止一个浏览器支持custom tabs,
所以我们通过PackageManager把所有能支持ACTION_VIEW的intent的
,同时支持custom tabs的service调用的包名列出来

> 另外,为了一个更好的用户体验
>
> 比如说这是一个知乎的链接,我们就应该用知乎app打开它(如果安装了的话)
>
> 所以最好在跳转的时候判断一下Uri...如果是zhihu就调用知乎app


回到刚才的代码...`onServiceConnected()`这里

service连接成功之后,我们就用`warmup()`来在当前情景下启动Chrome


然后看:

```java

public CustomTabsSession getSession() {
    if (mClient == null) {
        mCustomTabsSession = null;
    } else if (mCustomTabsSession == null) {
        mCustomTabsSession = mClient.newSession(null);
    }
    return mCustomTabsSession;
}

```

这里会创建一个新的session,这个session用于api里面所有的请求

> 你也可以传入一个CustomTabsCallback在创建一个新的session的时候,然后就能知道一个页面有没有加载成功

你可以用session.mayLaunchUrl()进行预加载

关于这个CustomTabsCallback的两个回调,其实就相当于`WebViewClient.onPageStarted()`和`WebViewClient.onPageFinished()`,基本上...

然后其实我们一般想做的是如果能打开custom tabs就打开,打不开就用自己的webView

> 你想用UC和QQ浏览器的话就跳过....

那么我们可以自己实现一个callback:

```java

public class WebviewActivityFallback implements CustomTabActivityHelper.CustomTabFallback {
    @Override
    public void openUri(Activity activity, Uri uri) {
        WebViewActivity.open(activity, uri.toString());
    }
}

```

然后就可以打开自己的webView了...


然后贴上一个具体打开custom tabs的代码:

```java

private void handleUrl(String url) {
    Log.d(TAG, "handleUrl: getAuthority " + Uri.parse(url).getAuthority());
    if (Uri.parse(url).getAuthority().contains("zhihu.com")) {
        // 如果打开的是知乎的链接 www.zhihu.com/zhuanlan.zhihu.com
        Intent activityIntent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
        final PackageManager pm = getPackageManager();
        final List<ResolveInfo> resolvedActivityList = pm.queryIntentActivities(
                activityIntent, PackageManager.MATCH_DEFAULT_ONLY);
        for (int i = 0; i < resolvedActivityList.size(); i++) {
            if (resolvedActivityList.get(i).activityInfo.packageName.equals("com.zhihu.android")) {
                // 装了知乎
                startActivity(activityIntent);
                return;
            }
        }
    }

    if (customTabsBuilder == null) {
        customTabsBuilder = new CustomTabsIntent.Builder(customTabActivityHelper.getSession());
        customTabsBuilder.setToolbarColor(ContextCompat.getColor(this, R.color.colorPrimary));
        customTabsBuilder.addDefaultShareMenuItem();
    }

    CustomTabsIntent customTabsIntent = customTabsBuilder.build();
    CustomTabActivityHelper.openCustomTab(this, customTabsIntent, Uri.parse(url), new WebviewActivityFallback());
}

```

就是检查一下有没有比custom tabs更适合打开这个链接的app 比如这里的知乎

没有的话就用custom tabs

就这样....

这就是我怎么在椰壳日报里尝试用custom tabs的,具体例子可见:[YukDaily](https://github.com/80998062/YukDaily)