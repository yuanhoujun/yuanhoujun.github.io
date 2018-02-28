title: 关于Git，你真的学会了吗？
date: 2018/02/01 22:13
comments: true
tags:
- Git
- 版本控制工具
- 开发工具
- 命令行
categories:
- 开发工具
- Git
---

“锋哥，Git有什么可说的，不就是`git add`添加，`git commit`提交嘛”  听说我要写一篇Git教程，小明不屑一顾地说。
“..."。

小明是我的一个学生。目前，是一名Android开发工程师。

过了几天，我又再次见到了小明。

“锋哥，今天，我在Github新建了一个版本库，本地提交后推送远程的时候，却被拒绝了，是怎么回事？”

以下是小明的操作记录：
```
git init
git add .
git commit -m "Init commit"
git remote add origin git@github.com:xiaoming/xxx.git
git pull origin master
```
以上操作触发了下面的错误：
```
From git@github.com:xiaoming/xxx.git
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
fatal: refusing to merge unrelated histories
```
“小明，注意看最后一句提示。翻译成中文的意思是 ‘拒绝合并不相关的历史’，这个问题有两个方案可以处理。"

* `git pull`命令其实是触发了拉取`git fetch`和合并`git merge`两个操作。而本地的版本库和远程版本库在第一次拉取或推送完成之前是毫不相关的，Git为了避免不必要的合并，默认不允许进行这样的操作。但你可以手动添加`--allow-unrelated-histories`强制进行合并，这是方案一。
```
git pull origin master --allow-unrelated-histories
```
* 再来看方案二，从你上面的操作来看，你只是在本地初始化了一个版本库，并完成了基础的提交。接下来，你希望和远程版本库建立关联，将提交推送到远程。这种情况下，其实你可能并不需要远程的默认数据（通常是一个空的README文件）。所以，你可以添加`-f`参数，将提交强制提交并覆盖远程版本库。
```
git push -f origin master
```

小明若有所思地点点头，这是小明第一次遇到Git问题。我想，接下来他应该会比较顺利了。

没想到，过了几天，我又收到了小明的消息。这一次，他发来的是对Git的抱怨。

“锋哥，Git好讨厌，提交日志出现了错误，也不能修改。你知道搜狗输入法有时候不够智能，输入太快不小心就输错了...😓” 

“🙂，你这孩子，别轻易下结论哈。其实，Git是允许修改提交记录的。使用Git最舒服的一点就是：Git永远都会给你反悔的机会。这一点，其它的版本控制工具是做不到的！”

“哦，原来是这样啊！那快说说看，要怎么做？” 小明已经一副迫不及待的表情了。

“`git commit`命令中有一个参数叫`--amend`就是为解决这个问题而生的。因此，如果是最近的提交，你只需要按照下面的命令操作即可。”

```
git commit --amend -m "这是新的提交日志"
```

看完我的消息，小明给我发来一个微笑的表情。小明的抱怨让我想起一句好气又好笑的农村俗语 “屙屎不出怪茅坑”，哈哈。

本以为一切可以风平浪静了。没想到，过了一个月左右，突然接到了小明的紧急电话。电话那头，小明似乎心情很急躁。

“锋哥，我不小心进行了还原操作，我写的代码全丢了。几千行的代码啊，明天晚上就要发版本了，有办法找回来吗？”

听到这个消息，我心里盘算，大约有50%的概率应该是找不回来了。这孩子比较粗心，可能根本就没提交到版本库。但如果他正好提交到了版本库，兴许还有救。因此，我安慰他说 “小明，别急！你打开TeamViewer，我远程帮你看看”

连上机器后，我使用`history`命令看到小明在提交之后使用了`git reset --hard xxx`命令进行重置。`--hard`是`git reset`命令中唯一一个不安全的操作，它会真正地销毁数据，以至于你在`git log`中完全看不到操作日志。可是，Git真的很聪明，它还保存了另外一份日志叫`reflog`，这个日志记录了你每次修改HEAD的操作。因此，你可以通过下面的命令对数据进行还原：
```
git reflog

// 使用这个命令，你看到的日志大概是这样
c8278f9 (HEAD -> master) HEAD@{0}: reset: moving to c8278f9914a91e3aca6ab0993b48073ba1e41b2b
3e59423 HEAD@{1}: commit: a
c8278f9 (HEAD -> master) HEAD@{2}: commit (amend): v2 update
2dc167b HEAD@{3}: commit: v2
2e342e9 HEAD@{4}: commit (initial): Init commit
```

