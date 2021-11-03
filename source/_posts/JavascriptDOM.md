---
title: 5.Javascript DOM & BOM
date: 2019-03-18 12:22:11
tags: js
---

Document Object Model is a platform and language-neutral interface that allows programs and scripts to dynamically access and update the content structure, and style of a document.  

html is root.

### node

```
Node:
  nodeName, nodeType, nodeValue

Node type:
  element node: 对应的编号是1
  attribute node: 2
  text node: 3

Node value:
  element node: null
  attribute node: 属性值
  text node: 文本内容
```

例子：

```html
<div id="box">
   Hello world
</div>
```

```
Node type:
  element node: <div> 
  attribute node: id="box"  
  text node: Hello world 
```

```javascript
1.元素节点
var obj = document.getElementById('box');
console.log(obj.nodeName); // DIV，注意nodeName的输出是大写的
console.log(obj.nodeType); // 1
console.log(obj.nodeValue); //元素节点的value都是输出null

2.属性节点
//DOM中的attributes属性返回指定节点的属性集合
var att = obj.attributes[0];
console.log(att.nodeName); // id
console.log(att.nodeType); // 2
console.log(att.nodeValue); // box

3.文本节点
console.log(node.nodeName); // 文本节点的nodeName都是输出#text
```

### node的获取
例1：

```html
<div id="box">
  <a href="#">link</a>
</div>
```

获取全部子节点：

```javascript
var obj = document.getElementById("box");
var nodes = obj.childNodes;

/*IE has 1 childnodes, non-IE has 3 childnodes
div后面的空白处算一个text node，所以有text node、a 、text node三个子节点
但如果上面html语句去除回车写成一行, nonIE: the length = 1
*/
console.log(nodes.length);
```

```html
<span>hello</span>
<!-- 
  如果想得到span元素的内容
  用node.innerHTML
  而不是用node.nodeValue,这个对于元素节点永远输出null
 -->
```

例2：

```html
<div id="box">
  <p>hello</p>
  <a href="#">link</a>
</div>
```

```javascript
var obj = document.getElementById("box");

通过obj.firstChild，obj.lastChild可以得到其第一个和最后一个的子节点
通过obj.parentNode可以获得父节点
通过obj.previousSibling、nextSibling获得同级别的前一个、下一个节点
```

### attributes node

例1：

```html
<div class="section" id="box"></div>
```

```javascript
var obj = document.getElementById("box");
console.log(obj.attributes.length); // 2

//通过[]的方式获取某个attribute节点
alert(obj.attributes['id'].nodeName); // id
alert(obj.attributes['id'].nodeType); // 2
alert(obj.attributes['id'].nodeValue); // box
```

### node的操作

1)创建元素

```html
<head>
  <script type="text/javascript">
    window.onload = function(){
      //创建一个元素节点
      var title = document.createElement('h1');
      title.innerHTML = "hello world";
      //添加元素到页面中
      document.body.appendChild(title);

      var link = document.createElement('a');
      link.href = "#";
      link.innerHTML = "google";
      //把这个元素添加到title元素里，成为了其子元素
      title.appendChild(link);

      //创建文本节点
      var text = document.createTextNode('<h1>Natsume</h1>');
      document.body.appendChild(text);
      //页面上显示的是<h1>Natsume</h1>,而不是Natsume
    }
  </script>
</head>
<body>
</body>
```

2)插入元素

```html
<head>
  <script type="text/javascript">
    window.onload = function(){
      var obj = document.createElement('li');
      obj.innerHTML = '1.xxx';

      var oView = document.getElementById('view');
      oView.insertBefore(obj, oView.firstChild);
      oView.appendChild(obj);
    }
  </script>
</head>
<body>
  <ul id="view">
    <li>2.xxx</li>
  </ul>
</body>
```

页面效果：
```
1.xxx
2.xxx
1.xxx
```

3)应用例子

```html
<head>
  <style type="text/css">
    #box {
      width: 100px;
      height: 500px;
      border: 1px solid black;
      overflow: auto; /*滚动*/
    }
    .section {
      ...
    }
  </style>
</head>
<body>
  <div id="box">
    <!-- <div class="section">
      <a href="#">
        <img src="img/img_0.png">
        icon1
      </a>
      <p>description</p>
    </div> -->
  </div>
</body>
```

```javascript
window.onload = function(){
  var oBox = document.getElementById('box');

  var oDiv = null; //<div>
  var oA = null; //<a>
  var oImg = null; //<img>
  var oText = null;// icon1
  var oP = null;//<p>
  //广告，想要插入到某处，停留5s后消失
  var oAd = null;

  for(var i = 0; i < 10; i++){
    if(i == 3){
      //oAd的设置
    }
    oDiv = document.createElement('div');
    oDiv.className = "section";
    oBox.appendChild(oDiv);

    oA = document.createElement('a');
    oA.href = '#';

    oImg = document.createElement('img');
    oImg.src = "img/img_" + i + ".png";
    oA.appendChild(oImg);

    oText = document.createTextNode("icon" + i);
    oA.appendChild(oText);
    
    oP = document.createElement('p');
    oP.innerHTML = "description";
    oDiv.appendChild(oP);
  }

  setTimeout(function(){
    oBox.removeChild(oAd);
  }, 5000);
}
```

### node属性

