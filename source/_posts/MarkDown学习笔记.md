---
title: 'MarkDown 语法'
date: 2017-02-17 10:28:54
tags: MarkDown
categories: coding
---

## 标题

```markdown
# 一级标题

## 二级标题

### 三级标题
```

## 换行

```markdown
这段文字换行  

第二行
```

## 列表

### 无序列表

```markdown
* 1
* 2
* 3
```

### 有序列表

```markdown
1. A
2. B
3. C
```

## 引用

```markdown
> 沉舟侧畔千帆过  
> 病树前头万木春
>> 这里是二级引用
```

## 图片和超链接

```markdown
[Google](http://Google.com)

![pic](http://g.hiphotos.baidu.com/image/pic/item/203fb80e7bec54e7f0e0839fb7389b504fc26a27.jpg)
```

## 粗体和斜体

```markdown
**这里是粗体**  
*这里是斜体*
```

## 表格

```markdown
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
```

## 代码框

### 代码行

```markdown
`init`表示初始化
```

### 代码块

```markdown
```javascript
"use strict";

var http = require("http");

http.createServer(function(request,response){

  response.writeHead(200, {'Content-Type': 'text/plain'});

  response.end('Hello World\n');
}).listen(8888);

console.log('Server running at http://127.0.0.1:8888/');
```
