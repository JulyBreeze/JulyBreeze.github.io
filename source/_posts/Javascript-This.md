---
title: 3.Javascript-This
date: 2019-03-14 17:03:02
tags: js
---

### this

随着使用场景的不同，this发生改变.  
**谁调用函数，this指向谁**。**匿名函数的指针永远指向window**。

* Global context/Scope  
this refers to the global object window both in strict mode or not.
* Function scope
this depends on how the function is called.  
In strict mode, if this was not defined by the execution context, it remains undefined.

```javascript
//由window调用
function f1(){
  return this;
}
//in browser f1() === window;
//in Node, f1() === global

function f2(){
  'use strict';
  return this;
}

//f2() === undefined
```

```html
<input type="button" value="button" id="btn" />
<script>
  function fn(){
    console.log(this);
  }
  fn(); //this refers to window

  //1.
  var bt = document.getElementById("btn");
  bt.onclick = function() {
    console.log(this); //when click button, this refers to bt object
  }
  //2.
  bt.conclick = fn; //assign this function to bt object, this refers to bt object
  //3.
  bt.onclick = function(){
    console.log(this); //refers to bt
    fn();//这时this refers to window
    //因为点击按钮后，执行匿名函数，匿名函数调用了fn
    //所以fn()里面console输出的是window
  }
</script>
```

点击哪个按钮，哪个按钮就呈active:

```html
<input type="button" value="button1" />
<input type="button" value="button2" />
<input type="button" value="button3" />

<script>见下</script>
```

```javascript
  var btns = document.getElementByTagName("input");
  for(var i = 0; i < btns.length; i++){
    btns[i].onclick = function() {
      //clear the styles of all the buttons
      for(var j = 0; j < btns.length; j++){
        btns[j].className="";
      }
      this.className="active";
    }
  }

```

### call apply bind

都是Function对象自带的三个方法，作用是**改变函数中的this指向**。

相同点:

* 第一个参数都是this要指向的对象
* 都可以利用后续参数传参

不同的是:

* call、apply立即调用; bind返回对应的函数，稍后调用
* call() accepts an argument list, apply() accepts a array or array-like object.

**1.call**  
Allows for a function/method belonging to one object to be assigned and called by a different object.

```javascript
//全是optional
某个func/method.call(thisArg, arg1, arg2 ...)
```

thisArg的取值有四种情况:  
a) 无，或者null，defined：this指向window对象  
b) 传递另一函数的函数名：this指向这个函数的引用(function是object，所以func name就是对func的引用)   
c) 传递string、number、boolean等基础类型：this指向其对应的包装boxed对象  
d) 传递一个对象：this指向这个对象  

```javascript
function a() {
  console.log(this);
}

function b(){}
var c = {name : "jonh"}

//以下三个都是输出window
a.call();
a.call(null);
a.call(undefined);

a.call(b); //function b(){}

a.call(1); // Number
a.call(" "); // String
a.call(true); // Boolean

a.call(c); //Object
```

例1：
```javascript
function add(x, y){
  console.log(x + y);
}

function subtract(x, y){
  console.log(x - y);
}

add.call(subtract, 3, 2); // 5
```
解释：
相当于把subtract替换成add，就等同于add(3, 2)

例2：
```javascript
//这是Product的constructor
function Product(name, price) {
  this.name = name;
  this.price = price;
}

//允许Food call Product这个constructor
function Food(name, price) {
  Product.call(this, name, price);
  this.category = "food";
}

function Toy(name, price) {
  Product.call(this, name, price);
  this.category = "toy";
}

var cheese = new Food('feta', 5);
var fun = new Toy('robot', 40);
```
解释：
call() provides a new value of this to the function/method Product.  
With call, you can write method once, then inherit it in another object Toy and Fodd, without having to rewrite this method for new object.即可以理解为把this指针传入call()，那么就可以在当前的context下使用Product这个method了。


例3：
```javascript
var animals = [{species: 'Lion', name: 'King'}, {species: 'whale', name: 'happy'}];

for(var i = 0; i < animals.length; i++){
  //匿名函数
  (function(i)) {
    this.print = function() {
      console.log(i + ',' + this.species + ':' + this.name);
    }

    this.print();
  }).call(animals[i], i);
}
```
  
2.apply()

