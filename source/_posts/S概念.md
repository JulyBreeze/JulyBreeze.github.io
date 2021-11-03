---
title: 1.JS 重点概念
date: 2019-03-19 09:02:34
tags: js
---
### Closure

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures

闭包就是能够读取其它函数内部变量的函数，本质上，闭包是将函数内部和外部连接的桥梁。
最大用处：一个是在外部可以读取函数内部的变量，另一个是让这些变量的值始终保持在内存中。

但是不能滥用，因为闭包会使函数的变量都保存在内存中。
例1：

```javascript
function f1(){
  var n = 9;
  //addOne前面没有var，表示这是一个全局变量
  //且是匿名函数
  addOne = function(){
    n += 1;
  }

  function f2(){
    alert(n);
  }

  return f2; //返回的是函数
}

var res = f1();
res(); //output 9
addOne();
res(); //output 10
```

解释：f2是闭包函数，运行了两次。  
f1的局部变量n一直保存在内存中，没有在f1被调用完成后被删除(garbage collection)。因为f1是f2的父函数，f2被赋给全局变量，所以f2一直在内存中，因此f1也在内存中，

例2：

```javascript
function makeAdder(x){
  return function(y){
    return x + y;
  }
}

//makeAdder相当于是一个函数工厂，用它创建了两个新函数。
var add5 = makeAdder(5); // x = 5
var add10 = makeAdder(10); // x = 10

console.log(add5(2)); //7
console.log(add10(2)); //12
```

add5和add10都是闭包，它们共享相同的函数定义，但是保存了不同的词法环境。  
在add5的环境中，x为5。而在add10中，x则为10。

**闭包是由函数以及创建该函数的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量**
A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function’s scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time. 这个闭包英文定义好像和我理解的不太一样，好像指的是一种block的感觉。

### OOP

js中OOP是基于构造函数constructor和原型链prototype chain的。  

#### prototype 

Every JavaScript object has a prototype object (or null, but this is rare) associated with it. The object inherits properties from the prototype. **每个JS object对应一个原型对象prototype，并从prototype继承属性和方法。**

可以这样理解：prototype就是让所有的对象实例可以共享其所包含的属性和方法

每个object具有私有属性__proto__，该属性指向该object对应的prototype。该prototype也有一个自己的原型对象，层层向上直到一个object的prototype为null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。

```javascript
var obj1 = {x : 1}
obj1.__proto__ === Object.prototype; // true
```

#### I.原型链prototype chain

原型链是实现继承的主要方法。  

Js object有一个指向一个prototype的链。当试图访问一个object的属性时，它不仅仅在该object上搜寻，还会搜寻该object的prototype，以及该object的prototype的prototype，依次层层向上搜索，直到找到一个名字匹配的属性或到达prototype chain的末尾。

几乎所有的JS object都是位于原型链顶端的Object的instance。

例子：

```javascript
//父构造函数
function ParentType () {
  this.property = true;
}

ParentType.prototype.getParentValue = function () {
  return this.property;
}
//子构造函数
function ChildType() {
  this.child_property = false;
}

//继承，子的prototype是父的实例
ChildType.prototype = new ParentType();

//继承后又添加了新方法
ChildType.prototype.getChildValue = function() {
  return this.child_property;
}

var instance = new ChildType();
alert(instance.getParentValue()); //true;
```

解释：定义了两个类型ParentType和ChildType，每个类型有一个属性和方法。  
继承是通过创建ParentType的实例，并将实例赋给ChildType.prototype实现的。**实现继承的本质是重写原型对象，代之以一个新类型的实例。** 即原来存在于ParentType的实例中的属性和方法，现在也存在于ChildType.prototype中。

{%asset_image fig.png%}

我目前的理解：  
可以把prototype看作是一个“class”，object继承这个“class”，所以每个object都有一个proto指向其prototype。  
在js中，继承的实现是通过把这个“class”重新赋值为父类的instance，这样就实现了继承。
“class”的method都是用这个“class”的.形式来添加的。

#### II.构造函数

##### 1.基本概念

规则：  
* constructor的name的第一个字母通常大写。
* 函数体内使用this关键字，**指向所要生成的object实例，而不是全局。**
* 生成object的时候，必须使用new命令来调用constructor。

```javascript
function Person(name, height){
  this.name = name;
  this.height = height;
}

var boy = new Person('John', 180);
```

上述代码依次执行下面的步骤:

```html
1) 建立了一个constructor function, Person
2) new创建了一个空object instance，名为boy
这个boy object的原型会指向Person的prototype，而不是Person本身！！！即：
  boy.prototype === Person.prototype is true
3) 然后这个boy object赋给constructor内部的this关键字。
即让constructor内部的this关键字指向这个object实例。
4)最后，开始执行构造函数内部代码
```

如果没有用new，此时this指向全局作用域：

```javascript
function Person() {
  this.height = 180;
}

var boy = Person();

console.log(boy.height);
//Cannot read property 'height' of undefined
//即boy is undefined

console.log(window.height); //180
```

为了保证constructor必须与new命令一起使用，可以在constructor内部使用strict模式。  
在严格模式中，函数内部的this不能指向全局对象，如果指向了全局，this默认等于undefined：

```javascript
function Person(name, height) {
  'use strict';
  this.name = name;
  this.height = height;
}

var boy = Person('John', 180);
//在声明boy时就出错了
//TypeError Cannot set property 'name' of undefined
```

##### 2.构建object的四种方法

1)语法结构创建object

