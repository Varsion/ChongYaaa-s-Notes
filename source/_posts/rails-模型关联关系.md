---
title: Rails 模型关联关系
tags:
  - Active Record
  - 模型
  - 模型关联
id: '1677'
categories:
  - - Ruby on Rails
date: 2020-11-15 17:30:39
---

Rails 中的 **Active Record** 拥有很多强大的功能，主要功能包括：表示模型之间的关系、通过相关联的模型表示继承的层次结构、数据验证等...

## 关联类型

### belong_to

belong_to 关联会在两个模型之间创建一对一关系，声明所在的模型实例属于另一个模型实例，比如有博客和用户两个模型，并且一篇博客的作者只能是一个用户，就需要在博客的模型中声明。

```
class Blog < ApplicationRecord
# ...
belongs_to :user
# ...
end
```

在 belong_to 关联声明中必须使用单数形式。如果在上面的代码中使用复数形式定义 user 关联，应用会报错，这是因为 Rails 自动使用关联名推导类名，推导出的类名也就变成了复数。

### has_one

has_one 关联也是建立了两个模型之间的一对一关系，但是语意和结果同 belongs_to 不一样。这种关联表示模型的实例包含或拥有另一个模型的实例，例如一条用户登陆信息只能对应者一个用户：

```
class Account < ApplicationRecord
# ...
has_one :user
# ...
end
```

同时根据使用需求，可能还需要为 user 的某列创建唯一性索引或外键约束

### has_many

has_many 关联建立两个模型之间的一对多关系。在 belongs_to 关联的另一端使用该关联。has_many 关联表示模型的实例有零个或多个另一模型的实例。

```
 class User < ApplicationRecord
   # ...
   has_many :blogs
   # ...
 end
```

### has_many :through

has_many :through 关联场用于建立两个模型之间的多对多关联。这种关联表示一个模型的实例可以借由第三个模型，拥有零个或多个另一个模型的实例。

例如，一篇博客可以设置多个标签：

```
class Blog < ApplicationRecord
has_many :blogs_tags,class_name: "BlogsTags"
has_many :tags,through: :blogs_tags
end
###
class Tag < ApplicationRecord
has_many :blogs_tags, class_name: "BlogsTags"
has_many :blogs,through: :blogs_tags
end
###
class BlogsTags < ApplicationRecord
belongs_to :blog
belongs_to :tag
end
```

[![](http://img.varsion.cn/blog-img/2020/11/image-5.png)](http://img.varsion.cn/blog-img/2020/11/image-5.png)

### has_one :through

has_one :throught 关联建立两个模型之间的一对一关系。这种关联表示一个模型通过第三个模型拥有另一模型的实例。例如，每个商家都有一个账户，每个账户都有一个账户历史：

```
class Supplier < ApplicationRecord
  has_one :account
  has_one :account_history, through: :account
end
 
class Account < ApplicationRecord
  belongs_to :supplier
  has_one :account_history
end
 
class AccountHistory < ApplicationRecord
  belongs_to :account
end
```

[![](http://img.varsion.cn/blog-img/2020/11/image-8.png)](http://img.varsion.cn/blog-img/2020/11/image-8.png)

### has_and_belongs_to_many

has_and_belongs_to_many 关联直接建立两个模型之间的多对多关系，不借由第三个模型。

例如，应用中有装配体和零件两个模型，每个装配体有多个零件，每个零件又可用于多个装配体，这时可以按照下面的方式定义模型：

```
class Assembly < ApplicationRecord
  has_and_belongs_to_many :parts
end
 
class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end
```

![](http://img.varsion.cn/blog-img/2020/11/image-9.png)

原理上讲，has_and_belongs_to_many 并不是不借由第三个模型，而是，创建了第三张默认的表来对相应的字段链接进行绑定从而达到多对多关联的效果

* * *