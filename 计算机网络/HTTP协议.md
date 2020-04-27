# Http协议



## HTTP介绍

**请求消息**

请求消息格式：请求行（request line）、消息头（header）、空行和消息体四个部分

- 请求行（起始行）：方法，URL地址，协议版本
- 消息头Headers：如Host,User-Agent,Accept,Accept-Language,Accept-Encoding,Connection,Content-Type,Content-Length
- 消息体Body：不是所有的请求都有一个 body：例如获取资源的请求，GET，HEAD，DELETE 和 OPTIONS，通常它们不需要 body。

**响应消息**

HTTP响应格式：状态行、消息头、空行和消息体。

- 状态行：版本，状态码，状态文本  如：HTTP/1.1 404 Not Found
- 消息头Headers：同上
- 消息体Body

**HTTP协议请求方法**：options,head,get,post,put,delete,trace,connect


```
http请求头:
- Accept：浏览器可接受的MIME类型。
- Accept-Charset：浏览器可接受的字符集。
- Accept-Encoding：浏览器能够进行解码的数据编码方式，比如gzip。Servlet能够向支持gzip的浏览器返回经gzip编码的HTML页面。许多情形下这可以减少5到10倍的下载时间。
- Accept-Language：浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时要用到。
- Authorization：授权信息，通常出现在对服务器发送的WWW-Authenticate头的应答中。
- Connection：表示是否需要持久连接。
- Content-Length：表示请求消息正文的长度。
- Cookie：这是最重要的请求头信息之一
- From：请求发送者的email地址，由一些特殊的Web客户程序使用，浏览器不会用到它。
- Host：初始URL中的主机和端口。
- If-Modified-Since：只有当所请求的内容在指定的日期之后又经过修改才返回它，否则返回304“Not Modified”应答。
- Pragma：指定“no-cache”值表示服务器必须返回一个刷新后的文档，即使它是代理服务器而且已经有了页面的本地拷贝。
- Referer：包含一个URL，用户从该URL代表的页面出发访问当前请求的页面。
- User-Agent：浏览器类型，如果Servlet返回的内容与浏览器类型有关则该值非常有用。
- Content-Type：字符的编码，1.0版规定，头信息必须是 ASCII 码，后面的数据可以是任何格式。因此，服务器回应的时候，必须告诉客户端，数据是什么格式，这就是Content-Type字段的作用。Content-Type: text/html; charset=utf-8

```



https://www.ruanyifeng.com/blog/2016/08/http.html



## HTTPS介绍

**HTTPS**（Hypertext Transfer Protocol Secure：超文本传输安全协议）是一种透过计算机网络进行安全通信的传输协议。HTTPS 经由 HTTP 进行通信，但利用 SSL/TLS 来加密数据包。加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。

- 对称加密：密钥只有一个，加密解密为同一个密码，且加解密速度快，典型的对称加密算法有DES、AES等；
- 非对称加密：密钥成对出现（且根据公钥无法推知私钥，根据私钥也无法推知公钥），加密解密使用不同密钥（公钥加密需要私钥解密，私钥加密需要公钥解密），相对对称加密速度较慢，典型的非对称加密算法有RSA、DSA等。

HTTPS 默认工作在 TCP 协议443端口，它的工作流程一般如以下方式：

- 1、TCP 三次同步握手
- 2、客户端验证服务器数字证书
- 3、DH 算法协商对称加密算法的密钥、hash 算法的密钥
- 4、SSL 安全加密隧道协商完成
- 5、网页以加密的方式传输，用协商的对称加密算法和密钥加密，保证数据机密性；用协商的hash算法进行数据完整性保护，保证数据不被篡改。

## HTTP和HTTPS区别

- HTTPS需要申请CA证书
- HTTP协议运行在TCP之上，传输内容明文，HTTPS运行在SSL/TLS之上，SSL/TLS运行在TCP之上，传输内经过加密
- 端口不通，HTTP80，HTTPS443
- HTTPS可以有效的防止运营商劫持



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

  400 Bad Request 请求参数有误，包含语法错误，无法被服务器解析

  401 Unautorized 请求要求用户的身份认证

  403 Forbidden 服务器理解请求客户端的请求，但是拒绝执行此请求

  404 Not Found 请求资源不存在

  405 请求方法不允许