可以看到，我们在版本`3e59423`进行了`git reset`操作，最新版本是`3e59423`。因此，我们可以再次通过`git reset`命令回到这个版本：
```
git reset --hard 3e59423
```
以上操作完成后，你会惊喜地发现，丢失的数据居然神奇般地回来了。

“🌺 🌺 🌺”

“下次别这样操作了哈。另外，你怎么一次性丢失这么多代码。一定要记得勤提交。” 小明出现这样的问题，与平时的不规范操作也是分不开的。因此，最后我还不忘嘱咐了他一句。

“好的，我知道了。对了，我一个还有比较疑惑的问题。`git checkout`和`git reset`到底有啥区别？我以前用SVN的时候`git checkout`是用来检出代码的，在Git中可以用它切换分支或者指定版本，但`git reset`同样可以做到。难道两者是完全一样的吗？” 小明在QQ中给我发来了回复消息。

“这是一个比较有深度的问题，解释这个问题需要一点时间。接下来，你仔细听”

## 理解Git工作空间
理解这个问题之前，先来简单学习一些Git基础知识。Git有三种状态：
* 已提交（commited）：数据已完全保存到本地数据库中
* 已修改（modified）：修改了文件，但还没有保存到数据库中
* 已暂存（staged)：对一个已修改的文件做了标记，将包含在下一次提交的版本快照中

