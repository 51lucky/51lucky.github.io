---
title: hexo深度优化
toc: true
comments: true
categories: web
tags: hexo
abbrlink: 854043ca
date: 2017-04-10 16:26:52
---

安装：利用基于 Node.js 的 Hexo 可以快速搭建一个博客网站。搭建前必须安装 [Node.js](https://nodejs.org/en/) 和 [Git](https://git-scm.com/)。Node.js 是一款开源且跨平台的服务器端和网络应用，使用 JavaScript 开发。Git 是一款免费、开源的分布式版本控制系统。安装过程中全点下一步即可。之后，请参照[官方中文文档](https://hexo.io/zh-cn/docs/index.html)进行 Hexo 的安装。

网站初始化：在指定文件夹初始化网站文件。在你想存储网站文件的文件夹中，鼠标右键后，打开 `Git Bash Here`，然后输入 `hexo init blog`，`blog` 是文件名，可任意取。成功后会出现 `INFO Start blogging with Hexo!` 提示信息。Windows 用户由于自带的记事本简陋难用，故建议安装 [Notepad++](https://notepad-plus-plus.org/) 以编辑 .yml 文件。
<!-- more -->

## 深度优化

### 文章链接唯一化

也许你会数次更改文章题目或者变更文章发布时间，在默认设置下，文章链接都会改变，不利于搜索引擎收录，也不利于分享。唯一永久链接才是更好的选择。安装此插件后，不要在 `hexo s` 模式下更改文章文件名，否则文章将成空白。

```
npm install hexo-abbrlink --save
```

在 `站点配置文件` 中查找代码 `permalink:`，将其更改为:

```
permalink: posts/:abbrlink/  # “posts/” 可自行更换
```

在 `站点配置文件` 中添加如下代码：

```
# abbrlink config
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32 
  rep: hex    # 进制：dec(default) and hex
```

可参照样例以选择：

```
crc16 & hex
https://post.zz173.com/posts/66c8.html

crc16 & dec
https://post.zz173.com/posts/65535.html

crc32 & hex
https://post.zz173.com/posts/8ddf18fb.html

crc32 & dec
https://post.zz173.com/posts/1690090958.html
```

### 搜索引擎优化

#### 添加网站地图

```
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```

在 `站点配置文件` 中添加如下代码。

```
# hexo sitemap 
sitemap:
path: sitemap.xml
baidusitemap:
path: baidusitemap.xml
```

配置成功后，会生成 `sitemap.xml` 和 `baidusitemap.xml`，前者适合提交给谷歌搜素引擎，后者适合提交百度搜索引擎。其次，在 `robots.txt` 中添加下面的一段代码：

```
Sitemap: http://51lucky.me/sitemap.xml
Sitemap: http://51lucky.me/baidusitemap.xml
```

#### 添加蜘蛛协议

`robots.txt` 放置在 `\source` 目录下。

```
#hexo robots.txt
User-agent: *
Allow: /
Allow: /archives/

Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/

Sitemap: http://51lucky.me/sitemap.xml
Sitemap: http://51lucky.me/baidusitemap.xml
```

#### 限制出站链接

网络爬虫会在当前页面搜索所有的链接，故有可能跳到别的网站。`nofollow` 标签是由谷歌领头创新的一个 “反垃圾链接” 的标签，并被各大搜索引擎广泛支持，引用 `nofollow` 标签的目的是：用于指示搜索引擎不要追踪（即抓取）网页上的带有 `nofollow` 属性的任何出站链接，以减少垃圾链接的分散网站权重。

```
npm install hexo-autonofollow --save
```

在 `站点配置文件` 中添加如下代码。

```
nofollow:
    enable: true
    exclude: # 例外的链接，可将友情链接放置此处
    - exclude1.com
    - exclude2.com
```

#### 主动推送新链接

解决百度爬虫被禁止访问的问题，提升网站收录质量和速度。

```
npm install hexo-baidu-url-submit --save
```

在 `站点配置文件` 中添加如下代码。

```
baidu_url_submit:
  count: 5 ## 比如3，代表提交最新的三个链接
  host: 51lucky.me ## 在百度站长平台中注册的域名
  token:  ## 请注意这是您的秘钥， 请不要发布在公众仓库里!
  path: baidu_urls.txt ## 文本文档的地址， 新链接会保存在此文本文档里
```

### 404页面

在`source`目录下，新建`404.html`文件或者`404.md`文件，以404.html为例接入[腾讯404公益广告](http://www.qq.com/404/)

```html
---
layout: false
title: 404
---
<!DOCTYPE HTML>
<html>
<head>
  <title>404 - 51lucky</title>
  <meta name="description" content="404错误，页面不存在！">
  <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="robots" content="all" />
  <meta name="robots" content="index,follow"/>
</head>
<body>
  <script type="text/javascript" src="//qzonestyle.gtimg.cn/qzone/hybrid/app/404/search_children.js" charset="utf-8" homePageUrl="/" homePageName="回到我的主页"></script>
</body>
</html>
```

### 加速国内访问

同时部署到 `Coding.net`，在 DnsPod 设置默认访问 `you.coding.me`，国外访问 `you.github.io`即可。

如果域名已经备案，可以使用腾讯COS和腾讯CDN来加速你的blog。

## 一键图床

适用于两种平台的免费软件，可以一键上传图片到[七牛](https://portal.qiniu.com/signup/choice?code=3lnzd7a8tzxoy),[腾讯COS](https://www.qcloud.com/product/cos)等服务器，同时获取外链。

1. MAC 平台：[U 图床](http://www.lzqup.com/) & [PhotoCloud](https://github.com/liufsd/PhotoCloud)
2. Windows 平台：[MPic](http://mpic.lzhaofu.cn/)

## 错误信息

### LF will be replaced

Windows 提交命令的时候出现 `warning: LF will be replaced by CRLF in XXXXXXXXXXXXXX` 的警告。输入命令：

```
git config --global core.autocrlf false
```

## 参考链接

[使用 Hexo 搭建博客的深度优化与定制](http://blog.tangxiaozhu.com/p/45374067/)