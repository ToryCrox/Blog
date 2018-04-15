---
title: Git笔记
date: 2018-03-23 22:54:09
tags: Git
---

> [25个 Git 进阶技巧][1]
> [Git常用命令备忘][2]
 
## 命令及技巧

### git log查看提交记录

```bash
git log git log <file> # 查看该文件每次提交记录 
git log -p <file> # 查看每次详细修改内容的
git log -p -2 # 查看最近两次详细修改内容的
git log --stat # 查看提交统计信息
```

### git reflog查看被撤消的提交
```bash
#恢复
git reset --hard [hash]
```

### git stash暂存
`git stash`用好很强大

```bash
git stash #暂存修改，注意要先git add
git stash apply #应用暂存修改
git stash pop   #应用暂存修改并删除
git stah list   #查看暂存的修改
git stash apply [stash@{0}] #应用指定的暂存修改
```
<!--more-->
### 提交

`git commit --amend` 添加到最后一次提交

### 推送远程

正常情况下push

```bash
git push origin master
```

第一次可以推送并关联默认分支
```bash
git push -u origin master
```
这样以后每次就可以只用通过`git push`来推送了

如果需要进行`code review`, 需要改成下面这样

```bash
git push origin HEAD:refs/for/master
```

> `refs/for/master`需要经过`code review`之后才可以提交；`refs/heads/master`不需要`code review`

### 远程更新

在pull代码的时候，最好加上--rebase
```bash
git pull --rebase
```

因为git pull --rebase因为se这个命令做了以下内容：

> - 把你 commit 到本地仓库的内容，取出来放到暂存区(stash)（这时你的工作区是干净的，这样就不会产生冲突啦）
> - 然后从远端拉取代码到本地，由于工作区是干净的，所以不会有冲突
> - 从暂存区把你之前提交的内容取出来，跟拉下来的代码合并


### git blame查看某一行代码的修改历史

通过以下方法查看文件中每行的最近个性
```bash
git blame file_name
```

结果如：

```txt
0584cb5 (tory 2018-03-01 23:14:43 +0800  2) apply from: "config.gradle"
```

然后通过`git show commitID` 查看历史

或者:
> 在Android Studio中右键文件行号部分，选择`Annotate`

### git cherry-pick

`git cherry-pick`用于把另一个本地分支的`commit`修改应用到当前分支。

[git cherry-pick简介](http://blog.csdn.net/hudashi/article/details/7669462)


### 移除某文件夹的版本控制
以`bin`目录为例，如果`bin`已经被加入到

先添加到`.gitignore`里面
```bash
bin/
```

预览要删除的文件
```bash
git rm -r -n --cached "bin/"
```

执行命令:
```bash
git rm -r --cached  "bin/"
```

提交并推送到远程

```bash
git commit -m "remove bin folder all file out of control"    //提交
git push origin master   //提交到远程服务器
```

### 分支管理
```bash
#查看分支
#查看本地分支
git branch
#查看远程分支
git branch -r
#查看本地和远程分支
git branch -a

#创建分支
git branch 分支名
#切换
git checkout 分支名
#创建+切换分支
git checkout -b 分支名

#删除分支
git branch -d 分支名
```


## 基本配置

### 配置

设置Git的`user name`和`email`：

```bash
$ git config --global user.name "xxx"
$ git config --global user.email "xxx@xxx.com"
```

初始化项目:

```bash
git init
```

### SSH Keys

生成ssh key：

```bash
ssh-keygen
```
在`~/.ssh/id_rsa.pub`中把公钥复制到`github`的`ssh key`的配置项中

添加后，在终端（Terminal）中输入以下内容，测试与github的连接是否正常
```bash
ssh -T git@github.com
```
    
oschina的则是输入以下内容:
```bash
ssh -T git@git.oschina.net
```

### 关联远程仓库
> 使用下面的bash命令，将Http方式的项目改成为**SSH**方式

关联添加远程地址
```bash
git remote add origin "你项目的的ssh地址"
```
重新设置远程地址
```bash
cd "你项目的目录文件夹"
git remote set-url origin "你项目的的ssh地址"
```
### 设置commit模板
在主目录下新建`commit.template`文件，填入以下内容

```bash
BUG ID: ALHWWY-XXXX or none
DESCRIPTION: 修复xxx模块的xxx错误
```
然后设置提交模板

```bash
git config  --global commit.template ~/commit.template
git config --global core.editor vim #设置提交commit message的默认编辑器
```

### Shadowscoks代理

```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```


  [1]: http://www.imooc.com/article/1089 "25个 Git 进阶技巧"
  [2]: http://www.imooc.com/article/1111 "Git常用命令备忘"


### 查看配置信息

system（系统级别）->global（用户级别，也就是所有仓库）->local（当前仓库）
```bash
#查看系统config
git config --system --list

#查看当前用户（global）配置
git config --global  --list

#查看当前仓库配置信息（需要在仓库中使用）
git config --local  --list
```

### .gitignore

GitHub官方的所有.gitignore文件
https://github.com/github/gitignore