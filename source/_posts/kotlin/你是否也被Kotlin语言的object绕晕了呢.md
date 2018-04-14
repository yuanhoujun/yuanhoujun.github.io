title: 你是否也被Kotlin语言的object绕晕了呢
date: 2018/04/14 12:18
comments: true
tags:
- Kotlin
- object
- 编程语言
categories:
- Kotlin
- 基础知识
---

![文 | 欧阳锋](https://upload-images.jianshu.io/upload_images/703764-499d2f2f3e299b80.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

近日，在笔者的Kotlin语言交流群中。的确发现了一些同学对**object**的用法有一些疑问。于是，出现了下面这样错误的用法：
![](https://upload-images.jianshu.io/upload_images/703764-a0f15e2bb073278a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

很自然的想法，c是一个接口类型的成员变量，访问外部类的成员变量，这不是理所应当的吗？

即使查看Kotlin官方文档，也有这样一段描述：
>Sometimes we need to create an object of a slight modification of some class, without explicitly declaring a new subclass for it. Java handles this case with anonymous inner classes. Kotlin slightly generalizes this concept with object expressions and object declarations.

核心意思是：Kotlin使用object代替Java匿名内部类实现。

很明显，即便如此，这里的访问应该也是合情合理的。从匿名内部类中访问成员变量在Java语言中是完全允许的。

这个问题很有意思，解答这个我们需要生成Java字节码，再反编译成Java看看具体生成的代码是什么。

借助[JD-GUI](http://jd.benow.ca/)，我们可以看到下面的内容：

```
public final class Outer
{
  private String a;

  public static final class c
    implements Moveable
  {
    public static final c INSTANCE;
    
    static
    {
      c localc = new c();INSTANCE = localc;
    }
    
    public void move()
    {
      Moveable.DefaultImpls.move(this);
    }
  }
}
```

很有意思，我们在Kotlin类中object部分的代码最终变成了下面这个样子：

```
 public static final class c implements Moveable {
    public static final c INSTANCE;
    static {
      c localc = new c();INSTANCE = localc;
    }
    
    public void move() {
      Moveable.DefaultImpls.move(this);
    }
  }
```

这是一个静态内部类，很明显，静态内部类是不能访问外部类成员变量的。可是问题来了，说好的匿名内部类呢？

这里一定要注意，如果你只是这样声明了一个object，Kotlin认为你是需要一个静态内部类。而如果你用一个变量去接收object表达式，Kotlin认为你需要一个匿名内部类对象。

因此，这个类应该这样改进：

```
class Outer {
    private var a: String? = null
    
    // 用变量c去接收object表达式
    private val c = object: Moveable {
        override fun move() {
            super.move()
            // 改进后，这里访问正常
            println(a)
        }
    }
}
```

>为了避免出现这个问题，谨记一个原则：如果object只是声明，它代表一个静态内部类。如果用变量接收object表达式，它代表一个匿名内部类对象。

## object能干啥？
很自然地想到，Kotlin的object到底有什么作用。其实，从上文的表述来看。很明显，object至少有下面两个作用：
* 简化生成静态内部类
* 生成匿名内部类对象

其实，object还有一个非常重要的作用，就是生成单例对象。如果你需要在Kotlin语言中使用单例，非常简单，只需要使用object关键字即可。

```
object Singleton {
    fun f1() {
        
    }
    
    fun f2() {
        
    }
}
```

这种方式声明object和上面的方式略有区别，其最终会生成一个名为Singleton的类，并在类中生成一个静态代码块进行单例对象生成：

```
public final class Singleton
{
  public static final Singleton INSTANCE;
  
  public final void f1() {}
  
  public final void f2() {}
  
  static
  {
    Singleton localSingleton = new Singleton();INSTANCE = localSingleton;
  }
}
```

在Kotlin语言中对方法进行访问的时候最终其实是通过**INSTANCE**实例进行中转的。

在Kotlin语言中还有一个很常用的object叫做伴随对象。所谓的伴随对象只不过是名字叫做**Companion**的object而已。它主要用于类中生成类似Java的静态变量，Kotlin语言针对这个变量会认为你只是希望生成一个静态变量，而不希望引入多余的类。如果你是和Java语言混合开发的话，可以使用一个注解生成和Java语言静态变量完全一样的效果。

## 简单总结
Kotlin语言中使用object命名的方式的确容易让人误认为只要使用这个关键字就是生成了一个对象。而从上文的表述当中，你会发现，其实不同的使用姿势将产生不同的效果。因此，在日常使用中一定要学会随机应变。如果遇到了不明白的问题，不妨来看看这篇文章是否已经解答了你的问题。如果没有，请在文章下方留言告诉我。

## 欢迎加入Kotlin交流群
如果你也喜欢Kotlin语言，欢迎加入我的Kotlin交流群： 329673958 ，一起来参与Kotlin语言的推广工作。
