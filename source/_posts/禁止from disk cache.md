---
title: 禁止from disk cache
date: 2023-02-01 11:53:27
tags: 其他
description: 浏览器回退到当前页面时，需要从接口调取数据而非缓存的解决办法......
---

# 浏览器回退到当前页面时，需要从接口调取数据而非缓存的解决办法？
今天遇到了一个bug，从页面A跳转到页面B的时候提交了一份表单信息使得用户信息发生了变化，但是从页面B返回到页面A的时候，获取到的用户信息还是提交之前的状态。
后来发现是因为页面发起请求获取信息的时候，直接从浏览器拿了缓存。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9578e8302e641de91e30b1372e8c14e.png)
# 什么是from disk cache
from disk cache意思是从磁盘中获取缓存，这是因为请求的时间在缓存的时间长度内（max-age），导致会先从缓存中拿数据，缓存中没有才会向服务器发起请求。它跟304的区别是，304会再跟服务器确认一次数据，确保数据没有变化，而**from disk cache没有跟服务器确认，就直接使用浏览器的缓存。**
# 解决方法
1. http关闭缓存
```javascript
ajax.get('/url',{headers:{'Cache-Control':'no-cache'}})
```
2. 动态改变请求路径
```javascript
ajax.get(`/url?time=${new Date().getTime()}`)
```
使得每次请求的路径都不同，则不会去获取缓存。	
