---
title: 使用Ripper解析Ruby代码
tags:
  - Ruby
  - Ruby原理剖析
id: '1738'
categories:
  - - Ruby
date: 2020-12-17 18:27:08
---

下述示例代码展示了如何使用 Ripper 解析 Ruby 脚本：

```ruby
require "ripper"
require "pp"

code = <<STR
10.times do n
puts n
end
STR

puts code
pp Ripper.sexp(code)
```

可以通过 Ripper.sexp 方法显示 Ruby 解析代码过程中的相关信息，运行该代码可以得到该结果：

![](http://img.varsion.cn/blog-img/2020/12/image.png)

这段输出结果是 Ruby 代码的一种文本表示，在 Ruby 解析代码时，随着语法规则的不断匹配，代码文件中的词条会被转变为一种复杂的内部数据结构：抽象语法树（AST）。被用来记录 Ruby 代码的结构和语法的意义 。

取最后 block 里的代码部分，也就是 puts n 语句：

![](http://img.varsion.cn/blog-img/2020/12/image-3.png)

![](http://img.varsion.cn/blog-img/2020/12/image-4.png)

当使用 Ripper 显示词条信息的时候，源码文件的行和列是以数字类型显示的。例如 [2,1] 表示 Ripper 是在源码的第二行第一列中找到了 puts 调用。也可以看到 Ripper 把 AST 中的每个节点都输出为一个数组，比如：

```
[:@ident, "puts", [2, 1]]
```

对于 puts n 语句，Ruby 现在对其含义已经有了详细的 "描述"（AST），而不是一些模糊的词条流。可以看到函数调用（command）及随后的标识符节点，标明了是哪个函数被调用。

Ruby 使用了 args_add_block 节点，因为可以给一个命令 （command），也就是函数调用，传递块 （block），这个 args_add_block 节点也依然会被保存在 AST 中。

最后，展示了整个 Ruby 代码示例的 AST

![](http://img.varsion.cn/blog-img/2020/12/image-6.png)

上图中的 method add block 是指总在调用一个需要块参数的方法 10.times do 树节点 call 显然是代表实际的方法调用 10.times。**Ruby 把对代码的理解以节点的方式依次保存在了 AST 中。**

* * *