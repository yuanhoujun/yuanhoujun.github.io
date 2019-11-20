---
title: 理解Kotlin语言独有的位置注解，让注解控制更精准
date: 2019-11-15 11:42:45
tags:
- Kotlin
- 注解
categories:
- Kotlin
comments: true
---

> 在Kotlin语言编写的代码中，你应该看到过类似这样的注解`@file:JvmName(...)`，这有点难以理解，正常的注解不会存在类似`@file:`这样的前缀，在Java语言中也没有类似的语法。那么，这到底有什么作用呢？<!-- more --> 由于其特殊的作用，我把它称之为”位置注解“。

Kotlin语言是一门将语法简化到极致的编程语言，我们一起来看一段简单的代码：

```
class Person {
    var name: String? = null
}
```

这段极其简单的代码，经过Kotlin编译器的处理，等价于下面这段Java代码：

```
public final class Person {
   @Nullable
   private String name;

   @Nullable
   public final String getName() {
      return this.name;
   }

   public final void setName(@Nullable String var1) {
      this.name = var1;
   }
}
```

虽然在Kotlin语言中，看起来只是声明了一个成员变量，实际上编译后不仅声明了一个成员变量`name`，还生成了与之对应的`setter/getter`方法。

这个时候，问题来了，如果我们在`Person`类的`name`属性上方添加一个注解，会出现什么问题呢？

```
class Person {
   @Callable
   var name: String? = null
}
```

我们刚才说到，实际生成的字节码中包含了`setter/getter`方法，那么这个注解可能出现的位置就有4个地方：

* 属性（成员变量name）
* setter方法
* setter方法参数name
* getter方法

用代码来表示，具体可能出现的位置如下图所示：

```
public final class Person {
   // 位置一：属性
   @Callable
   @Nullable
   private String name;
	
   // 位置二：setter方法
   @Callable
   public final void setName(/*位置三：setter方法参数*/ @Nullable String var1) {
      this.name = var1;
   }
   
   // 位置四：getter方法
   @Callable
   @Nullable
   public final String getName() {
      return this.name;
   }
}
```

这个时候编译器晕菜了，它无法确定你到底想要让注解出现在什么位置。那么，这种情况下，Kotlin编译器究竟会怎么做呢？感兴趣的同学不妨自己做做实验。

那么，是否有办法使注解准确地出现在指定位置呢？答案是：当然有！位置注解恰好就是用来解决这个问题的。

我们将上面的代码添加位置注解，修改为下面这样：

```
class Person {
    @field:Callable
    var name: String? = null
}
```

通过添加位置注解`@field`，	`@Callable`注解将准确出现在属性定义的位置，如下所示：

```
public final class Person {
   // 注解将出现在这里
   @Callable
   @Nullable
   private String name;
	
   public final void setName(@Nullable String var1) {
      this.name = var1;
   }
   
   @Nullable
   public final String getName() {
      return this.name;
   }
}
```

对应上述其它三个位置的位置注解分别是：

* set: 对应setter方法位置
* get：对应getter方法位置
* setparam：对应setter方法参数位置

除此之外，Kotlin还提供了以下几个位置注解，对应其它不同使用场景：

### @file:
这个注解用在文件级别，每一个Kotlin文件对应一个或多个Java类，当对应一个类的时候，可通过添加该位置注解，结合上一节课讲到的注解`@JvmName`一起使用，可改变生成的Java类名。

你可以理解为这个注解实际作用的位置就是最终编译生成的Java类。

```
@file:JvmName("FooKt")
fun foo() {
    println("Hello, world...")
}
```

最终生成的代码类似下面这样，生成的类名恰好是注解上方所填写的名称：

```
@JvmName("FooKt")
public final class FooKt {
	public final void foo() {
		...
	}
}
```

### @param:
这个注解的作用是使注解出现的位置定位到构造函数的参数上面。

大家知道，在Kotlin语言中，如果在构造函数参数前面添加`var`或`val`关键词，在对应类中会生成相应的属性、setter、getter方法。

为了让注解准确地出现在其构造函数参数的位置，这个注解就应运而生了！

我们继续来看一个例子：

```
class Person(@param:Callable var name: String)
```

添加上述位置注解后，最终生成的注解就会出现在构造函数参数的位置，如下所示：

```
public final class Person {
   @NotNull
   private String name;

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final void setName(@NotNull String var1) {
      this.name = var1;
   }
	
   // 注解最终出现在了这里
   public Person(@Callable @NotNull String name) {
      super();
      this.name = name;
   }
}
```

### @property:
这是一个特殊的位置注解，这个注解对于Java端是不可见的，其代表的位置是对应属性的Property对象。这样说起来有点抽象，我们来看一个例子，先来了解一下`Property`到底是什么东西。

我们继续以`Person`类为例，通过下面一段代码去访问它：

```
fun main(args: Array<String>) {
    val person = Person("Scott")
    
    val propertyName = person::name
    // 这里将打印 name: falsee
    println("${propertyName.name}: ${propertyName.isConst}")
}
```

上述代码中，propertyName对应的就是`Person`类中`name`属性的Property实例，简单来说就是，Property保存了对应属性的相关信息，代表了当前属性。通过Property可以获取到当前属性的相关信息（包括变量的名称，是否常量，是否延迟初始化等等）。

如果在构造函数的`name`前面添加位置注解`@property:`，注解生成的位置会稍微有点难以理解。访问这个注解的唯一方法就是通过其Property实例，我们一起来试一下：


