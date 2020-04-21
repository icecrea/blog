# Java基础类面试题小结

## java的反射机制，怎么实现的？

## Java 到底是值传递还是引用传递？

https://www.zhihu.com/question/31203609



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

