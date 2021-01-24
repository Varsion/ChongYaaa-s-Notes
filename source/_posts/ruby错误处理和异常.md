---
title: Ruby错误处理和异常
tags:
  - Ruby
id: '676'
categories:
  - - Ruby
abbrlink: cad046e
date: 2020-09-09 14:53:15
---

## 引发和捕获异常

异常(exception)是一种特殊对象。是`Exception`类或者子类的实例。引发一个异常意味着停止程序的正常执行，之后要么处理初心的问题，要么完全退出程序。

处理问题还是退出程序，取决于是否使用了`rescue`子句。如果程序中有该子句，程序控制流将会移交该子句控制。

## `rescue`关键字

引发异常并不意味着程序的终结。可以对异常进行处理，处理发生的问题并保持程序的运行。

`rescue`代码段用于挽救关键程序，它被限定于`begin`和`end`关键字的范围，并在中间的位置放置一个`rescue`子句

```
print "Enter a number"
n = gets.to_i
begin
  result = 100 / n
rescue
  puts "Error Number,was it 0?"
  exit
end
puts "100/#{n} is #{result}"
```

如果这段程序输入0，除法运算 100 / 0 会引发`ZeroDivisionError`异常，由于已经在`begin/end`中包含了一个`rescue`语句，控制流能顺利转入`rescue`语句。错误信息会被打印输出，而后程序正常退出

假如输入的是0以外的数据,运算将会成功,程序控制也将会跳过`rescue`语句,然后在那之后将会恢复执行.

`begin/end`的存在是为了限制`rescue`的控制粒度(细腻程度).

如果在一个方法中没有使用`begin/end`进行包装,就容易对整个方法的所有代码进行异常控制,控制粒度不明确,会引发不必要的麻烦.

## 显示的引发异常

为了引发一个异常,可以使用`raise`加上想要引发的异常的名称.加入没有提供异常的名称(并且加入没有重新引发一个不同的异常)Ruby则会引发一个通用的`RuntimeError`

也可以给`raise`指定第二个参数,他被用于在异常引发是的说明信息

```
def Hello
  raise NameError,"go out"
end
```

## 捕获异常

为了将异常对象赋值给一个变量,可以配合`rescue`指令使用特殊的操作符`=>`

异常对象如其他对象一样可以相应消息,尤其有用的是`backtrace`和`message`方法

*   `backtrace`可以返回一个字符串数组,用于表示异常出发时的调用栈:它包括方法名称,文件名和代码行数,并展示了异常发生时代码执行的整个路线图.
*   只要信息存在`message`方法就可以返回在`raise`提供的信息

```
...
  rescue ArfumentError => e
puts e.backtrace
puts e.message
...
```

### 被引发的是一个异常还是一个异常类?

在编程的角度上,异常触发的是类:使用`raise ZeroDivisionError`而不是`raise ZeroDivisionError.new`

但实际上是异常类的实例被引发了.这个语法引发了一个类,因为那笔实例化看起来更为贴切和抽象,假如在`rescue`子句中测试自己捕获的异常对象,可以看到类和实例中的关系是断开的.

不适用`new`也达到了相同的效果,并使得代码有一个较高层次的样式,以提供足够的信息里展示代码余小宁的过程,从而不必关心其操作的细节.

* * *