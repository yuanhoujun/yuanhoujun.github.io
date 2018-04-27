title: 测测你的Kotlin基础
date: 2016/04/27 15:09
comments: true
tags:
- Kotlin
- 编程语言
categories:
- Kotlin
- 基础知识
---


![文 | 欧阳锋](https://user-gold-cdn.xitu.io/2018/4/25/162fabff1c2028ae?w=2250&h=500&f=png&s=60530)

> 本次测试满分160分，测测看，你能拿几分 \<\<\<

#### 1）Kotlin语言有基本数据类型吗？（5分）

#### 2）Kotlin中有哪些访问控制符，分别代表什么意思？默认访问控制符是什么？（5分）

#### 3）Kotlin接口是否允许有方法实现？是否允许声明成员变量？（5分）

#### 4）Sealed类有什么作用？（5分）

#### 5）Kotlin语言中如何实现类似Java创建匿名内部类对象？（10分）

#### 6）Kotlin的扩展相对继承有什么优势？扩展方法的执行是否也遵循多态？（10分）

#### 7）如果一个类同时实现多个接口，接口中存在同名方法，如何解决冲突？（5分）

#### 8）Kotlin语言中是否存在static关键字，如果没有，如何声明静态变量，并实现与Java互通（5分）

#### 9）使用Kotlin语言是否一定不会出现空指针异常？为什么？（10分）

#### 10）Kotlin语言中推荐使用什么方式判断两个对象是否相等？如何判断两个对象是同一个对象？（5分）

#### 11）如果使用Foo\<out T: TUpper\> 这种方式声明泛型，使用Foo<*>这种方式接收该对象实例，代表什么意思？如何理解Kotlin泛型，与Java有什么区别？（10分）

#### 12）如何自定义setter/getter方法？（5分）

#### 13）使用语句`var x = null`声明变量x是否合法？如果合法，x的具体类型是什么？(5分)

#### 14）下面这段代码的输出结果是什么？（10分）

```
val list = listOf(1, 2, 3)
list.add(4)
println(list)
```

#### 15）下面这段代码的执行结果是什么？（5分）

```
// Kotlin端
object A {
    fun init() {
        println("A init")
    }
}

// Java端
A.init()
```

#### 16）下面代码的执行结果是什么？（5分）

```
fun sum(a: Int, b: Int) = { a + b }

println(sum(1, 3))
```

#### 17）下面代码的执行结果是什么？（5分）

```
println(null is Any)
println(null!! is Nothing)
```

#### 18）下面代码的执行结果是什么？（10分）

```
class A {
    init() {
        f()
    }
    
    val a = "a"
    
    fun f() {
        println(a)
    }
}

fun main(args: Array<String>) {
    A()
}
```

#### 19）下面代码的执行结果是什么？（10分）

```
println(127 as Int? === 127 as Int?)
println(128 as Int? === 128 as Int?)
```

#### 20）下面代码的执行结果是什么？如果运行异常，应该怎样修改才能达到预期效果？（10分）

```
 (1..5).forEach {
    if (it == 3) break
    println(it)
 }
```

#### 21）下面代码的执行结果是什么？如果运行异常，应该怎样修改，为什么要这样修改？（10分）

```
val A.x: Int = 3

println(A().x)
```

#### 22）下面这段代码的执行结果是什么？（10分）

```
fun isOdd(x: Int) = x % 2 != 0

fun length(s: String) = s.length

fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}

fun main(args: Array<String>) {
    val oddLength = compose(::isOdd, ::length)
    val strings = listOf("a", "ab", "abc")
    println(strings.filter(oddLength))
}
```

**注：本篇例子Kotlin版本为1.2.31，更新版本可能存在部分差异**

#### 下面是你的基础等级：

得分|评价
:---:|:---:
0 ~ 80|基础较差
80 ~ 108 | 基础较好
108 ~ 160 | 基础很棒

# 查看答案方法
微信扫描下方二维码关注**欧阳锋工作室**，回复“Kotlin测试题答案”即可获取当前测试题答案

# 欢迎加入Kotlin交流群
如果你也喜欢Kotlin语言，欢迎加入我的Kotlin交流群： 329673958 ，一起来参与Kotlin语言的推广工作。