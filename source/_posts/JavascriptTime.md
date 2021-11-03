---
title: 4.Javascript
date: 2019-03-18 12:20:14
tags: js
---
### 获取元素

```javascript
getElementById();
getElementsByTagName(); //return a list
getElementsByClassName(); //return a list
getElementsByName(); //return a list
```

例：

```html
<!-- 单选框的name必须是一样的 -->
<input type="radio" name="gender" />
<input type="radio" name="gender" />
```

```javascript
var genders = document.getElementsByName("gender");
document.onclick = function(){
  ...
}
```

### 获取element的style
```html
<head>
  <style type="text/css">
  #box {
    width:300px;
    height: 300px;
    background: green;
  }
  </style>
  <script type="text/javascript">
    window.onload = function(){
      var btn = document.getElementById('bt);
      var box = document.getElementById('box');
      btn.onclick = function(){
        //alert.(btn.style.width) doesn't work
        //because style can only access in inline style

        //IE browser
        alert(btn.currentStyle.width);

        //other browser
        var style = getComputedStyle(btn);
        alert(style.width);

        //不知道是哪个浏览器，所以采用兼容的方法
        var width = '';
        if(window.getComputedStyle(btn)){
          var style = getComputedStyle(btn);
          width = style.width;
        }else{
          width = btn.currentStyle.width;
        }
        alert(width);
      }
    }
  </script>
<head>
```
</br>
上面这样写很麻烦，所以可以封装成一个函数：

```javascript
//att是要获取的object的属性
function getStyle(obj, att){
  if(window.getComputedStyle(obj)){
    //因为传进来的att是string，而实际调用是getComputedStyle(obj).width
    //所以用[]
    return getComputedStyle(object)[att];
  }else{
    return obj.currentStyle[att];
  }
}
```

### wrap封装
可以把常用的，比如上面的getStyle函数单独放在一个js文件中，然后引入到html中：

```html
<head>
  <script type="text/javascript" src="xxxx"></script>
</head>
```

这里的封装其实就是jquery的原理：  
在外部js文件中加入：
```javascript
//function name is $
function $(id){
  return document.getElementById(id);
}

$('btn').onclick = ...
```


也可以把window.onload = function(){}一起封装在外部js文件中:

```javascript
function $(input) {
  if(typeof input === 'string'){
    return document.getElementById(input);
  }else if(typeof input === 'function'){
    window.onload = input;
  }
}
```

然后就可以在html中这样使用：

```html
<script type="text/javascript" src="xxxx"></script>
<script type="text/javascript">
  $(function(){
    //点击btn，使其width增加10px
    $('btn').onclick = function(){
      //因为style.width是一个string，所以需要先变成int格式
      //+10后再变成string格式( + 'px')
      $('btn').style.width = parseInt(getSyle($('btn'), 'width')) + 10 + 'px';
    }
  })
</script>
```


### Timer
1)setInterval()
```javascript
//重复操作的定时器，每间隔time触发function一次，time最小只能使10ms
//@return 时钟对象
setInterval(function, time/毫秒)
clearInterval(时钟对象)
```

```html
<script type="text/javascript">
$(fucntion(){
  var timer = null;
  var index = 0;
  //开始计时
  $('start').onclick = function(){
    timer = setInterval(speak(), 1000);
  }

  function speak(){
    index++;
    console.log(index);
  }

  $('stop').onclick = function(){
    clearInterval(timer);
  }
})
</script>

<input type="button" name="start" id="start" value="start"/>
<input type="button" name="stop" id="stop" value="stop"/>
```
但每点一次start button，就开启一个定时器，这样console里面显示的速度会越来越快。  
解决方法，只有timer为空的时候才创建：

```javascript
$('start').onclick = function(){
  if(timer == null){
    timer = setInterval(speak(), 1000);
  }
}
//但这样如果点击stop停止计数后，再点击start，就没有反应了
//所以还要在stop里添加一句：
$('stop').onclick = function(){
    clearInterval(timer);
    timer = null; //这里需要手动重新设为null
  }
```

2)setTimeout()

```javascript
//只执行一次
setTimeout()
clearTimeout()
```

### 应用-轮播图

假设轮播5张图。  
轮播图下的5个圆点是通过< p >里的< i >实现的

