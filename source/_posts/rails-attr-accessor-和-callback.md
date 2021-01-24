---
title: Rails attr_accessor 和 callback
tags:
  - Active Record
  - Rails
  - Ruby
id: '1812'
categories:
  - - Ruby
  - - Ruby on Rails
abbrlink: 6052f971
date: 2021-01-04 09:58:10
---

开发的时候遇到的一个愚蠢的、耗费了超多时间的错误

在一条数据创建时，需要额外生成一些无法在表单提交中填入的数据，就比如说：

现在一个组织注册，需要自动生成该组织的邀请码，组织管理员可以通过将该邀请码发送给其他用户从而邀请其他用户加入自己的组织

看，很简单的一个业务，仅仅需要一个 before_create 即可完成的一个数据回调，然后按照自己的理解，将改字段添加了 attr_accessor

![](http://img.varsion.cn/blog-img/2021/01/image-1.png)

![](http://img.varsion.cn/blog-img/2021/01/image-3.png)

然后，无止境的 bug 就涌出来了，数据可以正常创建，但就是这个回调始终无法运行，甚至去找了好多相关的帖子，都没有对这种情况的描述，直到刚刚，抱着尝试的心态将其注释掉，好的，完美运行。

然后，现在的疑问是 attr_accessor 和 callback 之间有什么关联影响嘛？

## 有关attr_accessor

attr_accessor 可以将类中的一个属性暴漏在外，可以直接通过其属性名对他的值进行操作。相当于为这个属性分配了getter 和 setter 方法。

我在一篇博客中看到了这样的描述（[What is attr_accessor in Rails?](https://rubyinrails.com/2014/03/17/what-is-attr_accessor-in-rails/)

![](http://img.varsion.cn/blog-img/2021/01/image-5.png)

即，如果我把这个属性设置为 attr_accessor 就默认不把其添加到数据库中，而仅仅是将其作为一个类属性或者对象属性进行操作使用。

而和 callback 的关系，也自然无法将其存到数据库中了。

* * *