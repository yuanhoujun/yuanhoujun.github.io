title: Kotlin 代理模式
date: 2016/08/22 00:25
comments: true
tags: Android, Kotlin, 代理模式, 代理属性, 延迟加载
categories: Kotlin
---

代理模式是23种经典设计模式之一，代理模式被认为是继承的更好替代解决方案；因为代理比继承更加灵活，在Java语言中，通过反射可以实现动态代理，动态代理可以实现**AOP**编程，即：可以动态地往已有类中添加逻辑；比如：实现事务的自动提交，异常的自动捕获，热修复等等; 

在Kotlin语言中，代理模式是默认支持的，不需要任何额外的代码，你只需要记住一个关键字**by**。我们不妨来试一下:

```
interface Base {
    fun sayHi()
}

class BaseImpl : Base {
    override fun sayHi() {
        println("BaseImpl->sayHi")
    }
}
class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
    val b = BaseImpl()
    val derived = Derived(b)
    derived.sayHi()
}
```

这里Derived作为BaseImpl的代理类，拥有BaseImpl类中的所有方法，Derived将代理BaseImpl类执行BaseImpl类中的所有方法，就像继承自BaseImpl类一样。这样说起来有点抽象，来看一下Kotlin编译器具体为我们做了一些什么。但是，怎么看呢？教大家一个方法！
大家都知道，Kotlin和Java均是JVM语言，最终均转换到同样的Java字节码，这样我们就可以先将Kotlin编译为.class文件，再反编译为.java文件，看看对应的Java代码，我们就可以看到更多的细节。下面是最终反编译生成的Java代码:

```
public final class Derived implements Base {
  public Derived(@NotNull Base b) {
    this.$$delegate_0 = b;
  }
  public void sayHi() {
    this.$$delegate_0.sayHi();
  }
}
```

这里，我们可以清楚地看到，Kotlin编译器为我们动态添加了一个成员变量$$delegate_0，这个成员变量代表被代理的对象，这里对应的是BaseImpl对象，Derived里面的sayHi方法最终调用是代理对象的sayHi方法，即Kotlin编译器帮我们提供了一个非常漂亮的代理模式实现。

# 代理属性
在一些情况下，我们可能希望某些属性延迟加载，即在我们正在需要的时候才对它赋值；亦或者我们希望可以随时监听属性值的变化；在上述这些场景中，代理属性就可以发挥作用了。

代理属性的语法格式如下：

```
class DelegateProperty {
	val d: String by Delegate()
}
class Delegate {
    operator fun getValue(thisRef: Any? , property: KProperty<*>): String {
        return "Invoke getValue() , thisRef = $thisRef , property name = ${property.name}"
    }
    operator fun setValue(thisRef: Any? , property: KProperty<*> , value: String) {
        println("Invoke setValue() , thisRef = $thisRef , property name = ${property.name} , value = $value")
    }
}

fun main(args: Array<String>) {
    val dp = DelegatedProperty()
    dp.d = "Value0" // Invoke setValue() , thisRef = DelegatedProperty@2ef1e4fa , property name = d , value = Value0
   
    println(dp.d) // Invoke getValue() , thisRef = DelegatedProperty@2ef1e4fa , property name = d
}
```

这里的代理是如何实现的呢？我们知道，Kotlin的属性值会自动生成set/get方法，而代理类通过代理set/get方法生成相应的代理方法，这里的方法对应关系如下：

```
// thisRef对应代理对象的引用，property对应代理属性的反射属性封装
// 注意这里的代理方法一定要添加operator关键字，operator关键字是重载操作符关键字，后续的文章中会讲到，敬请期待
get() -> operator fun getValue(thisRef: Any? , property: KProperty<*>)
set() -> operator fun setValue(thisRef: Any? , property: KProperty<*> , value: T)
```

Kotlin标准库提供了一些常用代理的方法实现，即上文提到的几种代理，先来看第一种：延迟加载。
### 延迟加载
Kotlin提供了一个lazy方法用于实现延迟加载，lazy方法有一个lambda表达式参数，用于对属性进行初始化赋值，而一旦完成赋值，该lambda表达式将不会再次调用。lambda表达式调用发生在第一次使用该属性的时候，即实现了属性赋值的延迟加载。来看一个简单的例子:

```
// 使用标准库实现的lazy函数，实现属性的延迟加载
private val lazyValue: String by lazy {
    println("调用该初始赋值表达式完成赋值")
    // 这里是实际赋值
    "Hello, world"
}
fun main(args: Array<String>) {
    // 仅在第一次会调用lazy方法的lambda表达式
    println(lazyValue) // 打印：调用该初始赋值表达式完成赋值
    println(lazyValue) // 打印： Hello, world, 再次调用将不再调用lambda表达式
}
```

lazy方法是一个线程安全的延迟加载方法，为了加深大家的理解，根据上面的原理，我们尝试自己来实现一个非线程安全的延迟加载方法，看具体实现：

```
private object UNINITIALIZE_VALUE

class MyLazy<T>(initialize: ()->T) {
    private var value: Any? = UNINITIALIZE_VALUE
    private val initialize = initialize
    operator fun getValue(thisRef: Any? , property: KProperty<*>): T {
        if(value == UNINITIALIZE_VALUE) {
            value = initialize()
        }
        return value as T
    }
    operator fun setValue(thisRef: Any? , property: KProperty<*> , value: T) {
        this.value = value
    }
}
// 为了和标准库区分，使用__lazy命名
fun <T>  __lazy(initialize: () -> T): MyLazy<T> = MyLazy(initialize)

var lazyValue1 by __lazy {
    println("自定义lazy初始化赋值表达式被调用")
    "Hello , world"
}

fun main(args: Array<String>) {
    // 自定义延迟加载函数__lazy
    println(lazyValue1)
    lazyValue1 = "Other value"
    println(lazyValue1)
}
```

由此可见，实现一个延迟加载接口并不复杂，最重要的是要理解延迟加载的过程以及实现原理。总结实现延迟加载接口，需要注意三个地方：

* 需要提供初始化lambda表达式参数，用于初始赋值
* 需要实现代理属性对象的setValue/getValue方法，如果是val则只需要实现getValue即可
* 需要严格确保属性不会被多次初始化 

### Observable属性
Kotlin标准库还提供了一个可观察属性，这个属性使用观察者模式实现，如果属性值发生变化则会调用相应的回调lambda接口通知使用者，先看一个具体的例子:

```
var observableValue by Delegates.observable("Initial value") {  prop , old , new ->
    println("$old -> $new")
}

fun main(args: Array<String>) {
    println(observableValue)    // 打印：Initial value
    observableValue = "Hello"   // 打印: Initial value -> Hello
    println(observableValue)    // 打印：Hello
}
```

这里的具体实现，感兴趣的同学请参看文章开头的方法进行追踪！

### Storing Properties in a Map
这也是Kotlin标准库提供的一个非常有用的特性，它主要用于JSON数据的解析。看官方的例子：

```
class User(val map: Map<String, Any?>) {
	val name: String by map
	val age: Int by map
}

val user = User(mapOf(
	"name" to "John Doe",
	"age" to 25
))
```

该方法比较简单，这里就不再赘述了！

# 总结
至此，关于代理的介绍可以暂时告一段落了！
代理模式是一个非常经典设计模式，在解决某些问题中可以发挥事半功倍的效果。幸运的是，Kotlin语言原生支持代理模式，实现代理模式如同声明一个属性一样简单。而且，代理模式的设计也非常漂亮，仅仅使用一个关键字by极尽简约之美。在日常编码中，一定要灵活运用代理模式，比如实现延迟加载，实现属性观察等等。[KotterKnife](https://github.com/JakeWharton/kotterknife) 是一个非常经典的代理模式的实现例子，有兴趣的同学可以clone该仓库，查看源码，领会代理模式的优美。

# 欢迎加入Kotlin交流群
如果你也喜欢Kotlin语言，欢迎加入我的Kotlin交流群： 329673958 ，一起来参与Kotlin语言的推广工作。

# 文章源码地址
Kotliner: [https://github.com/yuanhoujun/Kotliner](https://github.com/yuanhoujun/Kotliner),
别忘了点击仓库右上方的star哦！