```javascript
//都是optional
//argsArray是一个array或者array-like object
某func.apply(thisArg, [argsArray])
```

例1：
```javascript
var array = ['a', 'b'];
var elements = [0, 1, 2];
array.push.apply(array, elements);

console.info(array); // ["a", "b", 0, 1, 2];
```
解释：
如果直接用array.push(elements)，那么elements是当作一个single element加入到array中的。如果用concat()则会产生一个新的数组，而不是在原数组上修改。利用上述代码就可以实现在原来array中push进elements的所有元素。

例2：
```javascript
var numbers = [1,2,3,4,5];
var max = Math.max.apply(null, numbers);
//equal to max(numbers[0], numbers[1]...)

var min = Math.min.apply(null, numbers);
```

例3：
```javascript
function Product(name, price) {
  this.name = name;
  this.price = price;
}

//如果Food和Product里的相同的参数是一一对应的，就可以用arguments，用apply()即可
//但如果Product里的参数是price,name，用call()来实现。
function Food(name, price,category) {
  Product.apply(this, arguments);
  this.category = category;
}

var cheese = new Food('feta', 5, "food");
```


3.bind()
create a new function that has its 'this' keyword set to the provided value, with a given sequence of arguments.

```javascript
function.bind(thisArg, arg1, arg2...)
```
简单说来，就是把thisArg的this绑定到function上，返回一个bound function。


例1：bound function 绑定函数
```javascript
this.num = 9;
var module = {
  num: 100,
  getNum: function(){
    return this.num;
  }
}

module.getNum(); //100

var getVal = module.getNum;
getVal(); //9, because this function gets invoked at the global scope

//create a bound function,把this绑定到boundGetNum函数
var boundGetNum = getVal.bind(module);
boundGetNum(); //100
```

例2，和setTimeout()一起使用：
这个不明白
```javascript
function LaterBloomer(){
  this.petalCount = Math.floor(Math.random() * 12) + 1;
}

LaterBloomer.prototype.declare = function(){
  console.log('beautiful flower with' + this.petalCount + ' petals');
}

LaterBloomer.prototype.bloom = function(){
  //括号里的this指向类实例，即flower？
  window.setTimeout(this.declare.bind(this), 1000);
}

var flower = new LaterBloomer();
flower.bloom();
//after 1 second, triggers the 'declare' method
```


### arguments object
arguments is an array-like object inside function that contains the values that are passed to that function.(array-like means that arguments has a length property and indexed from zero, but doesn't have other built-in methods).  
Each argument can be reassigned.

convert arguments to array 方法：

```javascript
var arr = Array.prototype.slice.call(arguments);
var arr = [].slice.call(arguments);
var arr = Array.from(arguments);
var arr = [...arguments];
```

例：
```javascript
//concatenate several strings
function myConcat(separator){
  var arr = Array.prototype.slice.call(arguments, 1);
  return arr.join(separator);
}

myConcat(',', 'red', 'orange', 'blue');
//return "red, orange, blue"
```

### 常见的event

```javascript
var obj = document.getElementById("box");
//单击事件（按下，松开后触发）、双击事件
obj.onclick = function(){}
obj.ondblclick = function(){}
//鼠标按下事件、松开
obj.onmousedown = function(){}
obj.onmouseup = function(){}
//鼠标移入 移除
obj.onmouseover = function(){}
obj.onmouseout = function(){}
//鼠标按住后的移动
obj.onmousemove = function(){}
```

### 自定义属性  
不能获取元素的style(除非是直接写在该元素的tag中)，如果想要在改元素上进行某些事件的触发来改变style，可以借助自定义属性。

例如：点击图片，就toggle两种头像

```html
<script>
 .bt {
   width: 10px;
   height: 10px;
   background: url(img/icon0.png);
   float:left;
   margin: 5px;
 }
</script>
<div class="bt"></div>
<div class="bt"></div>
<div class="bt"></div>
```

```javascript
var btns = getElementByClassName("bt");
for(var i = 0; i < btns.length; i++){
  //下面这个isShow就是自定义属性
  btns[i].isShow = true;

  btns[i].onclick = function() {
    this.isShow = !this.isShow;
    //backgroundImage设置或返回元素的背景图像
    this.style.backgroundImage = this.isShow ? "url(img/icon0.png)" ? "url(img/icon1.png)";
  }
}
```