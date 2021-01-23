---
title: Ruby - 对象消息拦截
tags:
  - Ruby
  - 回调和钩子
id: '1557'
categories:
  - - Ruby
date: 2020-10-17 11:17:15
---

当发消息给对象的时候，对象会在方法查找路径上找到与消息同名的方法执行，如果找不到这样的方法，就会引发 NoMethodError 异常，除非为该对象提供了一个 method_missing 方法。

### 使用 method_missing 实现委托

可以将 method_missing 作为对象行为自动拓展的一种方式。例如，一个对象，他从某种方面上来说是一个容器，但是也会有一些其他特性。

或许是一个药箱，作为药物的集合，有另外的一些属性需要单独的存储和处理，既包含了药物，又包含了这些药物的元数据。

```
 class DrugBox
   attr_accessor :drug_name,:drug_date
   def initialize
     @drugs = []
   end
   def method_missing(m,*args,&block)
     @drugs.send(m,*args,&block)
   end
 end
```

现在可以在药箱中执行操作了：

```
drug = DrugBox.new
drug << drug_for_clod
drug << drug_for_stomach
other = drug.select { drugs drug.main_ingredient == "other" }
```

虽然实例中并没有 << 和 select 方法，因此这些消息会因为 method_missing 被传递到 @drug 数组中。仍然可以在 DrugBox 中直接定义任何方法，甚至可以直接覆盖数组方法，不过 method_missing 免去了定义一道并行处理有序集合方法的麻烦。

> **Ruby 的方法委托技术**
> 
> 在 method_missing 的例子中，将（不能识别的）消息的处理委托给了数组 @drugs 。Ruby中有许多将一个对象的获得委托给另外对象的机制。

### 源头 BasicObject # method_missing

在 BasicObject 中的 method_missing，是定义在类体系树中非常顶层的几个方法之一。由于所有的方法最终都是来源于 该基类，故而所有对象都拥有 method_missing 方法。

有两个方式可以覆盖默认的 method_missing 方法。一是可以打开 BasicObject 类，然后再次定义 method_missing 方法；第二种方法更为通用，在最顶层定义 method_missing 方法，然后将它作为 Object 私有的实例方法装入。

如果使用第二种方法，除了 BasicObject 的真正实例之外的所有对象都会找到这个新版本的方法。

但是，如果定义了自己的 method_missing 方法，就失去了对变量的智能识别而智能识别方法了。

### method_missing、respond_to?、和respond_to_missing?

需要知道的是 method_missing 和 respond_to? 并不等同，可以参考下面的例子

```
class Persom
  attr_accessor :name,:age

  def initialize(name,age)
    @name,@age = name , age
  end

  private def method_missing(m, *args,&block)
    if /set_(.*)/.match(m) # 匹配方法
      self.send("#{$1}=",*args)
    else
      super 
    end
  end
end
```

因此 Person 对象拥有了 set_age 方法，但是该对象并不能响应他：

```
p = Person.new("Dvaid", 22)
p.set_age(20)
puts p.age
puts p.respond_to?(:set_age)
```

