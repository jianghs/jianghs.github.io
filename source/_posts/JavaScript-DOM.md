---
title: JavaScript DOM
date: 2019-08-09 15:26:22
tags: JavaScript
categories: coding
---
## 语法

### 对象

* 属性 Object.property
* 方法 Object.method()

如 Person.age\Person.get()

#### 类别

1. 内建对象：语言本身的对象，如 new Date(),new Array()等。

2. 宿主对象：browser or node

3. 用户自定义对象。

## DOM

### 文档 - D

DOM 将编写的网页文档转换成文档对象。

### 对象 - O

windows 对象对应这浏览器本身，这个对象的属性和方法通常统称为 BOM （浏览器对象模型）

### 模型 - M

某种事物的表现形式。

### 节点

#### 元素节点

DOM 的原子。

这些节点构成了文档的结构。

#### 文本节点

表示 DOM 的具体文本。

#### 属性节点

对元素更具体的描述。

所有的属性节点都被元素节点包含。

### CSS

```css
selector {
  property : value;
}
```

节点树上的各个元素将继承父元素的样式属性。

#### class 属性

```html
<h1 class="special">hello world</h1>
```

```css
.special {
  font-style : italic;
}
```

#### id 属性

独一无二的标识符。

```html
<h1 id="special">hello world</h1>
```

```css
#special {
  font-style : italic;
}
```

#### 获取元素

1. getElementById: 通过元素 ID
2. getElementByTagName: 通过标签名字，返回一个数组
3. getElementByClassName: 通过类名字，返回一个数组

### 获取和设置属性

getAttribute

setAttribute

## 最佳实践

平稳退化：确保网页在没有 JavaScript 的情况下也能正常工作。

分离 JavaScript：把网页结构内容与脚本动作行为分开。

向后兼容性：确保老浏览器不会因为脚本死掉。

性能考虑：脚本执行性能最优。

## 创建动态标记

### createElement

创建元素节点

### appendChild

### createTextNode

创建文本节点
