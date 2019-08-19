## Rxjava

参考：

最为经典的扔物线：给 Android 开发者的 RxJava 详解（[引用1](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html)， [引用2](http://gank.io/post/560e15be2dca930e00da1083 )） 以及[RxSamples](https://github.com/rengwuxian/RxJavaSamples#toc1)

- [RxJava 到底是什么](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html#toc_1)
- [RxJava 好在哪](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html#toc_2)
- [API 介绍和原理简析](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html#toc_3)
- [RxJava 的适用场景和使用方式](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html#toc_25)



入门篇：

[RxJava 与 Retrofit 结合的最佳实践](http://gank.io/post/56e80c2c677659311bed9841)，简单的网络请求过程，从Retrofit的使用到转换成Rxjava的使用



进阶篇：

[手写极简版的Rxjava](https://juejin.im/post/5cce6fb05188254177317fdc)





学习笔记：

1.Observable 与 Observer的关系如同 Button 与OnClickListener的关系 被观察者与观察者

2.RxJava 还内置了一个实现了 Observer 的抽象类：Subscriber，在 RxJava 的 subscribe 过程中，Observer 也总是会先被转换成一个 Subscriber 再使用，区别有俩点：新增了onStart、unsubscribe方法

3.map 一对一的转换关系 ，flatmap 



[Rxjava2](https://www.jianshu.com/p/0cd258eecf60)