1)放在tag里的自定义属性  
格式：data-自定义属性的name

```html
<div class="section" data-index='0' data-isShow='true'></div>
```

2)获取attributes
```javascript
var obj = document.getElementById('box');

var atts = obj.attributes;
atts['id'].nodeValue;

obj.getAttribute('data-index');
```

3)设置node attributes
```javascript
obj.type = 'button';
obj.style.color = 'red';
//@param: 'attribute name' ,'value'
obj.setAttribute('style', 'color: red');
obj.setAttribute('class', 'section');
```

4)删除node attributes
```javascript
obj.removeAttribute('class');
```

#### 应用例子

自动生成表格，并可以删除某行

```html
<style>
  table{
    ...
  }
  table td{
    border: 1px solid gray;
  }
</style>
<body>
  输入行数：<input type="text" id="row" placeholder="row" /><br/>
  输入列数：<input type="text" id="col" placeholder="column" /><br/>
  <input type="button" id="btn" value="生成表格"/>
  <!-- <table>
    <tr>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td></td>
    </tr>
  </table> -->
<body>
```

```javascript
window.onload = function(){
  var oRow = $('row');
  var oCol = $('col');
  var oBtn = $('btn');

  oBtn.onclick = function(){
    //获取<input>输入框里输入的值
    var _row = oRow.value;
    var _col = oCol.value;

    //add table
    var _table = document.createElement('table');
    document.body.appendChild(_table);
    //add row
    for(var i = 0; i < _row; i++){
      var _tr = document.createElement('tr');
      _table.appendChild(_tr);

      //add column
      for(var j = 0; j < _col; j++){
        var _td = document.createElement('td');
        //显示几行几列
        _td.innerHTML = (i + 1) + '-' + (j + 1);
        _tr.appendChild(_td);
      }

      var _deleteTd = document.createElement('td');
      _tr.appendChild(_deleteTd);

      //add the delete row operation
      var _deleteBtn = document.createElement('input');
      _deleteBtn.type = 'button';
      _deleteBtn.value = '删除该行';
      _deleteTd.appendChild(_deleteBtn);

      _deleteBtn.onclick = function(){
        //this refers to deleteBtn
        // this.parentNode is _deleteTd
        // this.parentNode.parentNode is the corresponding row
        _table.removeChild(this.parentNode.parentNode);
      }
    }
  }
}
```

### BOM
Browser Object Model  
BOM提供了与浏览器窗口进行交互的对象。由于BOM主要用于管理窗口与窗口之间的通讯，因此其核心对象是window。  BOM由一系列相关的对象构成，并且为每个对象都提供了很多方法与属性:
* window (好像后面几个都是window对象的属性)
* location
* navigator，提供了与browser有关的信息，取决于浏览器类型
* screen
* history

BOM缺乏标准；JavaScript语法的标准化组织是ECMA，DOM的标准化组织是W3C。

在使用框架时，每个框架都有自己的window对象，top对象指向最外围的框架，即整个浏览器窗口。parent对象表示包含当前框架的框架，self对象回指window。


例子：

```html
<input type="button" value="open" id="btn1"/>
<input type="button" value="close" id="btn2"/>
```

```javascript
openedWindow = null;
$('btn1').onclick = function(){
  // window.document

  //第二个参数是打开方式是以新标签打开页面
  //window.open('','_blank'); //打开空白页
  openedWindow = window.open('https://www.google.com','_blank');
}

$('btn2').onclick = function(){
  openedWindow.close();
  //关闭本窗口
  window.close();
}
```


#### window对象

```
window.navigator.userAgent可以看到浏览器的信息
window.location 当前窗口的完整url
```

#### location对象

locatioin是window对象和document对象的属性，保存着当前文档的信息，并将URL解析为独立的片段

```html
属性名        例子 
hash         #xxx
hostname     www.abc.com
host         www.abc.com:80
href         http://www.abc.com
pathname     /xxx/
port         8080
search       ?xxx=xxxxx 
```


改变浏览器位置，这样点击后退按钮可以导航到前一个页面：
```javascript
location.href = "xxx"
```

如果用这个，则不会在历史纪录里产生新记录，不能回到前一个页面：
```javascript
location.replace("xxx)
```



#### history对象
保存着上网的历史记录。是window对象的属性。

React中的history对象的属性
{% asset_img reactHISTORY.png React-History-Object %}


### 总结

DOM里的常见属性和方法:

```javascript
//obj是通过getElementByXX获取的节点
atts = obj.attributes
nodes = obj.childNodes

//上面可以用length来获取数量
atts.length
nodes.length

//通过[]获取某个attribute节点,可以是具体的attribute name，或者index
obj.attributes['id'].nodeName;
obj.attributes['id'].nodeType;
obj.attributes['id'].nodeValue;
obj.attributes[0];
obj.getAttribute('xxxx'); //获取属性的value

obj.setAttribute('xxxname', 'xxxvalue');
obj.removeAttribute('xxxname');

obj.firstChild
obj.lastChild
obj.parentChild
obj.previousSibling
obj.nextSibling
```

```javascript
document.createElement('h1');
document.body.appendChild(obj);
document.createTextNode('xxxx');

obj1.appendChild(obj2);
obj1.removeChild(obj2)
//把obj2插入到obj1的第一个子节点之前
obj1.insertBefore(obj2, obj1.firstChild);
```