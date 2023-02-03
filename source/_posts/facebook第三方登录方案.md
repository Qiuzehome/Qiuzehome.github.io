---
title: facebook第三方登录方案（javascript SDK）
date: 2023-02-01 11:57:26
tags: JavaScript
description: facebook第三方登录方案
---
# 创建应用
- 1.登录[facebook开发者平台](https://developers.facebook.com/apps/?show_reminder=true)
- 2.创建应用，填写相关信息->选择facebook登录->选择web->将网站加入白名单
 **客户端id**可创建后点击应用进去查看
# 开发流程
### 方案1
在创建应用时，到添加白名单的时候这里会有一份流程可以按着它的流程来添加代码开发，这里就不多介绍

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ld0LN921-1659717799116)(http://doc.linghit.io/server/index.php?s=/api/attachment/visitFile&sign=9864c875004006a760cea2cc7cdade97)\]](https://img-blog.csdnimg.cn/66a696a9a47f41689b1481d3f7c17189.png)

### 方案2
由于某些问题，步骤4中的添加按钮无法使用sdk提供的<<fb></fb>组件，这里提供另一种方法(可能是服务端加载顺序问题导致，暂时没去处理，先采用了别的方法进行开发)
- 1.引入sdk
```html
//src中的zh_TW是语言版本，可替换成其他如zh_CN
<script src="https://connect.facebook.net/zh_TW/sdk.js" async defer crossorigin="anonymous"></script>
```
- 2.初始化sdk
```javascript
FB.init({
	appId: FB_ID,//客户端id
	cookie: true,
	xfbml: true,
	version: 'v14.0'
});
```
- 3.登录
自定义dom调用该方法即可
```javascript
  FB.login(
        function (response) {
		//获取登录状态，token,id
            if (response.status === 'connected') {
			//请求api的/me接口获取用户更多信息，fields传入想获取的信息
                FB.api('/me?fields=id,name,email', function (fb_user) {
                    console.log(fb_user)//获取用户信息
                });
            } else {
                Toast('登錄失敗，請重新登錄');
            }
        },
        {scope: 'email'}//读取权限
    );
```
# 踩坑
- 1.需兼容sdk加载失败情况（翻墙网络不稳定等）
- 2.sdk加载的方案添加或更换域名需手动在开发者平台增加域名
- 3.fb强制采用https，本地开发不方便调试
![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-S5DeXrPI-1659717799116)(http://doc.linghit.io/server/index.php?s=/api/attachment/visitFile&sign=2facf4a586daba7e9d30fa8ceecfb7b4)\]](https://img-blog.csdnimg.cn/6070bd6f688945e9bb7a9cfdf71e6fdb.png)

#相关文档
- 开发文档：https://developers.facebook.com/docs/facebook-login/web