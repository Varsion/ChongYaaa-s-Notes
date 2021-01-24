---
title: 在Rails6 中集成前端UI框架
tags:
  - Rails
  - Rails6
id: '1754'
categories:
  - - Ruby
  - - Ruby on Rails
abbrlink: 775c5c69
date: 2020-12-19 19:48:42
---

最近使用 Rails6 进行开发的时候，页面仍然是在不停的用 bootstrap 造轮子，所以就在想可不可以将一些前端UI框架集成到 Rails中，例如： [AmazeUI](http://amazeui.shopxo.net/)、[LayUI](https://www.layui.com/)等等...

因为近些版本Rails都在使用 Webpack 进行静态资源打包，所以一些地方改起来就非常的复杂，也在考虑是用 包管理工具集成管理 还是 将UI框架直接杂糅进Rails 做成 Gem 之类的。

## 使用 Yarn 集成 AmazeUI

为了效果演示，我创建了一个静态页面演示控制器：

![](http://img.varsion.cn/blog-img/2020/12/image-7.png)

然后将AmazeUI的演示页面设为主页面查看效果。

通过yarn添加 amazeui 的资源文件

![](http://img.varsion.cn/blog-img/2020/12/image-10.png)

然后记得将 app/assets/stylesheets/application.css 的文件类型 改为 .scss，即改成 app/assets/stylesheets/application.scss。

将刚刚导入的 AmazeUI 引入到页面中：

```
@import "amazeui/dist/css/amazeui";

```

添加这行代码即可引入 AmazeUI 的CSS样式，那么 AmazeUI 的JS组件应该如何引入呢？

同时，如果使用 AmazeUI 的JS组件的话 有些地方需要依赖到 jQuery ：

通过 yarn 安装 jQuery

![](http://img.varsion.cn/blog-img/2020/12/image-12.png)

然后需要在 config/webpack/environment.js 里添加以下代码，才可以引入 jQuery：

```
const webpack = require('webpack')

environment.plugins.prepend('Provide',
  new webpack.ProvidePlugin({
    $: 'jquery/src/jquery',
    jQuery: 'jquery/src/jquery'
  })
)

```

然后接下来的一个问题是 Fontawesome 未找到。在 FireFox 浏览器中给了详细的报错：

![](http://img.varsion.cn/blog-img/2020/12/image-17.png)

查了很多资料总结之后，发觉这些应该是引入 CSS 的时候，没有将 CSS 连带的图标/字体包 一起引入，而且在页面路径下寻找这些的时候，直接去网站根目录下寻找了，

![](http://img.varsion.cn/blog-img/2020/12/image-22.png)

找了很久的解决办法，是将本地资源引用切换成网络资源，可以解决问题，但是初次加载页面会很慢（实际使用的时候，可能要在服务器端做个资源映射吧，明明资源就在哪里，就是引用不到，很烦。

修改 node_modules/amazeui/dist/css/amazeui.css 文件中的部分。

也只是简单的把文件的url替换了，是在 AmazeUI 的 [issue](https://github.com/amazeui/amazeui/issues/277) 里找到的 CDN 加速网站，的确是搜了好久都没有这几个文件的 CDN资源。

或者，可以将 fonts 资源文件夹 放到 public 文件夹中，也可以解决。

感觉 Rails6 或者 webpack 在一些静态资源的引用上还不是很舒服（可能是我写垃圾页面代码写多了的缘故

然后可以启动服务器，可以看到根路由下的页面的样式，已经是 AmazeUI 的样式了。

![](http://img.varsion.cn/blog-img/2020/12/image-16-1024x537.png)

* * *