---
title: Sass
date: 2019-02-10 00:58:56
tags: CSS
categories: CSS
---
Sass is a CSS preprocessor.  
Sass code -> Sass compiler-> Compiled CSS

Sass has two syntax: Sass syntax(indentation sensitive), SCSS syntax. So usually choose SCSS syntax.

SCSS  
### 1.variables and nest
variables starts with $ dollar sign
```html
<nav class="clearfix">
  <ul class="navigation">
    <li><a href="#"> About us</a></li>
    <li><a href="#"> Pricing</a></li>
    <li><a href="#"> Contact</a></li>
  </ul>
  
  <div class ="buttons">
    <a class="btn-main" href="#"> Sign Up</a>
    <a class="btn-hot" href="#"> Get a quote</a>
  </div>
</nav>
```
```css
*{
  margin: 0;
  padding: 0;
}

$color-primary: #f9ed69; 
// yellow
$color-secondary: #f08a5d;
$color-tertiary: #b83b5e;
$width-button: 150px;
nav {
  margin: 30px;
  background-color: $color-primary;

  &::after{
    content: "";
    clear:both;
    display: table;
  }
}

/* we can move this class into nav
this is to fix float collapse
.clearfix::after{
  content: "";
  clear:both;
  display: table;
}
*/
.navigation {
  list-style: none;
  float:left;
  li {
    margin-left:30px;
    display: inline-block;
    
    &:first-child{
      margin-left:0px;
    }
    
    a:link{
      text-decoration: none;
      text-transform: uppercase;
      color:$color-tertiary;
    }
  }
}

.buttons {
  float: right;
}

.btn-main:link,
.btn-hot:link{
  padding: 10px;
  display:inline-block;
  text-decoration:none;
  border-radius:100px;
  text-transform: uppercase;
  width: $width-button;
}

.btn-main{
  &:link{
    background-color: $color-secondary;
  }
  
  &:hover{
    background-color: darken($color-secondary, 15%);
  }
}

.btn-hot{
  &:link{
    background-color: $color-tertiary;
  }
  
  &:hover{
    background-color: lighten($color-tertiary, 15%);
  }
}
```

```css
//compile to css, it is:
.navigation li:first-child{
}
```

### 2.Mixin, Extend, Function  
#### 1)mixin: @mixin xxx + @include xxx
mixins is a piece of code written as mixins
so this piece of code will be put into the place where we call mixins

```css
@mixin clearfix {
  &::after{
    content: "";
    clear:both;
    display: table;
  }
}
//can also have argument
@mixin style-link-text($col) {
  text-decoration: none;
  text-transform: uppercase;
  color:$col;
}

nav {
  margin: 30px;
  background-color: $color-primary;
  
  @include clearfix
}

.btn-hot:link{
  padding: 10px;
  display:inline-block;
  text-align: center;
  border-radius:100px;
  width: $width-button;
  @include style-link-text($color-text-light);
}
```
#### 2) Function: @function xx 
```css
@function divide($a, $b) {
  @return $a / $b;
}

nav {
  margin: divide(60, 2) * 1px;
  background-color: $color-primary;
  
  @include clearfix
}
```
#### 3) Extend: %xxx + @extend %xxx
```css
%btn-placeholder {
  padding: 10px;
  display:inline-block;
  text-align: center;
  border-radius:100px;
  width: $width-button;
  @include style-link-text($color-text-light);
}

.btn-main{
  &:link{
    @extend %btn-placeholder;
    background-color: $color-secondary;
  }
  
  &:hover{
    background-color: darken($color-secondary, 15%);
  }
}

.btn-hot{
  &:link{
    @extend %btn-placeholder;
    background-color: $color-tertiary;
  }
  
  &:hover{
    background-color: lighten($color-tertiary, 15%);
  }
}
```
#### The difference of extends and mixins
The element that use extend% will be put on the place where the @extend is after compile, while mixin paste its content to the place where uses it, which does not follow 'Don't repeat yourself' rule. So use extend.  
Eg:
```css
//btn-main:link and btn-hot:link put on here(%btn-placeholder) after compile:

.btn-main:link, .btn-hot:link {
  padding: 10px;
  display:inline-block;
  text-align: center;
  border-radius:100px;
  width: $width-button;
  @include style-link-text($color-text-light);
}
```


### 3.Installation
```
npm install node-sass --save-dev
npm install live-server -g
```
The second one is used to automatically run HTML/CSS file

Add the test script in package.json, eg:
```json
//the content after :, 
// node-sass, file need to be compiled, file to store the compiled css code.
"scripts": {
    "compile:sass": "node-sass sass/main.scss css/style.css -w"
  },
//-w means watch whether the sass file changes or not
//if it changes, automatically compile
```
Then run on the terminal
```
npm run compile:sass
live-server
```

