---
title: 正确使用Kotlin注解，兼容Java代码
date: 2019-11-12 12:40:50
tags:
- Kotlin
- Jvm
- 注解
categories:
- Kotlin
thumbnailImage: http://youngfeng.com/assets/images/kotlin/jvmanno/thumb.jpg
thumbnailImagePosition: bottom
autoThumbnailImage: yes
comments: true
---

> 大多数情况下，你不需要关注这个问题。但是，如果你的代码中包含了部分Java代码，理解这些注解将帮助你解决很多棘手问题。

<!-- more -->

产生这个问题的根本原因在于：Kotlin语言与Java语言的设计思路不同，部分特性属于Java语言独有，例如静态变量。部分特性属于Kotlin语言独有，例如逆变和协变。

> 为了抹平这些差异，Kotlin语言提供了一个绝佳的思路，通过添加注解可以改变Kotlin编译器生成的Java字节码，使之按照Java语言可以理解的方向进行，从而实现兼容。

**问题答疑：Kotlin语言与Java字节码有什么关系？为什么Kotlin编译器会生成Java字节码？**

不管是Kotlin语言还是Java语言都是建立在JVM平台上面的编程语言，其最终都需要编译成JVM可以识别的Java字节码才能被正确执行。这也是为什么Kotlin语言与Java可以完全互通的原因之一，不要将Java与Java平台混为一谈。

接下来我们看第一个注解，也是最常见的注解

### @JvmField
Kotlin编译器默认会将类中声明的成员变量编译成私有变量，Java语言要访问该变量必须通过其生成的getter方法。而使用上面的注解可以向Java暴露该变量，即使其访问变为公开（修饰符变为public)。

我们来做一个实验：

