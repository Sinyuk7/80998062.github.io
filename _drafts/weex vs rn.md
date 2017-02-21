![http://west.cdn.gui-quan.com/media/upload/bc29ac5a498cc6b03166d91d0569f4f4517202e2](http://west.cdn.gui-quan.com/media/upload/bc29ac5a498cc6b03166d91d0569f4f4517202e2)

weex 跟 rn 一样都是跨平台方案，但是 

- rn 讲究的是 *learn once, write anywhere*，虽然 大部分代码可以通用，但在写的时候不少module、接口还是有平台特殊性，需要区分 **Android** 和 **iOS**。
- rn 没有成熟的 web 方案，虽然有一些社区的，但并不成熟
- rn 在分 bundle、预加载、异步加载上做的并不友好，需要开发者自己做很多额外的事。
-  使用 rn 就必须使用 **Fresco**、**OkHttp** 这些依赖。
- rn 列表性能问题被诟病已久，且并没有任何打算加上的迹象。 

 对应地:

- weex 则强烈控制了一份代码运行三个平台，而且直接做了分bundle

异步加载这些事

- 图片加载也可以使用`adapter `方式，而不需要强行绑定 **Fresco**

   

> 那为什么吹rn的比吹weex的多呢

- 因为 rn 是大厂出的啊，本身推广也更好

weex的问题

- 不稳定，可能一个月不看，更新到新版本就跑不了了（最近的 **vue 2.0** 支持就导致 github 上大部分项目跑不了了）
- 对原生要求更高（rn 其实只要搭建起环境，就可以基本脱离 native 直接做前端开发了），不懂 native 是完全寸步难行的
- 文档差，官方文档层次不齐，有新的有老的，有一些东西甚至没有文档只能看代码
- 生态差，没有靠谱的开源项目参考 很多特性 **iOS** 早就有了，**Android** 版迟迟不出
- 代码风格、质量不行（自己看看 Android weex 源码就会发现了）