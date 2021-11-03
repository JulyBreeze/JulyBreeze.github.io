---
title: 2.JavaScript 数据结构
date: 2019-03-14 15:50:46
tags: js
---
### js在html中的位置

在html文档中，script-tag一般放在body里的最后。  
也可以放在head tag里，但为了能在所有文档加载完成后运行js，此时应该这样写：

```html
<head>
  <script type="text/javascript">
    //只会执行一次
    window.onload = function(){
      //js code
    }
  </script>
</head>
```

### Array

#### 1.basic

**同一个数组里的元素可以是不同的类型，数组是object类型**
1.create Array

```javascript
var arr1 = [];   //数组为空
var arr2 = [1,2,3,4,5];
var arr3 = [,,,]; //3个元素[IE浏览器、Chrome]或4个元素(不同的浏览器识别不同)
//new可以省略
var arr4 = new Array(); //数组为空
var arr5 = new Array(5); //创建数组，有5个空元素
var arr6 = new Array(1,2,3,4,5); //创建的数组是[1,2,3,4,5]
```

多维数组

```javascript
var a = [1,2,3];
var b = [4,5,6];
var matrix = [a, b];//first row is a, second row is b
console.log(matrix[0][1]) //output is 2

var matrix2 = [[1,2,3], [4,5,6]];
```

#### 2.常见methods  

```javascript
arr.length
value = arr[index]
index = arr.indexOf(value)

/*数组遍历*/
for(var i = 0; i < arr.length; i++){
  arr[i];
}

for(index in arr){
  arr[index]
}

arr.forEach((ele) => {});

//对首端元素进行操作
arr.shift() //delete the first element, return the deleted element.
arr.unshift(elements) //add element/elements to the front, return new length.

//对尾部元素进行操作
arr.pop() //delete the last element and return this element.
arr.push(elements) //add element/elements to the end, return new length.

arr.reverse() //change the original array
arr.sort() //sort
```

#### 3.其他methods  

**1) splice()** (意思：拼接结合)

```javascript
// 1.删除 remove n elements starting from index, return deleted elements
//如果不写，表示从index开始往后所有的元素都删除
arr.splice(index, [n]) 

// 2.替换 replace n elements starting from index with one newValue
// return the original replaced elements
arr.splice(index, n, newValue)

// 3.添加 add one or many newValues at index
arr.splice(index, 0, newValue)
arr.splice(index, 0, newValue1, newValue2,...)
```

实际使用:数组去重

法1) 对于每一元素，和其后面所有的元素一一比较，重复的就删除。

```javascript
for(var i = 0; i < arr.length; i++){
  for(var j = i + 1; j < arr.length; j++){
    if(arr[i] === arr[j]){
      arr.splice(j, 1);
      //因为删除一个元素后，length会变化
      //所以后面的元素会占据被删除元素的位置，因为有j++,所以这里要--
      j--;
    }
  }
}
document.write(arr);
```

其他方法：ES6里引入了Set和Map  
先排序，然后不和前面一个元素相等的元素就保留下来

```javascript
function uniques(arr){
  return Array.from(new Set(arr));
}

//this function doesn't work for objects, because objects are equal for sort
//filter()符合条件的留下
function uniques(arr){
  return arr.sort().filter(function(item, index, arr){
    //!index表示index = 0的元素
    return !index || item != arr[index - 1]
  });
}
```

最简单的写法：

```javascript
uniques = [...new Set(array)];
```

**2) join()**

```javascript
//join each element with spliter, the type of res is string
arr.join(spliter);

arr.join(); //default separate element with comma ','
arr.join(''); //arr=[1,2,3,4,5], res = "12345"
```

**3) sort()**

```javascript
//sort in-place and return the array
//默认情况下以string的形式比较
arr.sort();

//if want to sort numbers:
arr.sort(function(a, b) {
  return a - b;
}); //ascending order

//eg 虽然这个直接用sort和用下面的方法得到的结果是一样的
var arr = ['50px', '20px', '60px'];

arr.sort(function(a, b){
  return parseInt(a) - parseInt(b);
});
```

应用：shuffle array  
思路：随机生成一个[0, i]之间的值，称为j，然后交换这个j位置上的数和nums[i]，需要倒叙来

```javascript
//这个并不是真正意义上的shuffle，因为好像是不能平均概率
arr.sort(function() {
  return Math.random() - 0.5;
});

function shuffle(arr){
  for(int i = arr.length - 1; i > 0; i--)
    //得到一个[0, i]之间的数
    const j = Math.floor(Math.random() * (i + 1));
    const temp = array[i];
    arr[i] = arr[j];
    arr[j] = temp;
  }
  return arr;
}

//上面swap部分可以用下面的destruct来代替，但这个好像会比较慢一些
[arr[i], arr[j]] = [arr[j], arr[i]];
```

**4) slice()**(切下，薄片)  
是shallow copy

```javascript
//copy [n1, n2) and return a new array
arr2 = arr1.slice(n1, [n2]);//n2可以省略
```

深拷贝：见后面

**5) filter()**  
creates a new array filled with all elements that pass the function.  
把符合条件的留下！

