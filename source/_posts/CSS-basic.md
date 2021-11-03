---
title: CSS Basic
date: 2019-03-20 09:42:49
tags: css
---

### 插入css

三种方法：  
1)External style sheet

```html
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css">
</head>
```

2)Internal style sheet
```html
<head>
  <style>
    .box {...}
  </style>
</head>
```

3)Inline style
当style仅需要在一个element上应用一次时。

```html
<p style="...">xxx</p>
```

### 常见的css

1.margin   
其值可以有：auto、length(px,em等)、%

```css
margin-top:100px;
margin-bottom:100px;
margin-right:50px;
margin-left:50px;
```

简写属性：

```css
/* 1到4个值,中间以空格分开，顺时针：上 右 下 左 */
margin: 50px 50px 50px 50px;

/* 上 左右 下 */
margin: 50px 25px 50px;

/* 上下 左右 */
margin: 50px 50px;

/* 四个边距都是 */
margin: 50px;
```

2.padding   
其值: length(px,em等)、%

```css
padding-top:25px;
padding-bottom:25px;
padding-right:50px;
padding-left:50px;
```

```css
/* 1到4个值,中间以空格分开，同margin一样 */
padding
```

3.position  
Element可以使用的top、bottom、left、right属性定位。但必须先设定position属性。  
position有5个值:

```html
relative
absolute：element位置相对于最近的已定位的父元素，如果ele没有已定位的父元素，则其位置相对于<html>
fixed: 元素的位置想读与browser window是固定的，即便window是滚动的也不会移动
static
sticky：基于用户的滚动位置来定位
```

例：xxx相对于aaa左移20px
```html
<style>
  h2.pos_left{
    position:relative;
    left: -20px;
  }
</style>
<h2>aaa</h2>
<h2 class="pos_left">xxx</h2>

```


4.display & visibility

1)常用的值

```css
none: 此元素不会被显示
block: 此元素将显示为块级元素，此元素前后会带有换行符。
inline: 默认。此元素会被显示为内联元素，元素前后没有换行符。
inline-block:行内块元素
table: 元素会作为块级表格来显示（类似 <table>），表格前后带有换行符。
inherit:
...
```

2)display & visibility  
display可以隐藏某个元素，且隐藏的元素不会占用任何空间：

```css
display:none;
```

visibility可以隐藏某个元素，但隐藏的元素仍需占用与未隐藏之前一样的空间：

```css
visibility:hidden;
```

5.Dimension  
属性有：

```
height:
line-height:
max-height:
min-height:

width:
max-width:
min-widht:
```

6.float  
Float会使元素向左或向右移动，其周围的元素也会重新排列。往往用于图像。

一个浮动元素会尽量向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止。  
浮动元素之后的元素将围绕它，但浮动元素之前的元素将不会受到影响。

```css
float: left、right;
```

清除float：  
因为元素浮动之后，周围的元素会重新排列。为了避免这种情况，使用clear属性。  
clear属性指定元素两侧不能出现浮动元素。
```css
clear: both;
```

7.overflow  
控制content溢出元素框时显示的方式。

```
visible: 默认，内容不会被修剪，会呈现在元素框之外
hidden: 内容会被修剪，并且其余内容是不可见的。
scroll: 内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。
auto:   如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。
inherit: 从父元素继承 overflow 属性的值。
```


### 伪类和伪元素

```css
selector:pseudo-element {property:value;}
selector.class:pseudo-element {property:value;}
```

伪类
```
:hover
:link
:active
:target
:focus
:not()
```

伪元素
```
::before
::after
::selection
::first-letter
::first-line
```

#### 伪元素before和after
::before和::after{}里特有属性content，content必须有值，至少是空。用于在css渲染中向元素逻辑上的头部或尾部添加内容。这些添加不会出现在DOM中，不会改变文档内容，不可复制，仅仅是在css渲染层加入。

### 常见的trick

1.clearfix  
If an element is taller than the element containing it, and it is floated, it will overflow outside of its container.  
可以使用overflow: auto
```css
/* ::after是伪元素,可以在元素的内容之后插入新内容 */
clearfix ::after{
  content: "";
  display: table;
  clear: both;
}
```

2.absolute center position
```css
absCenter {
  postion: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```
