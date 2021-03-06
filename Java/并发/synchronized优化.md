# synchronized优化

## 管程实现

synchronized 同步代码块的语义底层是基于对象内部的监视器锁（monitor），分别是使用 monitorenter 和 monitorexit 指令完成。monitorenter 指令在编译为字节码后插入到同步代码块的开始位置，monitorexit 指令在编译为字节码后插入到方法结束处和异常处。JVM 要保证每个 monitorenter 必须有对应的 moniorexit。

monitorenter：每个对象都有一个监视器锁（monitor），当 monitor 被某个线程占用时就会处于锁定状态，线程执行 monitorenter 指令时尝试获得 monitor 的所有权，即尝试获取对象的锁。过程如下：

 	1. 如果 monitor 的进入数为0，则该线程进入 monitor，然后将进入数设置为1，该线程即为 monitor 的所有者；
 	2. 如果线程已经占有monitor，只是重新进入，则monitor的进入数+1；
 	3. 如果其他线程已经占用 monitor，则该线程处于阻塞状态，直至 monitor 的进入数为0，再重新尝试获得 monitor 的所有权




在 HotSpot JVM 中，monitor 由 ObjectMonitor 实现，其主要数据结构如下：
> ObjectMonitor() {
...
    _count        = 0;      // 记录个数
    _owner        = NULL;   // 持有monitor的线程
    _WaitSet      = NULL;   // 处于wait状态的线程，会被加入到_WaitSet
    _EntryList    = NULL ;  // 处于等待锁block状态的线程，会被加入到该列表
    ...
    }



ObjectMonitor 中有两个队列，\_WaitSet 和 _EntryList，用来保存 ObjectWaiter 对象列表（每个等待锁的线程都会被封装成 ObjectWaiter 对象），\_owner 指向持有 ObjectMonitor 对象的线程。
 	1. 当多个线程同时访问一段同步代码时，首先会进入 _EntryList，等待锁处于阻塞状态。
 	2. 当线程获取到对象的 monitor 后进入 The Owner 区域，并把 ObjectMonitor 中的 _owner 变量设置为当前线程，同时 monitor 中的计数器 count 加1。
 	3. 若线程调用 wait() 方法，将释放当前持有的 monitor，_owner 变量恢复为 null，count 减1，同时该线程进入 _WaitSet 集合中等待被唤醒，处于 waiting 状态。
 	4. 若当前线程执行完毕，将释放 monitor 并复位变量的值，以便其他线程进入获取 monitor。



## 对象头结构

**锁优化**在 JDK1.6 之后，出现了各种锁优化技术，如**轻量级锁、偏向锁、适应性自旋、锁粗化、锁消除**等。

对象头结构分为两部分：

1. Mark Word存储对象自身运行时数据，如下表，包括**hash code，gc分代年龄，锁标志，是否偏向锁，线程id等**：

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/对象头结构.png)

2. 存储指向方法区对象类型数据的指针，如果是数组对象的话，额外会存储数组的长度



## 重量级锁
monitor 监视器锁本质上是依赖操作系统的 Mutex Lock 互斥量 来实现的，我们一般称之为重量级锁。
OS实现线程切换需要**从用户态切换到内核态**，成本高耗时。
重量级锁的锁标志位为'10'，指针指向的是 monitor 对象的起始地址

## 轻量级锁
在没有多线程竞争的前提下，减少传统的重量级锁使用OS的互斥量而带来的性能消耗
轻量级锁提升性能的经验依据是：**对于绝大部分锁，在整个同步周期内都是不存在竞争的**。如果没有竞争，轻量级锁就可以使用 **CAS 操作**避免互斥量的开销，从而提升效率。

### 轻量级锁加锁过程：
1.线程进入同步代码块，jvm在**当前线程栈桢中建立锁记录lock record**，存储**锁对象当前Mark word的拷贝**，owner指向对象对象的mark word.

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/轻量级锁加锁过程.png)

