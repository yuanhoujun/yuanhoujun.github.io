title: 那些年，我们看不懂的那些Kotlin标准函数
date: 2018/04/27 15:21
comments: true
tags:
- Kotlin
- 编程语言
categories:
- Kotlin
- 基础知识
---


> Kotlin标准库中提供了一套用于常用操作的函数。最近，在我的Kotlin交流群中有人再次问到了关于这些函数的用法。今天，让我们花一点时间，一起看一下这些函数的用法。

# Ready go >>>
**注：这里所说的标准函数主要来自于标准库中在文件Standard.kt中的所有函数。**

### run#1

```
@kotlin.internal.InlineOnly
public inline fun <R> run(block: () -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

contract部分主要用于编译器上下文推断，这里我们忽略掉这部分代码。

观察源码发现，run方法仅仅是执行传入的block表达式并返回执行结果而已（block是一个lambda表达式）。

**因此，如果你仅仅需要执行一个代码块，可以使用该函数**

看一个例子：

```
val x = run {
           println("Hello, world")
           return@run 1
        }
println(x)

// 执行结果
Hello，world
1
```

### run#2

```
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

这个函数跟上面的函数功能是完全一样的。不同的是，block的receiver是当前调用对象，即在block中可以使用当前对象的上下文。

**因此，如果你需要在执行的lambda表达式中使用当前对象的上下文的话，可以使用该函数。除此之外，两者没有任何差别**

看一个例子：

```
class A {
    fun sayHi(name: String) {
        println("Hello, $name")
    }
}

class B {

}

fun main(args: Array<String>) {
    val a = A()
    val b = a.run {
        // 这里你可以使用A的上下文
        a.sayHi("Scott Smith")
        return@run B()
    }
    println(b)
}

// 执行结果
Hello，Scott Smith
b@2314
```

从例子中，我们可以看到，这个函数还可以用于对数据类型进行转换。

### with 
```
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

这个函数其实和run函数也是做了一样的事情。不同的是，这里可以指定block的接收者。

**因此，如果你在执行lambda表达式的时候，希望指定不同的接收者的话，可以使用该方法**

```
class A {
    fun sayHi(name: String) {
        println("Hello, $name")
    }
}


fun main(args: Array<String>) {
    val a = A()
    with(a) {
        // 这里的接收者是对象a，因此可以调用a实例的所有方法
        sayHi("Scott Smith")
    }
}

```

### apply

```
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

可以看到，这个方法是针对泛型参数的扩展方法，即所有对象都将拥有该扩展方法。相对于run#2方法，apply不仅执行了block，同时还返回了receiver本身。

**这在链式编程中很常用，如果你希望执行lambda表达式的同时而不破坏链式编程，可以使用该方法**

看一个例子：

```
class A {
    fun sayHi(name: String) {
        println("Hello, $name")
    }
    
    fun other() {
        println("Other function...")
    }
}


fun main(args: Array<String>) {
    val a = A()
    a.apply { 
        println("This is a block")
        sayHi("Scott Smith")
    }.other()
}

// 执行结果
This is a block
Hello, Scott Smith
Other function...
```

### also

```
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}
```

这个函数跟with又很像，不同的是，block带有一个当前receiver类型的参数。在block中，你可以使用该参数对当前实例进行操作。

**这个函数和with完全可以互相通用，with函数可以直接在当前实例上下文中对其进行操作，而also函数要通过block参数获取当前类实例。因为用法完全一致，这里就不举例了**

### let

```
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

如果你使用过RxJava，可能会感到似曾相识，这其实就是RxJava的map函数。这个函数也是针对泛型参数的扩展函数，所有类都将拥有这个扩展函数。

**如果你希望对当前数据类型进行一定的转换，可以使用该方法。该方法的block中同样可以使用当前receiver的上下文**

看一个例子：

```
class Triangle {}

class Rectangle {}

fun main(args: Array<String>) {
    val tr = Triangle()
    val rect = tr.let { it ->
        println("It is $it")
        return@let Rectangle()
    }
    println(rect)
}

