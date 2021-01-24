---
title: Ruby - 将方法作为对象使用
tags:
  - Ruby
  - 可调用和可运行对象
id: '1528'
categories:
  - - Ruby
abbrlink: 62e4ed3f
date: 2020-10-14 15:34:09
---

## lambda

和 Proc.new 一样，lambda 方法也会返回一个 Proc 对象：

```
lam = lambda { puts "A lambda" }
lam.call
# A lambda
```

应当注意的是，lambda 创建的 Proc 对象和 Proc.new 创建的有一些不同，这是 Proc 类 独有的一个 lambda 风格：

lambda 需要一个明确的创建过程，不论在何处隐式的创建 Proc 对象，都是常规的 proc 而不是 lambda 。

lambda 和 proc 的另外一个不同是对待 return 返回关键字的方式。lambda 中的return 会触发整个 lambda 主体立即退出 lambda 所在的代码上下文。proc 中的 return 会从 proc 被执行所在的方法中返回。

```
def Test_re
  lam = lambda { return }
  lam.call
  puts "hello"
  p = Proc.new { return }
  p.call
  puts "Qus?"
end
Test_re
# hello
```

这个代码片段只会输出 hello 而不会输出第二条打印消息。

值得注意的是，proc 内部的 return 触发了闭合方法的返回，当没有位于方法内部调用包含 return 的 proc 时，会触发一个严重的错误。

最后，最为重要的是，lambda 不能在调用的时候使用错误的参数数量，并不能像 proc 一样。

在实践中，Ruby最常调用的不是 proc 或者 lambda，而是方法。

在我看了，方法调用看作是一个层级里的传递过程：发送消息给对象，然后对象执行所匹配的命名了的方法。

## 将方法作为对象使用

方法自己并不能作为对象，触发对他们进行一些处理，如果要将他们作为对象使用，就要涉及 _对象化 objectify_。

### 捕获对象方法

通过 method 方法，并将方法名作为参数传递给它（以字符串或者符号的形式），就可以得到 Method 对象

```
class C
  def talk
    puts "Hello"
  end
end
c = C.new
meth = c.method(:talk)
```

这样就拥有了一个 Method 对象，具体来说是一个绑定的 Method 对象：不是抽象层的 talk 方法，而是具体的 talk 方法绑定到了对象 c 上，如果发送 call 消息给 meth，它会知道要通过 c 扮演 self 角色来调用自己

也可以将方法从对象上解除绑定，然后绑定到其他对象上，需要另一个对象与原始对象同类或者为其子类。

```
class D < C
end
d = D.new
unbound = meth.unbind
unbound.bind(d).call
```

如果要直接得到一个未绑定的方法对象，而不在已绑定的方法上调用 ubbind 方法，可以通过 instance_method 从类中得到更多，而不使用类实例

```
unbound = C.instance_method(:talk)
```

可以说，在有了这个未绑定的方法之后，就能够使用 bind 方法将它绑定到 C 或者 C的子类的任意实例上。

### 方法对象的基本原理

假如已经得到某个类体系，并在体系中重新定义了某个方法

```
class A
  def a_meth
    puts "def in A"
  end
end
class B < A
  def a_meth
    puts "def in B"
  end
end
class C < B
end

c = C.new
c.a_meth
```

需要寻求一种方式，通过执行类中的方法版本，让最低一层的类来响应上两个类级别的类方法。

默认情况下，最低一层的实例是不能达到这个要求的，因为他会遍历方法查找路径，所以会执行第一次匹配的方法：

```
# 输出为
def in B
```

但是可以强制使用解绑操作和绑定操作来解决这个问题

```
A.instancd_method(:a_meth).bind(c).call
# def in A
```

可以直接将该方法保存到 C 类的某个方法中，然后直接在 c 上调用该方法。

这个例子中传达了一个矛盾的状态：如果需要控制对象去响应已经重定义过的方法，那么就应该重新审查自己的代码，而不管已经创建的类或者模块体系。

同时，方法是可调用对象，可以将他们从实例对象上分离出去（解绑）

* * *