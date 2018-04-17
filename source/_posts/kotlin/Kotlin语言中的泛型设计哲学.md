title: Kotlin语言中的泛型设计哲学
date: 2018/04/17 23:16
comments: true
tags:
- Kotlin
- 泛型
categories:
- Kotlin
- 基础知识
---

![文 | 欧阳锋](https://upload-images.jianshu.io/upload_images/703764-912c5c8fce46d69d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>Kotlin语言的泛型设计很有意思，但并不容易看懂。关于这个部分的官方文档，我反复看了好几次，终于弄明白Kotlin语言泛型设计的背后哲学。这篇文章将讲述Kotlin泛型设计的整个思考过程及其背后的哲学思想，希望可以解答你心中的疑问。不过，可以预见地，即使看完，你也未必完全明白这篇文章在说什么，但至少希望你通过这篇文章可以快速掌握Kotlin泛型的用法。

## Kotlin泛型的设计初衷
我们认为，Kotlin是一门比Java更优秀的JVM编程语言，Kotlin泛型设计的初衷就是为了解决Java泛型设计中一些不合理的问题。这样说可能不够直观，看下面这个例子：

```
 List<String> strs = new ArrayList<>();
// 这里将导致编译错误，Java语言不允许这样做
 List<Object> objs = strs;
```

很明显，String和Object之间存在着安全的隐式转换关系。存放字符串的集合应该可以自由转换为对象集合。这很合理，不是吗？

如果你这样认为的话，就错了！继续往下看，我们扩展这个程序：

```
List<String> strs = new ArrayList<>();
List<Object> objs = strs;
objs.add(1);

String s = strs.get(0);
```

很明显，这不合理！我们在第一个位置存入了整型数值1，却在取的时候将它当成了字符串。strs本身是一个字符串集合，用字符串接收读取的数据的逻辑是合理的。却因为错误的类型转换导致了不安全写入出现了运行时类型转换问题，因此，Java语言不允许我们这样做。

大多数情况下，这种限制没有问题。可是，在某些情况下，这并不合理。看下面的例子：

```
interface List<T> {
    void addAll(List<T> t);
}

public void copy(List<String> from, List<Object> to) {
   to.addAll(from);
}
```

这是一个类型绝对安全的操作，但在Java语言中这依然是不允许的。原因是，泛型是一个编译期特性，一旦指定，运行期类型就已经固定了。换而言之，泛型操作的类型是不可变的。这就意味着，List<String>并不是List<Object>的子类型。

为了允许正确执行上述操作，Java语言增加了神奇的通配符操作魔法。

```
interface List<T> {
  void addAll(List<? extends T> t);
}
```

**? extends T**意味着集合中允许添加的类型不仅仅是T还包括T的子类，但这个集合中可以添加的类型在集合参数传入addAll时就已经确定了。因此，这并不影响参数集合中可以存放的数据类型，它带来的一个直接影响就是addAll方法参数中终于可以传入泛型参数是T或者T的子类的集合了，即上面的copy方法将不再报错。

这很有意思，在使用通配符之前我们并不能传入类型参数为子类型的集合。使用通配符之后，居然可以了！这个特性在C#被称之为**协变**（covariant）。

**协变**这个词来源于类型之间的绑定。以集合为例，假设有两个集合L1、L2分别绑定数据类型F、C，并且F、C之间存在着父子关系，即F、C之间存在着一种安全的从**C->F**的隐式转换关系。那么，集合L1和L2之间是否也存在着**L2->L1**的转换关系呢？这就牵扯到了原始类型转换到绑定类型的集合之间的转换映射关系，我们称之为“可变性”。如果原始类型转换和绑定类型之间转换的方向相同，就称之为“协变”。

用一句话总结**协变**：如果绑定对象和原始对象之间存在着相同方向的转换关系，即称之为**协变**。

PS：以上关于**协变**的概念来自笔者的总结，更严谨的概念请参考[C#官方文档](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/covariance-contravariance/index)。

文章开头我们将不可变泛型通过通配符使其成为了可变泛型参数，现在我们知道这种行为叫做**协变**。很明显，**协变**转换中写入是不安全的。因此，**协变**行为仅仅用于读取。如果需要写入怎么办呢？这就牵扯到了另外一个概念**逆变**（contravariance）。

**逆变**与**协变**恰恰相反，即如果F、C之间存在着父子转换关系，L1、L2之间存在着从**L1->L2**的转换关系。其绑定对象的转换关系与原始对象的转换关系恰好相反。Java语言使用关键字super（？super List）实现**逆变**。

举个例子：假设有一个集合List<? super String>，你将可以安全地使用add(String)或set(Int，String)方法。但你不能通过get(Int)返回String对象，因为你无法确定返回的对象是否是String类型，你最终只能得到Object。

因此，我们认为，**逆变**可以安全地写入数据，但并不能安全地读取，即最终不能获取具体的对象数据类型。

为了简化理解，我们引入官方文档中 [Joshua Bloch](https://baike.baidu.com/item/Josh%20Bloch) 说的一句话：
>Joshua Bloch calls those objects you only read from Producers, and those you only write to Consumers. He recommends: "For maximum flexibility, use wildcard types on input parameters that represent producers or consumers"

Joshua Bloch是Java集合框架的创始人，他把那些只能读取的对象叫做生产者；只能写入的对象叫做消费者。为了保证最大灵活性，他推荐在那些代表了生产者和消费者的输入参数上使用通配符指定泛型。

相对于Java的通配符，Kotlin语言针对**协变**和**逆变**引入两个新的关键词**out**和**in**。

**out**用于**协变**，是只读的，属于生产者，即用在方法的返回值位置。而**in**用于**逆变**，是只写的，属于消费者，即用在方法的参数位置。

用英文简记为：**POCI** = Producer Out , Consumer In。

如果一个类中只有生产者，我们就可以在类头使用out声明该类是对泛型参数T**协变**的：

```
interface Link<out T> {
    fun node(): T
}
```

同样地，如果一个类中只有消费者，我们就可以在类头使用in声明该类是对泛型参数T**逆变**的：

```
interface Repo<in T> {
    fun add(t: T)
}
```

**out**等价于Java端的**? extends List**通配符，而**in**等价于Java端的**? super List**通配符。因此，类似下面的转换是合理的：

```
interface Link<out T> {
    fun node(): T
}

fun f1(linkStr: Link<String>) {
    // 这是一个合理的协变转换
    val linkAny: Link<Any> = linkStr
}

interface Repo<in T> {
    fun add(t: T)
}

fun f2(repoAny: Repo<Any>) {
    // 这是一个合理的逆变转换
    val repoStr: Repo<String> = repoAny
}
```

## 小结：协变和逆变
**协变**和**逆变**对于Java程序员来说是一个全新的概念，为了便于理解，我用一个表格做一个简单的总结：

-|协变|逆变
:---:|:---:|:---:
关键字|out|in
读写|只读|可写
位置|返回值|参数
角色|生产者|消费者

## 类型投影
在上面的例子中，我们直接在类体声明了泛型参数的协变或逆变类型。在这种情况下，就严格限制了该类中只允许出现该泛型参数的消费者或者生产者。很显然，这种场景并不多见，大多数情况下，一个类中既存在着消费者又存在着生产者。为了适应这种场景，我们可以将协变或逆变声明写在方法参数中。Kotlin官方将这种方式叫做 [类型投影（Type Projection）](https://kotlinlang.org/docs/reference/generics.html#use-site-variance-type-projections)。

这里我们直接使用官方文档的例子：

```
class Array<T>(val size: Int) {
    fun get(index: Int): T { /* ... */ }
    fun set(index: Int, value: T) { /* ... */ }
}

fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}

val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" } 

// 由于泛型参数的不变性，这里将出现问题
copy(ints, any) 
```

很明显，我们希望from参数可以接收元素为Any或其子类的任意元素，但我们并不希望修改from，以防止出现类似文章开头的问题。因此，我们可以在from参数中添加out修饰，使其协变：

```
fun copy(from: Array<out Any>, to: Array<Any>) {
}
```

一旦添加out修饰符，你就会发现，当你尝试调用set方法的时候，编译器将会提示你在out修饰的情况下禁止调用该方法。

注：Java语言在使用”协变“的情况下，from参数依然可以调用set方法。从这里可以看出，Kotlin语言在泛型安全控制上比Java更加精细。

## 星号投影
除了上述明确的类型投影方式之外，还有一种非常特殊的投影方式，称之为星号投影（star projection）。

在某些情况下，我们并不知道具体的类型参数信息。为了适应这种情况，Java语言中我们会直接忽略掉类型参数：

```
class Box<T> {
     public void unPack(T t) {
          ...
     }
}

// 在不确定类型参数的情况下，我们会这样做
Box box = new Box();
```

在Kotlin语言中，我们使用星号对这种情况进行处理。因为，Kotlin针对泛型有严格的读写区分。同样地，使用*号将限制泛型接口的读写操作：
* `Foo<out T: TUpper>`，这种情况下，T是协变类型参数，上边界是TUpper。Foo<\*>等价于Foo<out TUpper>，这意味着你可以安全地从Foo<\*>读取TUpper类型。
* `Foo<in T>`，在这种情况下，T是逆变类型参数，下边界是T。Foo<\*>等价于Foo<in Nothing>，这意味着在T未知的情况下，你将无法安全写入Foo<\*>。
* `Foo<T: TUpper>`，在这种情况下，T是不可变的。Foo<\*>等价于你可以使用Foo<out TUpper>安全读取值，写入等价于Foo<in Nothing>，即无法安全写入。

## 泛型约束
在泛型约束的控制上，Kotlin语言相对于Java也技高一筹。在大多数情况下，泛型约束需要指定一个上边界。这同Java一样，Kotlin使用冒号代替extends：

```
fun <T: Animal> catch(t: T) {}
```

在使用Java的时候，经常碰到这样一个需求。我希望泛型参数可以约束必须同时实现两个接口，但遗憾的是Java语言并没有给予支持。令人惊喜的是，Kotlin语言对这种场景给出了自己的实现：

```
fun <T> swap(first: List<T>, second: List<T>) where T: CharSequence, 
                                                    T: Comparable<T> {
    
} 
```

可以看到，Kotlin语言使用where关键字控制泛型约束存在多个上边界的情况，此处应该给Kotlin鼓掌。

## 总结
Kotlin语言使用**协变**和**逆变**来规范可变泛型操作，out关键字用于协变，代表生产者。in关键字用于逆变，代表消费者。out和in同样可以用于方法参数的泛型声明中，这称之为类型投影。在针对泛型类型约束的处理上，Kotlin增加了多个上边界的支持。

Kotlin语言最初是希望成为一门编译速度比Scala更快的JVM编程语言！为了更好地设计泛型，我们看到它从C#中引入了**协变**和**逆变**的概念。这一次，我想，它至少同时站在了Scala和C#的肩膀上。

## 欢迎加入Kotlin交流群
如果你也喜欢Kotlin语言，欢迎加入我的Kotlin交流群： 329673958 ，一起来参与Kotlin语言的推广工作。


