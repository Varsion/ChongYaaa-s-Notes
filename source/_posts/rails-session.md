---
title: Rails Session
tags:
  - Rails
  - Session
id: '1655'
categories:
  - - Ruby on Rails
date: 2020-11-11 10:17:30
---

### CookieStore - Session默认的存储方式

Rails 的 Session 是默认存储在 CookieStore 中的，这是一个安全的做法，CookieStore 会基于 secure_key_base 对 Session 的内容进行加密，并将最终的加密以 Base64 的结果返回作为 Cookie。

CookieStore 是 Rails哲学提倡的最佳实践中的一个，不需要依赖服务器端的持久化数据库（Redis / Memcache / MySQL）即可实现。

当我们的 Rails App 运行时，有一个名为_appname_session 的 Cookie 会一直在我们的项目中：

[![](http://img.varsion.cn/blog-img/2020/11/image-3.png)](http://img.varsion.cn/blog-img/2020/11/image-3.png)

#### CookieStore的优缺点

优点：

*   使用简单，无需额外的后端存储，开箱即用；
*   相对与其他存储方式，读写很快，只需解密，无 IO 请求；
*   因为 Session 加密存储在用户浏览器，所以不会因为后端存储丢失而导致 Session 失效；
*   我们依然可以通过改变 secure_key_base 的方式来使之前的 Session 失效；

缺点：

*   浏览器 Cookie 限制，加密过后的内容不能超过 4K，所以我们不能在 Session 里面放过多内容；
*   如果你的静态资源没有独立域名的话，静态页面的请求 Header 也会多余带这个 Session 的值；
*   Session 存储过大内容到了客户端，会导致用户访问服务错误，且难以被发现；
*   如果那种验证的 code 存在 Session 里面，会有重现攻击（Replay attacks）风险

一些提示：

*   flash 也是存在Session里的
*   有关用户信息，只需要将 user_id 存于 Session即可，不要将整个 user 对象放进去。

### 用户Session

正常的持久化登陆流程采用了Session流程：

```ruby
module Concerns
	module UserSession

	def self.included base
		base.class_eval do
			helper_method :logged_in?
			helper_method :current_user
		end
	end

	def signin_user user
		session[:user_id] = user.id
	end

	def logout_user
		session[:user_id] = nil
	end

	def logged_in?
		!!session[:user_id]
	end

	def current_user
		if logged_in?
			@current_user = User.find(session[:user_id])
			# @current_user = @current_user  User.find(session[:user_id])
		else
			nil
		end
	end

	end
end
```

通过对Session的读写和销毁实现用户登陆逻辑。

* * *

文章参考了：[Rails 默认 Session 的存储方式：CookieStore](https://ruby-china.org/topics/39083)