2.jvm通过**cas操作尝试将对象头中的 Mark Word 更新为指向 Lock Record 的指针**

3.如果更新成功，那么这个线程就拥有了该对象的锁，对象的 Mark Word 的锁状态为轻量级锁00。此时线程堆栈与对象头的状态如图所示： 

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/轻量级锁加锁过程2.png)

4.如果更新失败，JVM 首先检查**对象的 Mark Word 是否指向当前线程的栈帧**
如果是，就说明当前线程已经拥有了该对象的锁，那就可以直接进入同步代码块继续执行
如果不是，就说明这个锁对象已经被其他的线程抢占了，当前线程会尝试自旋一定次数来获取锁。如果自旋一定次数 CAS 操作仍没有成功，那么轻量级锁就要升级为重量级锁（锁的标志位转变为'10'），Mark Word 中存储的就是指向重量级锁的指针，后面等待锁的线程也就进入阻塞状态

## 偏向锁

**轻量级锁是在无多线程竞争的情况下，使用 CAS 操作去消除互斥量；偏向锁是在无多线程竞争的情况下，将这个同步都消除掉。**

偏向锁提升性能的经验依据是：**对于绝大部分锁，在整个同步周期内不仅不存在竞争，而且总由同一线程多次获得**。偏向锁会偏向第一个获得它的线程，如果接下来的执行过程中，该锁没有被其他线程获取，则持有偏向锁的线程不需要再进行同步。这使得线程获取锁的代价更低。

**偏向锁的获取过程：** 

1、线程执行同步块，锁对象第一次被获取的时候，JVM 会将锁对象的 Mark Word 中的锁状态设置为偏向锁（锁标志位为'01'，是否偏向的标志位为'1'），同时通过 CAS 操作在 Mark Word 中记录获取到这个锁的线程的 ThreadID

2、如果 CAS 操作成功。持有偏向锁的线程每次进入和退出同步块时，只需测试一下 Mark Word 里是否存储着当前线程的 ThreadID。如果是，则表示线程已经获得了锁，而不需要额外花费 CAS 操作加锁和解锁

3、如果不是，则通过CAS操作竞争锁，竞争成功，则将 Mark Word 的 ThreadID 替换为当前线程的 ThreadID



## 其他优化

### 1、适应性自旋

**自旋锁**：互斥同步时，挂起和恢复线程都需要切换到内核态完成，这对性能并发带来了不少的压力。同时在许多应用上，共享数据的锁定状态只会持续很短的一段时间，为了这段较短的时间而去挂起和恢复线程并不值得。那么如果有多个线程同时并行执行，可以让后面请求锁的线程通过自旋（CPU忙循环执行空指令）的方式稍等一会儿，看看持有锁的线程是否会很快的释放锁，这样就不需要放弃 CPU 的执行时间了。

**适应性自旋**：在轻量级锁获取过程中，线程执行 CAS 操作失败时，需要通过自旋来获取重量级锁。如果锁被占用的时间比较短，那么自旋等待的效果就会比较好，而如果锁占用的时间很长，自旋的线程则会白白浪费 CPU 资源。解决这个问题的最简答的办法就是：指定自旋的次数，如果在限定次数内还没获取到锁（例如10次），就按传统的方式挂起线程进入阻塞状态。JDK1.6 之后引入了自适应性自旋的方式，如果在同一锁对象上，一线程自旋等待刚刚成功获得锁，并且持有锁的线程正在运行中，那么 JVM 会认为这次自旋也有可能再次成功获得锁，进而允许自旋等待相对更长的时间（例如100次）。另一方面，如果某个锁自旋很少成功获得，那么以后要获得这个锁时将省略自旋过程，以避免浪费 CPU。

### 2、锁消除

### 3、锁粗化





参考

《Java并发编程的艺术》

https://nicky-chen.github.io/2018/05/14/synchronized-principle/

https://juejin.im/entry/5acde01a51882555867fc924