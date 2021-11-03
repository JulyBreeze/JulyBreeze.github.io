---
title: 2.JS 单线程 & 异步
date: 2019-03-19 11:47:11
tags: js
---
### 单线程

js是单线程的，即所有任务需要排队，前一个任务结束，才会执行后一个任务。为了解决如果前一个任务执行时间过长的问题，产生了任务队列task queue。

任务分两种：同步synchronous，异步asynchronous.  
同步：在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；  
异步：不进入主线程、而进入task queue的任务，只有task queue通知主线程，某个async任务可以执行了，该任务才会进入主线程执行。

具体过程：  
* 所有sync都在主线程上执行，形成一个执行栈execution context stack
* 主线程之外，还存在一个任务队列task queue。只要某async任务有了运行结果，就在task queue之中放置一个事件。
* 一旦execution context stack中的所有同步任务执行完毕，系统就会读取task queue，
* 那些对应的异步任务结束等待状态，进入stack开始执行。

### 异步

异步执行可以用回调函数callback实现。

1)setTimeout()和setInterval()  
最基本的函数，二者可以改变一个队列函数的执行顺序。

```javascript
var nums = [1,2,3];
for(var i in nums){
  setTimeout(function(){console.log(nums[i])}, 0);
  console.log(nums[i]);
}

//output
1 2 3 3个3
```

解释：
js是单线程，还维护一个setTimeout队列，未执行的setTimeout任务就按顺序放入这个队列中。等待普通的任务队列中的任务执行完才开始按顺序执行积累在setTimeout中的任务。所以上题，先打印1 2 3，然后将setTimeout里的任务放入setTimeout队列。等待1 2 3打印完毕后，执行里面那个function 3次，输出三个3。

2)ajax  
Asynchronous JavaScript and XML  
核心是XMLHttpRequest对象(XHR)。XHR为向服务器发送请求和解析服务器相应提供了流畅的接口，使用XHR对象获取新数据，然后通过DOM将数据插入到页面中，做到不刷新页面也能获取数据。

3)fetch()


### Promise

Promise represents the eventual completion/failure of an async operation. 它是一个constructor，function里有两个arguments，这两个参数是functions：resolve(成功后的callback)、reject(失败后的callback)。  
Promise.prototype属性中有then()，所以Promise创建的实例可以使用.then()

```javascript
var p1 = new Promise(function(resolve, reject) {
  ...
});

//使用arrow function
const p2 = new Promise((resolve, reject) => {
  resolve(someValue);
  // or
  reject('error reason');
})
```

规范：

* 一个promise有三种状态: 等待pending、已完成fulfilled、已拒绝rejected
* promise必须实现then方法  
then必须返回一个promise，同一个promise的then可以调用多次，并且回调的执行顺序跟它们被定义时的顺序一致。  
then方法接受两个参数，第一个参数是成功时的回调，在promise由pending态转换到fulfilled态时调用; 第二个是失败时的回调，在promise由pending态转换到rejected态时调用。  
同时，then可以接受另一个promise传入，也接受一个“类then”的对象或方法，即thenable对象。

具体例子：

```javascript
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("success");
  },100);
});

p.then((successMessage) => {
  console.log(successMessage);
});
```

### async await

async/await从上到下顺序执行，像写同步代码一样  

1)async  

* 建立在Promise的基础上。
* 非阻塞  
async函数里面如果有异步过程会等待，但是async函数本身会马上返回，不会阻塞当前线程。可以简单认为，async函数工作在主线程，同步执行，不会阻塞界面渲染。async函数内部由await关键字修饰的异步过程，工作在相应的协程上，会阻塞，等待异步任务的完成再返回。
* **返回的是Promise对象**
如果return一个直接量，async会把这个直接量通过Promise.resolve()封装成Promise对象后返回  
如果没有return，相当于返回了Promise.resolve(undefined);


2)await  

* await只能放在async函数内部使用
* await后面跟着是Promise对象

```javascript
async function f(){
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done"), 1000);
  });

  let res = await promise; //wait until promise resolve
  alert(res);
}

f();
```

### AJAX

#### 1.基本概念  

1)XML，可扩展标记语言Extensible Markup Language, 用来传输和存储数据。  
2)XMLHTTP是一组API函数集，可被JavaScript等web浏览器内嵌的脚本语言调用，通过HTTP在client和server之间收发XML或其它数据。XMLHTTP可以动态地更新网页，即不重新加载页面的情况下更新网页，无需重新从服务器读取整个网页。  
XMLHttpRequest对象用于在后台与服务器交换数据。

#### 2.ajax的具体实现例子  

```javascript
//创建请求对象
var request = new XMLHttpRequest();
//设置请求
//@params: get/post, 请求地址，是否异步(true表示异步)
request.open('GET', 'Json1.json', true)
request.send();
request.onreadystatechange = function(){
  /*
  0: 初始化，还没有调用open()方法
  1： 载入，已调用send()方法，正在发送请求
  2： 载入完成，send()方法完成，已经收到response
  3： 正在解析response
  4： 解析完成，可以在client端使用
  */
  if(request.readyState == 4){
    if(request.status == 200){
      //把json转换为js对象
      //jsondata里面是：people：[{object1}，{2}，{3}]
      var jsondata = JSON.parse(request.response);
      
      var people = jsondata.people;

      //渲染
      var ul = document.createElement('ul');
      for(var i = 0; i < people.length; i++){
        var li = document.createElement('li');
        li.innerHTML = people[i].firstName;
        ul.appendChild(li);
      }
      document.body.appendChild(ul);
    }
  }
}
```

#### 3.封装

```javascript
//第二个参数是获取数据后的操作
function ajax_get(url, successFunc){
  var request = window.XMLHttpRequest ? new XMLHttpRequest() : new ActiveXObject('Microsoft.XMLHTTP');
  request.open('GET', url, true);
  request.send();
  request.onreadystatechange = function(){
    if(request.readyState == 4){
      if(request.status == 200){
        if(successFunc) successFunc(request.response);
        //上面的if也可以写成：
        //successFunc && successFunc(request.response);
      }else{
        console.log("cannot obtain data");
      }
  }
}

//使用以上的封装函数
ajax_get('xxxx', function(response){
  console.log(response);
});
```

#### 4.Get请求

要发给服务器的参数放在url的?后面，多个参数用&隔开。

```html
username=xxx&password=xxxx
```

Comet是对Ajax的扩展，是服务器几乎实时的向客户端推送数据。实现方法有长轮询和HTTP流。

#### 5.跨域资源共享

XHR对象只能访问与包含它的页面位于同一个域的资源。

##### CORS

cross origin resource sharing  
用自定义的HTTP头部让浏览器和服务器进行沟通。头部包含请求页面的源信息（协议、域名和端口）。

##### JSONP

JSON with padding(填充式JSON)

由两部分组成：callback函数和数据：  
* 回调函数：当响应到来时，应该在页面中调用的函数。回调函数的名字一般是在请求中指定的。  
* 数据：传入callback里的JSON数据。

例子：
http://xxx.com/json/?callback=handleResponse

这里指定了回调函数的名字叫handleResponse().

JSONP通过动态< script >来使用，因为script元素不受限制可以从其他域中加载资源。  
优点：直接访问响应的文本，支持在浏览器和服务器之间双向通信。  
缺点：当从其他域中加载执行时，可能会在相应中夹带恶意代码；确定JSONP请求是否失败不容易。