{% tabbed_codeblock @JvmField http://youngfeng.com %}
<!-- tab Kotlin -->
class Person {
    @JvmField
    var name: String? = null
}
<!-- endtab -->
  
<!-- tab Java -->
public class Client {

    public static void main(String[] args) {
        Person p = new Person();
        // 在添加@JvmField注解之前，这样访问会报错
        // 只能通过p.getName()的方式进行访问
        String name = p.name;
    }
}
<!-- endtab -->
{% endtabbed_codeblock %}

在Person类中我们定义了一个成员变量`name`，在添加`@JvmField`属性前我们试图通过`p.name`的方式进行访问，编译器会报错。因为，默认生成的成员变量`name`是私有的。而添加该注解之后我们却可以正常访问了。

由此可见，`@JvmField`注解的确使生成的字节码发生了变化，我们将字节码用Java语言的形式表示，具体发生的变化类似下面的代码展示：

{% tabbed_codeblock 添加@JvmField注解的变化 http://youngfeng.com %}
<!-- tab before -->
public final class Person {
   private String name;

   public final String getName() {
      return this.name;
   }

   public final void setName(@Nullable String var1) {
      this.name = var1;
   }
}
<!-- endtab -->
  
<!-- tab after -->
public final class Person {
   public String name;
}
<!-- endtab -->
{% endtabbed_codeblock %}

**注：before与after分别为添加注解前与添加之后**

以上场景是将`@JvmField`注解添加到普通变量上方，如果添加到伴随对象的成员变量上方，会发生什么呢？我们来试试看：

{% tabbed_codeblock 添加到伴随对象 http://youngfeng.com %}
<!-- tab Kotlin -->
class Person {
    var name: String? = null

    companion object {
        @JvmField
        val GENDER_MALE = 1
    }
}
<!-- endtab -->
  
<!-- tab Java -->
public static void main(String[] args) {
  // 未添加之前
  // int gender = Person.Companion.getGENDER_MALE();
  // 添加之后，可直接访问
   int gender = Person.GENDER_MALE;
   System.out.println(gender);
}
<!-- endtab -->
{% endtabbed_codeblock %}

同样地，添加注解之后我们可以通过点语法直接对其进行访问。

由此可见，`@JvmField`注解会使伴随对象在伴生类中生成静态成员变量，通过伴生类类名可直接对其进行访问。

### 结论
`@JvmField`注解可改变字节码的生成，其作用的目标是类成员变量或伴随对象成员变量。作用在类成员中可使该变量对外暴露，通过点语法直接访问。即将私有成员变量公有化（public），并去掉setter/getter方法。作用在伴随对象成员变量中，可以使该伴随对象中的变量生成在伴生对象中，成为伴生对象的公有静态成员变量，通过伴生类可直接访问。

那么问题来了，如果该注解作用在私有成员变量上方会发生什么呢？请大家自行做实验验证。


### @JvmStatic
这个注解与`@JvmField`非常容易出现混淆，两者都可以作用在伴随对象成员变量上方，我们来试试看，如果同样作用在伴随对象成员变量中，会出现什么情况。

添加`@JvmField`注解的效果，上面我们已经看到了，我们直接将注解修改为`@JvmStatic`试试看：

{% tabbed_codeblock 添加到伴随对象 http://youngfeng.com %}
<!-- tab Kotlin -->
class Person {
    var name: String? = null

    companion object {
        @JvmStatic
        val GENDER_MALE = 1
    }
}
<!-- endtab -->
  
<!-- tab Java -->
public static void main(String[] args) {   
    // 1) 这样访问报错
    int gender = Person.GENDER_MALE;
    // 2) 这样访问正常
    int gender = Person.Companion.getGENDER_MALE();
    // 3) 这样访问也正常
    int gender = Person.getGENDER_MALE();

    System.out.println(gender);
}
<!-- endtab -->
{% endtabbed_codeblock %}

切换到Java代码，你可以看到，我一共提供了三种访问方式。第一种访问方式是通过点语法直接访问，编译器报错，由此可见，`@JvmStatic`注解并没有在伴生类中生成静态的公有成员变量。第三种方式可以正常访问，证明该注解在伴生类中生成了静态的公有getter方法。第二种方式可以正常访问，证明该注解不会破坏伴随对象中原有成员的访问方式。

由此，我们可以大胆猜测，`@JvmStatic`注解的作用应该是生成静态的setter/getter方法，而不会改变属性（成员变量）的访问权限。

为了进一步验证我们的猜想，我们将`val`修改为`var`试试看。

```
public static void main(String[] args) {
    // 1) 这样访问报错
    int gender = Person.GENDER_MALE;
    // 2) 这样访问正常
    int gender = Person.Companion.getGENDER_MALE();
    // 3) 这样访问也正常
    int gender = Person.getGENDER_MALE();

    // 4) 以下访问正常
    Person.setGENDER_MALE(1);

    System.out.println(gender);
}
```

第四种方式调用正常，证明我们的猜测没有错，`@JvmStatic`仅会改变伴随对象或对象（object）中setter/getter方法的生成方式，而不会改变属性访问权限，这是与注解`@JvmField`的本质区别。

**注意：由于`@JvmField`不仅会改变属性的访问权限，同时也会改变setter/getter方法的生成，细心的同学应该已经注意到了。一旦添加了`@JvmField`注解，setter/getter方法也消失了（变量可以通过点语法直接访问，setter/getter方法也就没必要存在了）。而`@JvmStatic`仅仅是使setter/getter方法变为静态方法，同时生成位置放置到伴生类中。这与`@JvmField`的处理方式有些冲突（`@JvmField`会直接删除掉setter/getter方法）。为了避免冲突，Kotlin语言禁止将这两个注解混淆使用。**

以上是将`@JvmStatic`与`@JvmField`作用在伴随对象成员变量上的区别。实际上，`@JvmStatic`不仅可以修饰属性（成员变量），还可以修饰方法，修饰方法的作用与修饰属性的作用一致，都是将方法变成静态类型。

为了更直观地表示两种的区别，我们用一个表格完整展示两个注解的区别：

注解|作用位置|作用
:---:|:---:|:---:
`@JvmField`|类属性或对象属性|使属性修饰符成为public
`@JvmStatic`|对象方法（包括伴生对象）|使用方法成为静态类型，如果作用在伴生对象方法中，其方法会成为伴生类的静态方法


### @JvmName
这个注解可以改变字节码中生成的类名或方法名称，如果作用在顶级作用域（文件中），则会改变生成对应Java类的名称。如果作用在方法上，则会改变生成对应Java方法的名称。

{% tabbed_codeblock @JvmName http://youngfeng.com %}
<!-- tab Kotlin -->
@file:JvmName("FooKt")

@JvmName("foo1")
fun foo() {
    println("Hello, Jvm...")
}
<!-- endtab -->
  
<!-- tab Java -->
// 相当于下面的Java代码
public final class FooKt {
   public static final void foo1() {
      String var0 = "Hello, Jvm...";
      System.out.println(var0);
   }
}
<!-- endtab -->
{% endtabbed_codeblock %}

可以看到第一个注解`@file:JvmName("FooKt")`的作用是使生成的类名变为`FooKt`，第二个注解的作用是使生成的方法名称变为`foo1`。

**注意：该注解不能改变类中生成的属性（成员变量）的名称。**

这里的注解中，我们看到了一个特殊的前缀`@file:`，这个注解前缀是Kotlin语言特有的一种标识，其作用是标记该注解最终会作用在生成的字节码的具体位置（属性、setter、getter等），关于这个部分，大家可以先跳过，下一篇文章将给大家详细讲解。

### @JvmMultifileClass
说完了上面这个注解，就不得不提到`@JvmMultifileClass`这个注解，这个注解通常是与`@JvmName`结合使用的。其使用场景比较单一，看下面的例子：

{% tabbed_codeblock @JvmMultifileClass http://youngfeng.com %}
<!-- tab Util1 -->
@file:JvmName("Utils")

fun isEmpty(str: String?): Boolean {
    return null == str || str.length <= 0
}
<!-- endtab -->
  
<!-- tab Util2 -->
@file:JvmName("Utils")

fun isPhoneNumber(str: String): Boolean {
    return str.startsWith("1") && str.length == 11
}
<!-- endtab -->
{% endtabbed_codeblock %}

编译以上代码，Kotlin编译器会提示错误`Error:(1, 1) Kotlin: Duplicate JVM class name 'Utils' generated from: package-fragment, package-fragment`，即生成的类名出现了重复。可是，如果我们就是希望声明使用多个文件，但方法生成到同一个类中呢？`@JvmMultifileClass`就是为解决这个问题而生的。

我们在上面代码的基础上分别添加注解`@JvmMultifileClass`试试看:

{% tabbed_codeblock @JvmMultifileClass http://youngfeng.com %}
<!-- tab Util1 -->
@file:JvmName("Utils")
@file:JvmMultifileClas

fun isEmpty(str: String?): Boolean {
    return null == str || str.length <= 0
}
<!-- endtab -->
  
<!-- tab Util2 -->
@file:JvmName("Utils")
@file:JvmMultifileClass

fun isPhoneNumber(str: String): Boolean {
    return str.startsWith("1") && str.length == 11
}
<!-- endtab -->

<!-- tab Java -->
// 生成的代码相当于下面这段Java代码
public final class Utils {
   public static final boolean isEmpty(@Nullable String str) {
      return Utils__A1Kt.isEmpty(str);
   }

   public static final boolean isPhoneNumber(@NotNull String str) {
      return Utils__A2Kt.isPhoneNumber(str);
   }
}
<!-- endtab -->
{% endtabbed_codeblock %}

这个注解在处理多个文件声明，合并到一个类的场景中发挥着举足轻重的作用。如果你有这样的需求，一定要谨记这个注解。

### @JvmOverloads
由于Kotlin语言支持方法参数默认值，而实现类似功能Java需要使用方法重载来实现，这个注解就是为解决这个问题而生的，添加这个注解会自动生成重载方法。我们来试一下：

{% tabbed_codeblock @JvmOverloads http://youngfeng.com %}
<!-- tab Kotlin -->
@JvmOverloads
fun foo(x: Int, y: Int = 0, z: Int = 0): Int {
    return x + y + z
}
<!-- endtab -->
  
<!-- tab Java -->
// 生成的代码相当于下面这段Java代码
public static final int foo(int x, int y, int z) {
  return x + y + z;
}
   
public static final int foo(int x, int y) {
  return foo(x, y, 0);
}

public static final int foo(int x) {
  return foo(x, 0, 0);
}
<!-- endtab -->
{% endtabbed_codeblock %}

由此可见，通过这个注解可以影响带有参数默认值方法的生成，添加该注解将自动生成带有默认值参数数量的重载方法。这是一个非常有用的特性，方便Java端可以更高效地调用Kotlin端代码。

### @Throws
由于Kotlin语言不支持CE（Checked Exception），所谓CE，即方法可能抛出的异常是已知的。Java语言通过`throws`关键字在方法上声明CE。为了兼容这种写法，Kotlin语言新增了`@Throws`注解，该注解的接收一个可变参数，参数类型是多个异常的KClass实例。Kotlin编译器通过读取注解参数，在生成的字节码中自动添加CE声明。

为了便于理解，看一个简单的例子：

{% tabbed_codeblock @JvmOverloads http://youngfeng.com %}
<!-- tab Kotlin -->
@Throws(IllegalArgumentException::class)
fun div(x: Int, y: Int): Float {
    return x.toFloat() / y
}
<!-- endtab -->
  
<!-- tab Java -->
// 生成的代码相当于下面这段Java代码
public static final float div(int x, int y) throws IllegalArgumentException {
      return (float)x / (float)y;
}
<!-- endtab -->
{% endtabbed_codeblock %}

可以看到，添加了`@Throws(IllegalArgumentException::class)`注解后，在生成的方法签名上自动添加了可能抛出的异常声明（throws IllegalArgumentException），即CE。

这个注解在保证逻辑的严谨性方面非常有用，但如果你的工程中仅使用Kotlin代码，可以不用理会该注解。在Kotlin语言的设计哲学里面，CE被认为是一个错误的设计。

### @Synchronized
这个注解很容易理解，顾名思义，主要用于产生同步方法。Kotlin语言不支持`synchronized`关键字，处理类似Java语言的并发问题，Kotlin语言建议使用同步方法进行处理。

Kotlin团队认为同步的逻辑应该交给代码处理，而不应该在语言层面处理：

![](http://youngfeng.com/assets/images/kotlin/jvmanno/syn.png)

但为了兼容Java，Kotlin语言支持使用该注解让编译器自动生成同步方法：

{% tabbed_codeblock @Synchronized http://youngfeng.com %}
<!-- tab Kotlin -->
@Synchronized
fun start() {
    println("Start do something...")
}
<!-- endtab -->
  
<!-- tab Java -->
// 生成的代码相当于下面这段Java代码
public static final synchronized void start() {
  String var0 = "Start do something...";
  System.out.println(var0);
}
<!-- endtab -->
{% endtabbed_codeblock %}

### @JvmWildcard
这个注解主要用于处理泛型参数，这涉及到两个新的知识点：**逆变**与**协变**。由于Java语言不支持协变，为了保证安全地相互调用，可以通过在泛型参数声明的位置添加该注解使用Kotlin编译器生成通配符形式的泛型参数（`？extends ...`)。

看下面这段代码：

```
class Box<out T>(val value: T)

interface Base
class Derived : Base

fun boxDerived(value: Derived): Box<Derived> = Box(value)
fun unboxBase(box: Box<Base>): Base = box.value
```

按照正常思维，下面的两个方法转换到Java代码应该是这样：

```
Box<Derived> boxDerived(Derived value) { …… }
Base unboxBase(Box<Base> box) { …… }
```

但问题是，Kotlin泛型支持型变，在Kotlin中，我们可以这样写`unboxBase(Box(Derived()))`，而在Java语言中，泛型参数类型是不可变的，按照上面的写法显然已经做不到了。

正确转换到Java代码应该是这样：

```
Base unboxBase(Box<? extends Base> box) { …… }
```

为了使这样的转换正确生成，我们需要在泛型参数的位置添加上面的注解:

```
fun unboxBase(box: Box<@JvmWildcard Base>): Base = box.value
```

### @JvmSuppressWildcards
这个注解的作用与`@JvmWildcard`恰恰相反，它是用来抑制通配符泛型参数的生成，即在不需要型变泛型参数的情况下，我们可以通过添加这个注解来避免生成型变泛型参数。

{% tabbed_codeblock @JvmSuppressWildcards http://youngfeng.com %}
<!-- tab Kotlin -->
fun unboxBase(box: Box<@JvmSuppressWildcards Base>): Base = box.value
<!-- endtab -->
  
<!-- tab Java -->
// 生成的代码相当于下面这段Java代码
Base unboxBase(Box<Base> box) { …… }
<!-- endtab -->
{% endtabbed_codeblock %}

正确使用上述注解，可以抹平Kotlin与Java泛型处理的差异，避免出现安全转换问题。

### @Volatile @Transient
这两个注解恰好对应Java端的两个关键字`volatile`与`transient`，前者主要用于解决多线程脏数据问题，后者用于标记序列化对象中不参与序列化的属性。

这两个注解比较简单，就不举例说明了。在遇到类似需要与Java互通的场景时，只需要将其关键字替换为该注解即可。

以上就是我们日常开发过程中能够遇到的所有注解了，在Kotlin 1.3版本中，还增加了一个新的注解`@JvmDefault`用于在接口中处理默认实现的方法。接口中允许有默认实现是从JDK 1.8版本开始的，为了兼容低版本JDK，Kotlin语言新增了该注解用于生成兼容性字节码，但该注解目前仍处于实验阶段，名称或行为均可能发生改变，建议大家先不要使用，推荐大家始终使用JDK 1.8及其以上版本。

### 最佳实践
如果在工程中必须存在部分Java代码，为了实现完美调用，一定要谨慎并正确地使用上述注解。要充分理解Kotlin编译器与Java编译器生成的字节码差异。

如果是由于现存Java库仅兼容Java字节码，导致部分框架在遇到Kotlin语言生成的字节码时会出现解析错误，不能正常使用。这个时候要尝试检查是否需要通过上述注解矫正字节码的生成，使Java库能够正常使用。

如果是新工程，建议大家全部使用Kotlin代码，避免出现上述注解，减少阅读上的困难。目前，Kotlin版本已经非常稳定了，请大家放心使用。

阅读更多技术文章，请关注微信公众号”欧阳锋工作室“
![](http://youngfeng.com/assets/images/mpwexin.jpg)

参与Kotlin技术讨论，请添加唯一官方QQ交流群：329673958
