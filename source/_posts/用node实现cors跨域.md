---
title: 用node实现cors跨域
date: 2023-02-01 11:35:59
tags: node
description: 用node去实现cors解决跨域......
---
# 1. 关于跨域
浏览器从一个域名的网页去请求另一个域名的资源时，域名、端口、协议任一不同，会导致跨域拿不到响应内容。
**当一个请求 url 的协议、域名、端口三者之间任意一个与当前页面 url 不同即为跨域**
https://(协议)www.baidu.com(主机)8080(端口号)
早起开发者们为了解决这一问题使用了JSONP的方法进行处理（具体可看[JSONP的原理，以及如何使用node实现](https://blog.csdn.net/weixin_43240744/article/details/124183510?spm=1001.2014.3001.5501)）。但毕竟还是存在很多局限，于是有了cors。
# 2.CORS
当浏览器进行跨域请求的时候，会在**请求里添加头部origin，表明自己协议，主机，端口。当服务器收到这个客户端发送的请求之后，如果需要允许能够访问，就需要添加头部信息Access-Control-Arrow-Origin到响应里面。浏览器收到传回来的这个头部信息就知道能不能进行跨域请求了。**
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
  $.ajax({ type: "get", url: "http://127.0.0.1:8888/get" }).then(
    (data) => {
      console.log(data);
    }
  );
</script>
```
```javascript
let http = require('http')
let server = http.createServer()
server.listen(8888)
server.on('request', (req, res) => {
    var result = {
          code: 0,
          msg: "请求成功"
      }
      res.end(JSON.stringify(result))
})
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7f023ab315dc4493a1aa85638f21d1fd.png)
我们在8080端口中运行一个html并且对8888端口的服务器发起请求，由于两者不在同一个端口所以发生了跨域。这段报错的意思是没有发现Access-Control-Arrow-Origin这个请求头，我们在Network里面也确实没发现这个头部。
![在这里插入图片描述](https://img-blog.csdnimg.cn/af6fb1d47f1c44debaec5d5c5333703e.png)
针对这种情况，我们可以通过设置请求头Access-Control-Allow-Origin，允许跨域。
```javascript
let server = http.createServer((req, res) => {
    res.writeHead(200, {
        'Access-Control-Allow-Origin':'http://127.0.0.1:8080'//可以是*，也可以是跨域的地址
    })
})
```
这时我们再去发起请求，可以看到响应头中的Access-Control-Allow-Origin属性，指向我们的客户端地址，这里我们也可以设置*即允许所有地址。并且我们这里也不再发生跨域的错误了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e59f6b1d5b74dc08041a8a6923adb3d.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/96cdde35da8d447581ec35c0dac56e19.png)
### 其他请求方式
如果我们将请求方式换成**put**请求会发现，它又发生跨域了
![在这里插入图片描述](https://img-blog.csdnimg.cn/7310fa657eec48438ce926ce62ab0b87.png)
这是因为浏览器为了**防护用户恶意修改浏览器数据，使用了例如put，delete等方法，我们可以通过设置请求头字段Access-Control-Allow-Methods**，使得浏览器允许这些方法跨域
```javascript
let server = http.createServer((req, res) => {
    res.writeHead(200, {
        'Access-Control-Allow-Origin': 'http://127.0.0.1:8080',//可以是*，也可以是跨域的地址
        'Access-Control-Allow-Methods': ['GET', 'POST', 'PUT']
    })
})
```
这时候再去发起请求就可以正常拿到响应内容了
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ce4db89e5da441f818d46517150be91.png)
