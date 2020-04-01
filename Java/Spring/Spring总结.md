# Spring总结

IOC 控制反转：



DI依赖注入，实现控制反转的一种方式。

Bean&Context概念



AOP：在每个切面上完成通用的功能。如通用参数校验。



Spring框架组件：

核心的：

Core - 所有组件实现的核心

Beans，Context实现IOC,DI的基础

AOP：实现面向切面编程

Web:包括MVC



AOP如何实现的？

通过代理模式，在调用对象某个方法时，实现插入的切面逻辑。

实现方式：动态代理，也叫运行时增强。如jdk代理，cglib代理。还有静态代理，编译或者类加载时织入，如aspectj。了解具体注解和使用方式。



placeHolder动态替换：替换发生时间，bean



事务传播类型：

required supports 

## SpringAOP的实现

## Spring容器初始化的流程？bean的生命周期？

## SpringMVC的执行流程？Spring的事务控制？

## Spring常见的注解？自定义注解的实现









