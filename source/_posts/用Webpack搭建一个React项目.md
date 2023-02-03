---
title: 用Webpack搭建一个React项目
date: 2023-01-31 16:14:12
tag: 工程化
description: 手把手展示如何用webpack搭建一个react项目......
---
# 先搭建一个简单的项目
## 初始化

cmd输入npm init，设置相关信息初始化项目。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621210211102.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621210251849.png)

<!--more-->
## 安装所需的依赖
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621211133316.png)
## 新建文件webpack.config.js配置出入口以及module
```javascript
const path = require('path')
module.exports = {
    entry: '/src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        rules: [{
            test: /\.(js|jsx)$/,
            exclude: /node_modules/,
            use: ['babel-loader'],
            include: path.resolve(__dirname, 'src')
        }]
    }
}
```
在对应的目录下新建一个index.js作为入口文件并编辑以下代码作为示范
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021062121184932.png)
```javascript
//index.js代码
document.getElementById('main').innerText = 'my react'
```
## 打包并引入
在终端执行webpack命令，或在package.json配置命令并执行进行打包
打包成功后对应目录下会生成打包后的文件，在html文件内引入打包生成的bundle.js进行验证
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021062121240835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzI0MDc0NA==,size_16,color_FFFFFF,t_70)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="main"></div>
</body>
<script src="../dist/bundle.js"></script>
</html>
```
运行发现引入成功了，这时候一个由webpack搭建的简单项目已经成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621212743254.png)

## 开始引入react
安装react，react-dom
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621213659995.png)
编写react
```javascript
import React from 'react'
import ReactDom from 'react-dom'
ReactDom.render(
    <div>Hi React</div>,document.getElementById('main')
)
```
执行webpack打包，编译报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621213940965.png)
这里无法识别react代码，还需要配置babel，分别安装@babel/preset-react，babel-plugin-transform-react-remove-prop-types并新建一个.babelrc进行配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621214258746.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621214409518.png)

```javascript
{
    "presets": [
        "@babel/preset-react"
    ],
    "plugins": [
        "transform-react-remove-prop-types"
    ]
}
```
再执行webpack打包，打包成功后打开引入js的html，发现成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210621214624998.png)

