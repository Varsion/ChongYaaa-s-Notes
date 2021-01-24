---
title: Ruby - 基础匿名函数 Proc
tags:
  - Proc
  - Ruby
  - 匿名函数
  - 可调用和可运行对象
id: '1006'
categories:
  - - Ruby
abbrlink: f4495147
date: 2020-10-13 20:37:19
---

最简单的来说，可调用对象的概念表现为：通过接受消息 call 的对象与某些代码关联，并执行这些关联代码以获得预期的结果。

Ruby中最主要的可调用对象是 Proc 对象、lambda 表达式、方法对象。Proc 对象是自包含的代码序列，可以创建、存储、作为参数传递，还可以使用 call 方法执行。

## Proc 对象

使用 Proc.new 创建 Proc 实例。通过实例化 Proc 类并包含代码块，创建 Proc 对象：

```
pr = Proc.new { puts "Proc's Block" }

pr.call
# Proc's Block
```

当一个代码块被应用到对 Proc.new 的调用中后，变为了 Proc 对象的主体，调用该对象的时候，这个代码块就会被执行。

## Proc 和 代码块的区别

创建 Proc 对象是，总要提供一个代码块，但不是没个代码块都可以作为 Proc 的主要成分。

```
[1,2,3].each { x puts x * 10 }
```

涉及一个代码块但是并没有创建 proc。事情要比之前的复杂一些，这样的语法可以捕获一个代码块，并将其对象化成为proc：

```
def call_a_proc(&block)
  block.call
end

call_a_proc { puts "proc's block" }
```

通过使用相似的特殊语法，proc 能够代替在方法调用时的代码块：

```
p = Proc.new { x puts x.upcase }
%w{ David Black }.each(&p)
# DAVID BLACK
```

在很大程度上，代码块和 proc 的关系实质上是语法与对象的问题。

### **语法（代码块）和对象（proc）**

一个重要的且常见的误解是认为 Ruby 的代码块不是对象。

```
[1,2,3].each { x puts x * 10 }
```

这是一个微不足道的例子，其中包含一个接受者、点运算符、方法名和代码块：

接受者是一个对象，而代码块不是，当然，代码块页是方法调用语法的一部分。

因为代码块和 proc 的交互操作，代码块的语法相比参数列表的情况复杂一些，Proc的实例是对象，而代码块要包含任何逻辑都需要创建 proc。这就是 Proc.new 要携带代码块的原因：在它被调用更多时候，如何找到 proc 应该执行哪些逻辑的方式。

又一个重要的含义是，代码块是句法构造，而不是方法参数。为方法提供参数的问题在于代码块是否独立出现，就像一个参数列表是独立出现还是缺失的。当提供了一个代码块，并不是将代码块作为参数传递给方法，他只是代码块本身而已。现在从另外的角度，近距离的看一下转换机制，它允许代码块作为 proc 被捕获，同时也允许 proc 暂时作为代码块的替身。

## 代码块与 proc 相互转换

代码块和 proc 相互转换非常容易，因为代码块存在的目的就是被执行，而 proc 是对象，他的任务就是执行之前定义好的代码块。

### 捕获代码块作为 proc

```
def capture_block(&block)
  block.call
end
capture_block { puts "Inside the block" }
```

该方法会捕获自己的代码块作为 proc 对象然后调用该对象，这是对 Proc.new 隐式调用的一种情况，与使用代码块相同，通过它创建的 proc 绑定到了参数 block 上。

