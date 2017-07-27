---
title: Python爬虫（一）：第一个网络爬虫
date: 2017-04-24 09:15:22
tags:
categories:
---

### 如何获取一个页面
1. 引入Python自带的`urllib`包中的`request`模块。
2. 打开一个网站页面。
3. 将页面内容读取并输出。

```
from urllib.request import urlopen

html = urlopen("https://www.baidu.com/")
print(html.read())

```

### beautifulsoup
上述例子给出了获取一个页面的整个内容。对于一些复杂的网页，有时候我们只是想取到里面的某些内容，所以上述例子显然是不够的，这时候我们需要引入另一个包`beautifulsoup`,通过这个包我们可以很方便的获取我们想要的内容。

#### 安装
Windows：命令行下切换到Python对应的安装目录下，通过自带工具`pip`安装，安装命令如下：
```
pip install beautifulsoup4
```

一个例子：
以下即可获取到百度首页的所有div及其内部元素信息。
```
from urllib.request import urlopen
from bs4 import BeautifulSoup

html = urlopen("http://www.baidu.com/")
bsObj = BeautifulSoup(html.read())
print(bsObj.div)

```

### 添加异常处理
为了使程序更加健壮，我们需要增加异常处理。

```
from urllib.request import urlopen
from urllib.error import HTTPError
from bs4 import BeautifulSoup

def getcontent(url):
    try:
        html = urlopen(url)
    except HTTPError as e:
        return None

    try:
        bsObj = BeautifulSoup(html.read())
        result = bsObj.h1
    except AttributeError as e:
        return None

    return result

result = getcontent("http://www.baidu.com")
if result == None:
    print("结果为空")
else:
    print(result)
```
