---
title: Python爬虫（二）：beautifulsoup进阶
date: 2017-04-28 09:23:12
tags:
categories:
---
### 通过属性标签搜索
假设页面中包含如下两段程序，我们可以通过`beautifulsoup`获取指定的span。

```
<span class="green"></span>
<span class="red"></span>
```

一个例子：
需要获取指定页面的所有
```
from urllib.request import urlopen
from bs4 import BeautifulSoup

html = urlopen("http://www.pythonscraping.com/pages/warandpeace.html")
bsObj = BeautifulSoup(html)

nameList = bsObj.find_all("span", {"class":"green"})
for name in nameList:
    print(name.get_text())
```

#### find_all

搜索当前 tag 的所有 tag 子节点，并判断是否符合过滤器条件。
```
find_all( name , attrs , recursive , text , **kwargs )
```
name 参数：

查找所有名字为 name 的 tag。
```
bsObj.find_all("title")
```

keyword 参数：

搜索指定名字的属性时可以使用的参数值包括`字符串`,`正则表达式`,`列表`,`True`.
```
bsObj.find_all(id="link2")
```

按 css 搜索：

按照CSS类名搜索tag的功能非常实用,但标识CSS类名的关键字 class 在 Python 中是保留字,使用 class 做参数会导致语法错误.从 Beautiful Soup 的4.1.1版本开始,可以通过 class_ 参数搜索有指定CSS类名的tag.

class_ 参数同样接受不同类型的`过滤器` ,`字符串`,`正则表达式`,`方法`或 `True` .
```
bsObj.find_all("a", class_="sister")
```

text 参数：

通过 text 参数可以搜搜文档中的字符串内容.与 name 参数的可选值一样, text 参数接受 字符串 , 正则表达式 , 列表, True .
```
bsObj.find_all("a", text="Eric")
```

limit 参数：

限制返回结果数量。
```
bsObj.find_all("a", limit=2)
```

resursive 参数：

调用tag的 find_all() 方法时,Beautiful Soup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 recursive=False .
```
bsObj.find_all("a", recursive=False)
```

#### find
```
find( name , attrs , recursive , text , **kwargs )
```
基本用法和`find_all()`一样，只是少limit限制。
