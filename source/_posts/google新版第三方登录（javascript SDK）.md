---
title: google新版第三方登录（javascript SDK）
date: 2023-02-01 11:55:59
tags: JavaScript
description: google新版第三方登录
---

# 注册流程
- 1.登录谷歌[开发者平台](https://console.cloud.google.com/home/recommendations)，点击左上角打开选择项目弹窗选择项目，若没有项目则新建项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c8141a6d75c40cc83b76d6d9203be2f.png)


- 2.创建完项目后，若是新账号需要先填写OAuth consent screen相关信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/99ccb7262c26431c99766baea62fa70e.png)

- 3.创建凭证
创建凭证 Credentials —> Create OAuth client ID —> Web application
然后填写h5应用的相关信息和谷歌登录授权的白名单域名和授权的页面路由。创建成功后会拿到谷歌给的Client ID。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2c601f745abc4b99afa99142835d5928.png)


# 开发流程
- 1.引入sdk
```html
<script src="https://accounts.google.com/gsi/client" async defer></script>
```
- 2.自定义按钮
```html
<div id="buttonDiv"></div>
```
- 3.初始化
```javascript
/*
fn :function 登录回调函数
*/
const google_init=(fn)=>{
 google.accounts.id.initialize({
        client_id: GOOGLE_ID,//客户端ID(创建应用第三步中获得的id)
        callback: fn
    });
    google.accounts.id.renderButton(
        document.getElementById('buttonDiv'),//自定义按钮
        {theme: 'outline', size: 'large', logo_alignment: 'center'} // customization attributes
    );
}
```
- 4.登录
登录回调拿到响应数据中的credential进行解析可以获取到账户更多信息
```javascript
const google_load = (response) => {
	console.log(response)
    const responsePayload = decode_jwt(response.credential);
	console.log(responsePayload)
};
//解析token
const decode_jwt = (token) => {
    var base64Url = token.split('.')[1];
    var base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    var jsonPayload = decodeURIComponent(
        window
            .atob(base64)
            .split('')
            .map(function (c) {
                return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
            })
            .join('')
    );
    return JSON.parse(jsonPayload);
};
google_init(google_load)
```
# 踩坑
- 1.需兼容sdk加载失败情况（翻墙网络不稳定等）
- 2.sdk加载的方案添加或更换域名需手动在开发者平台增加域名
#相关文档
- 开发文档：https://developers.google.com/identity/gsi/web 