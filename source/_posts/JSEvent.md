---
title: 6.Js Event
date: 2019-04-13 12:37:21
tags: js
---

### 基本概念

Js与HTML交互是通过event实现的。

### 事件对象
当触发DOM上的某个event时，会产生一个event object. Event object记录事件发生的相关信息，包括导致事件的元素，事件的类型等。  
事件对象只有在事件发生时才会产生，无法手动创建。事件对象只能在处理函数内部访问，处理函数结束后该对象会自动销毁。

```javascript
var obj = document.getElementById('box');
obj.onclick = function(ev) {
  //不同的浏览器兼容，ev是事件对象
  ev = window.event | ev;

  /*
  event的属性
  IE：srcElement
  google,safari: srcElement/target
  */
  //事件源
  var self = ev.target || ev.srcElement;
  //self.id 事件源的名称
}
```

一些event属性：

```javascript
//鼠标相对于浏览器页面位置
ev.clicentX
ev.clientY

//鼠标相对于事件源的位置
ev.offsetX
ev.offsetY
```

#### 例子
鼠标保持按下状态，元素跟随鼠标移动：  
可以通过onmousemove事件获取鼠标相对于浏览器的坐标，然后把其赋值给元素的style中的left和top属性。

```html
<style>
  #box {
    width:100px;
    height:100px;
    position: absolute;
    background: green;
    left:0;
    top:0;
  }
</style>
<div id="box"></div>
```

```javascript
window.onload = function(){
  var obj = document.getElementById('box');

  obj.onmousedown = function(){
    document.onmousemove = function(ev){
      ev = window.event || ev;

      obj.style.left = ev.clientX + 'px';
      obj.style.top = ev.clientY + 'px';
    }
    document.onmouseup = function(){
      document.onmousemove = null;
    }
  }
}
```

但是上面的写法，拖动时不管鼠标点击的是方框的哪里，最后鼠标都是在方框的左上角位置。所以需要保存鼠标相对于元素的位置：
```javascript
window.onload = function(){
  var obj = document.getElementById('box');

  obj.onmousedown = function(ev){
    ev = window.event || ev;

    var downX = ev.offsetX;
    var downY = ev.offsetY;

    document.onmousemove = function(ev){
      ev = window.event || ev;
      //元素距离browser的位置
      obj.style.left = ev.clientX - downX + 'px';
      obj.style.top = ev.clientY - downY + 'px';
    }

    document.onmouseup = function(){
      document.onmousemove = document.onmouseup = null;
    }
  }
}
```


### 事件冒泡dubbed bubbling
事件会从最内层的元素开始发生，一直向上传播，直到document对象。

举例:  
```html
<div id="view1">
  <div id="view2">
    <div id="view3">
    </div>
  </div>
</div>
```

```javascript
window.onload = function(){
  var view1 = document.getElementById('view1');
  var view2 = document.getElementById('view2');
  var view3 = document.getElementById('view3');
  
  //点击view3时，触发顺序 view3 -> view2 -> view1 -> document
  view3.onclick = function(ev){
    //如果想阻止冒泡
    //兼容写法
    ev.stopPropagation ? ev.stopPropgation() : ev.cancelBubble = true;
    alert('view3');
  }

  view2.onclick = function() {alert('view2');}
  view1.onclick = function() {alert('view1');}
  document.onclick = function() {alert('document');}
}
```

实际应用：  
1.输入框点击时会出现下拉表

```html
<div id="search">
  <input type="text" id="text" placeholer="" />
  <ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
  </ul>
</div>
```

```javascript
window.onload = function(){
  var input = document.getElementById('text');
  var ul = document.getElementsByTagName('ul')[0];
  var lis = document.getElementsByTagName('li');

  //method1
  input.onfocus = function(){
    ul.style.display = 'block';
  }

  input.onblur = function(){
    //设置setTimeout，因为当鼠标离开输入框要去点击下拉菜单的条目时
    //相当于失去焦点，所以需要设定一个事件后执行
    setTimeout(function(){
      ul.style.display = 'none';
    }, 50);
    
  }

  //给下拉菜单里每个条目都添加点击事件，这样很麻烦
  /*
  for(var i = 0; i < lis.length; i++){
    ...
  }
  */

  //method2：利用冒泡在父元素只绑定一次点击
  //因为点击任何li都会冒泡到ul，所以直接写ul.onclick
  ul.onclick = function(ev){
    //想知道是哪个li条目被点击，通过事件源获取
    ev = window.event || ev;
    var self = ev.target || ev.srcElement;
    input.value = self.innerHTML;
  }
}
```