```javascript
var obj = {a : 1};
//原型链： obj -> Object.prototype -> null

var arr = ['hello', 'world'];
//原型链： arr -> Array.prototype -> Object.prototype -> null

function f1(){
  return 1;
}
//原型链：f1 -> Function.prototype -> Object.prototype -> null
```

2)构造器new创建object

```javascript
function Graph(){
  this.vertices = [];
  this.edges = [];
}

Graph.prototype = {
  addVertex : function(v){
    this.vertices.push(v);
  }
}

var g = new Graph();
//g的自身属性有 vertices edges
//当g被实例化时，g.[[Prototype]]指向了Graph.prototype
```

3)Object.create()
创建新对象，并使用现有的对象来提供新创建的对象的__proto__：

```javascript
Object.create(proto, [propertiesObj])
```

例1：

```javascript
var a = {a: 1}
var b = Object.create(a);
//a就是b的原型对象
//b -> a -> Object.prototype -> null

var c = Object.create(null);
//c -> null, c没有继承Object.prototype
```

例2：

```javascript
//父类
function Shape(){
  this.x = 0; 
  this.y = 0;
}

Shape.prototype.move = function(x, y) {
  this.x += x;
  this.y += y;
  console.log("shape moved");
}

//子类
function Rectangle() {
  Shape.call(this); //call super constructor
}

//继承
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;

var rect = new Rectangle();
console.log(rect instanceof Rectange); //true;
console.log(rect instanceof Shape); //true

rect.move(1, 1); //"shape moved"
```

4)class构建(ES6引入的)  
新的关键字包括 class, constructor，static，extends 和 super  
注意class里面的function{}后不要加任何东西！

```javascript
class Polygon {
  constructor(height, width){
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon{
  constructor(sideLength) {
    //super is used to access and call functions on an object's parent.
    //super needs to be called first
    //相当于给height和width赋值为sideLength
    super(sideLength, sideLength);
  }
  get area() {
    return this.height * this.width;
  }
  set sideLength(newLength) {
    this.height = newLength;
    this.width = newLength;
  }
}
```

#### III.具体例子

```javascript
function Foo(){ };
var f1 = new Foo();

Foo.prototype.x = "hello";
f1.x;   // hello，因为只有函数创建的对象才能使用继承的prototype属性
Foo.x; // undefined，因为函数本身不能使用其prototype属性中的x
```

The prototype is only used for properties inherited by objects/instances created by that function.  
**The function itself does not use the associated prototype** (but since the function itself is an object, it inherits from the prototype of it's creator function, typically the javascript system Function object).
Foo本身是Object，它继承它的creator的prototype。

#### IV.instanceof operator

The instanceof operator tests whether the prototype property of a constructor appears anywhere in the prototype chain of an object.

```javascript
object instanceof constructor
```

例1：

```javascript
//constructor
function Car(model, year){
  this.model = model;
  this.year = year;
}

var auto = new Car('Accord', 1998);

//判断constructor Car()的prototype是否在object auto的chain上
auto instanceof Car;//true
auto instanceof Object;//true

//因为Car.prototype和Object.prototype都在auto的原型链上
```

例2：关于Function和Object的关系  
很难理解，但先记着吧

```javascript
Function instanceof Object; //true
Object instanceof Function; //true
```

解释：  
先有Object.prototype。Function.prototype继承Object.prototype而产生。  
Function、Object和其它constructor继承Function.prototype而产生。  
好像突然理解了！又不记的怎么理解的了？？


### let const var

在ES6之前，JS只有函数作用域和全局作用域。{}限定不了var声明变量的使用范围：

```javascript
{
  var i = 1;
}
console.log(i); //1
```

而且var是变量提升，即无论声明在何处，都会被提至其所在作用域的顶部。

#### let

1)let可以声明作用域为block scope，即{}的变量

```javascript
{
  let i = 1;
}
console.log(i); //Uncaught ReferenceError: i is not defined
```

所以let常配合for循环使用。  
因为JS中的**for循环体比较特殊，每次执行都是一个全新的独立的块{}作用域**，用let声明的变量传入到for循环体的作用域后，不会因此发生改变，不受外界的影响。例如：

当i是var时，这个在setTimeout()中有类似例子和解释：

```javascript
for(var i = 0; i < 10; i++){
  setTimeout(fuction() {
    console.log(i);
  }, 0);
}
//输出10个10
```

如果改成let：

```javascript
//i在for循环体局部作用域中使用的时候，变量会被固定，不受外界干扰
for(let i = 0; i < 10; i++){
  setTimeout(fuction() {
    console.log(i);
  }, 0);
}
//输出 0 1 2 3 ... 9
```

补充：  
上面例题中的setTimeout()中的time设为0，即便如此，it's placed on a queue and scheduled to run at the next opportunity, not immediately. Currently executing code must complete before functions on the queue are executed, the resulting execution order may not be as expected.

2)let没有变量提升和暂时性死区  
使用let命令声明变量之前，该变量都是不可用的，这称为暂时性死区temporal dead zone/TDZ。必须等let声明语句执行完之后，变量才能使用:

```javascript
console.log(x); //Uncaught ReferenceError ...
let x = 1;
// 这里就可以安全使用x
```

#### const

const也是block scope，暂时性死区，没有变量提升，不可以重复声明。  
* 对于简单类型数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。
* 对于复合类型数据（主要是对象和数组），const存放的是地址，这个地址相当于一个指针，指向一个对象。地址是固定的，但可以修改这个对象的属性。


```javascript
const person = {
  name: 'Jonh',
  age: 30
}

person.age = 31;
console.log(person); // {name: 'Jonh', age: 31}

person = {
  name: 'Max'
} // Uncaught TypeError: Assignment to constant variable.
```
