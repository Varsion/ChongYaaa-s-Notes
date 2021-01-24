---
title: Ruby中Super向上追溯方法路径
tags:
  - Ruby
  - Super方法回溯
id: '618'
categories:
  - - Ruby
abbrlink: b99e89a0
date: 2020-09-03 15:52:16
---

在方法定义主体中，`Super`关键字可以在当前执行的方法查找路径的方法中，跳转到下一个高级定义。

```
module M
  def Report
    puts "this is in module M"
  end
end

class C
  include M
  def Report
    puts "this is in class C"
    super
    puts "back from the super call"
  end
end

c = C.new
c.report
```

该份代码的输出结果是：

```
this is in class C
this is in module M
back from the super call
```

一个C的实例接收到了`report`消息，方法查找过程将会从类C开始，非常肯定这里是有一个`Report`方法，并且被执行了。

在方法内部有一个`super`调用，这说明即使对象找到了一个方法去响应该消息，也必须去继续搜索和查找以进行下一次匹配。

如果类C中不存在该方法，那么模块M中的`report`方法将会首先被匹配到。如果某个合适版本的方法在查找路径的后期被覆盖，`super`关键字提供了一种调用这个方法的方式。

有时，在编写一个子类的时候，存在类中的一个方法的功能已经几乎满足了用户的期望。但也不完全是这样，使用`super`可以同时拥有钩子和包装原始方法机制。

同时`super`处理传递参数的方式如下：

*   无参数列表调用`super`自动转发被调用的方法的实参
*   空参数列表调用`super()`不传递参数到更高一级的调用中。即使当前方法接受了传参
*   指定参数的调用`super(a,b,c)`精准转发这些参数

* * *