这三种状态对应Git三个工作区域：Git版本库、暂存区和工作区
![](http://upload-images.jianshu.io/upload_images/703764-6459c27004beb536.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Git版本库是Git用来保存项目的元数据和对象数据库的地方，使用`git clone`命令时拷贝的就是这里的数据。

工作目录是对某个版本独立检出的内容，这些数据可以供你使用和修改。

暂存区在Git内部对应一个名为index的文件，它保存了下次将要提交的文件列表信息。因此，暂存区有时候也被叫作 “索引”。

一个基础的Git工作流程如下：
1）在工作区修改文件
2）使用`git add`将文件添加到暂存区，也就是记录到`index`文件中
3）使用`git commit`将暂存区中记录的文件列表，使用快照永久地保存到Git版本库中

## 理解HEAD
解释这个问题，你还需要简单理解HEAD是什么。简单来说，HEAD是当前分支引用的指针，它永远指向该分支上最后一次提交。为了让你更容易理解HEAD，你可以将HEAD看作上一次提交数据的快照。

如果你感兴趣，你可以使用一个底层命令来查看当前HEAD的快照信息:
```
git ls-tree -r HEAD

100644 blob aca4b576b7d4534266cb818ab1191d91887508b9	demo/src/main/java/com/youngfeng/snake/demo/Constant.java
100644 blob b8691ec87867b180e6ffc8dd5a7e85747698630d	demo/src/main/java/com/youngfeng/snake/demo/SnakeApplication.java
100644 blob 9a70557b761171ca196196a7c94a26ebbec89bb1	demo/src/main/java/com/youngfeng/snake/demo/activities/FirstActivity.java
100644 blob fab8d2f5cb65129df09185c5bd210d20484154ce	demo/src/main/java/com/youngfeng/snake/demo/activities/SecondActivity.java
100644 blob a7509233ecd8fe6c646f8585f756c74842ef0216	demo/src/main/java/com/youngfeng/snake/demo/activities/SplashActivity.java
```

这里简单解释一下每个字段的意思：100644表示文件模式，其对应一个普通文件。blob表示Git内部存储对象数据类型，另外还有一种数据类型tree，对应一个树对象，中间较长的字符串对应当前文件的SHA-1值，这部分不需要记住，简单了解即可。

所以，简单来说，HEAD对应一个树形结构，存储了当前分支所有的Git对象快照：
![](http://upload-images.jianshu.io/upload_images/703764-639a489c0930506a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们用一个表格简单来总结一下以上知识点：

HEAD|Index(暂存区)|工作区
:--:|:--:|:--:
上一次提交的快照，下一次提交的父节点|预期的下一次提交快照|当前正在操作的沙盒目录

理解`git reset`和`git checkout`区别主要是理解Git内部是怎么操作以上三棵树的。

接下来，我们用一个简单的例子来看一下使用`git reset`到底发生了什么。先创建一个Git版本库并触发三次提交：
```
git init repo
touch file.txt
git add file.txt
git commit -m "v1"

echo v2 > file.txt
git add file.txt
git commit -m "v2"

echo v3 > file.txt
git add file.txt
git commit -m "v3"
```

以上操作完成后，版本库现在看起来是这样的：
![](http://upload-images.jianshu.io/upload_images/703764-553e3fc8b81023c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来执行命令`git reset 14ad152`看看会发生什么。以下是命令执行完成后看到的结果：
```
git log --abbrev-commit --pretty=oneline
### This is output ###
14ad152 (HEAD -> master) v2
bcc49f4 v1

git status -s
### This is output ###
 M file.txt

cat file.txt
### This is output ###
v3
```

可以看到版本库中文件版本回退到了V2，工作区文件内容同之前的版本V3一致；为了确认暂存区发生了什么变化，我们再使用一个底层命令对比一下暂存区数据和版本库数据是否一致：
```
# 查看暂存区信息
git ls-files -s
### This is output ###
100644 8c1384d825dbbe41309b7dc18ee7991a9085c46e 0	file.txt

# 查看版本库快照信息
git ls-tree -r HEAD
### This is output ###
100644 blob 8c1384d825dbbe41309b7dc18ee7991a9085c46e	file.txt
```

可以看到当前版本库和暂存区信息是完全一致的，HEAD指向了v2提交，用一个图形来表示整个过程，应该是这样：
![](http://upload-images.jianshu.io/upload_images/703764-5d21495a27e774c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看一眼上图，理解一下刚刚发生的事情：首先，HEAD指针发生了移动，指向了V2，并撤销了上一次提交。目前，版本库和暂存区都保存的是第二次提交的记录，工作区却保存了最近一次修改。稍微联想一下，你就会发现，这次的`git reset`命令恰好是最近一次提交的逆向操作。让数据完全回到了上一次提交前的状态。所以，如果你想撤销最近一次提交，可以这么做。

####  增加--soft参数测试
以上是我们对`git reset`命令的第一次尝试，在下一轮尝试前，先执行`git help reset`看看`reset`命令的用法：
```
git reset [-q] [<tree-ish>] [--] <paths>...
git reset (--patch | -p) [<tree-ish>] [--] [<paths>...]
git reset [--soft | --mixed [-N] | --hard | --merge | --keep] [-q] [<commit>]
```

看最后一句发现，`reset`命令后面还可以接5个不同的参数: `--soft`、`--mixed`、`--hard` 、`--merge`、`--keep`。这里我们主要关注前面三个，其中`--mixed`其实刚刚已经尝试过，它和不带参数的`git reset`命令是同样的效果。换而言之，`--mixed`是`git reset`命令的默认行为。接下来执行`git reset --soft 14ad152`看看会发生什么。命令执行完成后，按照惯例，我们同样使用基础命令看看发生了什么变化：
```
git log --abbrev-commit --pretty=oneline
### This is output ###
14ad152 (HEAD -> master) v2
bcc49f4 v1

git status -s
### This is output ###
M  file.txt

cat file.txt
### This is output ###
v3
```
奇怪了？为什么会和上次不带任何参数的执行结果完全一致？难道Git出现了设计错误。相信你看到结果一定会有这样的疑问，其实不然！因为，这里我用文本粘贴了输出结果，忽略了命令的字体颜色，其实这里第二条命令输出结果中的M颜色与上一次执行结果是不一样的。为了让你看到不同，看下面的截图：
![](http://upload-images.jianshu.io/upload_images/703764-c76ba02b7fb8b5ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个颜色表示：file.txt文件已经被添加到了暂存区，使用`git commit`命令就可以完成提交。为了严谨，我们依然使用上面的底层命令看看版本库和暂存区信息是否一致。注意：这里的结果应该是不一致才对，因为版本库记录的文件版本是v2，而暂存区记录的文件版本其实是v3。
```
git ls-tree -r HEAD
### This is output ###
100644 blob 8c1384d825dbbe41309b7dc18ee7991a9085c46e	file.txt

git ls-files -s
### This is output ###
100644 29ef827e8a45b1039d908884aae4490157bcb2b4 0	file.txt
```
可以看到，两个命令执行输出的SHA-1并不一致，验证了我们的猜想。

这里我们可以得出一个结论：`--soft`和默认行为(`--mixed`)不一样的地方是：`--soft`会将工作区的最新文件版本再做一步操作，添加到暂存区。使用这个命令可以用来合并提交。即：如果你在某一次提交中有未完成的工作，而你反悔了，你可以使用这个命令撤销提交，等工作做完后继续一次性完成提交。

####  增加--hard参数测试
接下来我们对最后一个参数进行测试，这也是小明在使用过程出现问题的一个参数。执行命令`git reset --hard 14ad152`，看看发生了什么：
```
git log --abbrev-commit --pretty=oneline
### This is output ###
14ad152 (HEAD -> master) v2
bcc49f4 v1

git status -s
### This is output ###
>>> No output <<<

cat file.txt
v2
```

注意看，这次使用`git status -s`完全看不到输出，这就证明：当前工作区，暂存区，版本库数据是完全一致的。查看文件内容，发现文件回到了v2版本。通常情况下，如果你看到这种情况，一定会吓一跳，你最近一次提交的数据居然完全丢失了。的确，这是Git命令中少有的几个真正销毁数据的命令之一。除非你非常清楚地知道自己在做什么，否则，请尽量不要使用这个命令！

我们依然用一张图，完整地描述这个命令到底发什么了什么：
![](http://upload-images.jianshu.io/upload_images/703764-dea7f52471f35ebe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，相对于默认行为，`--hard`将工作区的数据也还原到了V2版本，以至于V3版本的提交已经完全丢失。

# git checkout
接下来看`git checkout`,  按照惯例，先执行`git checkout 14ad152`看看会发生什么：
```
git log --abbrev-commit --pretty=oneline
### This is output ###
14ad152 (HEAD -> master) v2
bcc49f4 v1

git status -s
### This is output ###
>>> No output <<<

cat file.txt
v2
```

可以看到，又出现了神奇的一幕，这一次`git checkout`命令的执行结果的确和`git reset --hard`完全一致。这是否意味着两者就没有任何区别了呢？当然也不是。严格来说，两者有两个“本质”的区别：
* 相对而言，`git checkout`对工作目录是安全的，它不会将工作区已经修改的文件还原，`git reset`则不管三七二十一一股脑全部还原。
* 另外一个比较重要的区别是，`git checkout`并不移动HEAD分支的指向，它是通过直接修改HEAD引用来完成指针的指向。

第二个不同点相对比较难理解，我们用一张图来更直观地展示二者的区别：
![](http://upload-images.jianshu.io/upload_images/703764-c0e4b18961fe668f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

简单来说，`git reset`会通过移动指针来完成HEAD的指向，而`git checkout`则通过直接修改HEAD本身来完成指向的移动。

# 命令作用于部分文件
`git reset`和`git checkout`还可以作用于一个文件，或者部分文件，即带文件路径执行。这种情况下，两个命令的表现不太一样。我们来试试看，先执行` git reset 14ad15 -- file.txt`命令尝试将文件恢复到V2版本。命令执行完成，按照惯例用一些基础命令来看看发生了什么：
```
git log --abbrev-commit --pretty=oneline
### This is output ###
4521405 (HEAD -> master) v3
14ad152 v2
bcc49f4 v1

git status -v
### This is output ###
diff --git a/file.txt b/file.txt
index 29ef827..8c1384d 100644
--- a/file.txt
+++ b/file.txt
@@ -1 +1 @@
-v3
+v2

cat file.txt
v3
```

可以看到，版本库和工作区的数据都没有发生变化。唯一发生变化的是暂存区，暂存区记录下一次提交的改动将导致数据从V3恢复到V2版本!
![](http://upload-images.jianshu.io/upload_images/703764-d9199360f0dbe0b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里我们可以这样理解：执行这条命令后，Git先将暂存区和工作区的文件版本恢复到V2，再将工作区的文件版本恢复到V3。与`--hard`不一样的地方是：这个命令并不会覆盖工作区已经修改的文件，是安全操作。

执行带路径的`git checkout`命令和`git reset`命令有一些细微的差别，相对于`git reset`，`git checkout`带路径执行会覆盖工作区已经修改的内容，导致数据丢失，是一个非安全操作。

针对上面的所有实验，我们用一个简单的表格来总结他们的区别，以及操作是否安全：
#### 不带路径执行
命令行|HEAD|暂存区|工作区|目录安全
:---:|:---:|:---:|:---:|:---:
git reset [--mixed] |YES|YES|NO|YES
git reset --soft |YES|YES|NO|YES
git reset --hard |YES|YES|YES|NO
git checkout |Modify|YES|YES|YES

#### 带路径执行
命令行|HEAD|暂存区|工作区|目录安全
:---:|:---:|:---:|:---:|:---:
git reset -- |NO|YES|NO|YES
git checkout|NO|YES|YES|NO

**注意：执行非目录安全的命令操作的时候，一定要慎重，除非你非常清楚自己在做什么！**

“小明，你明白了吗？” 消息发送过去之后，等了很久却一直没有响应。
“哎，这孩子！估计听睡着了... 😆”

自从这次问到Git的问题后，已经两年过去了，小明再没有问到关于Git的问题。而就在昨天，突然又收到了小明的消息。

# 也许你应该试试Git Flow
“锋哥，我现在已经是Android Leader了。现在安卓团队一共6个人，我们现在在做一个社交类应用，在Git管理方面我还是发现了一些问题。其中一个问题就是，现在版本库有好多分支，其中开发主要在develop分支。主干分支是master主要用于版本发布。可还有一些分支却显得非常混乱，有什么办法改善这种情况吗？”

“关于Git的分支设计，目前有一个公认比较好的设计叫 [Git Flow模型](https://github.com/nvie/gitflow)。关于Git Flow模型，你可以查看这篇文章 [http://nvie.com/posts/a-successful-git-branching-model/](http://nvie.com/posts/a-successful-git-branching-model/) 了解一下"

# 一个idea，一次提交
"好的！还有一个困扰了我很久的问题是，大家的提交日志写的比较笼统。在查找问题的时候非常不便，而且大部分同学一次性提交好多文件，导致解决问题的时候不能准确定位到具体是哪一次提交导致的。我告诉大家，一次提交改动要尽可能小。但当别人问到具体的提交规则的时候我又不知道从何说起..."

“这是一个很好的问题 。中国程序员普遍存在的一个问题是，恨不得把这辈子能提交的代码一次性搞定。甚至有人用多次提交太麻烦的借口来搪塞问责人。简单来说，可以用一句话概括提交原则：一个idea，一次提交。另外，你说的没错，提交必须尽可能小，注释必须尽可能表述准确！”

给小明讲了这么多Git，我忍不住半开玩笑地问他，“小明，你现在还觉得Git简单吗？”

小明发了一个无奈的表情！说道，“以前是我才疏学浅，略知皮毛，不知道Git原来还有这么多玩法，忍不住为Git的发明者点赞了。对了，锋哥，Git到底是谁开发的？”

## Git的最大功臣，其实不是Linus
”关于Git的故事，互联网上其实已经烂大街了。我简单给你介绍一下吧！Git的诞生其实是一个偶然，其初始使命是为Linux内核代码管理服务的。早年的时候Linux内核源码是用Bitkeeper版本控制工具管理的。可是，后来因为某些利益关系，Bitkeeper要求Linux社区付费使用。这一举动激怒了Linus，也就是Linux的创始人，他决定自己开发一个分布式版本控制系统。几周时间下来，Git的雏形就诞生了，并且开始在Linux社区中应用开来。虽然Linus是Git的创始人，可是背后的最大功臣却是一个日本人 [Junio C Hamano](https://en.wikipedia.org/wiki/Junio_Hamano)。Linus在Git开源版本库的提交只有258次，而Junio C Hamano却提交了4000多次。也就是说，在Linus开发后不久项目的管理权就交给了这个日本人。关于 [Junio C Hamano](https://en.wikipedia.org/wiki/Junio_Hamano)，你感兴趣的话可以Google了解一下。他现在在Google工作，如同Linus一样非常低调。“

“这个故事也告诉我：不要用技术去挑战一个程序员 @_@ ”

这个故事讲完，小明与Git的故事就已经告一段落了。其实，还有一些比较常见的问题，小明并没有问到过。这里，我为你准备了一个附录，给你介绍一些常用的小命令帮你解决日常小问题。它很有用，一定要拿笔记下来，或者收藏这篇文章备用。

# 常见问题
**问题一：公司的Git服务器是搭建在一个内网服务器上面的，我想把代码同时提交到OsChina上面，以便在家拉取代码，远程办公，怎么办？**
Git本身是一个分布式的版本管理系统，实现这个需求非常简单，使用`git remote add`命令添加多个远程版本库关联即可。
```
git remote add company git@xxx
git remote add home git@xxx
```

**问题二：在拉取远程代码的时候，如果本地有代码还没有提交，Git就会提示先提交代码到版本库。可暂时我又不想提交，怎么办？**
针对这个问题，Git提供了一个临时区域用于保存不想提交的记录，对应的命令是`git stash`。通常情况下，你可以这样操作：
```
# 将暂时还不想提交的数据保存到临时区域，保存成功后，工作区将和版本库完全一致
git stash
# 还原stash数据到工作区
git stash apply
# 以上操作完成后，stash数据依然保存在临时区域中，为了删除这部分数据，使用如下命令即可。
git stash drop
# 如果你想在还原数据的同时从临时区域删除数据，可以这样操作：
git statsh pop
# 以上两个命令如果不接任何参数将删除掉所有的临时区域数据，如果你只想删除其中一条记录，指定对应索引数据即可。
git stash pop/drop stash@{index}
# 查看临时区域所有数据，使用如下命令：
git stash list
```

**问题三：作为项目负责人，我希望迅速找出问题代码的“元凶”，有什么办法吗？**
针对这个问题，最好的答案是`git blame`，使用这个命令并指定具体文件它将显示文件每一行代码的最近修改记录，你可以清晰地看到最近代码的修改人。

**问题四：部分Team Leader会要求使用`git rebase`合并代码，这有什么好处吗？**
我们用一个简单的思维来理解这个问题，最常见的合并操作是使用`git merge`，而这样操作会在合并分支生成一次新的提交，并且会严格记录分支提交日志，在长期开发过程中，日志就会呈现多条线路展示，给阅读带来一定的障碍。而使用`git rebase`会使整体代码提交记录始终像在单一分支开发一样，仅使用一条线路展示。但使用`git rebase`是有一定陷阱的，这个问题需要一定的时间才能说清楚，如果需要了解两个命令的详细区别，我推荐你阅读这篇文章 [Rebase 代替合并](https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/rebase)。

# 总结
Git是一个非常优秀的版本控制系统，我极力推荐你在日常开发中使用。这篇文章从小明的角度解释了几个常见问题的解决方案，毫无悬念地，你可能还会遇到其它的一些问题。遇到问题，你可以尝试使用Google搜索解决方案；也可以在文章下方给我留言，我非常乐意为你解答Git问题。

---
我是欧阳锋，版本控制，我使用Git。了解欧阳锋，从这里开始：[欧阳锋档案馆](https://www.jianshu.com/p/0c5d14fbaf1a)。