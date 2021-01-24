---
title: Ruby方法查找
tags:
  - Ruby
  - 方法查找
id: '581'
categories:
  - - Ruby
abbrlink: c0e023d
date: 2020-09-02 19:55:46
---

每当对象接收到消息的时候，预期的结局就是在对象的类或者超类中，再往上追溯，到达Object甚至BasicObject中，或者在混合到类的模块中，执行与消息同名的方法。

如果类和混和后的模块中都定义了相同名称的方法，这种发生混淆的例子，对象最终会选择哪个方法执行呢？

## 方法查找的基本原理

```
module M
  def report
    puts "'report' method in module M"
  end
end

class C
  include M
end

class D < C
end

obj = D.new
obj.report
```

实例方法`report`被定义在模块M中，而模块M被混合到了类C中。类D是C的子类，obj是D的一个实例。通过这样的层级关机，对象`obj`可以访问到`report`方法。

**方法查找对象视角**

```
对象Obj，发送了"report"消息，需要在方法查找路径中找到`report`方法进行调用

`Obj`是类D的一个实例，D中并没有包含`report`方法

类D是类C的子类，类C也没有包含`report`方法

类C中混合了模块M，M中定义了`report`方法

调用`report`方法
```

最后在搜索结束前，找到了该方法，如果没有找到该方法，则会触发一个错误。错误条件是由`method_missing`触发

**方法查找距离**

![](http://img.varsion.cn/blog-img/2020/09/QEHD@B43V@AY_LFABWV.png)

按照分布的方式，说明了类D的对象中方法的搜索路径，示例中，在模块M中成功的搜索到了目标。图中，展示了如果方法没有被搜索到，将会搜索更远的距离。当消息`report`被发送到对象，就开始的方法搜索，如图中箭头所指向的不同类和混合的模块。

## 同名方法的多次定义

在类中定义一个方法两次，第二次的定义将会取代第一次。这个在模块中也依然如此，其遵循如下规则：

在任意指定时间内，针对每一个类和模块仅仅拥有一个同名方法。

当然对象的法则与类和模块的法则是相似的：在特定的时间，对象仅可以看到一个既定方法中的其中的一个版本。

假如对象的查找路径中有多个同方法，会执行第一次找到的方法块。

```
module InterestBearing
  def calculate_interest
      puts "this is in module"
    end
end

class BankAccount
  include InterestBearing
  def calculate_interest
    puts "this is in class"
  end
end

account = BankAccount.new
account.calculate_interest
```

执行结果如下

```
this is in class
```

一个对象在其查找路径中有两个或多个同名方法的另一种情况是：

当一个类混合了两个或多个模块时，将搜索到方法的多个实现。这样的例子包含的逆序查找模块，这意味着，最新混合到类中的模块将会最先被搜索到。假如最新被混合的模块中包含一个同名方法，其方法在早先被混合的模块中出现过。最新被混合的模块中的方法版本将会占有最高的优先级，因为其是**最短路径**。

```
module M
  def report
    puts"this is M"
  end
end

module N
  def report
    puts "this is N"
  end
end

class C
  include M
  include N
end

obj = C.new
obj.report
```

执行结果如下

```
this is N
```

N模块中的`report`方法是obj在查找路径中命中的最近混合模块中的方法。

因此模块N中的方法占有最高优先级。

**包含一个模块多次**

```
class C
  include M
  include N
  include M
end
obj = C.new
obj.report
```

但是，该代码块的执行结果仍然是`this in N`

### Prepend(前置)的工作原理

每次在类中include包含一个模块，都会影响那个类的实例。必须处理对应方法名的消息过程。

对于`prepend`来说也是一样的，只不过，如果前置了一个模块在该类中，对象会首先在该模块中查找，而后才是在类中查找

## 查找方法规则

![](http://img.varsion.cn/blog-img/2020/09/image-1.png)

类D的实例在其方法查找路径中，跨越包含和前置的两个模块进行方法查找

* * *