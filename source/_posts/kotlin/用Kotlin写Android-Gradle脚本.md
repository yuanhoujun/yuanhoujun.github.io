title: 也许你应该试试用Kotlin写Gradle脚本
date: 2018/01/26 10:48
comments: true
tags:
- Kotlin
- Groovy
- Gradle
- 脚本
categories:
- Kotlin
- 杂谈
---

![文 | 欧阳锋](http://upload-images.jianshu.io/upload_images/703764-1ba63891256d9db2.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>Android应用开发中，离不开Gradle脚本的构建。大部分Android开发同学忽视了脚本的力量，甚至有很大一部分同学不知道Gradle脚本是什么，用什么语言编写的；当然，也有相当一部分同学知道Gradle脚本是使用Groovy语言编写的，但对于Groovy语言却一窍不通，只是勉强可以看懂Gradle脚本。正所谓，知其然，但并不知其所以然...

换个角度看问题，熟练掌握Gradle脚本还需要精通Groovy语言，这对Android开发同学来说的确是一个不小的挑战。这种Java + Groovy的开发套餐对于普通的Android开发者来说的确存在一定的知识断层，显而易见的是，部分同学写的Gradle脚本简直“不堪入目”。时间回到去年5月份，Google IO大会上宣布了一个重磅消息，Android官方开始支持使用Kotlin语言进行应用开发。其实，在这个时间节点上，我已经在生产环境使用Kotlin开发Android将近一年。对于我来说，这无疑是一个让人欣喜若狂的消息。但，惊喜还远远不止这些，过了一段时间，我又看到了这篇文章 [Kotlin Meets Gradle](https://blog.gradle.org/kotlin-meets-gradle)。很有诗意的标题：当Gradle邂逅Kotlin，文章的核心意思是：**Gradle团队正在尝试使用Kotlin语言作为Gradle脚本的官方开发语言。**

>我想，也许，Android开发者的春天就要到了！

# Why use Kotlin ?
在写Gradle脚本的时候，最痛苦的莫过于没有任何提示，唯一的调试手段就是使用**print**方法打印调试日志。正如 [Kotlin Meets Gradle](https://blog.gradle.org/kotlin-meets-gradle) 文中所说，当你使用Kotlin语言编写Gradle脚本的时候，你会发现一切都变得有趣起来。突然：
* 脚本代码可以自动补全了
* 源码之间可以互相跳转了
* 插件源码更容易看懂了
* 重构（Refactoring）也可以支持了
...

当然，惊喜还不止这些，当你开始决定使用Kotlin语言的时候，仿佛一切都变得美好了起来！

# Let's start
好了，废话不多说，接下来我们开始尝试用Kotlin语言编写Gradle脚本。由于当前 [kotlin-dsl](https://github.com/gradle/kotlin-dsl) 正处于预发布状态（**kotlin-dsl**的最新版本是0.14.2，对应Gradle插件版本4.5），IDE的支持也不完善，为了更好的体验该功能，推荐大家使用如下配置：

### 实验室配置
**操作系统：**macOS 10.13.2
**Android Studio：** 3.1 Canary 9
**Gradle Wrapper：** 4.5
**Gradle Plugin：** 3.1.0-alpha9
**Kotlin**：1.2.21

# 操作步骤
首先，按照以往步骤创建一个Android工程：
![](http://upload-images.jianshu.io/upload_images/703764-58f93d293addaba0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来，改造开始，Gradle Script Kotlin脚本以`.gradle.kts`后缀结尾。因此，我们先将工程根目录`settings.gradle`更名为`settings.gradle.kts`。
![](http://upload-images.jianshu.io/upload_images/703764-429cf07b4fcaf7bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个地方的错误有两个原因：
* Kotlin语言中，单引号只能包裹字符，不能包裹字符串
* Kotlin语言中，方法调用使用括号。仅在使用`infix`修饰的方法中可以省略括号。这里显然是一个正常调用方法。因此，我们修改为：
```
include("app")
```
接下来，修改根目录的`build.gradle`脚本，用同样的方式修改后缀，方法修改为括号调用，修改后的内容如下：
```
buildscript {
    repositories {
        google()
        jcenter()
    }
    dependencies {
        // 这里修改为括号调用即可
        classpath("com.android.tools.build:gradle:3.0.1")
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}
```
> 注意：在修改后缀名称的时候IDE会出现警告提示，这里可以忽略，选择**continue**即可。

由于我们手动修改了`build.gradle`脚本，为了保证工程可以使用这个脚本，需要在`settings.gradle.kts`中添加一行代码，让Gradle知道使用`build.gradle.kts`脚本构建。因此，最后的`settings.gradle.kts`代码如下：
```
include("app")
rootProject.buildFileName = "build.gradle.kts"
```

最后一步，修改app模块`build.gradle`文件，这也是最复杂的一步，修改完后缀名后，你会看到整个脚本全部被红色标识错误：
![](http://upload-images.jianshu.io/upload_images/703764-fcca2611413c7e31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


别慌！还是一样的方式，这里我们先将这里的所有代码注释掉。在最上方逐一对应修改，`apply plugin`部分修改为:
```
plugins {
    id("com.android.application")
}
```

接下来，修改`android {}`闭包部分。这里有两个小技巧，由于目前IDE的支持不是很完善，在输入的时候稍微等待一段时间，IDE会给出相应的提示。另外，如果没有提示，例如`android {}`闭包就没有任何提示，输入完成后展开右侧gradle面板，选择`gradle/buid setup/init`，双击执行：
![](http://upload-images.jianshu.io/upload_images/703764-4710f3eea9bac093.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在底部面板可以看到任务执行是否成功。注意，即使任务执行成功，脚本依然可能被红色标识，这是IDE支持不完善导致的，可以忽略。

修改完成后的内容如下：
```
android {
    compileSdkVersion(27)
    buildToolsVersion("27.0.2")

    defaultConfig {
        applicationId = "com.youngfeng.kotlindsl"
        minSdkVersion(15)
        targetSdkVersion(27)
        versionCode = 1
        versionName = "1.0"
        testInstrumentationRunner = "android.support.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        getByName("release") {
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro")
        }
    }
}
```

注意：你在使用的过程中，依然可能会遇到无论如何都不生效的问题。这个时候别着急，使用`./gradlew assembleDebug`命令调试，查看终端找到错误原因。Windows用户去掉`./`执行即可。
![](http://upload-images.jianshu.io/upload_images/703764-1e9abe6a47044d09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


最后的依赖部分，同样地，全部修改为括号调用即可。这里就不赘述了，文章的最后部分会提供操作视频，在使用过程中有任何问题可以打开操作视频参考，如果依然不能解决，可以在文章下方给我留言，我会在第一时间给你答复。修改后的内容如下：
```
dependencies {
    implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
    implementation("com.android.support:appcompat-v7:26.1.0")
    implementation("com.android.support.constraint:constraint-layout:1.0.2")
    testImplementation("junit:junit:4.12")
    androidTestImplementation("com.android.support.test:runner:1.0.1")
    androidTestImplementation("com.android.support.test.espresso:espresso-core:3.0.1")
}
```

通过上面的步骤，从Groovy转换到Kotlin的步骤已经全部完成，你可以在终端输入`./gradlew assembleDebug`测试是否可以正常构建了：
![](http://upload-images.jianshu.io/upload_images/703764-2ec3894242b02f2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 统一依赖管理
上面的步骤虽然完成了脚本的转换，但依赖的管理依然是混乱的，为了实现类似
 [Snake](https://github.com/yuanhoujun/Android_Slide_To_Close) 工程的统一依赖管理，我们还需要做一些工作。

Gradle官方提供了使用 [buildSrc](https://docs.gradle.org/current/userguide/organizing_build_logic.html#sec:build_sources) 目录实现自定义任务和插件逻辑，这里我们可以使用它完成依赖的统一处理，一个完整的buildSrc目录结构如下：
![](http://upload-images.jianshu.io/upload_images/703764-a000e0ab037ab1a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Deps类中，可以这样定义依赖结构：
```
object deps {
    object plugin {
        val gradle = "com.android.tools.build:gradle:3.1.0-alpha09"
        val kotlin = "org.jetbrains.kotlin:kotlin-gradle-plugin:1.2.21"
    }

    object kotlin {
        val stdlibJre7 = "org.jetbrains.kotlin:kotlin-stdlib-jre7:1.2.21"
    }

    object android {
        object support {
            val compat = "com.android.support:appcompat-v7:27.0.2"
            val constraintLayout = "com.android.support.constraint:constraint-layout:1.0.2"
        }

        object test {
            val junit = "junit:junit:4.12"
            val runner = "com.android.support.test:runner:1.0.1"
            val espressoCore = "com.android.support.test.espresso:espresso-core:3.0.1"
        }
    }
}
```

定义之后，我们就可以在脚本中直接引用了：
```
dependencies {
    implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
    implementation(deps.kotlin.stdlibJre7)
    implementation(deps.android.support.compat)
    implementation(deps.android.support.constraintLayout)
    testImplementation(deps.android.test.junit)
    androidTestImplementation(deps.android.test.runner)
    androidTestImplementation(deps.android.test.espressoCore)
}
```
是不是漂亮了许多？

至此，整个转换过程就顺利完成了，为了保证转换的成功率，我推荐使用文章开头的实验室配置。如果版本过低，不保证可以转换成功。最新版本的**kotlin-dsl**会跟随最新版本的Gradle插件发布，因此一定要使用最新版本。另外，目前IDE对kts的支持依然不完善，即使正确的写法也会报错，这个一定要注意，不要被IDE欺骗了。

### 更详细的操作，请看视频教程
腾讯视频：[用Kotlin写Android Gradle脚本](https://v.qq.com/x/page/x0539lvfmm2.html)

#### 一些建议
虽然使用Kotlin语言写脚本是一件非常美妙的事情，但目前依然存在一些问题：
* IDE支持不完善
*  [kotlin-dsl](https://github.com/gradle/kotlin-dsl) 正在快速开发中，语法变动较大
* 缺少官方文档
* 互联网上缺少相关资料，遇到问题很难追踪

因此，目前我并不推荐你在生产环境中使用，但可以作为日常学习练手之用。预计1.0版本的发布在今年6月份左右，正式版本发布后，我推荐你立即将Gradle脚本转换到Kotlin语言。

#### 遇到问题，看这里 ==>
在使用的过程中，按照文章同样的步骤，你依然可能会遇到很多问题。因此，我为你整理了目前互联网上可以参考的资料，你可以收藏这篇文章。遇到问题别慌，来这里查找答案。

关于**kotlin-dsl**的开发路线图，请看这篇文章：[https://blog.gradle.org/kotlin-scripting-update](https://blog.gradle.org/kotlin-scripting-update)

如果你在使用过程中，遇到了任何问题，并且确定是 **kotlin-dsl** 的bug，请点这里：[https://github.com/gradle/kotlin-dsl](https://github.com/gradle/kotlin-dsl) 并推送 **issue**

如果你遇到了知识盲点，并且在Google找不到答案。可以来 [Slack](https://kotlinlang.slack.com/)#gradle频道反馈，我在 [Slack](https://kotlinlang.slack.com/) 的昵称是**Scott Smith**，也欢迎你给我发送私信消息。
![](http://upload-images.jianshu.io/upload_images/703764-995957884721bca9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本篇文章例子完整代码，请点击这里：[https://github.com/yuanhoujun/gradle-script-kotlin-example](https://github.com/yuanhoujun/gradle-script-kotlin-example)

kts文档正在编写当中，具体进度，请点这里：[https://github.com/gradle/kotlin-dsl-docs](https://github.com/gradle/kotlin-dsl-docs)

# 欢迎加入Kotlin交流群
如果你也喜欢Kotlin语言，欢迎加入我的Kotlin交流群： 329673958 ，一起来参与Kotlin语言的推广工作。
