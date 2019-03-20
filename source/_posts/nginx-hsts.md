---
title: nginx ingress controller 的HSTS配置
---

今天遇到一个奇怪的问题，在`k8s`里通过`nginx ingress controller`设置的`ingress` 里使用`tls`字段设置了`https`证书之后，在浏览器里所有的`子域名都被定向到了https`，而我们的证书不支持泛域名，所以访问就会报错

定位了一番，发现是`nginx`的`HSTS`配置导致的，`HSTS`是`HTTP Strict Transport Security`,是现代浏览器都支持的`Web安全协议`, 作用是强制客户端使用`HTTPS`与服务器创建连接.

采用`HSTS`协议的网站将保证浏览器始终连接到该网站的HTTPS加密版本，不需要用户手动在URL地址栏中输入加密地址,而这与在服务端做重定向不同（nginx与大部分负载均衡可以配置将http请求定向到https）

HSTS的作用是强制客户端（如浏览器）使用HTTPS与服务器创建连接。服务器开启HSTS的方法是，当客户端通过HTTPS发出请求时，在服务器返回的超文本传输协议`响应头`中包含`Strict-Transport-Security`字段。非加密传输时设置的HSTS字段无效。

比如，https://xxx 的响应头含有`Strict-Transport-Security: max-age=31536000; includeSubDomains`。这意味着两点：
在接下来的一年（即31536000秒）中，浏览器只要向xxx或其**子域名**发送HTTP请求时，必须采用HTTPS来发起连接。比如，用户点击超链接或在地址栏输入 http://xxx/ ，浏览器应当自动将 http 转写成 https，然后直接向 https://xxx/ 发送请求。
在接下来的一年中，如果 xxx 服务器发送的TLS证书无效，用户不能忽略浏览器警告继续访问网站

`nginx ingress controller` 默认开启了`HSTS`并且启用了`includeSubDomains`, 这导致了我描述的问题，解决的方法很简单，就是在nginx的配置`configMap`里添加`hsts-include-subdomains: "false"`，注意：不能加在`annotations`里，参考[文档](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#hsts)

附上在`Chrome`里检查`HSTS`的方法, 访问`chrome://net-internals/#hsts`,在`Query HSTS/PKP domain`里输入域名即可查询

![](https://ws3.sinaimg.cn/large/006tKfTcly1g19j656iykj309r08tgmd.jpg)

可惜的是没找到好的方法清空已经获取的`HSTS`配置，默认是`15724800` 秒（半年），即便是清空浏览器缓存也不行，已经踩坑的苟且方法是可以用浏览器的`隐身模式`访问