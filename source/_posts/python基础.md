---
title: python基础
date: 2018-03-21 21:04:19
tags: python
---


## 参考
>- [Python教程 - 廖雪峰的官方网站][1]
>- [ python3-cookbook][2]
>- 中文教程及自动化测试介绍https://my.oschina.net/u/1433482/blog/634218?fromerr=oGg6OFhY


<!--more-->
**Python的设计哲学是“优雅”、“明确”、“简单”**
> Python开发者的哲学是“用一种方法，最好是只有一种方法来做一件事”

**安装**
> Python有两个版本，一个是2.x版，一个是3.x版，这两个版本是不兼容的
> Python的官方网站下载Python 3.5对应的[64位安装程序][3]或[32位安装程序][4]
>特别要注意勾上`Add Python 3.5 to PATH`，然后点“`Install Now`”即可完成安装

安装两个版本的python：

```bash
# 使用默认版本的Python
py
# 使用Python 27
py -2
# 使用Python 35
py -3
```
pip命令

```bash
py -m pip install itchat
# 指定特定版本的pip
py -3 -m pip install itchat
```

## 基础

基本语法

- `#`开头的语句是注释
- 冒号`:`结尾时，缩进的语句视为代码块，建议**4个空格**的缩进
- **大小写敏感**
- 文件头标注
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```

方法

- 输出`print('The quick brown fox', 'jumps over', 'the lazy dog')`
- 输入`name = input('please enter your name: ')`
- Python允许用`'''...'''`的格式表示多行内容
```python
print('''line1
line2
line3''')
```

### 数据类型

 - 整数
 - 浮点数   可以使用科学计数法，如1.23x10<sup>9</sup>就是`1.23e9`
 - 字符串
 - 布尔值   只有`True`、`False`两种值(注意**大小写**)   布尔值可以用`and`、`or`和`not`运算
 - 空值     `None` 不是0
 - 两种除法 `/`除法计算结果是浮点数(即使是两个整数恰好整除) `//`称为地板除，两个整数的除法仍然是整数

### Python的字符串  

 - 字符串是以Unicode编码的
 - `ord()`函数获取字符的整数表示，`chr()`函数把编码转换为对应的字符
 - `encode()`方法可以编码为指定的`bytes`，`decode()`把`bytes`变为`str`
```python
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'

>>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
'中文'
```
- 要计算str包含多少个**字符**，可以用`len()`函数,要查看字节数可结合encode
- Python中，采用的格式化方式和C语言是一致的，用`%`实现，举例如下：
```python
>>> '%2d-%02d' % (3, 1)
' 3-01'
```


### 定义函数

- `def`语句，依次写出函数名、括号、括号中的参数和冒号:
- 如果你已经把`my_abs()`的函数定义保存为`abstest.py`文件了，那么，可以在该文件的当前目录下启动Python解释器，用f`rom abstest import my_abs`来导入`my_abs()`函数，注意`abstest`是文件名（不含`.py`扩展名）
```python
def my_abs(x):
    if x >= 0:
        return x
    else:
        return -x
```
- `import`用来导入包，如`import math`语句表示导入math包
- 函数可以快返回*多个值*
```python
import math

def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny
```
- 可以传入默认参数，类似php,注意默认参数要为**不可变对象**
```python
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
```
- `*args`是可变参数，`args`接收的是一个`tuple`；`**kw`是关键字参数，`kw`接收的是一个dict。
>**可变参数**既可以直接传入`func(1,2,3)`，又可以先组装list或tuple，再通过`*args`传入：`func(*(1, 2, 3))`； 
>**关键字参**数既可以直接传入：`func(a=1,b=2)`，又可以先组装`dict`，再通过`**kw`传入：`func(**{'a': 1, 'b': 2})`。

**高级特性**

切片（Slice）  
- **`L[0:3]`**表示，从索引0开始取，直到索引3为止，但不包括索引3
- **`L[-2,0]`**表示倒数
- `list`,`tuple`,字符串者可以切片

