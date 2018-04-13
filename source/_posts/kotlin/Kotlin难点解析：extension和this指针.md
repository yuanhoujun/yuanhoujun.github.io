title: Kotlin难点解析：extension和this指针
date: 2018/4/14 00:11
comments: true
tags:
- Kotlin
- 扩展
- this
categories:
- Kotlin
- 基础知识
---

>扩展（extension）是Kotlin语言中使用非常简单的一个特性。这篇文章并不是要讲解扩展的基本用法，而是解决在一些复杂场景中，扩展容易让人产生迷惑的一些问题。除了扩展，本篇文章还将讲解this指针在Kotlin语言中的基础用法。

![](https://upload-images.jianshu.io/upload_images/703764-119913eaea78702b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 扩展函数难点解析
大多数场景下，你都能轻松搞定Kotlin扩展。可是，看看下面这个题目，你还能脱口而出，告诉我答案是什么吗？

```
open class E {

}

open class E1: E() {

}

open class A {

    open fun E.f() {
        println("E.f in A")
    }

    open fun E1.f() {
        println("E1.f in A")
    }

    fun call(e: E) {
        e.f()
    }
}

class A1: A() {

    override fun E.f() {
        println("E.f in A1")
    }

    override fun E1.f() {
        println("E1.f in A1")
    }
}

fun main(args: Array<String>) {
    // a）
    A().call(E())

    // b）
    A1().call(E())

    // c）
    A().call(E())

    // d）
    A().call(E1())
}
```

问题：请告诉a，b，c，d位置代码执行的输出结果是什么？

对于这个问题，恐怕你在纸上写写画画半天也不一定能给出正确答案吧。关于这个问题，其实我之前的一篇文章 [ [Kotlin] Lambda and Extension](https://www.jianshu.com/p/d7a40c6a60d1) 中有提到过。可是，我认为这篇文章关于这部分的解释不够清晰，有必要再详细阐述一次。

Ok，let's started。

为了解决这个问题，官方提出了两个新的概念：**dispatch receiver**和**extension receiver**。

* dispatch receiver：中文翻译为**分发接收者**。所谓的分发接收者，就是声明这个扩展方法所在的类。即：在哪个类中声明，那个类就是你的分发接收者。
* extension receiver：中文翻译为**扩展接收者**。所谓的扩展接收者，就是你实际扩展的那个类。举个例子：你针对Int类扩展了一个方法add，这个add方法的扩展接收者就是Int类实例。

为了简化，这里我们将**dispatch receiver**简称为**DR**，将**extension receiver**简称为**ER**。

还记得多态的概念吗？多态是一种运行时概念，即对象的类型要等到运行时才能最终确定。因此，一些语言中也将多态叫做类型延迟加载。解决上面这个问题我们需要关注就是扩展函数是否会产生多态行为。

这里我们将产生多态行为的技术叫做**动态解析**，与之相反的行为称之为**静态解析**。

为了解决上面的问题，你需要记住下面这个规则：
* DR类型是动态解析的
* 与之相反，ER类型是静态解析的

先看上面例子的a、b部分，很显然：
* a代码中f函数的DR是类A，ER是类E
* b代码中f函数的DR是类A1，ER是类E

参照上面的规则，由于DR类型是动态解析的。在A1类中我们重写了E的扩展函数f，运行时最终会执行A1类中扩展的f方法。a部分很明显会输出A类中扩展的f方法。因此，最终的输出结果如下：

```
E.f in A
E.f in A1
```

继续看c、d部分，c、d部分的DR都是A，而对于ER，c、d分别是E、E1。参照上面的规则，ER是静态解析的。在call方法声明的地方，我们传入的对象类型是E，这就决定了无论扩展方法是来自E还是其子类，将始终执行E类的扩展方法。因此，c、d部分将输出同样的结果：

```
E.f in A
E.f in A
```
由此可见，如果你牢记上述两条规则，解决问题将变得非常容易。为了加强你的记忆，我用一个表格总结上面的知识点：

-|DR|ER
:---:|:---:|:---:
概念|扩展方法声明所在的类|声明扩展方法的类
解析方式|动态解析|静态解析

PS：由于新版本Kotlin中针对扩展函数也加入了override关键字，这非常有助于DR和ER的理解。如果你在使用Kotlin，强烈建议你更新到最新版本。

![](https://upload-images.jianshu.io/upload_images/703764-ab52dd83e6235da6.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 不太一样的this指针
在Java语言中，如果你在内部类中需要外部类的引用可以将this写在类名后面。可是，试试看Kotlin，果断不行。

为了获得外部类的引用，Kotlin语言引入了@符号。举个例子：

```
class Outer {
    inner class Inner {
        fun f() {
            println(this@Outer)
        }
    }
}
```

可以看到，为了获取外部类的引用，只需要在@后面接外部类的名称即可。

如果对应一个扩展函数，this引用指向是什么呢？先说答案，扩展函数中的this指针指向ER，即实际扩展的那个类对象。

```
fun Outer.foo() {
  println(this)
}
```

这里的this指向foo函数的接收者Outer类实例。

this指针还有一种场景是用在lambda表达式中，这是一种比较特殊的使用场景。lambda表达式本身没有任何接收者，如果是在全局声明一个lambda表达式，将不能使用this指针。而如果是在某个类或者扩展方法中使用this指针，将指向实际所在类或者扩展方法的接收者。

如果你习惯了Kotlin语言的这种表达方式，this指针的指向就不再是一个问题了。在你习惯这种用法之前，我用一个表格简单总结一下this指针的用法：

位置|指向
:---:|:---:
类中|默认指向当前类实例，使用@操作符指向具体外部类实例
扩展函数|默认指向扩展函数的接收者
lambda表达式|默认指向实际所在类实例或所在扩展函数的接收者

## 总结
关于扩展，大多数情况下，你不会遇到文章开头那种复杂的情况。如果遇到了这种情况，只要清楚地区分DR和ER，并牢记DR和ER的解析方式，就能轻松应对了。对于this指针，与Java语言不一样的地方是，为了引用具体类的实例，Kotlin语言使用@符号。个人认为，这种表述方式更自然。如果遇到某些比较复杂的情况，只需要弄清楚接收者，问题就引刃而解了。