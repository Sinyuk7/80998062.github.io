---
layout:     post
title:      "Gradle Recipes 1"
subtitle:   "笔记笔记"
date:       2015-12-01
author:     "Sinyuk"
header-img: "img/2015-12-01.jpg"
tags:
    - Android
    - Gradle

---

#Gradle Recipes for Android 1

> 赶紧看完这本书


我们一直看到的一个术语:Gradle wrapper.

它能够让一个客户端在没有安装Gradle的前提下就能运行它.

通过gradle/wrapper目录下的gradle-wrapper.jar和gradlewrapper.
properties文件来执行这一过程.

gradle-wrapper.properties 可能是这样的

```groovy

#Mon Dec 28 10:00:20 PST 2015

distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-2.10-all.zip

```
- distributionUrl: 说明wrapper会下载和安装2.10版本的Gradle
- 在第一次运行之后,Gradle的部署文件会被缓存在zipStorePath路径下(位于zipStoreBase目录)
- 之后所有的编译用的都是缓存的版本

我们通过命令行来执行wrapper

也就是

- ./gradlew 在Unix上
- gradlew.bat 在Windows上


##加入依赖

我们在app目录的build.gradle文件里加入其它库

```groovy

dependencies {
	compile fileTree(include: ['*.jar'], dir: 'libs')
	testCompile 'junit:junit:4.12'
	compile 'com.android.support:appcompat-v7:23.3.0'
}

```

> 看完这本书估计我也就会一点Groovy了
> 很急很关键

- 一个依赖完整的语法是这样的

`testCompile group: 'junit', name: 'junit', version: '4.12'`

- 而我们用的一般都是简写

`testCompile 'junit:junit:4.12'`

- 当然你还可以把版本号这样写

`testCompile 'junit:junit:4.+'`

> 不推荐


- 文件和目录的依赖

```groovy

dependencies {
	compile files('libs/a.jar', 'libs/b.jar')
}

```

你可以运行这个任务来输出所有的依赖

`> ./gradlew androidDependencies`

```

:app:androidDependencies
debug
\--- com.android.support:appcompat-v7:23.3.0
+--- com.android.support:support-vector-drawable:23.3.0
| \--- com.android.support:support-v4:23.3.0
| \--- LOCAL: internal_impl-23.3.0.jar
+--- com.android.support:animated-vector-drawable:23.3.0
| \--- com.android.support:support-vector-drawable:23.3.0
| \--- com.android.support:support-v4:23.3.0
| \--- LOCAL: internal_impl-23.3.0.jar
\--- com.android.support:support-v4:23.3.0
\--- LOCAL: internal_impl-23.3.0.jar
debugAndroidTest
No dependencies

```

- 如果加了transitive标志,那么Gradle就不会下载这个库所需要的依赖

> 你需要手动添加

```groovy

dependencies {
	runtime group: 'com.squareup.retrofit2', name: 'retrofit', version: '2.0.1',
	transitive: false
}

```

- 同时你也可以用exclude和include关键字来手动删除和添加某些独立的库
- 
```grovvy

dependencies {
	androidTestCompile('org.spockframework:spock-core:1.0-groovy-2.4') {
		exclude group: 'org.codehaus.groovy'
		exclude group: 'junit'
	}
}

```

> 也就是说spock-core project这个项目不包括了Groovy和JUnit

- 加入module jar

```grovvy

dependencies {
	compile 'org.codehaus.groovy:groovy-all:2.4.4@jar'
	compile group: 'org.codehaus.groovy', name: 'groovy-all',version: '2.4.4', ext: 'jar'
}

```
下面是完整的语法,上面是简写;

> ext: for extension


![image](https://github.com/80998062/80998062.github.io/raw/master/img/in-post/2016-07-05/Gradle_Recipes_for_Android.jpg)


 - 当然你可以直接用Android Studio来管理依赖,那个更方便了
 - 然后你可以直接在那边搜索Maven上的库

> 那如果不是Maven上的呢


##Configuring Repositories


```
repositories {
	jcenter()
}

```

默认情况下,Android用的是JCenter和Maven Central仓库

你可以在repositories这个block里面添加你想下载东西的那个仓库 比如Jitpack

你也可以用一个本地的目录

```groovy

repositories {
	flatDir {
		dirs 'lib'
		}
}

```

