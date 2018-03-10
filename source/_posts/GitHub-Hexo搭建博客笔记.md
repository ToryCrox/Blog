---
title: GitHub+Hexo搭建博客笔记
date: 2018-03-04 22:50:40
tags: hexo
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

### Github上创建一个仓库
名称为：aleaf.github.io


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

#### 更换主题
theme下载：https://hexo.io/themes/

```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

打开站点_config.yml文件，配置主题为next:

```python
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

打开主题的_config.yml配置文件，不是站点主题文件，找到Scheme Settings：
next主题有三个样式，使用是Pisces

#### 设置为中文
找到主题的_config.yml，修改

```
language: zh-Hans
```