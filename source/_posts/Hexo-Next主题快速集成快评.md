---
title: Hexo Next主题快速集成快评
date: 2019-09-25 13:03:00
tags:
---

### 快评是什么
快评是由[深圳市一行代码科技有限公司](https://yhdm360.com)开发的一款社会化的评论系统，快评使用Markdown编辑器进行评论回复。目前，支持使用微信或手机号进行登录。

体验地址：[https://www.youngfeng.com/2019/09/25/Hexo-Next%E4%B8%BB%E9%A2%98%E5%BF%AB%E9%80%9F%E9%9B%86%E6%88%90%E5%BF%AB%E8%AF%84/](https://www.youngfeng.com/2019/09/25/Hexo-Next%E4%B8%BB%E9%A2%98%E5%BF%AB%E9%80%9F%E9%9B%86%E6%88%90%E5%BF%AB%E8%AF%84/)

<!-- more -->

### 集成步骤
1）注册并登录快评官网

官网地址：[https://kuaiping.yhdm360.com/](https://kuaiping.yhdm360.com/)

点击右上方登录按钮，使用手机号加验证码的方式登录。首次登录后会看到**申请免费使用**的按钮，点击申请免费使用，填入申请域名，域名校验通过后你将看到类似下面的页面：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925124239651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FwcGxlMzM3MDA4,size_16,color_FFFFFF,t_70)

点击下方按钮显示接入方法，你将看到类似下图页面：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925124450302.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FwcGxlMzM3MDA4,size_16,color_FFFFFF,t_70)
点击红色箭头所指向的按钮，复制生成的app id。

2）使用kuaiping-adapter快速集成

**kuaiping-adapter**: [https://github.com/yuanhoujun/kuaiping-adapter](https://github.com/yuanhoujun/kuaiping-adapter)

点击上方链接进入进入kuaiping-adapter版本库，按照下方文档复制next文件夹到themes文件夹中，合并到原有主题文件夹中。![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925124848382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FwcGxlMzM3MDA4,size_16,color_FFFFFF,t_70)
通过上面两个步骤，如果不出意外的话，恭喜你，你已经成功集成[快评](https://kuaiping.yhdm360.com)评论系统了。如果你还有任何的其它问题，请扫描下方二维码加群。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925125438656.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FwcGxlMzM3MDA4,size_16,color_FFFFFF,t_70)
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4ua3VhaXBpbmcueWhkbTM2MC5jb20vcHViL2NvbW1vbi9pbWFnZS93eGdyb3VwLnBuZw?x-oss-process=image/format,png)

**备注：如果上方二维码失效，请直接添加群主微信ouyangfeng_office，备注“快评”**

### 参考文档
快评官网：[https://kuaiping.yhdm360.com](https://kuaiping.yhdm360.com)

Kuaiping-adapter: [https://github.com/yuanhoujun/kuaiping-adapter](https://github.com/yuanhoujun/kuaiping-adapter)

Next: [https://github.com/iissnan/hexo-theme-next.git](https://github.com/iissnan/hexo-theme-next.git)
