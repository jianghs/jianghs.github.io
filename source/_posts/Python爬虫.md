---
title: Python爬虫
date: 2017-04-24 09:15:22
tags:
categories:
---

## Python爬虫（一）：第一个网络爬虫

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

####
