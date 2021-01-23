---
title: Ruby – 枚举 Enumerable (二)
tags:
  - Enumerable
  - Ruby
id: '856'
categories:
  - - Ruby
date: 2020-09-18 20:30:58
---

集合类型就是为遍历出现的，而他们可以包含特殊状态的独立对象：集合类型中的第一个或最后一个，最大一个或者最小一个。`Enumable`对象带来了许多用于元素处理的工具。

### `first`

就像这个名字一样`Enumable # first`方法返回可枚举对象迭代时的第一个元素。

通过`first`方法返回的对象和使用迭代时第一个得到的对象。换句话说，它是通过`each`方法第一个传递到代码块中的元素。与散列以两个元素的数组传递键值对的情况是一致的，获取散列的第一个元素会得到以两个元素形式表示的键值对数组。

需要知道的时，并没有和`Enumable # first`相对应的`Enumable # last`，是因为寻找迭代的末尾不像寻找其开端一样简单明了。

### `take`方法和`drop`方法

可枚举对象知道如何从它自身的起点得到指定数量的元素，相反的，他也知道如何排除指定数量的元素。`take`和`drop`方法从本质上完成的是相同的事情，他只使用指定更多位置拆分集合，而不同点在于返回值不同。

```
a = [1,2,3,4,5,6]
a.take(2)
# [1,2]
a.drop(2)
# [3,4,5,6]
```

`#`为输出

只要获取元素，就会得到这些元素。而排除一些元素，就会得到从原始集合中键取要排除的元素而剩下的元素。可以通过使用代码块以及`take`和`drop`的变化形式`take_while`和`drop_while`来约束操作，他们会根据代码块的返回值是否为真来决定获取元素的数量。

### `min`方法和`max`方法

`min`方法和`max`方法的用法同其名称一样，但是其的大小判断是通过`<=>`逻辑来判定的。

这里要讲的是他们的衍生方法：`min_by`、`max_by`以及`minmax`和其对应的`minmax_by`。

```
%w{ Ruby C C# PHP JavaScript}.max_by { lang lang.size}
# JavaScript
%w{ Ruby C C# PHP JavaScript}.minmax_by { lang lang.size}
# C  JavaScript
```

* * *