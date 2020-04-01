# Java基础类面试题小结

## java的反射机制，怎么实现的？

## Java 到底是值传递还是引用传递？

https://www.zhihu.com/question/31203609



## cookie与session机制

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



## 深拷贝和浅拷贝是什么？



## 如何重写hashcode和equals

如何重写equals()方法？

1. 自反性：A.equals(A)要返回true.
2. 对称性：如果A.equals(B)返回true, 则B.equals(A)也要返回true.
3. 传递性：如果A.equals(B)为true, B.equals(C)为true, 则A.equals(C)也要为true. 说白了就是 A = B , B = C , 那么A = C.
4. 一致性：只要A,B对象的状态没有改变，A.equals(B)必须始终返回true.



如何重写hashCode()方法？

如果你重写了equals()方法，那么一定要记得重写hashCode()方法



## jdk动态代理和cglib动态代理区别

jdk : 利用拦截器(拦截器必须实现InvocationHanlder)加上反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。JDK动态代理只能对实现了接口的类生成代理，实现InvocationHandler ，使用Proxy.newProxyInstance产生代理对象，不能针对类。

cglib : 利用ASM开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。CGLIB是针对类实现代理，主要是对指定的类生成一个子类，并覆盖其中方法实现增强，但是因为采用的是继承，所以该类或方法最好不要声明成final，对于final类或方法，是无法继承的。

## String、StringBuffer、StringBuilder区别

String 字符串常量
StringBuffer 字符串变量（线程安全）
StringBuilder 字符串变量（非线程安全）

String 是不可变的对象, 因此在每次对 String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象。StringBuffer 对象本身进行操作，而不是生成新的对象，再改变对象引用。

## java io使用了哪些设计模式

装饰器模式，适配器模式



## AQS?

AQS 全称是 AbstractQueuedSynchronizer，顾名思义，是一个用来构建锁和同步器的框架，它底层用了 CAS 技术来保证操作的原子性，同时利用 FIFO 队列实现线程间的锁竞争，将基础的同步相关抽象细节放在 AQS，这也是 ReentrantLock、CountDownLatch 等同步工具实现同步的底层实现机制。



## 如何用zk实现分布式锁？

