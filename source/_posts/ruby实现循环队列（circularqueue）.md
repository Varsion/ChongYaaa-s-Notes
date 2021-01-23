---
title: Ruby实现循环队列（CircularQueue）
tags:
  - Ruby
  - 循环队列
id: '358'
categories:
  - - Ruby
  - - 数据结构与算法
date: 2020-08-04 17:09:14
---

> 昨天进行了队列的部分，准备今天把循环队列单独拿出来做一次笔记，关于Ruby的循环队列。
> 
> _没有找到什么值得参考的博文和例子，就只能自己慢慢摸索这着写。_同时参考了该题目[662.设计循环队列](https://leetcode-cn.com/problems/design-circular-queue/)

根据循环队列的需要列出了大概的结构框架，然后需要针对每个函数进行分析，

```
class LoopQueue

    def initialize(size)

    end

    # 入队
    def enQueue(item) 
        
    end

    # 出队
    def deQueue
        
    end

    # 判空
    def is_empty? 
    
    end

    # 判满
    def is_full? 
    
    end    
end
```

先来考虑的结构初始化所需要的内容:开辟循环队列所需要的空间，并指定头尾两个节点:

```
    attr_reader :head, :last, :Maxsize
    def initialize(size)
        @Maxsize = ++size
        @head = nil
        @last = nil
        @queue = Array.new(@Maxsize)
    end
```

接下来考虑的是如何判满和判空

*   **满**：当队列添加元素到last的下一个元素是head的时候，也就是转圈子要碰头了，`(@last + 1) % @Maxsize == @head` 我们就认为队列满了。
*   **空**：当队列删除元素到 `head == last` 的时候，我们认为队列空了。

```
    def is_empty? 
        if @head == @last
            return true
        else
            return false
        end
    end

    def is_full? 
        if (@last + 1) % @Maxsize == @head
            return true
        else
            return false
        end
    end
```

最重要的两个操作是入队和出队，注意`%`的含义:

```
    # 入队
    def enQueue(item) 
        return false if is_full?
        @queue[@last] = item
        @last = (@last+1) % @Maxsize
        return true
    end

    # 出队
    def deQueue
        return false if is_empty?
        data = @queue[@head]
        @queue[@head] = 0 
        @head = (@head+1) % @Maxsize
        return data
    end
```

获取队首元素和队尾元素，这里注意获取队尾时`last`应当`-1`

```
    # 获取队首
    def front
        return -1 if is_empty?
        return @queue[@head]
    end
    # 获取队尾
    def rear
        return -1 if is_empty?
        return @queue[@last-1]
    end
```

完整的代码我放在了最后这里:

```
class LoopQueue
    attr_accessor :head, :last, :Maxsize
    def initialize(size)
        @Maxsize = size+1
        @head = 0
        @last = 0
        @queue = Array.new(@Maxsize)
    end

    def getSize
        return @Maxsize
    end
    

    # 入队
    def enQueue(item) 
        return false if is_full?
        
        @queue[@last] = item
        @last = (@last+1) % @Maxsize

        return true
    end

    # 出队
    def deQueue
        return false if is_empty?
        data = @queue[@head]
        @queue[@head] = 0 
        @head = (@head+1) % @Maxsize
        return data
    end

    # 获取队首
    def front
        return -1 if is_empty?
        return @queue[@head]
    end
    # 获取队尾
    def rear
        return -1 if is_empty?
        return @queue[@last-1]
    end
    
    # 判空
    def is_empty? 
        if @head == @last
            return true
        else
            return false
        end
    end

    # 判满
    def is_full? 
        if (@last + 1) % @Maxsize == @head
            return true
        else
            return false
        end     
    end    
end
```

这个代码并不符合题目[662.设计循环队列](https://leetcode-cn.com/problems/design-circular-queue/)的要求，我这里是根据我的编码习惯写出代码，完成题目的话，也仅仅是根据题目修改方法名称即可。

下面这个是符合题目要求的代码:

```
class MyCircularQueue

=begin
    Initialize your data structure here. Set the size of the queue to be k.
    :type k: Integer
=end
attr_accessor :head, :last, :Maxsize
    def initialize(k)
        @Maxsize = k+1
        @head = 0
        @last = 0
        @queue = Array.new(@Maxsize)
    end
=begin
    Insert an element into the circular queue. Return true if the operation is successful.
    :type value: Integer
    :rtype: Boolean
=end
    def en_queue(value)
        return false if is_full
        
        @queue[@last] = value
        @last = (@last+1) % @Maxsize

        return true
    end
=begin
    Delete an element from the circular queue. Return true if the operation is successful.
    :rtype: Boolean
=end
    def de_queue()
        return false if is_empty
        data = @queue[@head]
        @queue[@head] = 0 
        @head = (@head+1) % @Maxsize
        return true
    end
=begin
    Get the front item from the queue.
    :rtype: Integer
=end
    def front()
    return -1 if is_empty
        return @queue[@head]
    end
=begin
    Get the last item from the queue.
    :rtype: Integer
=end
    def rear()
    return -1 if is_empty
        return @queue[@last-1]
    end


=begin
    Checks whether the circular queue is empty or not.
    :rtype: Boolean
=end
    def is_empty()
        if @head == @last
            return true
        else
            return false
        end
    end
=begin
    Checks whether the circular queue is full or not.
    :rtype: Boolean
=end
    def is_full()
        if (@last + 1) % @Maxsize == @head
            return true
        else
            return false
        end
    end
end
```