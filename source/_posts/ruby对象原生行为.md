---
title: Ruby对象原生行为
tags:
  - Object
  - Ruby
id: '42'
categories:
  - - Ruby
date: 2020-07-23 12:31:16
---

> 只要创建对象存在，他就能相应一组消息，每个对象在被创建时都语句确切的能力
> 
> 在`irb`解释器中 `p Object.new.methods.sort` 可以查看其原始方法

![](http://img.varsion.cn/blog-img/2020/07/79@XODLAV@7W0346WNKZ2.png)

### **`object_id`表示唯一对象**

Ruby中每一个对象都有一个和他唯一关联的ID编号，可以通过请求`object_id`获得该对象的ID 。

```
class Tist
  def initialize(id)
    @id = id
  end

  def getID
    return @id
  end

end
```

```
load "class.rb"
ts1 = Tist.new("1")
ts2 = Tist.new("2")

puts ts1.object_id
puts ts2.object_id
```

![](http://img.varsion.cn/blog-img/2020/07/1.png)

上述尝试可以看出，同为一个类的对象`object_id`并不相等

在尝试确认两个对象是否相等时，可以采取判断`objectt_id`的方法

```
load "class.rb"
ts1 = Tist.new("1")
ts2 = Tist.new("2")

ts = ts1
puts ts1.object_id
puts ts.object_id
```

![](http://img.varsion.cn/blog-img/2020/07/image-20200723075513344.png)

ID编号和对象的相等性

Ruby中给对象指定一个ID编号能够使对象有唯一的身份标识

### **`respond_to?`方法查询对象的能力**

Ruby对象响应消息，在程序运行期间的不同时间点，依赖于对象和为对象定义的各种方法。

用户可以提前判断（在要求对象执行任务前）对想是否知道如何处理发送给他的消息，`respond_to?`对于所有对象都适用，通常和`if`语句联合使用

```
obj = Object.new
if obj.respond_to?("talk")
obj.talk
else
puts "error"
end
```

`respond_to?`使自省（_introspection_）或者反射（_reflection_）的一个例子

### **`send`方法发送信息给对象**

`send`方法类似于`.`方法调用对象函数

```
obj = Object.new
if obj.respond_to?("talk")
obj.send("talk")
else
puts "error"
end
```

> **使用`__send__`或者`public_send`代替send**

*   `__send__`是为了避免`send`方法和对象内建方法冲突
*   `send`和`__send__`可以调用对象的私有方法，`public_send`方法却不能

* * *