```javascript
var newArray = arr.filter(callback(element[, index[, array]])[, thisArg])
//举例，把array中的每个元素长度大于6的保存下来形成新的数组
const result = words.filter(word => word.length > 6)
```

**6) map()**  
create a new array with the results of calling a provided function on each element in the calling array.

```javascript
var arr = [1, 4, 9, 16];
const map1 = arr.map(x => x * 2);

console.log(map1);
//[2, 8, 18, 32]
```

#### 4.Array Scope

```html
<!-- 实现点击button，改变字体颜色为白色，背景颜色为蓝色的操作 -->
<input type="button" value="button1" />
<input type="button" value="button2" />
<input type="button" value="button3" />

<script>
  var btns = document.getElementsByTagName("input");
  btns[0].onclick = function() {
    btns[0].style.backgroundColor = 'blue';
    btns[0].style.color = 'white';
  }
  btns[1].onclick = function() {
    btns[1].style.backgroundColor = 'blue';
    btns[1].style.color = 'white';
  }
  btns[2].onclick = function() {
    btns[2].style.backgroundColor = 'blue';
    btns[2].style.color = 'white';
  }

  //这样太麻烦，用数组
  //这样写没有反应，为什么？
  for(var i = 0; i < btns.length; i++){
    btn[i].onclick = function() {
      btns[i].style.backgroundColor = 'blue';
      btns[i].style.color = 'white';
      alert(i); //不管点击哪个button，弹出都是3
    }
  }
</script>
```

原因：

```
  预解析： var i
  逐行解析：
    btns[0].onclick = {...}
    i++; //i = 1
    btns[1].onclick = {...}
    i++; //i = 2
    btns[2].onclick = {...}
    i++; //i = 3
    退出循环

先for循环，i变成了3，再点击button的操作才去访问i的值，此时i = 3
```

解决方法：**用this**

```javascript
for(var i = 0; i < btns.length; i++){
    btn[i].onclick = function() {
      this.style.backgroundColor = 'blue';
      this.style.color = 'white';
    }
}
```

#### 5.function的arguments属性

**函数中有一个属性arguments**，是一个存放传入的参数的数组

```javascript
function sum(){
  var res = 0;
  for(index in arguments){
    res += arguments[i]；
  }
  returrn res;
}

sum(1,2,3) //output 6
sum(1,2,3,4) //output 10
```

#### 6.数组排序

1) 直接插入排序  
思路：  
从index = 1开始，当前元素假设为A，把A和前面的一堆数依次比较，是从后往前比较。  
如果在前面的数中，数比A小，那么就把那个数后移一位。如果遇到比A大的数，假设为B，就break出这个比较的循环。  
把A赋值给B后面的一位上。

```javascript
var arr = [5,23,3,1,8,6,2,9,10];
var i, j;
for(i = 1; i < arr.length; i++){
  var temp = arr[i];

  for(j = i - 1; j >= 0; j--){
    //compare arr[i] with already sorted subarry in backward direction
    if(temp > arr[j])break;
    else arr[j + 1] = arr[j];
  }

  arr[j + 1] = temp;
}
```

2) 冒泡排序

```javascript
for(var i = 0; i < arr.length - 1; i++){
  for(var j = 0; j < arrlength - 1 - i; j++){
    if(arr[j] < arr[j + 1]){
      swap;
    }
  }
}
```

#### 7.深拷贝

法1) 递归  
递归base case是到了非object的层

```javascript
function deepClone(obj){
  let objClone = Array.isArray(obj) ? [] : {};

  if(obj && typeof obj === "object") {
    for(key in obj) {
      if(obj.hasOwnProperty(key)){
        //如果obj的子元素是object，就递归复制
        if(obj[key] && typeof obj[key] === "object") {
          objClone[key] = deepClone(obj[key]);
        }else{
          objClone[key] = obj[key];
        }
      }
    }
  }
  return objClone;
}

//补充method介绍
/*
hasOwnProperty() can be called on any object and takes a string as an input. 
It returns a boolean which is true if the property is located on the object
*/
```

法2) JSON.stringfy()和JSON.parse()  
  
* JSON.parse() parses a json string

```javascript
//if reviver is a function
//it prescribes how the value originally produced by parsing is transformed
JSON.parse(string, [reviver]);

//Example
const json = '{"name": "John", "age": 20}';
obj = JSON.parse(json);

console.log(obj.age); //output is 20
console.log(obj.name); //output is John
```

```javascript
function deepClone(obj){
  let _obj = JSON.stringify(obj);
  let objClone = JSON.parse(_obj);

  return objClone;
}
```

* JSON.stringify() converts a JavaScript object or value to a JSON string

```javascript
console.log(JSON.stringify([new Number(3), new String('hello'), new Boolean(true)]));
// expected output: "[3,"hello",true]"
```

### String

#### 1.basic

```javascript
var str1 = "hello world";
var str2 = new String("hello world");
```

#### 2.常见methods

```javascript
str.length

// 访问char
str[index]
str.charAt(index)

//search
str.indexOf(searchVal, [fromIndex])
没找到return -1;
```

#### 3.其他methods

