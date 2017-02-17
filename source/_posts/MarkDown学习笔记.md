---
title: MarkDown学习笔记
date: 2017-02-17 10:28:54
tags: MarkDown
---
# 一级标题

## 二级标题

### 三级标题

---
#### 换行（两个空格后回车）
这段文字换行  
第二行
---
#### 无序列表
* 1
* 2
* 3

#### 有序列表
1. A
2. B
3. C

---

#### 引用
> 沉舟侧畔千帆过  
> 病树前头万木春
>> 这里是二级引用
---

#### 图片和超链接

[Google](http://Google.com)

![pic](http://mouapp.com/Mou_128.png)

---
#### 粗体和斜体

**这里是粗体**  
*这里是斜体*

---
#### 表格
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

---
### 代码框
`init`表示初始化

```
"use strict";

var http = require("http");

http.createServer(function(request,response){

  response.writeHead(200, {'Content-Type': 'text/plain'});

  response.end('Hello World\n');
}).listen(8888);

console.log('Server running at http://127.0.0.1:8888/');

```