```
class Person(@property:Callable var name: String)
```

访问该注解的唯一方式是通过其Property实例，并且Java端无法访问到：

```
fun main(args: Array<String>) {
    val person = Person("Scott")

    val propertyName = person::name
    // 访问该注解的唯一方式
    println(propertyName.annotations.find { it.annotationClass == Callable::class })
}
```

那么，具体到字节码，该注解到底出现在了哪里呢？我们不妨来反编译看一看：

```
public final class Person {
   @NotNull
   private String name;
	
   // 注解出现在了这里，非常特殊的一个位置
   @Callable
   public static void name$annotations() {
   }

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final void setName(@NotNull String var1) {
      this.name = var1;
   }

   public Person(@NotNull String name) {
      super();
      this.name = name;
   }
}
```

可以看到注解出现在了Kotlin编译器生成的一个以属性名称加**$**与**annotations**后缀作为方法命名的静态方法上。这是Kotlin编译器约定的一个特殊方法，通过Property实例可以准确访问到这里。

而知道了这个约定命名方式之后，事实上Java端也可以通过特殊的方式来访问到该注解，严格来讲，Java无法访问并不准确。

### @receiver：
这也是一个非常特殊的位置注解，Kotlin支持扩展函数，即在不通过继承的情况下对原有类扩展函数或属性。扩展中有一个很重要的概念就是**receiver**，所谓的receiver，就是指被扩展类的实例本身。

但问题来了，扩展并不会改变原有类的代码，如何将注解放到**receiver**位置呢，这似乎是一个不可能完成的事情。

这就要说到扩展的实现原理了，扩展实际上对应Kotlin中的一个全局函数，当转换到字节码的时候，函数的第一个参数就是receiver本身。这样说起来可能比较抽象，我们直接来看一个例子：

我们先对`Person`类增加扩展函数`sayHi`: 

```
fun Person.sayHi(greet: String) {
    println("$greet, $name")
}
```

然后反编译查看最终得到的Java代码：

```
public static final void sayHi(@NotNull Person $receiver, @NotNull String greet) {
  String var2 = greet + ", " + $receiver.getName();
  System.out.println(var2);
}
```

可以看到Kotlin编译器生成了一个静态方法，静态方法的第一个参数就是`receiver`，对应扩展类实例本身，第二个参数是扩展函数实际的参数。

这就是Kotlin扩展的实现原理，其最终是通过增加静态函数来实现的，扩展函数的第一个参数永远指向被扩展类的实例，即receiver。而我们添加了位置注解`@receiver`之后，注解生成的位置就会出现在扩展函数第一个参数的位置，类似下面这样：

```
public static final void sayHi(@Callable @NotNull Person $receiver, @NotNull String greet) {
  String var2 = greet + ", " + $receiver.getName();
  System.out.println(var2);
}
```

这就是`@receiver`位置注解的作用，理解了扩展函数的原理，这个注解的作用就不难理解了。

### @delegate
这是今天我们要说的最后一个位置注解，这又是一个相对比较难理解的位置注解，因为在Java语言中并不存在类似的概念。在Kotlin语言中代理模式大行其道，Kotlin语言使用`by`关键字就可以轻松实现代理模式。

这里存在一个同样的问题，前面我们说过，在Kotlin类中声明一个属性实际会同时生成`setter/getter`方法，这样注解可能出现的位置除属性之外就是三处（setter/getter/setter参数)。而如果属性本身使用代理的方式生成，这里就多了一个位置：**代理类属性**的位置。

这样说，可能还不太直观，我们用官方的`lazy`实现来举一个例子。

我们在`Person`类中增加一个代理属性`gender`：

```
class Person(var name: String) {
    @delegate:Callable
    val gender by lazy { "male" }
}
```

老规矩，我们还是直接反编译得到Java代码再来分析：

```
public final class Person {
   // 注解出现在了这个位置
   // 也就是真正的代理类实例的位置
   @Callable
   @NotNull
   private final Lazy gender$delegate;

   @NotNull
   public final String getGender() {
      Lazy var1 = this.gender$delegate;
      return (String)var1.getValue();
   }

   public Person(@NotNull String name) {
      super();
      this.name = name;
      this.gender$delegate = LazyKt.lazy((Function0)null.INSTANCE);
   }
}
```

通过上面的代码，我们可以清晰地看到Kotlin编译器在类中生成真正的代理类实例属性，`gender`的值实际是从代理对象中获取的。这个位置注解的作用就是将注解精确地放置到代理类的实例属性上方。

### 默认位置注解优先级
位置注解在Kotlin语言中并不是强制要求的，我们可以不添加位置注解，在未添加位置注解的情况下，Kotlin语言会按照下面的优先级将注解放置到指定的位置（如果注解可以同时出现在多个位置的话）：

`param` > `property` > `field`

### 最佳实践
以上就是Kotlin语言中我们可以用到的所有位置注解，这是因为Kotlin语言将语法简化到了极致，我们才需要这些注解精确地告诉编译器需要将注解放置到哪里。如果你需要在代码中添加注解，应该始终记得增加位置注解，以便注解可以精确地放置到你想要放置的位置，避免出现一些不必要的麻烦。

阅读更多技术文章，请关注微信公众号”欧阳锋工作室“

![](http://youngfeng.com/assets/images/mpwexin.jpg)

参与Kotlin技术讨论，请添加唯一官方QQ交流群：329673958
