# Http协议



## HTTP消息格式

### 请求消息

请求消息格式：请求行（request line）、消息头（header）、空行和消息体四个部分

- 请求行（起始行）：方法，URL地址，协议版本
- 消息头Headers：如Host,User-Agent,Accept,Accept-Language,Accept-Encoding,Connection,Content-Type,Content-Length
- 消息体Body：不是所有的请求都有一个 body：例如获取资源的请求，GET，HEAD，DELETE 和 OPTIONS，通常它们不需要 body。

HTTP协议请求方法：options,head,get,post,put,delete,trace,connect

### 响应消息

HTTP响应格式：状态行、消息头、空行和消息体。

- 状态行：版本，状态码，状态文本  如：HTTP/1.1 404 Not Found
- 消息头Headers：同上
- 消息体Body



## HTTPS

**HTTPS**（Hypertext Transfer Protocol Secure：超文本传输安全协议）是一种透过计算机网络进行安全通信的传输协议。HTTPS 经由 HTTP 进行通信，但利用 SSL/TLS 来加密数据包。

HTTPS 默认工作在 TCP 协议443端口，它的工作流程一般如以下方式：

- 1、TCP 三次同步握手
- 2、客户端验证服务器数字证书
- 3、DH 算法协商对称加密算法的密钥、hash 算法的密钥
- 4、SSL 安全加密隧道协商完成
- 5、网页以加密的方式传输，用协商的对称加密算法和密钥加密，保证数据机密性；用协商的hash算法进行数据完整性保护，保证数据不被篡改。





## 转发和重定向区别：

**forward（转发）**：

是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,因为这个跳转过程实在服务器实现的，并不是在客户端实现的所以客户端并不知道这个跳转动作，所以它的地址栏还是原来的地址.

**redirect（重定向）**：

是服务端根据逻辑,发送一个状态码（302）,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.

https://www.cnblogs.com/Qian123/p/5345527.html


## HTTP状态码：

- 2xx 成功，操作被成功接收并处理

  200 请求成功

  202 服务器已经接受，尚未处理

  204 服务器成功处理请求，但不需要返回实体内容

- 3xx 重定向，需要进一步操作完成请求

  - 301永久重定向

  - 302 临时重定向

  - 304 未修改。

    自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。

    对客户端有缓存情况下服务端的一种响应。客户端在请求一个文件的时候，发现自己缓存的文件有 Last Modified ，那么在请求中会包含 If Modified Since ，这个时间就是缓存文件的 Last Modified 。因此，**如果请求中包含 If Modified Since，就说明已经有缓存在客户端。服务端只要判断这个时间和当前请求的文件的修改时间就可以确定是返回 304 还是 200** 。

- 4xx 客户端错误，请求包含语法错误或无法完成请求

  400 请求参数有误，包含语法错误，无法被服务器解析

  401 请求要求用户的身份认证

  403 服务器理解请求客户端的请求，但是拒绝执行此请求

  404 请求资源不存在

  405 请求方法不允许

- 5xx 服务端处理请求过程中有错误

  500 Internal Server Error 服务器不知道如何处理，服务器内部错误
  
  504 Gateway Timeout 网关超时。扮演网关或者代理的服务器无法在规定的时间内获得想要的响应。

## Cookie与Session机制

Cookies是服务器在本地机器上存储的小段文本并随每一个请求发送至同一个服务器。网络服务器用HTTP头向客户端发送cookies，在客户终端，浏览器解析这些cookies并将它们保存为一个本地文件，它会自动将同一服务器的任何请求缚上这些cookies 。cookie机制采用的是在客户端保持状态的方案。它是在用户端的会话状态的存贮机制，他需要用户打开客户端的cookie支持。

cookie的内容主要包括：**名字，值，过期时间，路径和域。**路径与域一起构成cookie的作用范围。若不设置过期时间，则表示这个cookie的生命期为浏览器会话期间，关闭浏览器窗口，cookie就消失。这种生命期为浏览器会话期的cookie被称为会话cookie。会话cookie一般不存储在硬盘上而是保存在内存里，当然这种行为并不是规范规定的。若设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie仍然有效直到超过设定的过期时间。



session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构来保存信息。

当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了一个session标识（称为session id），如果已包含则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。

两者不同？

1. 存取方式不同。Cookie中只能保管ASCII字符串，Session中能够存取任何类型的数据,也能够直接保管Java Bean乃至任何Java类,对象。能够把Session看做是一个Java容器类。

2. 隐私策略的不同。Cookie对客户端可见，隐私信息需要加密。session对客户端透明，不存在敏感信息泄露风险。

3. 服务器压力不同。Session是保管在服务器端的，每个用户都会产生一个Session，耗费内存。Cookie保管在客户端，不占用服务器资源。

4. 浏览器支持的不同。假如客户端禁用了Cookie，需要运用Session以及URL地址重写。需要注意的是一切的用到Session程序的URL都要进行URL地址重写，否则Session会话跟踪还会失效。

5. 跨域支持上的不同

   Cookie支持跨域名访问，例如将domain属性设置为“.biaodianfu.com”，则以“.biaodianfu.com”为后缀的一切域名均能够访问该Cookie。跨域名Cookie如今被普遍用在网络中，例如Google、Baidu、Sina等。而Session则不会支持跨域名访问。Session仅在他所在的域名内有效。

https://juejin.im/entry/5766c29d6be3ff006a31b84e





[304介绍](https://blog.csdn.net/huwei2003/article/details/70139062)

[探究！一个数据包在网络中的心路历程](https://mp.weixin.qq.com/s/iSZp41SRmh5b2bXIvzemIw)

