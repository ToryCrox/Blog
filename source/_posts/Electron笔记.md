---
title: Electron笔记
date: 2018-07-07 17:50:11
tags:
---


参考:

官网: https://electronjs.org/docs
[Electron: 从零开始写一个记事本app](https://www.jianshu.com/p/57d910008612)


## 环境安装

安装`node.js` : https://nodejs.org/en/

安装npm：

npm可能用不了，可以用cnpm, 官网https://npm.taobao.org/
安装命令:

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

**安装`electron`包**:

```bash
>cnpm install -g electron
OR
>npm install electron -g
```

**验证electron安装成功**: 运行`electron`

**安装Electron-forge**
这是一个类似于傻瓜开发包的Electron工具整合项目。具体介绍点击 这里。
```bash
cnpm install -g electron-forge
```


## 新建项目

通过以下命令创建项目:
```bash
electron-forge init [项目名]
```