1)substring()

```javascript
//[start, end)
str.substring(startIndex, endIndex)
//example:
str = '0123456';
str.substring(3, -4)//output is "012"
//-4 is converted to 0, so find [0,3)
```

应用例子：展开和收起

```html
<div id='#box'>
  <p>"Learning from examples is the best way to improve yourself !"</p>
  <a href="#">收起</a>
</div>
```

```javascript
  var content = document.getElementsByTagName('p')[0];
  var obj = document.getElementsByTagName('a')[0];
  var totalStr = content.innerHTML;

  obj.isShow = true;
  obj.onclick = function(){
    this.isShow = !this.isShow;
    if(this.isShow){
      content.innerHTML = totalStr;
    }else{
      content.innerHTML = content.innerHTML.substring(0, 10) + '...';
      obj.innerHTML = "展开”
    }
  }
```

2)split()

```javascript
//return an array, the length of array = limit
str.split([separator], [limit]);
str.split() // return the str itself in a array.
str.split('') // return an array with element as each char of str
```

应用例子：颠倒string

```javascript
str = "123456";
str = str.split('').reverse().join('');
//output is 654321
```

### Set 

```javascript
new Set([iterable])
```

常见methods：

```javascript
set.add(value);
set.has(value);

set.clear();
set.delete(val);

set.keys();
set.values(); //和keys()是一样的
set.entries(); //in insertion order，[val, val]
//properties
set.size;
```

和string的关系，把string的每个char变成set里的元素:

```javascript
var str = "hello";
var set = new Set(str); //['h','e','l','o']
set.size; // 4
```

### Map

1)建立

```javascript
new Map([iterable])
//可以是array或者其他可以iterable object whose elements are key-value pair.
//例如：array格式如下：
new Map([[ 1, 'one' ],[ 2, 'two' ]]))
```

2)常用methods：

```javascript
map.set(key, value);
map.size;//property,not methods

map.values();
map.keys();
map.entries();

map.has(key);
map.get(key);
map.delete(key);

map.clear(); //remove all the pairs
map.forEach(callback function)
```

例子:

```javascript
var map = new Map();
var keyStr = "string", keyObj = {}, keyFunc = function(){};

map.set(keyStr, "key is string");
map.set(keyObj, "key is obj");
map.set(keyFunc, "key is function");

map.get(keyStr); //"key is string"
map.size; //3

map.get("string"); //"key is string", because 'string' === keyStr
map.get({}); //undefined, because keyObj !== {}
map.get(function(){}); //undefined, because keyFunc !== function(){}
```

3)如何iterate map

```javascript
//of
for(var[key, val] of map){
  ...
}

for(var[key, val] of map.entries()){
  ...
}

for(var key of map.keys()){
  ...
}

//注意val在前，key在后
map.forEach(function(val, key) {...})；
```

4)与array的关系

```javascript
var arr = [['key1', 'value1'], ['key2', 'value2']];
var map = new Map(arr);

map.get('key1'); //return 'value1'

//把map转换成array
Array.from(map);
[...map];
```

5)clone map
深拷贝？

```javascript
var original = new Map([[1, 'one']]);
var clone = new Map(original);

clone.get(1); //'one'
original === clone; //false
```

6)merge maps

```javascript
//Merge two maps. The last repeated key wins.
var first = new Map(...)
var second = new Map(...)

var merged = new Map([...first, ...second]);
```

### Object

window object是js中的顶级对象，所有定义在global scope中的变量、函数都会变成window对象的属性和方法，在调用的时候可以省略window。

#### 基本包装boxed类型

3个特殊的引用类型：Boolean，Number，String
尽管它们是primitive type，本不应该有methods，所以称其为基本包装类型。  
但是有一些methods，例：

```javascript
var s2 = s1.substring(2);
```

引用类型和基本包装类型的区别是对象的生存期：  
使用new创建的reference type的实例在执行流离开当前作用域后被回收；但基本包装类型的对象，只存在于代码执行的时刻，然后就会被销毁，所以不能为基本类型添加属性和方法。

new调用基本包装类型的构造函数，和直接调用同名的转型函数是不一样的。
例：

```javascript
//number保存的是基本类型值25
var value = "25";
var num = Number(value); //转型函数
alert(typeof num) //"number"

//obj保存的是Number的实例
var obj = new Number(value); //构造函数
alert(typeof obj) //"object"
```

#### 对象object

1.创建

创建Object的实例，然后添加属性property和方法
```javascript
var person = new Object();
person.name = "John";
person.sayName = function() {
  alert(this.name);
}
```

或者可以直接写成：
```javascript
var person = {
  name: "John",
  sayName: function() {
    alert(this.name);
  }
}
```


2.Property的两种类型(这个大概知道就行)  
object property具有特性attributes

1)数据属性  
* [[Configurable]] 
* [[Enumerable]] 
* [[Writable]] 能否修改
* [[Value]]


2)访问器属性  
get and set函数

3)修改attribute  
可以通过这个来修改属性默认的特性，比如修改成只读等

```javascript
Object.defineProperty(property所在的object, property的名字，descriptor)
```
