---
layout:     post
title:      "Some DesignTime Attributes"
subtitle:   "Tricks"
date:       2016-11-03
author:     "Sinyuk"
header-img: "img/2016-11-03.jpg"
tags:
    - Android
    - Tools

---


# Some DesignTime Attributes

> 黑魔法来了

- 用法(肯定都知道啦....
 
```xml

<layout xmlns:tools="http://schemas.android.com/tools"></layout>

```

> 然后先说一些常见的:


## tools:igonre


```xml

<string name="show_all_apps" tools:ignore="MissingTranslation">All</string>>

```

> 用于:Lint


## tools:targetApi

确定你的Api level(可以是Integer或者是名字

```xml

<GridLayout tools:targetApi="ICE_CREAM_SANDWICH" >

```

> 用于:Lint



## tools:local

作为xml的根元素,指定语言.可以对非English的文件关闭拼写检查

```xml

<resources xmlns:tools="http://schemas.android.com/tools" tools:locale="es">

```

> 用于:Lint,Studio



## tools:context

root element 这样在设计时编辑器就会为该布局设置context所用的主题

```xml

<android.support.v7.widget.GridLayout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools"
    tools:context=".MainActivity" ... >

```

> 用于:Layout Editor,Lint


## tools:layout

用在<fragment>标签里,可以在设计时就预览界面

```xml

<fragment android:name="com.example.master.ItemListFragment" tools:layout="@android:layout/list_content" />

```

> 用于:Layout Editors


## tools:listitem / listheader / listfooter

这几个属性可以用在<ListView>标签里(包括其他AdapterView 比如<GridView> <ExpandableListView> 等等),来确定list items的布局.在预览的时候这个tool会填充一下假的数据作为内容.

```xml

<ListView
    android:id="@android:id/list"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:listitem="@android:layout/simple_list_item_2" />

```

> 用于:Layout Editors


## tools:showIn

根元素.用在一个**被**include的布局里.在设计时它父布局也会被渲染在这个界面


```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:text="@string/hello_world"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:showIn="@layout/activity_main" />
```

> 用于:Layout Editors



## tools:menu

根元素.用来确认显示在ActionBar上的menu.当然AS也会通过tools:context中的上下文来找到`onCreateMenu()`方法从而渲染出menu.

但是tools:menu的优先于它,它后面跟的是用逗号隔开的一串id(但是没有@/id等前缀)

> 当然你可以用menu.xml的文件名(但是不需要.xml的拓展名


```xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:menu="menu1,menu2" />

```

当然对于Recyclerview,还有tools:layoutManager等属性


> 用于:Layout Editors

## More

还有一些`tools:shrink`,`tools:discard`等等都是用在Gradle plugin上的,感觉一般用不到,这里不提了...