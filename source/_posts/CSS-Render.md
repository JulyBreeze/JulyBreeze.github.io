---
title: How CSS Render Website
date: 2019-02-08T17:34:23.000Z
tags: CSS
categories: CSS
---

# Basic Knowledge
1.Box Model  
{% asset_img box.png BOX %}

Padding is transparent inside of the box.  
Margin is the space between boxes, it's outside the box.  
Fill area is the area that get filled with background color/image.  

```css
box-sizing: border-box/content-box(default)
```

- content-box  
  width/height代表的是content，所以box的宽高 = content的宽高 + padding和border
  The width/height of box = set width/height + padding + border
- border-box  
  width/height代表的是box
  The padding and border are inclued in the box width/height. So the content width/height = box width/height - border - padding.
  使用这个，就更方便的设置整个box的size

2.Main three types of Box  
Block-level, inline, inline-block  
(All html element has display property)

```css
display: inline(default), block, inline-block, content, table...
```

- block块级元素   
  the block-box always occupies 100% of its parent's width
- inline  
  the inline-box only occupies the space that its content needs  
  no width/height property  
  padding and margin only are about left/right
- inline-blcok  
  与inline相同，但是这个有width/height属性
- content  
  the container disappear, making the child elements children of the element the next level up in the DOM

3.Position

```css
position: static(default)/relative/absolute/fixed
```

* relative: the element is positioned relative to its normal position.
* absolute: the element is positioned relative to its first positioned(not static) ancestor element
* fixed : the element is positioned relative to the browser window

```css
float: none(default)/left/right
```
使用absolute的元素无需考虑float.


- normal flow
- float  
  The element will float to leftmost/rightmost of its containing box
- absolute  
  和float不同的是，has no impact on surrounding content or element, may overlap them

4.Stacking contexts  
z-index, filter, opacity, etc. can creat stacking contexts

5.Pseudo-classes  
Add some special effect to selectors

```
selector : pseudo-class {}
selector.class : pseudo-class {}
```

Anchor < a > Pseudo-classes:

  ```css
  a: link {}  unvisited link
  a: visited {}
  a: hover {}
  a: active {}  selected link
  ```

# CSS Architecture
1.How to name class  
Many approaches to name the class to make the code maintainable and reuseable.  
Block Element Modifier的命名规则:
```css
.block {}
.block__element {}
.block__element--modifier {}
```

- block is the component that can be reuseable.  
- element is part of the block that can not reused outside of block.  
- modidifer is a different version of a block or element.

2.Folders and files organization  
Seven folders for partial Sass files; One main Sass file to import all other files into a compiled CSS stylesheet.

```
.base/
.components/
.layout/
.pages/   styles for specific page
.themes/
.abstracts/ put code that doesn't output any CSS, such as variables
.vendors/  put all third-party CSS
```

# Basic responsive design principles
1. Fluid grids and layouts  
To allow content to easily adapt to the current viewport width used to browse the website. Uses % rather than px for all layout-related lengths.  
2. Flexible/responsive images  
Images behave differently than text content, and so we need to ensure that they also adapt nicely to the current viewport.  
3. Media queries  
To change styles on certain widths(breakpoints), allowing us to create different version of website for different widths.

# Layout types
1. Float Layouts  
2. Flexbox  
3. CSS Grid

# Some tips using vsc extensions
```
.composition>(img.composition__photo.composition__photo--p1)*3  
 then click TAB
```

```html
<div class="composition">
  <img src="" alt="" class="composition__photo composition__photo--p1">
  <img src="" alt="" class="composition__photo composition__photo--p1">
  <img src="" alt="" class="composition__photo composition__photo--p1">
</div>
```