![](http://img.varsion.cn/blog-img/2020/10/image-2.png)

有一种方法是可以通过定义 respond_to_missing? 方法，可以让 method_missing 和 respond_to? 产生一样的效果。

下面是要添加到 Person 类中的定义

```
  def respond_to_missing?(m,include_private = false )
    /set_/.match(m)  super 
  end
```

现在，新的Person对象可以响应该方法了：

![](http://img.varsion.cn/blog-img/2020/10/image-4.png)

通过使用 respond_to? 的第二个参数，可以控制查询是否要包括私有方法。第二个参数将会被传递到 respond_to_missing? 方法中。如例子中所示，他默认为 false。

### 捕获 include 和 prepend 操作

如果想要捕获这些事件，即当他们发生时触发回调，那么可以定义特殊的方法 included 和 prepended。这两个方法都以包含或前置时的类或模块名作为唯一参数。

现在详细看一下 include 同时要理解一下 prepended 的运行方式与它相同。可以同时快速的测试一下 include。

```
module M
  def self.included(c)
    puts "I have been"
  end
end

class C
  include M  # I have been
end
```

当 M 被包含到 C 中，可以看到 M.included 的执行结果。

就像这个例子中一样，一个模块拦截自己被包含的事件是发生在什么情况下呢。常见的一种讨论是围绕着实例方法和类方法的不同展开的。当一个模块到类中，就确保了定义在模块中的实例方法将会变成类的实例可用的实例方法，但类对象不受影响。接下来，就会有问题产生了：

如果想通过在混合模块并添加实例方法对同时，添加类方法应该要怎么做呢。

由于 included 方法的存在，所以可以捕获 include 操作，然后利用这个机会添加类方法到正在包含的类中。

```
module M
  def self.included(cl)
    def cl.a_class_method
      puts "Now there is has new class method"
    end
  end

  def an_inst_method
    puts "This module is instance method"
  end
end

class C
  include M
end

c = C.new
c.an_inst_method
c.a_class_method
```

当类 C 包含模块 M 的时候，有两件事发生。第一，实例方法 an_inst_method 会出现在类的实例的查找路径中。第二，由于 M 的 included 的回调方法，类方法 a_class_method 被定义到了类对象 C 中。

### 拦截 extend

使用模块拓展独立对象，是在Ruby中利用对象灵活性和可定制属性的可用技术中最强大的一种。它也是受益于运行时的钩子：使用 Module # extended 方法，可以简历一个回调方法，只要对象执行其所包含的模块的 extend 操作，就可以触发回调执行：

```
module M
  def self.extended(obj)
    puts "Module #{self} is being used by #{obj}"
  end
  def an_inst_method
    puts "This module supplise this instance method"
  end
end

m_obj = Object.new
m_obj.extend(M)
m_obj.an_inst_method
# 输出
# Module M is being used by #<Object:0x00007fd6a3071508>
# This module supplise this instance method
```

#### 使用 included 和 extended 时单例类的行为

实际上，通过 extend 使用模块拓展对象与通过 include 包含模块对于对象的单例类来说都是一样的。无论使用那种方式，结局都是模块会被添加到对象的方法查找路径中，且刚好就在查找链中对象的单例类之后。

但是这两个操作会触发不同的回调方法，即 extended 和 included 。

```
module M
  def self.included(c)
    puts "#{self} included by#{c}"
  end
  def slef.extended(obj)
    puts "#{self} extended by#{obj}"
  end
end

obj = Object.new
puts "Include M in object's singleton class:"
class << obj
  include M
end

puts
obj = Object.new
puts "Extending object with M:"
obj.extend(M)
```

这两个回调都定义在模块 M 中：included 和 extended。每个回调都会根据方法内的行为打印输出。以刚刚创建的通用对象开始，在对象的单例类中包含模块 M ，然后重复过程，新建另外一个对象后直接使用M拓展该对象：

![](http://img.varsion.cn/blog-img/2020/10/image-6.png)

非常特殊的是，包含操作会触发 included 回调，而拓展操作会触发 extended，尽管在这个特殊的情况下两个操作的结果一样：模块 M 中的方法会被添加到讨论中的对象的方法查找路径。

### Class # inherited 拦截继承事件

可以通过为类定义 inherited 方法来与类继承事件挂钩。如果在指定类中定义了该方法，在继承该类的时候，inherited 会使用新类名作为调用的参数。

```
class C
  def self.inherited(subclass)
    puts "#{self} just got subclassby #{subclass}"
  end
end


class D < C
end

# C just got subclassby D
```

D在继承C类的时候自动触发了 inherited 的调用，inherited 是一个类方法，因此只要类定义过它，他的子类在被继承的时候也会自动调用他。然后一次想体系的下级递进。

#### inherited 回调方法的局限

当D继承C时，C是D的超类；但是除此之外，C的单例类也是D的单例类的超类。那就是D调用C的类方法的方式，但是这里并不会触发回调。

### Module # const_missing

该方法是一个常见的回调方法，在给定的模块和方法中，只要引用不可识别的常量，该方法就会被调用：

```
class C
  def self.const_missing(const)
    puts "#{const} is undefined-setting"
    const_set(const,1)
  end
end

puts C::A
puts C::A
```

![](http://img.varsion.cn/blog-img/2020/10/image-8.png)

得益于回调方法，当 C::A 的时候，他被自动定义，这里关注的是 puts 打印的常量值，puts 不必知道这个常量是否之前被定义过，第二次调用的时候，该常量已经被定义了。

### method_added 和 singleton_method_added

如果把 method_added 作为类方法定义在任意的类和模块中，它能够在定义任意的实例方法时被调用。

```
class C
  def self.method_added(m)
    puts "Method #{m} was just defined"
    # 定义回调方法
  end
  def a_new_method
# 定义实例方法时触发它
  end
end
# Method a_new_method was just defined
```

singleton_method_added 和 method_added 所做的事情相同，不过针对的是单例方法。同时，一个奇怪的现象，他触发的是他自己本身：

```
class C
  def self.singleton_method_added(m)
    puts "Method #{m} was just defined "
  end
end
# Method singleton_method_added was just defined 
```

在大多数情况下，应该将 singleton_method_added 用在对象上而不是类对象上：

```
obj = Object.new
def obj.singleton_method_added(m)
  puts "Singleton method #{m} was defined"
end
def obj.a_new_method
end
```

将基于类和基于对象的方式放到一起，可以通过定义对象的单例类相关的放，达到特定对象的效果：

```
obj = Object.new
class << obj
  def singleton_method_added(m)
     puts "Singleton method #{m} was defined"
  end
  def a_new_method
  end
end
```

这个代码的输出和之前的完全相同，最终，绕了一个圈之后，可以将 singleton_method_added 定义为普通类的实例方法，在这种方式中，类的每个实例都可以在创建单例方法的时候触发该回调。

回调方法的定义掌控着每个该类的实例。在这些定义的单例方法都会因此触发回调。

* * *