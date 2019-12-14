---
title: 深入理解java web-中文编码
date: 2019-12-11 20:19:12
tags: Java
categories: coding
---
## 编码

1. ASCII码：共128个，一个字节的低七位表示。
2. ISO-8859-1：单字节编码，最多表示256个
3. GB2312：双字节编码
4. GBK：为了扩展GB2312，与之兼容。
5. GB18030：可能是单字节、双字节、四字节。
6. UTF-16：始终用双字节表示，java内存的字符存储格式。
7. UTF-8：UTF-8采用了一种变长技术，每个编码区域有不同的字码长度。不同类型的字符可以由1~6个字节组成。

## java中需要编码的场景

1. io操作：StreamEncoder类负责编码，如果没有指定CharSet，会使用本地环境默认编码。

2. 内存操作：String.getBytes(指定编码)、CharSet.forName(指定编码)

## 几种编码格式的比较

1. GB2312与GBK互相兼容，但是GBK范围更大。

2. UTF-16效率更高，字节和字符转换快，适合在本地磁盘和内存使用。但是字节流损坏无法恢复，安全性较差。

3. UTF-8适合在网络传输使用，安全性较好。

## java web中涉及的编解码

看一段文本的大小，看字符本身的长度是没有意义的，即使是一样的字符采用不同的编码最终存储的大小也会不同，所以从字符到字节一定要看编码类型。

Java中一个char是16个bit，相当于两个字节，所以两个汉字用char表示在内存中占用相当于4个字节的空间。

### URL的编解码

浏览器编码URL将非ASCII字符按照某种编码格式编码成16进制数字后将每个16进制表示的字节前加上“%”。

不同的浏览器对URL的PathInfo和QueryString的编码是不一样的。

### 服务器的解码

1. 对URL的URI部分进行解码的字符集是在 connector 的\<Connector URIEncoding=”UTF-8”/>中定义的，如果没有定义，那么将以默认编码ISO-8859-1解析。
2. QueryString的解码字符集要么是 Header 中 ContentType 定义的 Charset，要么就是默认的 ISO-8859-1，要使用 ContentType 中定义的编码就要将 connector 的\<Connector URIEncoding=”UTF-8” useBodyEncodingForURI=”true”/>中的 useBodyEncodingForURI 设置为 true。

#### http header

header中的除url的其他参数，如果是非ASCIl字符，并且未进行编码直接放入header中，会出现乱码。

#### post表单

POST表单参数是通过HTTP的BODY传递到服务端的。当我们在页面上单击提交按钮时浏览器首先将根据ContentType的Charset编码格式对表单填的参数进行编码，然后提交到服务器端，在服务器端同样也是用ContentType中的字符集进行解码的。所以通过POST表单提交的参数一般不会出现问题，而且这个字符集编码是我们自己设置的。可以通过request.setCharacterEncoding(charset)来设置。

#### http body

编解码字符集可以通过response.setCharacterEncoding来设置。
访问数据库通过jdbc驱动设置编码格式，需和数据库编码保持一致。
