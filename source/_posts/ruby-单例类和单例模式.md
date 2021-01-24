---
title: Ruby - 单例类和单例模式
tags:
  - Ruby
  - 单例类
id: '985'
categories:
  - - Ruby
abbrlink: 8bf4c840
date: 2020-10-09 08:53:34
---

"单例"一词在Ruby中有第二个不同的含义：单例模式，描述一个类只能有一个实例。Ruby标准库中包含对单例模式的实现，通过 require "singleton 就可以使用。单例类并没有直接与单例模式相关联，"单例"这个词有着更多的重载的意思。

* * *

Ruby中涉及类和模块最常见的就是实例方法的定义，紧接着就是类的实例化以及这些实例方法的调用。

同时，还可以直接在独立对象上定义单例方法：

```
obj = Object.new
def obj.talk
    puts "hello"
end

obj.talk
```

几乎任何的对象都能将单例方法添加到其内部，基于独立对象的本质来定义行为的能力是Ruby设计的标志之一。

### 通过单例类双重决定

对象的单例方法存在与对象的单例类中，每个对象都有两个类

*   实例方法所属的类
*   它的单例类

对象能调用他原生类中的实例方法，也能够调用他单例类中的方法这是双重的。对象可调用方法的总数量等于定义在这两个类中的实例方法的数量，加上贯穿其祖先类（对象类的超类，类的超类，以此类推）或者任何混合(mix)或前置(prepend)模块的可用方法的数量。可以把对象的单例类看作是某些方法的特殊存储空间，是为该对象特质的，不能与其他对象共享的。

### 检查和修改单例类

单例类是匿名的：尽管是类对象的实例，但是他们会自动出现且不需要指定名字。但是，可以打开单例类的类定义主体，然后添加实例方法、类方法和常量，如同常规类一般。

使用 class 关键字的特殊形式可以做到这一点。

```
class C
    # method and constant def
end
```

未来进入单例类的定义主体，需要以这个特殊的符号

```
classs object
    # method and constant def
end
```

<< object 符号意味着 object 的单例类是匿名的。当位于单例对象定义主体中时，就可以定义方法，然后这些方法就会成为单例方法，只属于当前单例类的对象。

```
str = "A String"
class << str
    def twice
        self+" "+self
    end
end
puts str.twice
# A String A String
```

`twice`方法是字符串`str`的单例方法，同时，完全可以这样做

```
def str.twice
    self+" "+self
end
```

与上一代码块的区别只在于"撬开"了`str`的单例类，然后在那里定义的方法。

#### def obj.method和 class << obj def method 的区别

本质上是没有区别的，唯一区别是**常量解析方式不同**:

如果有一个顶层常量N，那也可以的定义一个常量N在对象的单例类中。

```
N = 1
obj = Object.new
class << obj
    N = 2
end
```

这两种方式的不同点在于对常量的解析方式

```
def obj.method
    puts N
end
# 输出 1 （外层的N

def << obj
    def method_s
        puts N
    end
end
# 输出 2 （内层的N 属于obj的单例类
```

这个区别在常量可见性对代码的影响上比较少见，大多数情况下，单例方法定义的两种符号都是可以互相替换的。

#### 使用 class << 定义类方法

```
class Ticket
    class << self
        def most_exp(*tickets)
            tickets.max_by(&:price)
        end
    end
end
```

使用`class << object`方式打开了对象的单例类，然后在这个特殊的例子，涉及的对象是类对象`Ticket`，在代码中调用`class << self`时`self`的值。把该方法定义在类定义快中，其结果是得到定义在`Ticket`的单例方法，可以说这是**类方法**。

同样的，类方法也可以这样定义

```
class << Ticket
    def method
# etc
```

因为在`class Ticket`定义的主体中，self就是Ticket，在主体的`class << self`与主体外的`class << Ticket`相同，但是在实践中通常只会看到`class << self`，任何时候，对象方法只需要打开的是`self`。

### singleton_class 方法

如果要直接引用对象的单例类，可以使用`singleton_class`方法，这个方法可以减少对`class << object`的反复使用。

下面展示了如何使用该方法来得到对象的单例类的祖先列表

```
string = "A String"
puts string.singleton_class.ancestors
```

* * *