```html
<head>
  <style type="text/css">
    /* first clear style */
    *{
      margin: 0;
      padding: 0;
    }

    img{
      display: block;
      border: 0;
    }

    #box {
      width: 600px;
      margin: 50px auto;
      position: relative;
    }

    #pageView {
      position: absolute;
      bottom: 10px;
      /* 让轮播图下方的那五个小圆点放在中间 */
      text-align: center;
      width: 100%;
      height: 100%;
    }

    #pageView i {
      width: 10px;
      height: 10px;
      display: inline-block;
      border: 2px solid gray;
      border-radius: 6px;
    }

    /* 选中的点的边框和颜色都是黑色 */
    #pageView .active{
      border-color: black;
      background: black;
    }
  </style>
</head>
<body>
  <div id="box">
    <img src="img/img_1.png" id="img" />
    <p id="pageView">
      <i class="active"></i>
      <i></i>
      <i></i>
      <i></i>
      <i></i>
    </p>
  </div>
</body>
```

实现的过程：需要给每个轮播图一个index，所以设定i tage的自定义属性index。每次点击一张圆圈时时，先暂停自动轮播，然后获取当前的index，展示当前的图片。过一定时间，轮播又自动开始。而轮播的实现就是update()函数的重复执行，即index++，展示index的图，然后index继续++，如此。

```javascript
$(function(){
  var oImg = $('img');
  var oPages = document.getElementsByTagName('i');

  var index = 0;
  var imgName ='';
  var timer = null;
  var currentPage = 0;

  //call functions
  //startTimer
  start();

  //page点击
  for(var i = 0; i < oPages.length; i++){
    //index是自定义属性
    oPages[i].index = i;
    oPages[i].onclick = function(){
      //stop timer
      stop();

      index = this.index;
      changeView();
      //start timer again
      start();
    }
  }

  //wrap functions
  function start(){
    if(timer == null){
      timer = setInterval(update, 2000);
    }
  }

  function stop(){
    clearInterval(timer);
    timer = null;
  }

  //setInterval里要被触发的函数
  function update(){
    index++;
    changeView();
  }


  function changView(){
    //这个if相当于动态数组，实现循环
     if(index > 4) index = 0;

    //改变图片
    imgName = 'img/img_' + (index + 1) + '.png';
    oImg.src = imgName;

    //改变页码，把原来的<i>上的active消除，然后把当前的i变成active
    oPages(currentPage).className = '';
    currentPage = index;
    oPages[currentPage].className = 'active';
  }
})
```

### Date

```javascript
$(function(){
  var nowTime = new Date();
  //nowTime.getTime() 是毫秒数
  nowTime.getFullYear();//
  nowTime.getMonth() + 1;// month[0,11], so + 1
  nowTime.getDate(); //日 
  nowTime.getDay(); //星期数
  nowTime.getHours();
  nowTime.getMinutes();
  nowTime.getSeconds();
})
```

#### 应用-倒计时

倒计时时间 = 结束时间 - 当前时间  
用Date()获取当前时间，设定结束时间，然后取得这两个时间的差值，从中提取出秒分时，利用innerHTML放入i tag中。这个操作每个1s就执行一次，因此用setInterval(function(), 1000)来实现。

```html
<head>
  <style type="text/css">
    i {
      display: inline-block;
      width: 20px;
      height: 20px;
      color: white;
      background: green;
      line-height: 20px;
      text-align: center;
    }
  </style>
</head>
<body>
  <i></i> : <i></i> : <i></i>
</body>
```

```javascript
$(function(){
  var oI = document.getElementsByTagName('i');
  setInterval(function(){
    
    var time = new Date();
    //current time
    var nowTime = time.getTime();

    //结束时间，假设是10号12：00
    time.setDate(10);
    time.setHours(12);
    time.setMinutes(0);
    time.setSeconds(0);
    var endTime = time.getTime();

    //转换成秒
    var diff = (endTime - nowTime) / 1000;
    var s = parseInt(diff % 60);
    var m = parseInt(diff / 60 % 60);
    var h = parseInt(diff / 60 / 60);

    oI[0].innerHTML = h;
    oI[1].innerHTML = m;
    oI[2].innerHTML = s;
  }, 1000);
})
```