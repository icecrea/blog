## Java线程生命周期

线程与进程的区别：

- 线程是CPU调度的基本单位
- 进程是系统资源分配的基本单位

## 操作系统线程生命周期：

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/通用线程生命周期.png)



五态模型：初始状态、可运行、运行、休眠、终止。

休眠状态：运行态线程调用操作系统阻塞api（如阻塞读文件），或等待事件（如条件变量），线程状态转到休眠状态，释放cpu使用权。

注意此处阻塞式api指的是java里的bio，如以阻塞方式调用读文件，socket这些操作系统级的api。

## Java线程生命周期

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/java线程生命周期.png)

Java线程有六种状态，与操作系统线程的区别：

1. 将可运行，运行状态合并，这两个状态在操作系统层面调度，jvm不关心，jvm把线程调度交给操作系统处理
2. 细化了休眠状态，blocked，waiting，timed-waiting，只要java线程处于这三个状态之一，这个线程永远没有cpu的使用权
3. 线程调阻塞式api，操作系统层面会到休眠态，但jvm层面，java线程状态仍runnable。jvm不关心操作系统调度状态，在jvm看来，等待cpu（操作系统层可运行态）和等待io（操作系统休眠态）没有区别，都是等待资源，所以归为runnable

## Java线程的转换

### 1. Runnable <> Blocked  

   只有一种场景，线程等待synchronized隐式锁。拿到锁后又转换到Runnable

### 2. Runnbale <> Waiting 
       a.已经获取了synchronized隐式锁的线程调用object.wait()方法
       b.调用无参thread.join方法。等待期间主线程会转到waiting状态，
       c.LockSupport.park方法，当前线程会阻塞，从runnbale转到waiting。 LockSupport.unpark可唤醒目标线程，从waiting转到runnable

### 3. Runnbale <> Timed_Waiting 
       a. 超时参数的thread.sleep(time)
       b. 已经获取了synchronized隐式锁的线程调用带时间的object.wait(time)方法
       c. 调用带时间的thread.join(time)方法。等待期间主线程会转到waiting状态
       d. 带时间的LockSupport.parkNanos方法
       e. 带时间的LockSupport.parkUntil方法

### 4. New - > Runnable

   ​	Java线程刚创建为New状态。可以通过继承Thread重写run方法，或者实现Runnbale接口重写run方法来创建Thread对象。New状态线程不会被操作系统调度，调用start方法进入Runnable状态。

### 5. Runnable - > Terminated

   线程执行完run方法或异常退出都会转为Terminated状态。如果想主动终止run方法，可以通过调用线程的interrupt()方法。该方法仅仅是通知线程，线程可以执行一些后续操作，也可以无视通知。

   线程有两种方法收到通知：

   - 被动触发：线程a处于waiting、 timed-waiting状态，调用线程a的interupt方法，a状态会回到runnable状态，同时触发InterruptedException异常。
   - 主动检测：线程a处于Runnable状态并且未阻塞在IO操作上，需要a主动检测，可以通过isInterrupted()方法检测自己是否被中断。



## 创建多少线程合适？

分析业务是CPU密集型还是IO密集型。

- cpu密集型，理论上线程数=cpu核数，工程上常设置cpu核数+1
- io密集型，需要分析cpu计算与io操作的耗时比，最佳线程数 = cpu数 * （1 + io耗时 / cpu耗时）

注：此处只是基本思路，具体方案需要压测分析指标。



参考：

《Java并发编程实战》王宝令