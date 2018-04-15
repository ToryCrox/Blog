---
title: GitHub+Hexo搭建博客笔记
date: 2018-03-04 22:50:40
tags: "hexo"
---

## 参考资料

[GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249?utm_source=com.evernote&utm_medium=social)

## 搭建步骤：
获得个人网站域名
- GitHub创建个人仓库
- 安装Git
- 安装Node.js
- 安装Hexo
- 推送网站
- 绑定域名
- 更换主题
- 初识MarkDown语法
- 发布文章
- 寻找图床
- 个性化设置
- 其他
- 附录

<!--more-->
### Github上创建一个仓库
名称为：aleaf.github.io


### 安装Node.js

Hexo基于`Node.js`，`Node.js`下载地址：[Download | Node.js](https://nodejs.org/en/download/) 下载安装包，注意安装Node.js会包含环境变量及npm的安装，安装后，检测Node.js是否安装成功，在命令行中输入 node -v :

### 安装Hexo

使用npm命令安装Hexo，输入：
```bash
npm install -g hexo-cli 
```
安装完成后，初始化我们的博客，输入：
```bash
hexo init blog
```

为了检测我们的网站雏形，分别按顺序输入以下三条命令：

```
hexo new test_my_site

hexo g

hexo s
```

这些命令在后面作介绍，完成后，打开浏览器输入地址：
```
localhost:4000
```

常用命令:
```
npm install hexo -g #安装Hexo
npm update hexo -g #升级 
hexo init #初始化博客

命令简写
hexo n "我的博客" == hexo new "我的博客" #新建文章
hexo g == hexo generate #生成
hexo s == hexo server #启动服务预览
hexo d == hexo deploy #部署

hexo server #Hexo会监视文件变动并自动更新，无须重启服务器
hexo server -s #静态模式
hexo server -p 5000 #更改端口
hexo server -i 192.168.1.1 #自定义 IP
hexo clean #清除缓存，若是网页正常情况下可以忽略这条命令
```

### 推送网站
打开站点的配置文件_config.yml，翻到最后修改为：

```bash
deploy:
  type: git
  repo: git@github.com:ToryCrox/aleaf.github.io.git
  branch: master
```

保存站点配置文件

```bash
npm install hexo-deployer-git --save
```

这时，我们分别输入三条命令：

```bash
hexo clean 
hexo g 
hexo d
```

### hexo配置

#### Next主题配置官方文档

> - [主题配置 - NexT 使用文档](http://theme-next.iissnan.com/theme-settings.html)  
> - [第三方服务集成 - NexT 使用文档](http://theme-next.iissnan.com/third-party-services.html)  
> - [内置标签 - NexT 使用文档](http://theme-next.iissnan.com/tag-plugins.html)  
> - [进阶设定 - NexT 使用文档](eme-next.iissnan.com/advanced-settings.html)

### 个性个配置
> - [hexo的next主题个性化配置教程](https://segmentfault.com/a/1190000009544924#articleHeader12)

#### 更换主题
theme下载：https://hexo.io/themes/

```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

打开站点`_config.yml`文件，配置主题为next:

```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

打开主题的_config.yml配置文件，不是站点主题文件，找到Scheme Settings：
next主题有三个样式，使用是Pisces

#### 设置为中文
找到主题的_config.yml，修改

```yml
language: zh-Hans
```

#### 个性化配置

主要个性主题配置文件`_config.yml`

社交主页设置, 找到`social`
```yml
social:
  GitHub: https://github.com/ToryCrox
```

#### 增加侧栏菜单条目

默认的侧栏菜单条目有：首页、归档、标签、关于、搜索等。如果你想要增加其他的菜单条目，修改主题配置文件`_config.yml`里的`Menu Settings`中的`menu`和`menu_icons`两个地方

设置侧栏的位置:修改 主题配置文件 中的 `sidebar` 字段:
```yml
sidebar:
  position: left
```
见:http://theme-next.iissnan.com/getting-started.html#sidebar-settings

#### 启用搜索功能
自定义站点内容搜索Local Search

1. 安装 `hexo-generator-searchdb`，在站点的根目录下执行以下命令：
```bash
$ npm install hexo-generator-searchdb --save
```

2. 编辑 站点配置文件，新增以下内容到任意位置：
```yml
search:
  path: search.xml
  field: post Toggle strikethrough
  format: html
  limit: 10000
```

3. 编辑 主题配置文件，启用本地搜索功能：
```yml
# Local search
local_search:
  enable: true
```

#### 添加标签页面

1. 使用命令新建一个标签页面
```bash
hexo new page tags
```

2. 编辑刚新建的页面，将页面的类型设置为 tags ，主题将自动为这个页面显示标签云。页面内容如下：
```yml
title: 标签
date: 2014-12-22 12:39:04
type: "tags"
```

3. 修改菜单，编辑主题配置文件 ， 添加 tags 到 menu 中
```yml
menu:
  home: /
  archives: /archives
  tags: /tags
```


[主题配置 - NexT 使用文档](https://link.zhihu.com/?target=http%3A//theme-next.iissnan.com/theme-settings.html)

2. 编辑刚新建的页面，将页面的类型设置为 tags ，主题将自动为这个页面显示标签云。页面内容如下：
```
title: 标签
date: 2014-12-22 12:39:04
type: "tags"
```

3. 修改菜单，编辑主题配置文件 ， 添加 tags 到 menu 中
```ini
menu:
  home: /
  archives: /archives
  tags: /tags
```

#### 让首页不显示全文
有两种方法，参考: [Hexo Next 阅读全文设置 Next主题怎么让首页不显示全文](http://www.5isjyx.com/coding/201704/nextreadthefulltext.html)

1. `themes/next` 目录下的 `_config.yml` 文件，找到这段代码

```yml
# Automatically Excerpt. Not recommend.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: false
  length: 150
```

把 `enable` 的 `false` 改成 `true` 就行了，然后 `length` 是设定文章预览的文本长度

2. 写 md 文章的时候，可以在内容中加上 `<!--more-->`，这样首页和列表页展示的文章内容就是 `<!--more-->` 之前的文字，而之后的就不会显示了。

> 虽然第一种方法配置简单，但是显示的预览会乱，所以我选第二种方法

#### 设置首页文章数量

首页默认显示10篇文章，会导致首页很长，可以在站点`_config.yml`文件中，搜索`per_page`，修改显示数量
```yml
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 5
  order_by: -date
```

参考: [Hexo程序archive页面数量设置](http://www.yuzhewo.com/2015/11/21/Hexo%E7%A8%8B%E5%BA%8Farchive%E9%A1%B5%E9%9D%A2%E6%95%B0%E9%87%8F%E8%AE%BE%E7%BD%AE/)