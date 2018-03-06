title: Android两行代码实现仿微信滑动返回效果
date: 2018/3/6 09:57
comments: true
tags:
- Android
- 滑动返回
- 开源
categories:
- 开源
---

![](http://upload-images.jianshu.io/upload_images/703764-017d5942c296de5b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>iPhone滑动关闭页面是一个非常讨喜的设计。滑动关闭可以让你聚焦屏幕内容，而不需要因为返回突然切换思维到屏幕下方寻找返回按钮。事实上，在使用Android手机的时候，我经常这样做。原因是，Android不同机型的返回按钮位置不一样。以至于在更换机型后我常常找不到返回按钮，需要一段时间的适应期。而滑动关闭就可以有效地避免这个问题，目前已经有很多类型的Android应用开始支持滑动关闭，比如你熟悉的微信、快手等都已经支持了滑动返回效果。使用 [Snake](https://github.com/yuanhoujun/Android_Slide_To_Close) 框架你只需要两行代码就可以搞定滑动关闭集成...

如果你还不知道Snake是什么，请关注简书下面的文章：
* [Snake 让你轻松实现类似iOS滑动关闭功能](https://www.jianshu.com/p/3619d65739b4)
* [Snake版本升级到0.0.5啦](https://www.jianshu.com/p/f7d5e422fa79)
* [将滑动关闭进行到底](https://www.jianshu.com/p/7cf6864c9bde)
* [Snake版本再升级，支持类iPhone X上滑退出到桌面功能](https://www.jianshu.com/p/71c27a671500)

![Snake](http://upload-images.jianshu.io/upload_images/703764-44c47eb5362b573c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 初体验
如果你需要在Activity中实现滑动关闭效果，使用如下两个步骤即可：
* 在你的Application中对Snake进行初始化：`Snake.init(this)`
* 在你的Activity类的onCreate方法中对其进行托管：`Snake.host(this)`

以上两个方法已经完成了Activity滑动关闭集成，为了开启滑动关闭功能，你还需要在Activity类顶部添加`@EnableDragToClose`注解

## Snake设计思路
为了保证Snake框架尽可能灵活，我使用了注解实现单页固定滑动参数配置。而全局配置则使用单独的**snake.xml**文件进行配置。同时，为了支持动态关闭和开启，在Snake类中提供了相关API用于动态控制滑动关闭和开启。

## 设计目标
看过Snake官方文档的同学会发现，Snake并不提供左滑关闭或者其它方向关闭页面的设置，Snake也没有提供不同的关闭效果设置。没有这样设计的原因很简单，因为这种关闭效果并不常见，这样的设计不过是哗众取宠，浪费时间，且增加使用难度。

我的目标是：尽可能简化Snake设计，仅提供必要API，且专注于滑动关闭效果实现。

## 新版本来了
这是本篇文章的重点，昨天，[Snake 0.3.0](https://github.com/yuanhoujun/Android_Slide_To_Close/blob/develop/docs/update_log_0.3.0.md) 版本已经发布了。

0.3.0版本主要针对Fragment提供了继承方式集成：
##### 使用方法
按照下面的对应关系，改变你的Fragment父类就可以完成滑动关闭集成:
* `android.app.Fragment` => `com.youngfeng.snake.app.Fragment`
* `android.support.v4.app.Fragment` => `com.youngfeng.snake.support.v4.app.Fragment`

**注意：使用继承方式集成的情况下，原来的API完全可以通用。你可以选择使用Snake的API进行滑动控制，也可以使用父类中的方法进行滑动控制，这取决于你自己。甚至实例创建你依然可以交给newProxy/newProxySupport接口。**

详细信息，请查看官方文档：[https://github.com/yuanhoujun/Android_Slide_To_Close](https://github.com/yuanhoujun/Android_Slide_To_Close)

## 交流群
QQ群：288177681
如果你在使用Snake的过程中，遇到任何问题，请使用QQ群联系我。

---
我是 [欧阳锋](https://www.jianshu.com/p/0c5d14fbaf1a)，开源的道路上，我与你同行。