![](http://img.varsion.cn/blog-img/2020/10/image-1.png)

语法元素（代码块）是创建一个对象的基础，从代码中创建 proc 的这个“影子”步骤也可以解释为对特殊的基于 & 语法的需求。方法调用可以同时包含参数列表和代码块。如果没有特殊的标记 & ，Ruby没有办法得知用户想要停止绑定参数到常规参数上，并将代码块通过转换变成 proc 并保存这个结果。

& 标记也会出现在执行另一个转换操作时：使用 Proc 对象代替代码块。

### 对代码块使用 proc

```
p = Proc.new { puts "This proc" }
capture_block(&p);
# This proc
```

使用 proc 作为代码块的关键在于可以真正的使用它代替代码块：将 proc 作为参数发送给正在调用的方法。如同在方法定义中使用 & 字符标记参数指明这个参数吧代码块转化为 proc 一样，也可以使用 & 表明 在方法调用是 proc 应该要完成的代码块任务。

因为使用 & 标记的 proc 正在作为代码块使用，因此不能再将代码块发送到同一个方法调用中（会导致 “ both block arg and actual block given" 错误

Ruby 不能决定哪一个实体（即 proc 和代码块）可以作为代码块，因为只能使用一个。

如同Ruby运算符，在 &p 中 & 是对方法的包装：从名称上来说，是 to_proc 方法。在 Proc 对象上调用 to_proc 则会返回 Proc 对象本身，相当于在字符串上调用 to_s 或在整数上调用 to_i。

在 capture_block(&p)中的 & 可以完成两件事：触发了对 p 的 to_proc 方法的调用，然后告诉Ruby将 Proc 对象的结果作为代码块的替身。

最后，因为 to_proc 是一个放，可以把它用在更为通过的方式中。

### to_proc 方法概述

理论上，可以在任何类中或者任何对象中定义 to_proc 方法，然后这些受影响的对象就可以运用 & 标记的技术。

```
class Person
  attr_accessor :name
  def self.to_proc
    Proc.new { persom person.name }
  end
end
d = Person.new
d.name = "David"
m = Person.new
m.name = "Varsion"
puts [d,m].map(&Person)
```

这段代码中，存在一个包含两个 Person 对象的数组，并对数组执行了 map 操作，这个 proc 被指定为参数列表中更多 &Person。当然 Person 不是一个 proc，它是一个类。为了使其可行，Ruby 请求 Person 转变为一个 proc ，这样就会隐式的调用 Person 的 to_proc 方法。

to_proc会产生一个简单的 Proc 对象，它携带一个参数然后在这个参数的基础上调用 name 方法。Person对象本身拥有 name 特性，然后用于测试这段代码而创建的 Person 对象有自己的名称，对 Person 对象数组映射 ([d,m]) ，其所有意义是为了收集这些对象的 name 特性，并将整个结果数组打印输出。

这个过程复杂，而且涉及松散。毕竟任何携带代码块的放都可以使用 &Person ，如果涉及一个非 Person 且没有 name 方法的对象，这回有点奇怪。但是在这个例子中，to_proc 可以作为一个强大的转换钩子。

## 简洁的 Symbol#to_proc

内置方法 Symbol # to_proc 在如下情况中使用：

```
%w { red yellow blue }.map(&:capitalize)
# ["RED","YELLOW","BLUE"]
```

:capitalize 被解释为一次发送到数组中的每个元素的消息，因此上述代码的作用含义和下面这段代码相等：

```
%w { red yellow blue }.map{ str str.capitalize}
```

但是正如代码展示的，第一段代码会更为整洁。&:capitalize 或者相似的构造，很清晰的解释了他的原理：符号 :capitalize 和 to_proc 的标识符 &。

Symbol # to_proc 还可以更好的用于不实用圆括号的情况

```
%w { red yellow blue }.map &:capitalize
```

### 实现 Symbol # to_proc

再研究一下 to_proc 的例子，它等同于

```
%w { red yellow blue }.map{ str str.capitalize}
```

同时也等同于

```
%w { red yellow blue }.map{ str str.send(:capitalize)}
```

通常来说，不用为他编写什么，因为如果能用常规的点运算符的语法调用，就不必费力的去使用 send 。但是基于 send 编写的版本可以指明使用 Symbol # to_proc 的实现方式。在本例代码块中的任务是为了发送符号 :capitalize 给数组的每个元素。这就意味着，通过 :capitalize#to_proc 创建的 Proc 一定会将 :capitalize 作为参数发送给它自己。

可以提出对这个 Symbol # to_proc 的简单实现：

```
class Symbol 
  def to_proc
    Proc.new( obj send(self)
  end
end
```

这个方法会返回一个 Proc 对象，该对象带有一个参数并且会发送 self （它是以后应用中任何符号）参数给自己。

## proc 作为闭包使用

当代码块作为到调用对象的主体时，有关变量作用域的问题会变得有趣起来

```
def multipy_by(m)
  Proc.new { x puts x*m }
end
mult = multipy_by(10)
mult.call(12) # 120
```

这个例子中，multipy_by 返回了一个 proc 这样就可以使用任何参数调用它，但是被乘数还是需要作为参数传递给 multipy_by 。变量 m 无论它是什么值，都会包含到代码中，并传递给 Proc.new，并因此作为乘数，在每次调用 multipy_by 后由 Proc 返回。

Proc 对象在不同的作用域中稍有不同，在为了调用 Proc.new 而构建代码块时，已经创建的局部变量仍然在其作用域中（使用任何代码块都一样）。并且，不管在何时何地调用它，这些变量都会保留在 proc 的作用域中。

Proc 对象会保存其上下文，包括上下文变量的赋值操作。

携带上下文的那个代码片段称为一个闭包。创建一个闭包如同打包一个箱子：无论在那里打开箱子，他所包含的东西与打包时一致。当打开一个闭包时，它包含当内容都是创建时所包含的。闭包非常重要，因为他们能够保存一个程序的部分运行状态。在方法返回时，离开作用域的变量可能会包含一些有用的信息。

### Proc 的形式参数和实际参数

下面是 Proc 的实例化过程，并携带一个有参数的代码块：

```
pr = Proc.new { x puts "Called whit #{x}" }
pr.call(100)
```

proc 与方法的不同在于参数处理的方式，因为他们并不关心参数的树立那个是否正确，如果调用多于一个参数，那么单一的参数会被绑定到第一个参数，其他参数将被丢弃。

当然，也可以使用一些方法来“吸收”参数和其他所有的参数列表的工具。

唯一重要的一点是，在参数处理上 proc 要比方法少一些麻烦。

* * *