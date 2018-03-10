---
title: 在 Windows 平台上打造出你的 Linux 开发环境
date: 2018-03-04 23:08:33
tags: windows
---

## 资料：
- [在 Windows 平台上打造出你的 Linux 开发环境][1]
- [Cmder简单使用小结][2]
- [Win下必备利器之Cmder][3]


## Cmder
官网地址: http://cmder.net/
cmder有两个版本，mini版和full版
mini版本，就几M大小，这个也就用来替代windows的cmd
full版本比较大，100多M，模拟了一些常用linux命令（比如ls、cat、more、cp、mv、rm、find、grep等），自带git,而且自带vim。（如果已经安装了git for windows可以只使用mini版，但是要把git的src/bin添加到path中去）

### 快捷键

- `start .`  或者`e.` 打开当前路径所在的文件夹
- `Alt+H` ：显示所有快速键清单

### 自定义aliases

打开Cmder目录下的`config`文件夹，里面的aliases文件就是我们可以配置的别名文件，只需将里面ls命令的别名按下列方式修改就可以在ls命令下显示中文。

例如：
```
ll=ls -la --show-control-chars -F --color $*
la=ls -a --show-control-chars -F --color $*
```

### 启动Cmder

因为她是即压即用的存在，所以点击Cmder.exe即可运行。很显然这般打开她，不怎么快捷，即便用Listary高效搜索到她，然后点击;我们可以这样做:

- 把 cmder 加到环境变量
可以把`Cmder.exe`存放的目录添加到系统环境变量；加完之后,`Win+r`一下输入cmder,即可。

- 添加 cmder 到右键菜单
在某个文件夹中打开终端, 这个是一个(超级)痛点需求, 实际上上一步的把 cmder 加到环境变量就是为此服务的, 在管理员权限的终端输入以下语句即可:

```cmd
Cmder.exe /REGISTER ALL
```

- 添加命令(配合listary)
在选项-命令中，添加一个命令，关键字填`cmder`,路径填`cmder.exe`(需要添加到环境变量中)，参数填`/START %path%`,这样在任意地方输入`cmder`就能在当前路径打开`cmder`了

### 添加右键

可以关注这个gist。在Cmder根目录新建一个init.bat，输入以下代码：
```bat
@echo off
SET CMDER_ROOT=%~dp0
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder" /ve /d "Cmder Here" /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder" /v "Icon" /d "\"%CMDER_ROOT%cmder.exe\"" /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder" /v "Extended" /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder\command" /ve /d "\"%CMDER_ROOT%cmder.exe\" \"%%V\"" /f
pause
```

以管理员身份运行init.bat即可。删除的话再在根目录新建一个uninit.bat，依然是以管理员身份运行。代码如下：
```
@echo off
Reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder" /f
pause
```

### 解决文字重叠问题

`Win + Ait + P` 唤出设置界面 `> mian > font > monospce`,去掉那勾勾即可

### 修改命令提示符号·λ·

Cmder预设的命列列提示符号是 λ ;如果用着不习惯，可以将这个字元改成Mac / Linux环境下常见的 $ 符号，具体操作如下：

编辑Cmder安装目录下的vendor\init.bat批处理文件(min版本15行)，把：
```
@prompt $E[1;32;40m$P$S{git}{hg}$S$_$E[1;30;40m {lamb} $S$E[0m
```
修改成以下即可：
```
@prompt $E[1;32;40m$P$S{git}{hg}$S$_$E[1;30;40m $$ $S$E[0m
```
这个亲测在`cmder.exe`可以，但在PowerShell.exe需要另行设置:

打开文件`config/cmder.lua`（prompt.lua也有版本是这个），将第二行中的 λ 修改为Linux下常用的 $ 即可；亲测可行(2016-01-13)。


## Chocolatey软件包管理系统

`Chocolatey`的哲学就是完全用命令行来安装应用程序，它更像一个包管理工具（背后使用 Nuget ）
安装chocolatey , 运行如下命令即可：

    @powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin

可能需要被墙了，需要挂代理
安装软件命令 choco install softwareName, 短写是 cinst softwareName
可安装的应用程序，可以参见其 Package列表
以下是window下开发常用的开发环境应用:
```
choco install autohotkey.portable    #安装 AutoHotkey (Portable)
choco install nodejs.install  #安装 node
choco install git.install     #安装 git
choco install ruby            #安装 ruby
choco install python          #安装 python
choco install jdk8            #安装 JDK8
choco install googlechrome    #安装 Chrome
choco install google-chrome-x64 #Google Chrome (64-bit only) 
choco install firefox         #安装 firefox
choco install notepadplusplus.install #安装 notepad++
choco install Atom                    #安装 Atom
choco install SublimeText3            #安装 SublimeText3
choco install wget
```

  [1]: http://www.oschina.net/news/46712/develop-on-windows-as-if-it-was-unix "在 Windows 平台上打造出你的 Linux 开发环境"
  [2]: https://github.com/Just1n/Posts/blob/master/Cmder简单使用小结.md "Cmder简单使用小结"
  [3]: http://www.cnblogs.com/jadeboy/p/5132423.html "Win下必备利器之Cmder"
  [4]: http://cmder.net/