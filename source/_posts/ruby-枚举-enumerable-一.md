---
title: Ruby - 枚举 Enumerable (一)
tags:
  - Enumerable
  - Ruby
id: '741'
categories:
  - - Ruby
date: 2020-09-18 11:06:32
---

所有集合类型对象的创建都不一样，但是他们中大多数都有共通的特性。在Ruby中，集合类型对象通常都包含`Enumerable`模块。

使用`Enumerable`的类有一种协定：该类必须定义一个名为`each`的实例方法，而同时该模块也赋予了该类一组集合类型的相关行为。

```
class C
    include Enumerable
  def each
    # code
  end
end
```

### 依赖`each`获得枚举能力

任何可枚举的类都具有一个`each`方法（必须）。其作用是将其中的元素逐个的做为参数传递给代码块（`yield`）

*   在数组中,`each`依次传递各个元素给代码块
*   在哈希中,则是以两个元素的数组来传递键值对
*   在文件处理的情况下,会依次传递文件中的第一行给代码块
*   范围则会首先判断迭代是否可用(如起点是浮点数的情况)然后会伪装成数组进行传递

而如果在自己的类中定义了`each`方法,就可以将其自定义,只要传递参数给代码块即可

但是无论怎样,只要实现了`each`,就可以通过他来调用`Enumerab`模块中的方法

```
class Rainbow
  include Enumerable
  def each
    yield "red"
    yield "orange"
    yield "yellow"
    yield "green"
    yield "blue"
    yield "indigo"
    yield "violet"
  end
end
```

`Rainbow`的每个实例都可以迭代这些函数,最简单的情况下可以这样使用`each`来遍历所有元素

```
Rainbow.new.each do color
   puts "Color is #{color}"
end
```

### 可枚举对象的搜索和选择

`Enumerable`提供了很多功能,可用于在集合类型对象中根据一些条件过滤和搜索其中的一个或多个元素

这些条件过滤和搜索方法中,所使用的全都是迭代器,全都需要提供一个代码块,而这个代码块就是一个选择过滤器,可以在该代码块中定义自己的过滤条件.整个方法的返回值也会不相同,可能会返回一个对象,一个包含匹配对象的数组(数组可能为空)或者没有任何条件匹配时返回`nil`

#### 使用`find`方法进行第一次匹配

通过将数组中的元素作为参数传递个代码块并执行条件判断

`find`(还有一个可用的同义方法`detect`)会定位数组中第一个条件判断为真的元素例如

```
[1,2,3,4,5,6,7,8].find {n n>5}
```

这行代码会返回`6`,即第一个符合条件判断为真的元素

`find`迭代整个数组,将每个元素依传递到代码块中.如果代码使用布尔值为真的形式作为返回值,那么当前的元素就获得了优先,`find`代码就会停止迭代,如果`find`并没有查找到符合的元素,将会返回`nil`

而如果只是为了测试该数组或者该对象中是否存在一个值,更建议的方法是使用`include`

```
[1,2,3,4,nil,5,6].find {n n.nil?}
```

如果在当前情境下使用`find`方法,无论是否存在元素为`nil`该代码都将返回`nil`这个时候`include`更能解决问题`array.include?nil`

#### `find_all`和`reject`获取所有匹配元素

`find_all`又名`select`会返回一个新集合类型的对象,其中包含了在代码块中所有匹配从原始集合类型对象中得到的元素,而不仅仅是第一个匹配到的元素

如果没有匹配到任何元素,会返回一个空类型集合对象

```
[1,2,3,4,5,6,7,8,9].find_all {n n>5 }
```

这段代码会返回`[6,7,8,9]`即原数组中所有符合`n>5`的元素

同时数组,散列,集合都有`select!`的bang方法版本,会让源集合只留下条件判断为真的数据,不过不存在`find_all!`必须使用`select!`

`reject`方法可以排除元素,即将所有符合条件判断的数据剔除

同时,`reject`存在bang方法`reject!`会直接在源数据上修改.适用于数组,哈希,集合

#### 基于三等号匹配的`grep`来选择元素

`Enumerab#grep`方法会基于`case`的相等性运算符`===`,从可枚举对象中选择元素,`grep`最常见的用户,即字符串匹配模式

`grep`的普遍性可以让我们做一些花哨的工作

```
array.grep(/o/)      # 利用正则匹配元素
array.grep(String)   #利用类型匹配元素
array.grep(50..100)  #利用范围匹配元素
```

总言之`array.grep(expression)`等同于:

```
array.select [ element expression === element}
```

#### `group_by`和`partition`

`group_by`操作包含一个代码块并返回一个散列

对于每个对象,代码块都会被执行一次,每个独立代码块返回的值,最终会作为返回值的散列键,而键对应的值则是一个数组,该数组的元素包含哪些可枚举对象中的元素是通过代码的返回值来确定的.

```
colors = %w{red orange yellow green blue}
colors.group_by {color color.size}
```

改代码的返回值则是如下的散列

![](http://img.varsion.cn/blog-img/2020/09/image-11.png)

根据其字符串的长度来分组

`partition`会基于代码块是否返回为真,来将可枚举对象的元素分割为两个数组

```
[1,2,3,4,5,6,7,8].partition {x x%2 == 0}
```

![](http://img.varsion.cn/blog-img/2020/09/image-12.png)

* * *