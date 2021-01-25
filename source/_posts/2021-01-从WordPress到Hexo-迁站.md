---
title: 从WordPress到Hexo-迁站
id: '12010110'
tags:
  - Hexo
  - WordPress
  - 迁站
categories:
  - - Developer
keywords:Hexo,WordPress,站点迁移
date: 2021-01-24 09:34:32
---

# 从 WordPress 到 Hexo

> 之前是一直有在用 `WordPress` ，建站、可视化编辑还有网站访问量包括对一些IP的记录对于博客管理来说很方便，但是前些时间因为 WordPress 和 一些插件总是一直在提示更新，更新之后又会莫名的失败导致网站短时间内崩溃，于是就有准备将博客迁移到 Hexo 来。

## 媒体资源迁移

​	Hexo的博客还是使用图床要来的更舒服一些，基本步骤，就是将 WordPress 的媒体资源全部按照原定目录结构上传到图床中，我这边使用的是 [七牛云](https://www.qiniu.com/) 实名认证之后 10G 的免费存储空间还是挺香的。

​	WordPress 的媒体资源默认是在 `blog/wp-content/uploads` 文件夹中 按照 `Year/Mouth` 组织的，相应的文件寻找路径记录在了数据表 `wp-posts` 中 

![image-20210124103004307](https:img.varsion.cn/blog-img/20210124103004.png)

#### 上传图床

我才用了比较蠢的方法，将整个 uploads 文件夹 down下来，然后按照目录上传，上传时只需要设置好每个文件夹的路径前缀就好

![image-20210124103324679](https:img.varsion.cn/blog-img/20210124103324.png)

#### 修改数据库路径

```sql
update wp_posts set post_content = replace(post_content, 'http://www.exm.com/wp-content/uploads/', 'http://img.exm.com/');
```

可使用该 SQL语句 将当前的网络图片链接替换成已经上传到图床中的图片链接。

## 文章迁移

WordPress 提供了文章导出的功能，可以将选定的文章信息导出到 xml

```shell
npm install hexo-migrator-wordpress --save
```

可以使用这个 [package](https://github.com/hexojs/hexo-migrator-wordpress) 进行xml数据转换成 hexo - md 命令如下

```shell
hexo migrate wordpress WordPress.yyyy-mm-dd.xml
```

### 部分修改

​	使用 `hexo migrate wordpress` 进行数据迁移时，发现了该插件对于 `<pre> </pre>` 标签的支持很差，并且会 将原文章中的 `\n` 给转义成 空格。

#### `<pre>`

修改`blog/node_modules/hexo-migrator-wordpress/migrator.js` 添加如下代码

```js
tomd.addRule('code_block', {
    filter: 'pre', //(node) => node.nodeName === 'PRE',
    // 'pre' || (node.innerHTML.toString().toLowerCase().indexOf('<pre ') != -1),
    replacement: function (content, node, options) {
      return '\n\n' + options.fence + '\n' + content + '\n' + options.fence + '\n\n';
    }
  });
```

大概是在这个位置：

![image-20210124105025207](https:img.varsion.cn/blog-img/20210124105025.png)

#### 保留 `\n`

在 `blog/node_modules/turndown/lib/turndown.cjs.js` 修改一行代码

```js
var text = node.data.replace(/[ \r\n\t]+/g, ' ');
// 修改为
var text = node.data.replace(/[ \r\t]+/g, ' ');
```

参考该 [issues](https://github.com/domchristie/turndown/issues/264)

#### 重写 `escape` 函数

在 `blog/node_modules/turndown/lib/turndown.cjs.js` 

```js
escape: function (string) {
   return escapes.reduce(function (accumulator, escape) {
     return accumulator.replace(escape[0], function(s) {
       if (!s.startsWith('<pre')) {
         s.replace(escape[0], escape[1]);
       }
       return s;
     })
   }, string)
 }
```

至此，只需要将 `wordpress-yyyy-mm-dd.xml` 放到博客的根目录，运行迁移命令即可。