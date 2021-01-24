---
title: 浅析Ruby对象
tags:
  - Ruby
  - Ruby原理剖析
id: '1824'
categories:
  - - Ruby
abbrlink: fdf54046
date: 2021-01-06 09:11:03
---

```
每个 Ruby 对象都是类指针和示例变量数组的组合
```

Ruby将每个自定义对象都保存在了名为 RObject 的 C 结构体中，下图展示了该结构体的样子：

![](http://img.varsion.cn/blog-img/2021/01/image-7.png)

顶部的 VALUE 是 RObject 的指针，RObject 结构体包含了 RBasic 结构体和自定义对象特有的信息，一组叫做 flags 的布尔值，用来存储各种内部专用的值，还有一个叫做 **klass** 的**类指针**

类指针标明对象是哪个类的实例。在 RObject 中，Ruby 保存着每个对象都包含的实例变量数组，numiv是实例变量的数目，而 ivptr 是实例变量的值数组的指针。

### 有关 klass 和 ivptr

声明了一个简单的 Ruby 类：

```
class User
  attr_accessor :first_name,:last_name
end
```

Ruby 需要在 RObject 中保存类指针，因为每个对象都需要记录创建它的类。当类创建实例对象时，Ruby 内部会在 RObject 中保存指向该类的指针。

当然，同一个类的不同实例对象 (RObject) 的 klass 的值都会指向其类的结构体 (RClass)。

![](http://img.varsion.cn/blog-img/2021/01/image-8.png)

通过类名 #<User 显示了 li 对象的类指针值，后面的16进制字符串实际是该对象的 VALUE 指针。

![](http://img.varsion.cn/blog-img/2021/01/image-11.png)

Ruby 使用实例变量数组来记录保存在对象中的值。

### 有关基本类型对象

在内部，Ruby 使用了和 RObject 不一样的结构体来保存每个基本数据类型的值，例如 RString、RArray、RRegexp...

这些不同的结构体都包含了同样的 RBasic 结构体。

### 有关简单立即值

为了优化性能，Ruby 保存小值证书、符合和其他一些简单立即值时并没有使用任何结构体，只是将它们放到 VALUE 内：

![](http://img.varsion.cn/blog-img/2021/01/image-13.png)

这里的 VALUE 并不是指针，而是立即值本身，对这些简单的数据来说，并不存在类指针。Ruby 使用保存在 VALUE 中的前几个比特的一串比特标记来记忆这些值的类。例如，全部小值整数都有 FIXNUM_FLAG 位标记。

![](http://img.varsion.cn/blog-img/2021/01/image-15.png)

一旦 FIXNUM_FLAG 被设置，Ruby 就知道这个 VALUE 是一个小值整数，是 Fixnum 类的实例，而不是指向结构体的指针。

同样的，位标记也会标示 VALUE 是否为符号型，诸如 nil、true、false 这些值也有各自的标记

同时基本类对象也拥有实例变量，可以通过 instance_variables 方法，将一些基本数据类型的值插入其实例变量数组。

![](http://img.varsion.cn/blog-img/2021/01/image-16.png)

在字符串对象中保存实例变量

在内部，Ruby 使用了一点 Hack 手法来为基本类型对象保存实例变量——因为这些对象内部并没有使用 RObject 结构体。当在基本类型对象中保存实例变量时，Ruby 会把它保存在名为 generic_iv_tbl 的特殊散列中。

该散列维护这人基本类型对象和另外一些散列的**指针映射**，而那些散列包含了基本类型对象的所有实例变量。

![](http://img.varsion.cn/blog-img/2021/01/image-17.png)

generic_iv_tbl 为基本类型对象保存了实例变量

* * *