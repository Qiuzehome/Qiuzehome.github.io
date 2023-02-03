---
title: 通过javascript与node代码实现JSONP解决跨域
date: 2023-02-01 11:34:29
tags: node
description: 用node去实现JSONP解决跨域......
---
# 一.什么是跨域
浏览器从一个域名的网页去请求另一个域名的资源时，域名、端口、协议任一不同，会导致跨域拿不到响应内容。
**当一个请求 url 的协议、域名、端口三者之间任意一个与当前页面 url 不同即为跨域**
https://(协议)www.baidu.com(域名)8080(端口号)
# 二.什么是JSONP
平时在一些官方文档中，如vue,jquery都会通过script标签引入cdn加载框架，不用去安装就可以使用。那为什么这样就不会引起跨域呢？这是因为当初**设计script的时候就允许它在其他源请求别的脚本**，所以早起开发者们就可以通过这个方法进行跨域，这种**通过script引入脚本的方法就是JSONP**。
# 三.通过js与node实现JSONP
我们通过代码去展示一下具体过程。
### 客户端代码：
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body></body>
</html>
<script src="https://code.jquery.com/jquery-3.0.0.min.js"></script>
<script>
  $.ajax({url:"http://127.0.0.1:8888/?callback=a",type:"get"}).then((data)=>{
      console.log(data)
  })
</script>

```
### 服务端代码
```javascript
const http = require('http')
const url = require('url')
let server = http.createServer((req, res) => {
    let urlObj = url.parse(req.url, true)
    let data = urlObj.query
    console.log(data)
    res.end(`console.log(${JSON.stringify(data)})`)
})
server.listen(8888)
```
服务端启动了一个8888端口的服务，响应了一段js代码。客户端通过ajax对8888端口发起请求结果我们在浏览器中看到了如下情况![在这里插入图片描述](https://img-blog.csdnimg.cn/47c8421e5f474e5da4f1396a101d241c.png)果不其然地跨域了，接下来我们用jsonp的方法来请求这段资源。
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body></body>
</html>
<script src="http://127.0.0.1:8888/?a=2"></script>

```
我们通过script标签引入8888端口的资源**由于script标签的type默认是text/javascript，因此会将引入的代码作为javascript语句执行。**这里执行了node响应的代码console.log(JSON.stringify(data))
![在这里插入图片描述](https://img-blog.csdnimg.cn/a6ce99fb272a4981bc1abbedeebada61.png)
所以得出我们是可以通过这种方法拿到服务端的数据的，并且服务端也能通过引入的地址上的参数，拿到客户端传过来的参数。如果我们想拿到服务端的数据并用到我们的代码逻辑中，我们可以通过服务端让客户端调用方法的方式拿到服务端的数据。如下。
### 客户端
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body></body>
</html>
<script>
  const a = (data) => {
    console.log(`this is ${data}`);
  };
</script>
<script src="http://127.0.0.1:8888/?callback=a"></script>

```
### 服务端
```javascript
const http = require('http')
const url = require('url')
let server = http.createServer((req, res) => {
    let urlObj = url.parse(req.url, true)
    let fn = urlObj.query.callback
    res.end(`${fn}('响应数据')`)
})
server.listen(8888)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/616a1fbf83fc48a8aa47a73c6997e710.png)
我们在客户端中写了一个a方法，引入服务端资源的时候传了一个callback的参数a，告诉服务端调用a方法。服务端拿到a，之后做出响应代码**a('响应数据')**，所以客户端代码中的a方法的参数被传入了‘响应数据’。这时候客户端就顺利拿到了服务端传出的数据了。
# 四.总结
通过这也可以看出这种方法的弊端，它只适用于get请求，因为我们向服务端传送的数据只能通过请求地址的链接参数发送给服务端。此外它还有以下缺点
1. 只适用于get请求
2. 请求失败的时候无法捕获错误，获取不到状态码
3. 缺乏安全性，怕服务端存在漏洞，受到xss攻击，返回的script代码受他人控制
4. 只支持跨域HTTP 请求这种情况，不能解决不同域的两个页面之间如何进行 Javascript 调用的问题