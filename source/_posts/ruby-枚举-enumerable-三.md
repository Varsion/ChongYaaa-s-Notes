---
title: Ruby – 枚举 Enumerable (三)
tags:
  - Enum
  - Ruby
id: '879'
categories:
  - - Ruby
date: 2020-09-28 10:42:40
---

## `Each`相关方法

`Enumable`对象有很多与`each`详细的方法，他们可以遍历整个集合并且传递其中的元素到代码块，直到完全遍历完成迭代才会停止。这些方法体系中的每个成员都有自己特定的语义和定位。

### `reverse_each`

该方法可反向迭代一个枚举对象。

```
[1,2,3].reverse_each { e puts e*10}
# 30
# 20
# 10
```

需要注意的是，不要将该方法用于无限的迭代器中，因为反向的概念需要依赖于最后一个元素，而对于反向迭代器而言，使用该方法是毫无意义的。

### `each_with_index`与`each.with_index`

`each_with_index`在遍历结合时，每次会传递一个额外的参数到代码块中，从命名上来讲，他是一个代表了元素序号的整数。

每一个可枚举对象都可以拥有该方法`each_with_index`，但并不是每个可枚举队形都可以使用索引。

宿主拥有基础的索引概念，而哈希没有，但是他确实与索引相关。

哈希的键作为索引，而通过`each_with_index`迭代生成的序号却作为额外的或者元数据的索引。

`Enumerable`#`each_with_index`可以正常使用，但是更多的时候是考虑调用each所得的枚举器中可用的#`with_index`方法

```
array = %w{red yellow blue}
array.each.with_index do color ,i
    puts "Color ID#{i} is #{color}"
end
```

### `each_slice`和`each_cons`方法

这两个方法是`each`的特殊版本，他们可以一次集合中一定数量的元素，并将诸多元素作为数组在每次迭代的时候传递给代码块。

```
ar.each_slice(3) {slice p slice}
puts ""
ar.each_cons(3) {cons p cons}
```

![](http://img.varsion.cn/blog-img/2020/09/image-37.png)

`each_slice`操作会将集合按照n或者小于n（如果剩余元素不足n个）的大小逐次划分为新集合传递到代码块中。相反的，`each_cons`则一次向前移动一个元素的位置，而每次迭代都把包含n个元素的数组传递到代码块中，当最后一个元素被传递过一次之后迭代停止。

### `cycle`方法

`Enumerable`#`cycle`会反复的将对象中的所有元素作为参数传递到代码块中。如果调用时提供了一个参数，循环的次数将有该参数决定。如果没有该参数，则会无限进行。

扑克牌案例

```
class PokeCard 
    SUITS = %w{ clubs aidmonds hearts spades }
    RANKS = %w{ 2 3 4 5 6 7 8 9 10 J Q K A }

    class Deck
        attr_reader :cards
        def initialize(n=1)
            @cards = []
            SUITS.cycle(n) do s
                RANKS.cycle(1) do r
                    @cards << "#{r} of #{s}"
                end
            end
        end
    end
end
```

`PokeCard`类定义了用于表示花色和牌级的常量，而`PokeCard::Deck`类定义了一个牌盒，所有卡牌都会保存在实例变量 `@cards`中，同时作为读取器属性。通过使用`cycle`很容易将两个甚至多副扑克牌排列。例如下面的代码会产生两幅扑克牌。

```
deck = PokeCard::Deck.new
```

* * *