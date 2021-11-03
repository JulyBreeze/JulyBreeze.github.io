---
title: HTML basics
date: 2019-06-21 11:56:23
tags: html
---

###文件结构

```html
<!DOCTYPE html>
<html>
  <head>
    里面的内容不会显示在browser
    里面有meta tag, title tag, 和用来引入外面静态文件的link tag
  </head>
  <body>
    显示在浏览器中的内容，
    一般有标题栏<header>和主体部分<main>
  </body>
</html>
```

### 常用tag

#### 1.div & span

1) 块级与行内元素的区别

* 块级占一行，能设置宽高，如果不设置宽高，其width默认为父元素的100%
* 行内，不能设置宽高。默认的width是内容的宽度。

常见块级元素：p, h1 - h6，排版标签一般都是。  
行内元素：span, b, i, u, font等，文本修饰标签一般都是

```html
<div>块级标签，通过CSS赋不同的样式</div>
<span>行内标签，通过CSS赋不同的样式</span>
```


#### 2.img

```html
<img src="xxx.jpg" alt="Smiley face" height="10" width="10">
```

#### 3.列表

1.无序  
< ul >和< li >是一组组标签，ul里面只能有li，不能有其他的tag，但li里可以有别的tag
```html
<!-- 显示在浏览器是实心小圆点 -->
<ul> ul is unordered list
  <li></li> li is list item
</ul>
```

2.有序  
组合标签

```html
<ol start="2" type="A"> 序号从2开始，样式是大写的ABCD..
  <li></li>
</ol>
```

3.定义标签  
组合标签

```html
<dl> defination list
  <dt></dt> defination title
  <dd></dd> defination description
</dl>
```

#### 4.表格table

```html
<table>
  <tr> 行
    <td></td> 列
    <td></td>
  </tr>
</table>
```

#### 表单form

```html
<!-- 
action: defined the action to be performed when the form is submitted
target: specifies if the submitted res will open in a new browser tab
method: specifies the HTTP method(get or post) when submitting the form data 
 -->
<form action="/xxx" target="_blank" method="get">
  <!--input输入框中会默认显示John
  每个input里都应该有name attribute, 用来承载输入数据的
  -->
  <input type="text" name="firstName" value="John"> 
  <input type="radio" name="gender" value="male" checked> Male
  <input type="submit" value="Submit">
</form>
```

关于什么时候用GET或POST？  
1)The default method when submitting form data is "GET".  
When GET is used, the submitted form data will be visible in the page address field:

```
/xxx?firstname=John
```

2)Always use "POST" if the form data contains sensitive or personal information.   
The POST method does not display the submitted form data in the page address field.

```html

```

####

```html

```

```html

```


```html

```


```html

```


```html

```


```html

```