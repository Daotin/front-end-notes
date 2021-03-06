# 55 详细梳理ajax跨域4种解决方案

## 前言

自动接触前端，跨域这个词就一直萦绕在耳畔。因为一般接手的项目都已经做好了这方面的处理，而且之前一直感觉对这方面模棱两可，所以今天就抽个时间梳理一下。

## 为什么需要跨域

跨域这个概念来自一个叫 “同源策略” 的东西。同源策略是**浏览器**（注意是浏览器，跟通信协议无关）上为了安全考虑实施的非常重要的安全机制。

`Ajax` 默认只能获取到同源的数据，对于非同源的数据，Ajax是获取不到的。

### 什么是同源？

> * 协议相同
> * 域名相同
> * 端口相同

举例来说，`http://www.example.com/dir/page.html`这个网址，协议是`http://`，域名是`www.example.com`，端口是`80`（默认端口可以省略）。这个网址，在这个地址中要去访问下面服务器的数据，那么会发生什么情况呢？

| URL | 结果 | 原因 |
| :--- | :--- | :--- |
| `https://www.example.com/dir/other.html` | 不同源 | 协议不同，https 和 http |
| `http://en.example.com/dir/other.html` | 不同源 | 域名不同 |
| `http://www.example.com:81/dir/other.html` | 不同源 | 端口不同 |
| `http://www.example.com/dir/page2.html` | 同源 | 协议，域名，端口都相同 |
| `http://www.example.com/dir2/page.html` | 同源 | 协议，域名，端口都相同 |

那么。想要获取非同源地址的数据，就要使用跨域。不论是 Ajax 还是跨域，都是为了访问服务器的数据。简单的来说， **Ajax 是为了访问自己服务器的数据，跨域是为了访问别人服务器的数据（比如获取天气信息，航班信息等）。**

### 同源策略的目的

为了保证用户信息的安全，防止恶意的网站窃取数据。

> “ 设想这样一种情况：A网站是一家银行，用户登录以后，又去浏览其他网站。如果其他网站可以读取A网站的 Cookie，会发生什么？
>
> 很显然，如果 Cookie 包含隐私（比如存款总额），这些信息就会泄漏。更可怕的是，Cookie 往往用来保存用户的登录状态，如果用户没有退出登录，其他网站就可以冒充用户，为所欲为。因为浏览器同时还规定，提交表单不受同源政策的限制。
>
> 由此可见，"同源政策"是必需的，否则 Cookie 可以共享，互联网就毫无安全可言了。”
>
> via@ 阮一峰

## 实现跨域的方式

> * 反向代理
> * JSONP
> * WebSocket
> * CORS（根本解决方案）

### 反向代理

反向代理就是使用自己的服务器，在后端请求目标服务器的数据，然后返回给客户端。相当于做了一把中间人的感觉。

反向代理服务器，最常用的就是`Nginx`。但是作为前端代码实现的`Node.js`也可以搭建反向代理服务器。

下面来简要介绍使用node服务进行反向代理。

> 要实现这个前提是，**前端开发环境必须运行在nodejs服务中**，所幸的是，现在前端的开发自动化工具都是建立在nodejs上的，所以这个前提也不是很重要。

比如我有一个后端接口：`http://39.105.136.190:3000/zhuiszhu/goods/getList`，可以获取一些商品列表数据，但是我运行的node项目是在 `localhost:3000` 下的，明显构成跨域。

我们根据项目使用的框架不同，处理的方式也不同。

#### 1、nodejs+express+http-proxy-middleware 插件代理

如果是express项目，可以使用`http-proxy-middleware` 来处理，这个插件主要用于将前端请求代理到其它服务器。

用法很简单。你可以参考插件github官网： [https://github.com/chimurai/http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)

首先需要在你的express项目中安装该插件：

```text
npm install --save-dev http-proxy-middleware
```

然后在 `app.js` 中进行代理设置（示例如下）：

```javascript
var express = require('express');
var proxy = require('http-proxy-middleware');

var app = express();

app.use('/zhuiszhu', proxy({target: 'http://39.105.136.190:3000/', changeOrigin: true}));
app.listen(3000);
```

之后再项目中再次发送ajax请求的时候，可以看到数据可以收到了。

```javascript
$.ajax({
    url: '/zhuiszhu/goods/getList',
    type: 'GET',
    success(res) {},
});
```

![image-20191107135547280](https://user-images.githubusercontent.com/23518990/68364644-89019e80-0169-11ea-93a1-b2da9e24b4a7.png)

#### 2、webpack-dev-server 代理

我们经常在vue开发项目的时候，会用webpack作为前端自动化构建工具的话，也会使用到`webpack-dev-server`的插件，那么可以引用`webpack-dev-server`来进行配置跨域方案。

webpack-dev-server是一个小型的nodejs服务器，是基于express框架的，用于实时监听和打包编译静态资源。其中里面有一个属性是proxy，是专门来配置代理请求接口的。

你只需要在`webpack.config.js`中 `devServer`中如下设置即可：

```javascript
devServer: {
        port: 3000,
        inline: true,
        proxy: {
            "/zhuiszhu": {
                target: "http://39.105.136.190:3000/",
                changeOrigin: true  //必须配置为true，才能正确代理
            }
        }
    },
```

其实可以看出， webpack 的 devServer.proxy 就是使用了非常强大的 http-proxy-middleware 包来实现代理的，所以本质上是相通的。

### JSONP

JSONP基本思想是，网页通过添加一个`<script>`元素，向服务器请求JSON数据，这种做法不受同源政策限制；服务器收到请求后，将数据作为参数放在一个指定名字的回调函数里传回来，这个回调函数的名字我们需要通过js定义好。

比如：当页面资源加载完毕时候，获取跨域的数据，并且制定回调函数的名字为`foo`：

```javascript
window.onload = function () {
    var script = document.createElement("script");
    script.setAttribute("type","text/javascript");
    script.src = "http://bbb.com?callback=foo;
    document.body.appendChild(script);
};

foo(data) {
    console.log(data); // data即为跨域获取到的数据
}
```

在服务器那边，需要将数据放入foo函数的参数中：

```javascript
foo('hello world')
```

使用JSONP需要注意：

> 1. 必须后端配置相应回调函数。
> 2. 只能发送`GET`请求。

### WebSocket

WebSocket是一种通信协议，使用`ws://`（非加密）和`wss://`（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。

由于发出的WebSocket请求中有有一个字段是`Origin`，表示该请求的请求源（origin），即发自哪个域名。

正是因为有了`Origin`这个字段，所以WebSocket才没有实行同源政策。因为服务器可以根据这个字段，判断是否许可本次通信。

### CORS

CORS全称“ Cross-origin resource sharing ”（跨域资源共享），相比JSONP， CORS允许任何类型的请求 。

> CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。
>
> 整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。
>
> 因此，实现CORS通信的关键是服务器。**只要服务器实现了CORS接口，就可以跨源通信。**

### 总结

综上，如果访问的别人服务器的资源，并且未设置JSONP，也未开放WebSocket白名单，也没有设置CORS接口，那么唯一的选择就是使用自己的服务器进行反向代理。

## 参考链接

[浏览器同源政策及其规避方法](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html%20)

[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html%20)

[express框架介绍](https://github.com/Daotin/Web/blob/master/14-Node.js/07-express%E6%A1%86%E6%9E%B6%E4%BB%8B%E7%BB%8D.md)

[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware%20)

本文完。

（啾咪 ^.&lt;）

