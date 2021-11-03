---
title: 1.JavaScrit Scope
date: 2019-03-14 14:55:45
tags: js
---

### Scope (Static Scoping)

* Global Scope  
contains variable defined out of any code block
* Local Scope  
local scope created by a code block{}  
The parent scope cannot access variables in the child scope.  
Child scope can access values defined in itself or any parent/ancestor scope
* Shadowing  
Occurs when a local variable shadows over a global variabe and take precedent.  
内部scope里的一个变量和global变量同名，那么在这个scope里优先使用local变量  
* Leaked Global  
When shadowing, js will go all the way up to the global scope to find a variable's value.  
If js does't find it in the global scope, js will create it as a global variable, which means a variable in the local scope leaks into global scope.

总结：  
父对象的所有变量对子对象都是可见，反之则不成立。  
(chain scope: 子对象会一级一级地向上寻找所有父对象的变量) 
</br>

例题：

```javascript
let getScore = function () {
  if (1 < 2) {
    score = 3;
  }
  console.log(score);
}
getScore();
console.log(score); // Prints 3
```

### 解析

在一个作用域里，js解析分两步:

* 预解析  
  在代码真正运行前，所有变量和函数，此时都是undefined;  
  遇到重名，只留下一个；变量和函数重名，只留下函数。
* 逐行解析(开始解析每句code)  

函数内部也是一个作用域，所以也会进行预解析和逐行解析，且函数中局部优先(Shadowing)。

### Examples

1.

```javascript
var a = 1;
function fn(){
  alert(a); //output 1
  a = 2;
}

fn();
alert(a); //output 2
```

```
预解析： var a; fn = function(){...}
逐行解析： 
  a = 1
  fn()调用
    预解析： none
    逐行解析：alert(a) //1
             a = 2
  alert(a) //2
```

2.
```javascript
var a = 1;
function fn(){
  alert(a); //output undefined
  var a = 2;
}

fn();
alert(a); //output 1
```

```
预解析： var a; fn = function(){...}
逐行解析： 
  a = 1
  fn()调用
    预解析： var _a
    逐行解析：alert(_a) //undefined
             _a = 2
  alert(a) //1
```

3.

```javascript
var a = 1;
function fn(a){
  alert(a); //output undefined
  a = 2;
}

fn();
alert(a); //output 1
```

```
预解析： var a; fn = function(){...}
逐行解析： 
  a = 1
  fn()调用
    预解析： var _a (函数传入的参数是local variable)
    逐行解析：alert(_a) //undefined
             _a = 2
  alert(a) //1
```

4.函数名和变量名相同时

```javascript
1. alert(a); 
2. var a = 1;
3. function a() {
4.   alert("1.function a");
5. }
6. alert(a); 
7. var a = 3;
8. alert(a); 
9. function a(){
10.  alert("2.function a");
11.}
12. alert(a); 
```

```
预解析：
  第2行：var a
  第3行：a = function(){alert("1.function a")}
  第7行：var a
  第9行：a = function(){alert("2.function a")}

  //变量和函数同名时，只留函数
逐行解析：
  第1行：alert(a) //output 整个function(){alert("2.function a")}
  第2行：a = 1
  第6行：alert(a) //output is 1
  第7行：a = 3
  第8行：alert(a) // output is 3
  第12行：alert(a) //output is 3
```