---
title: 0.JS Basics
date: 2019-04-17 16:08:17
tags: js
---

### 1.Five primitive types

Javascript is a scripting language(executed without being compiled).

Null, Undefined, Boolean, Number, String  
这些基本类型在内存中占据的空间大小是固定的，保存在stack内存中;  

reference type得值是object，存放在heap内存中。

```javascript
//求xxx是哪种基本类型
typeof xxx
//判断aa是否是bb的instance，或者说aa是否是bb这个type
aa instanceof bb
```

具体使用：

```javascript
console.log(typeof true); //boolean
```

```javascript
const Car = (model, year, make) {
  ...
}

var auto = new Car("a", 2000, "b");

console.log(auto instancof Car); //true
console.log(auto instanceof Object); //true
```

### 2.Execution context

定义了变量或函数有权访问的其他数据。 每个执行环境中的代码执行完毕后，该环境会被销毁。  
全局执行环境在浏览器中是window对象，当关闭网页或浏览器后，才会被销毁。

1) variable object  
每个exectuion context都有一个与之相关的variable object —— 环境中定义的所有变量和函数都保存在其中。

2) scope chain作用域链(从里向外)  
每次进入新的execution context，会创建一个scope chain。保证对execution context有权访问的所有变量和函数的有序访问。大概是由variable object组成的chain

scope chain的首端是当前执行的代码所在的execution context的variable object。  
如果这个环境是function，则其activation object就是variable object。scope chain:

```html
activation object(arguments对象) ——> 外部的context ——> 更外部的... 全局执行环境的variable object
```

搜索某个变量从chain的首端开始，往后直到找到为止。

举例：

```javascript
var color = "blue";

function changeColor() {
  var anotherColor = "red";

  function swapColor(){
    var tempColor = anotherColor;
    anotherColor = color;
    color = tempColor;
    //可以访问color, anotherColor,tempColor
  }
  //可以访问color和anotherColr，不能访问tempColor
  swapColors();
}
//只能访问color
changeColor();
```

父环境不可以获取子环境的变量，但子环境可以获取父环境的变量

3) 延长scope chain

在chain的前端增加一个variable object，两种方式：

* try-catch的catch语句
* with语句

例：

```javascript
function buildUrl(){
  var qs = "?debug=true";
  //The location object contains information about the current URL.
  //The location object is part of the window object，
  //it is accessed through the window.location property.
  with(location) {
    var url = href + qs;
  }

  return url;
}
```

解释：？？不是很理解

### 3.Garage collection

自动的，按照固定的时间间隔周期性的执行。

* mark and sweep 标记清除  
variable进入context，就标记为进入，离开就标记为离开。  
* reference counting引入计数  
跟踪记录每个object被引用的次数。当某个object被赋值给某个reference variable时，引用次数加1，反之减1。当引用次数为0时，就把内存空间回收。

### Reference type