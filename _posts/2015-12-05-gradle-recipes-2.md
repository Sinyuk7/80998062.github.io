---
layout:     post
title:      "Gradle Recipes 2"
subtitle:   "笔记笔记"
date:       2015-12-05
author:     "Sinyuk"
header-img: "img/2015-12-05.jpg"
tags:
    - Android
    - Gradle

---

#Gradle Recipes for Android 2

> 很关键这一张


##设置你的工程

- 我们可以在build.gradle文件里面自定义一些属性,用`ext`关键词

`ext`就是extra嘛

> 讲道理记这种东西都是知道它的全称之后就好记多了


> 举个栗子

```groovy

ext {
	def AAVersion = '4.0-SNAPSHOT' // change this to your desired version
}
dependencies {
	apt "org.androidannotations:androidannotations:$AAVersion"
	compile "org.androidannotations:androidannotations-api:$AAVersion"
}

```

- 如果你不想写在build.gradle文件里面,还有gradle.properties文件

比如你可以在里面:

```groovy

username=user_from_gradle_properties
password=pass_from_gradle_properties

```

然后你可以在build文件里面你可以这样写:


```groovy

ext {
	if (!project.hasProperty('user')) { // 1
		user = 'user_from_build_file'
	}
	if (!project.hasProperty('pass')) { // 1
		pass = 'pass_from_build_file'
	}
}

task printProperties() { // 2
	doLast {
		println "username=$user"
		println "password=$pass"
	}
}

```

1. 检查是否有这个属性
2. 执行一个打印的任务

```
> ./gradlew printProperties
:app:printProperties
username=user_from_build_file
password=pass_from_build_file

```

当然你也可以用命令行来插入属性,使用-P标志

```

> ./gradlew -Puser=user_from_pflag -Ppass=pass_from_pflag printProperties
:app:printProperties
username=user_from_pflag
password=pass_from_pflag

```


---

所以,我们有三种方式来设置属性

1. ext block
2. properties文件
3. 命令行



> 举个栗子

我们可以把sdk的version,support library的version什么的全部写在top-level的build.gradle文件里面

这样就可以用到所有的module上面


```groovy

ext {

    compileSdkVersion = 24
    buildToolsVersion = "24.0.3"
    applicationId = "com.sinyuk.yukdaily"
    minSdkVersion = 16
    targetSdkVersion = 24
    versionCode = 1
    versionName = "1.0"

    supportVersion = '24.2.1'

    dependencies = [
            "appcompat-v7"             : "com.android.support:appcompat-v7:${supportVersion}",
            "support-design"           : "com.android.support:design:${supportVersion}"
    ]
}

```

然后你可以这样用:

```groovy

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
   
    compile rootProject.ext.dependencies["appcompat-v7"]
    compile rootProject.ext.dependencies["support-design"]
}

```

当然当然 你可以新建一个比如说config.gradle

算了....不推荐这个方法 = =


##更新Gradle

有2个方法

- 用一个新的wrapper

你可以通过命令行修改版本号

```shell

> ./gradlew wrapper --gradle-version 2.12
:wrapper
BUILD SUCCESSFUL
Total time: ... sec

```

也可以显式的把wrapper的任务加入top-level的build文件里面

```groovy

task wrapper(type: Wrapper) {
	gradleVersion = 2.12
}

```

当你先有的wrapper版本太低了,Android Studio就不会和build文件同步了.也就是跑不了任何task


- 直接修改properties文件

除了命令行,wrapper的设置还取决于`gradle/wrapper`目录下的两个文件:

- gradle-wrapper.jar
- gradle-wrapper.properties

gradle-wrapper.properties文件会是这样的:

```groovy

#... date of most recent update ...
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-2.12-bin.zip

```

将distributionUrl里面的版本号换成你想要的就可以了


##共享设置

在top-level的build文件中:

> 默认情况下我们会有一个top-level的build文件和一个app module的build文件



```groovy

allprojects {
	repositories {
		jcenter()
	}
}

```

我们可以在`allprojects`语句块里面修改全局属性.

也就是说比如这里你设置了一个jcenter仓库,你在app module里面就不用重复`repositories`了


当然你也可以用`subprojects`语句块

比如说你所有的子工程都是Android库

那么你可以:

```groovy

subprojects {
	apply plugin: 'com.android.library'
}

```

- 附加题

如果你看一下`allprojects`的文档,会发现这个方法需要一个`org.gradle.api.Action`类型的参数

```groovy

void allprojects(Action<? super Project> action)

project.allprojects(new Action<Project>() {
	void execute(Project p) {
		// do whatever you like with the project
	}
});

```

这里的Action呢就像是Java8的Functional Interface一样,只有一个抽象的方法

比如说:

```java

@FunctionalInterface
public interface ITrade {
 	 public boolean check(Trade t);
}

```

你有一个ITrade的接口,然后你想用这个接口对一些实例进行检查

> 这里用了lambda表达式


你就可以:

```java

ITrade newTradeChecker = (Trade t) -> t.getStatus().equals("NEW");

// Or we could omit the input type setting:
ITrade newTradeChecker = (t) -> t.getStatus().equals("NEW")

```

然后我们转过头来看看gradle里面是怎么写的:

```groovy

allprojects {
	repositories {
		jcenter()
	}
}

```

是不是一样,`allprojects`方法会遍历所有的project,然后调用`repositories`方法,参数是`jcenter`


> 当然具体的可以看[Gradle源码
 ](https://github.com/gradle/gradle)
