---
title: Git协同开发流程
tags:
  - Developer
  - Git
id: '805'
categories:
  - - Developer
keywords: Git,Develop,Developer
abbrlink: 802ded82
date: 2020-09-18 13:36:49
---

![](http://img.varsion.cn/blog-img/2020/09/image-15.png)

开发流程

## 具体操作

每次项目开始时，我会将基础代码框架并配置需要用到的组件，然后将基础代码包更新到我自己的仓库中。

![](http://img.varsion.cn/blog-img/2020/09/image-16-1024x282.png)

### Fork

需要点击右上角的 forl 按钮，将主仓库的代码 fork 到自己的仓库中。

![](http://img.varsion.cn/blog-img/2020/09/image-19.png)

### Clone

然后转到自己的Github仓库，将Fork来的代码 clone 到本地。

![](http://img.varsion.cn/blog-img/2020/09/image-20.png)

然后在[Cmder](https://cmder.net/)，或者Git命令行中进行代码 clone，将代码 clone 本地。

![](http://img.varsion.cn/blog-img/2020/09/image-21.png)

### push

在完成自己的部分代码的开发之后，需要将自己的代码推到自己的远程仓库中

这边举一个例子：

![](http://img.varsion.cn/blog-img/2020/09/image-22.png)

这是一份已经 clone 到本地并完成的代码，假设我现在修改/新增了文件：test.md

![](http://img.varsion.cn/blog-img/2020/09/image-23.png)

我现在要如何将该文件推送到自己的Github，然后应该如何再将该文件的改动推送到源仓库。

首先是切换到 develop 分支，将 test.md 添加至本地仓库，并设置提交信息，然后推送到自己的仓库中。

![](http://img.varsion.cn/blog-img/2020/09/image-26.png)

依次的代码是：

```
git checkout develop
git add test.md
git commit -m "The test"
git pust origin develop # 第一次执行时会提示填写用户名和密码
```

这时候，我们会在自己的代码仓库中看到刚刚提交的修改（注意切换到 develop 分支

![](http://img.varsion.cn/blog-img/2020/09/image-28.png)

![](http://img.varsion.cn/blog-img/2020/09/image-30.png)

至此，我们已经将修改后的代码/文件推送到了自己的Github仓库。

### 提交主仓库请求

![](http://img.varsion.cn/blog-img/2020/09/image-31.png)

我们要 Pull Request 中提交请求。

![](http://img.varsion.cn/blog-img/2020/09/image-32-1024x259.png)

尤其是注意，要从自己的 develop 分支提交到源仓库的 develop 分支，下文会讲述详细的原因。

![](http://img.varsion.cn/blog-img/2020/09/image-33-1024x223.png)

然后写明提交内容，点击提交按钮：

![](http://img.varsion.cn/blog-img/2020/09/image-34.png)

我会再主仓库检查并核对代码之后，将其合并到主分支中。

![](http://img.varsion.cn/blog-img/2020/09/image-35.png)

## develop 分支和 master 分支

![](http://img.varsion.cn/blog-img/2020/09/image-36.png)

关于 develop 分支和 master 分支，master 分支是建立代码库是就拥有的分支，而 develop 分支是代码仓库建立后，建立该分支用来开发。

所有开发者开发好的功能会在源仓库的 develop 分支中进行汇总，当 develop 中的代码经过不断的测试，已经逐渐趋于稳定了，接近产品目标了。这时候，我们就可以把 develop 分支合并到 master 分支中，发布一个新版本。

当然我们也可以把 develop 分支命名为 demo 分支，或者其他好听的名字。但是其意义是不变的。

```
这里放上Github链接：欢迎大家的关注和Star
[CoderTH](https://github.com/coderTH)
[Varsion](https://github.com/Varsion)
```

* * *