// 执行结果
It is Triangle@78308db1
Rectangle@27c170f0
```

从例子中可以看到，我们成功地将三角形转换成了矩形，这就是let函数的作用。

### takeIf
```
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? {
    contract {
        callsInPlace(predicate, InvocationKind.EXACTLY_ONCE)
    }
    return if (predicate(this)) this else null
}
```

这个函数也是针对泛型参数的扩展函数，所有类都将拥有这个扩展。这个函数使用了一个预言函数作为参数，主要用于判断当前对象是否符合条件。
这个条件函数由你指定。如果条件符合，将返回当前对象。否则返回空值。

**因此，如果你希望筛选集合中某个数据是否符合要求，可以使用这个函数**

看一个例子：

```
fun main(args: Array<String>) {
    val arr = listOf(1, 2, 3)
    arr.forEach {
        println("$it % 2 == 0 => ${it.takeIf { it % 2 == 0 }}")
    }
}

// 执行结果
1 % 2 == 0 => null
2 % 2 == 0 => 2
3 % 2 == 0 => null
```

### takeUnless

```
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? {
    contract {
        callsInPlace(predicate, InvocationKind.EXACTLY_ONCE)
    }
    return if (!predicate(this)) this else null
}
```

这个函数刚好与takeIf筛选逻辑恰好相反。即：如果符合条件返回null，不符合条件返回对象本身。

看一个例子：

```
fun main(args: Array<String>) {
    val arr = listOf(1, 2, 3)
    arr.forEach {
        println("$it % 2 == 0 => ${it.takeUnless { it % 2 == 0 }}")
    }
}

// 执行结果
1 % 2 == 0 => 1
2 % 2 == 0 => null
3 % 2 == 0 => 3
```

看到了吗？这里的执行结果和takeIf恰好相反。

### repeat

```
@kotlin.internal.InlineOnly
public inline fun repeat(times: Int, action: (Int) -> Unit) {
    contract { callsInPlace(action) }

    for (index in 0 until times) {
        action(index)
    }
}
```

这个函数意思很明显，就是将一个动作重复指定的次数。动作对应一个lambda表达式，表达式中持有一个参数表示当前正在执行的次数索引。

看一个例子：

```
fun main(args: Array<String>) {
    repeat(3) {
        println("Just repeat, index: $it")
    }
}

Just repeat, index: 0
Just repeat, index: 1
Just repeat, index: 2
```

# 简单总结
最后，我们用一个表格简单总结一下这些函数的用法：
函数|用途|特点|形式
:---:|:---:|:---:|:---:
run#1|执行block，并返回执行结果|block中无法获取接收者上下文|全局函数
run#2|执行block，并返回执行结果|block中可以获取接收者上下文|扩展函数
with|指定接收者，通过接收者执行block|block中可以获取接收者的上下文，可以对接收者数据类型做一定转换|全局函数
apply|执行block，并返回接收者实例本身|block中可以获取接收者的上下文，可用于链式编程|扩展
also|执行block，并返回接收者实例本身|block中有一个参数代表接收者实例，可用于链式编程|扩展
let|执行block，并返回执行结果|block中有一个参数代表接收者实例，可以对接收者数据类型做一定转换|扩展
takeIf|根据条件predicate判断当前实例是否符合要求|如果符合要求，返回当前实例本身；否则返回null|扩展函数
takeUnless|根据条件predicate判断当前实例是否不符合要求|如果不符合要求，返回当前实例本身；否则返回null|扩展

# 搞定Receiver
理解上面这几个函数，最重要的一点是要理解Receiver。遗憾的是，Kotlin官方文档中并没有针对Receiver的详细讲解。关于这部分的讲解，请扫描下方二维码关注**欧阳锋工作室**，回复**搞定Receiver**查看文章。

# 欢迎加入Kotlin交流群
关于Kotlin，如果你有任何问题，欢迎加入我的Kotlin交流群： 329673958。当前群交流活跃，问题解答速度很快，期待你的加入。