解释：当子节点li被点击，click事件会从子节点向上冒泡，父节点捕获到事件之后，通过ev.target知道了被点击的li节点，从而进行处理。  
这个例子也是**事件代理Event Delegation**：  
当要给很多元素添加事件的时候，可以把事件添加到父节点，即将事件委托给父节点来触发处理函数。

### 事件捕获Event capturing

从document到触发事件的节点，即自上而下的去触发事件

```javascript
//true: event capture; false: event bubble
addEventListener('event', function, true/false)
removeEventListener('event');

//eg, 对于点击事件，当点击时，使用事件捕获触发function
obj.addEventListener('click', function(){...}, true);
obj.removeEventListener('click');
```

#### 例子

1.例1

```html
<div id="view1">
  <div id="view2">
    <div id="view3">
      <div id="view4">
      </div>
    </div>
  </div>
</div>
```

当点击view4时，依次显示"view1 - capture, view2, view3, view4"，然后依次显示"view4 - bubble, view3, view2, view1"
```javascript
window.onload = function(){
  var oview1 = document.getElementById('view1');
  var oview2 = document.getElementById('view2');
  var oview3 = document.getElementById('view3');
  var oview4 = document.getElementById('view4');

  //event capture
  oview1.addEventListener('click', function(){
    alert("view1 - capture");
  }, true);
  oview2.addEventListener('click', function(){
    alert("view2 - capture");
  }, true);
  oview3.addEventListener('click', function(){
    alert("view3 - capture");
  }, true);
  oview4.addEventListener('click', function(){
    alert("view4 - capture");
  }, true);

  //bubble
  oview1.onclick = function() {
    alert("view1 - bubble");
  });
  oview2.onclick = function() {
    alert("view2 - bubble");
  });
  oview3.onclick = function() {
    alert("view3 - bubble");
  });
  oview4.onclick = function() {
    alert("view4 - bubble");
  });

  //remove
  oview4.removeEventListener('click');
  oview3.onclick = null;
}
```

例2：键盘事件  
回车切换输出框

```html
<head>
  <script>
    window.onload = function(){
      var oUser = document.getElementById('user'); 
      var oPs = document.getElementById('ps'); 
      var oBt = document.getElementById('bt'); 

      oUser.onkeydown = function(ev){
        ev = ev || window.event;
        //enter的keyCode = 13
        if(ev.keyCode == 13){
          oPs.focus();
        }
      }

      oPs.onkeydown = function(ev){
        ev = ev || window.event;
        if(ev.keyCode == 13){
          login();
        }
      }

      function login(){
        ...
      }
    }
  </script>
</head>
<body>
  <input type="text" placeholder = "user" id="user" /></br>
  <input type="password" placeholder = "password" id="ps" /></br>
  <input type="button" value="log in" id="bt" />
</body>
```


### 事件默认行为

例子：点击鼠标右键，出现自定义的menu下拉菜单
```html
<head>
  <style>
    #menu{
      width: 50px;
      height: 80px;
      border: 1px solid gray;
      position: absolute;
      display: none;
    }
  </style>
  <script>
    window.onload = function(){
      var oMenu = document.getElementById("menu");

      //oncontextmenu右键触发
      document.oncontextmenu = function(ev){
        ev = window.event || ev;

        //menu需要跟随鼠标的位置
        oMenu.style.display = 'block';
        oMenu.style.left = ev.clientX + 'px';
        oMenu.style.right = ev.clientY + 'px';

        //禁止本来的右键弹出的菜单这个默认行为, 兼容写法
        ev.preventDefault() ? ev.preventDefault() : ev.returnValue = false;
      }

      //点击左键menu消失
      document.onclick = function(){
        oMenu.style.display = 'none';
      }
    }
  </script>
</head>

<body>
  <div id='menu'></div>
</body>
```

关于阻止默认事件的更简单的写法：
```javascript
window.onload = function(){ 
  var oMenu = document.getElementById("menu");
  //oncontextmenu右键触发
  document.oncontextmenu = function(ev){
    ev = window.event || ev;

    //menu需要跟随鼠标的位置
    oMenu.style.display = 'block';
    oMenu.style.left = ev.clientX + 'px';
    oMenu.style.right = ev.clientY + 'px';

    return false;
  }

  //点击左键menu消失
  document.onclick = function(){
    oMenu.style.display = 'none';
  }
}
```
