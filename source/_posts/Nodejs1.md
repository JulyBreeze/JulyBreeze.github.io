---
title: Nodejs 1
date: 2019-04-08 21:35:02
tags:
---

```javascript
const fs = require('fs');
fs.writeFileSync('hello.txt', 'hello world');
```

### Core Modules

```html
http: launch a server, send request
https: lauch SSL server(data is encrypted)
fs
path
os
```


### Node LifeCycle
https://nodejs.org/de/docs/guides/event-loop-timers-and-nexttick/

The event loop is what to perform non-blocking I/O operations by offloading减轻 operations to the system kernel whenever possible. 


### Example

例1
```javascript
const http = require("http");

//@parameters: request data, response
function requestListener (req, res) {
	...
}

//@parameters: a function to process request
// the parameters must in order like above
http.createServer(requestListener);

//the above code can be written as anonymous function
http.createServer(function(req, res) {
});
```

更简单的写法：
```javascript
const server = http.createServer((req, res) => {
  console.log(req);
  //process.exit();//退出event loop
  
  //send back as response
  res.setHeader('Content-Type', 'text/html');
  res.write('<h1>Hello World</h1>');
  res.write('<h2> this is the response<h2>')
  res.end();
})

//@parameters: port(default is 80)
server.listen(3000);
```

例2
解释：
当地址是localhost:3000时，显示一个输入框，点击send按钮后：  
如果没有第二个if，就会跳到/message这个地址，显示hello world；  
如果加上第二个if，则会保持在'/'地址，同时产生一个新文件里面有test字样。  
此时打开右键-network，可以看到出现message和302，表示重定位成功

```javascript
const http = require("http");
const fs = require('fs');

const server = http.createServer((req, res) => {
  const url = req.url;
  const method = req.method;

  if(url === '/') {
    res.write('<h1>Enter message</h1>');
    //it add the input data to the request and make it accessable via the assigned name
    res.write('<form action="/message" method="POST"><input type="text" name="message"><button type="submit">Send</button></form>');
    return res.end();//return from this anonymous
  }
  
  if(url === '/message' && method === 'POST'){
    fs.writeFileSync('message.txt', 'test');
    res.statusCode = 302;
    res.setHeader('Location', '/');
    return res.end();
  }
  res.setHeader('Content-Type', 'text/html');
  res.write('<h1> hello world </h1>');
  res.end();
})

server.listen(3000);

```

输入输入框的data是request data


### Streams & Buffers

The request is read by nodejs in chunks
```html

```


### HTTP Headers

```
case-insensitive name: value(without line breaks)
```

根据context的不同可分为四种类型：

* General header: apply to both request and response, with no relation to the data eventually transmitted in the body
* Request header: containing info about the resource to be fetched or about the client itself
* Response header: contain info about the response, like its location or about the server itself
* Entity header: contain info about the body of the entity, like content length or MIME-type