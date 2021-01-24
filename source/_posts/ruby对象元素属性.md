---
title: Ruby对象元素属性
tags:
  - Object
  - Ruby
id: '20'
categories:
  - - Ruby
abbrlink: 4c0576fa
date: 2020-07-23 11:25:49
---

Ruby中是没有属性(property/attribut)这样的东西。在Ruby中从对象外部不能直接访问实例变量或对实例变量赋值，需要通过该方法来访问对象的内部。假设有下面的例子：

```
class HelloWorld
  def initialize(myname = "Ruby")
    @name = myname
  end
end
bob = HelloWorld.new("ruby")
bob.name = "Bob"
puts bob.name
```

![](http://img.varsion.cn/blog-img/2020/07/image-20200720164044888.png)

在Ruby中，如果是括号内的参数，括号是可以直接省略的。所以，对于`bob.name`，由于Ruby中是没有属性的，其实我们调用的是`bob.name()`中`.name()`方法。对于`bob.name = "Bob"`实际是执行`bob.name = ("Bob")`方法，上面两个方法都没有定义所以会报错。另外比如一个简单的`1 + 1`并不是一个简单的数学运算，而是`1.+(2)`，是执行了数字1的`+`方法，其中参数是2。那么在Ruby中，如何定义对实例变量的访问和变更呢？

```
class HelloWorld
  def initialize(myname = "Ruby")
    @name = myname
  end

  def name
    return @name
  end

  def name= (value)
    @name = value
  end

end
bob = HelloWorld.new("Jhon")
bob.name = "Bob"
puts bob.name
```

上面的 `def name`方法就相当于我们所知道的getter方法，`def name=`就是我们所知道的setter方法。

但是如果实例变量太多，如果都这么定义岂不是很麻烦，所以Ruby为我们提供了简便定义的方法`attr_reader,attr_writer,attr_accessor`只要指定了变量名的符号，Ruby就会为我们定义响应的存储器。

定义

含义

attr_reader :name

只读(定义name方法)

attr_writeer :name

只写(定义name=方法)

attr_accessor:name

读写(定义以上连个方法)

```
class HelloWorld
  def initialize(myname = "Ruby")
    @name = myname
  end

  # def name
  #     return @name
  # end

  # def name= (value)
  #     @name = value
  # end
  attr_accessor :name
end
bob = HelloWorld.new("Jhon")
bob.name = "Bob"
puts bob.name
```

* * *