- 5xx 服务端处理请求过程中有错误

  500 Internal Server Error 服务器不知道如何处理，服务器内部错误
  
  504 Gateway Timeout 网关超时。扮演网关或者代理的服务器无法在规定的时间内获得想要的响应。

## HTTP1.0 / 1.1 / 2.0都有什么不同

优化方向：**带宽和延迟**。带宽已经不是瓶颈，延迟分为三个点：**浏览器阻塞HOL，DNS查询，建立TCP连接**

**HTTP1.0**

- 没有keep-alive机制（Connection: KeepAlive），无法复用tcp连接，每次请求都需要和服务器三次握手建立tcp连接，慢启动，完成请求后断开连接。三次握手在高延迟的场景下影响较明显，慢启动则对文件类大请求影响较大。
- 浏览器阻塞，Head of Line (HOL) Blocking。浏览器会因为一些原因阻塞请求。浏览器对于同一个域名，同时只能有 4 个连接（这个根据浏览器内核不同可能会有所差异），超过浏览器最大连接数限制，后续请求就会被阻塞。请求队列的第一个请求因为服务器正忙（或请求格式问题等其他原因），导致后面的请求被阻塞。
- 缓存：在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准

> **队头阻塞**（**Head-of-line blocking**或缩写为**HOL blocking**）在计算机网络的范畴中是一种性能受限的现象。它的原因是一列的第一个数据包（队头）受阻而导致整列数据包受阻。例如它有可能在缓存式输入的交换机中出现，有可能因为传输顺序错乱而出现，亦有可能在HTTP流水线中有多个请求的情况下出现。

**HTTP1.1**

- **长连接：KeepAlive机制**，支持复用tcp连接，通过header中的connection是close或者Keep-Alive控制

- **流水线：Pipeline机制**。HTTP/1.1的持续连接有非流水线方式和流水线方式 。流水线方式是客户在收到HTTP的响应报文之前就能接着发送新的请求报文。与之相对应的非流水线方式是客户在收到前一个响应后才能发送下一个请求。

- **缓存处理**：HTTP1.1引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略

- **Host头处理**：客户端请求的头信息新增了`Host`字段，用来指定服务器的域名，有了`Host`字段，就可以将请求发往同一台服务器上的不同网站，为虚拟主机的兴起打下了基础。

  在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）

- **带宽优化及网络连接的使用**：HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

- **错误状态响应码** :在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

**HTTP2.0**

- **多路复用**，而非有序并阻塞的。针对HTTP高延迟的问题，SPDY优雅的采取了多路复用（multiplexing）。多路复用通过多个请求stream共享一个tcp连接的方式，**解决了HOL blocking的问题**，降低了延迟同时提高了带宽的利用率。每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。
- **首部压缩**。HTTP 协议不带有状态，每次请求都必须附上所有信息。所以，请求的很多字段都是重复的，比如`Cookie`和`User Agent`，一模一样的内容，每次请求都必须附带，这会浪费很多带宽，也影响速度。一方面，头信息使用`gzip`或`compress`压缩后再发送；另一方面，客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段了，只发送索引号，这样就提高速度了。
- **二进制格式**。HTTP/1.1 版的头信息肯定是文本（ASCII编码），数据体可以是文本，也可以是二进制。HTTP/2 则是一个彻底的二进制协议，头信息和数据体都是二进制，并且统称为"帧"（frame）：头信息帧和数据帧。
- **服务器推送**。HTTP/2 允许服务器未经请求，主动向客户端发送资源，这叫做服务器推送（server push）。



https://juejin.im/entry/5981c5df518825359a2b9476



## TCP/IP的四层结构，协议

|            | 协议                 |      |
| ---------- | -------------------- | ---- |
| 应用层     | ping,dns,ospf,telnet |      |
| 传输层     | tcp,udp              |      |
| 网络层     | icmp,ip              |      |
| 数据链路层 | arp,rarp             |      |







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





[javaguide](https://github.com/Snailclimb/JavaGuide/blob/master/docs/network/计算机网络.md)

[304介绍](https://blog.csdn.net/huwei2003/article/details/70139062)

[探究！一个数据包在网络中的心路历程](https://mp.weixin.qq.com/s/iSZp41SRmh5b2bXIvzemIw)