列表生成式
- 要生成的元素x * x放到前面，后面跟for循环，就可以把list创建出来
```python
>>> [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
#全排列
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```
- `isinstance`函数可以判断一个变量是不是字符串，如 `isinstance(x, str)`

**安装第三方模块**
必须先知道该库的名称，可以在官网或者pypi上搜索,例如`pip install Pillow` 处理图片

**面向对象的编程**

```python
class Student(object): #表示Student类继承objec类
    def __init__(self, name, score): #实例化类方法，第一个参数永远是self，表示实例自己
        self.name = name
        self.score = score
    def print_score(self): //第一个参数必需是self，调用时不用传入
        print('%s: %s' % (self.name, self.score))
```
```python
bart = Student('Bart Simpson', 59)
lisa = Student('Lisa Simpson', 87)
bart.print_score()
lisa.print_score()
```

>- class后面紧接着是类名，即Student，类名通常是大写开头的单词，紧接着是`(object)`，表示该类是从哪个类**继承**下来的
>- `__init__` 方法用来创建实例,第一个参数永远是**`self`**，表示创建的实例本身，注意是两个下划线`"__"`
>- 和普通的函数相比，在类中定义的函数只有一点不同，就是第一个参数永远是实例变量`self`，并且，调用时，不用传递该参数

- 如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线`__`，在Python中，实例的变量名如果以__开头，就变成了一个私有变量（`private`），只有内部可以访问，外部不能访问
- 在Python中，变量名类似`__xxx__`的，也就是以双下划线开头，并且以双下划线结尾的，是特殊变量，特殊变量是可以直接访问的，不是`private`变量，所以，不能用`__name__`、`__score__`这样的变量名。

------------------------------
## 三方模块


### pip

**修pip改镜像**
在unix和macos，创建配置文件路径为：`$HOME/.pip/pip.conf`
在windows上，创建配置文件路径为：`%HOME%\pip\pip.ini`
在建立的pip配置文件内加入：

```ini
[global]
index-url = https://pypi.doubanio.com/simple #这里使用的是豆瓣的镜像站点
```

### 网页下载器

- urllib 官方 `import urllib.request` `import http.cookiejar`
- request 第三方
> [python 3.3 摸拟登录 小例][5]



### 网页解析器

- `beautiful Soup`第三方插件
> 下载 http://wwww.crummy.com/software/BeautifulSoup/
>- 安装: `pip install beautifulsoup4`
>- 安装lxml解析器(可选)， `pip install lxml`
>- 测试: `import bs4`


pip安装的时候总是超时，可以建个文件 `~/.pip/pip.conf`, 内容如下
```ini
[global]
index-url = http://pypi.v2ex.com/simple
```

>- [python3.4学习笔记(十七) 网络爬虫使用Beautifulsoup4抓取内容][6]

### Progressbar

参考
> http://python.jobbole.com/83976/

clone下来`https://github.com/coagulant/progressbar-python3.git`运行

	git clone https://github.com/coagulant/progressbar-python3.git
	python setup.py install


### 安装Pillow

```
pip install Pillow
```

### 安装win32crypt
下载路径
https://pypi.python.org/pypi/pywin32
https://sourceforge.net/projects/pywin32/files/pywin32/

  [1]: http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000 "Python教程 - 廖雪峰的官方网站"
  [2]: http://python3-cookbook.readthedocs.org/zh_CN/latest/preface.html "python3-cookbook"
  [3]: https://www.python.org/ftp/python/3.5.0/python-3.5.0-amd64.exe
  [4]: https://www.python.org/ftp/python/3.5.0/python-3.5.0.exe
  [5]: http://blog.csdn.net/keenweiwei/article/details/9041307 "python 3.3 摸拟登录 小例"
  [6]: http://www.cnblogs.com/zdz8207/p/python_learn_note_17.html "python3.4学习笔记&#40;十七&#41; 网络爬虫使用Beautifulsoup4抓取内容"