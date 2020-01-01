# Java多线程常用类总结：FutureTask、CountDownLatch、CyclicBarrier

## Future用法

使用线程池，常用的是ThreadPoolExecutor的execute()方法，但该方法无法返回结果。那如何获取结果？就用到了submit()方法和Future接口。

ThreadPoolExecutor的submit()方法继承于抽象父类AbstractExecutorService，父类中提供了三种实现，如下图。返回都是Future接口，但第一种情况没有返回值。

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/abstractexecutorservice公共方法.png)

Future接口如下图，可以查看任务是否取消或完成，取消任务。通过get()方法获取结果，支持超时机制。需注意get()方法为同步方法，若任务没执行完，线程会阻塞，进入WAITING状态。

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/Future接口方法.png)

## FutureTask用法

FutureTask是一个工具类，实现了Future和Runnable接口，有两个构造函数。

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/FutureTask工具类.png)

FutureTask实现了Runnable接口，所以可以将它作为任务提交给ThreadPoolExecutor执行或直接Thread执行。

FutureTask实现了Future接口，也能用来获得任务执行结果。

```java
// 创建FutureTask
FutureTask<Integer> futureTask
  = new FutureTask<>(()-> 1+2);
// 创建线程池
ExecutorService es = 
  Executors.newCachedThreadPool();
// 提交FutureTask 
es.submit(futureTask);
// 获取计算结果
Integer result = futureTask.get();

// 创建并启动线程
Thread T1 = new Thread(futureTask);
T1.start();
// 获取计算结果
Integer result = futureTask.get();
```

对于一些串行的任务，需要提高效率，可以分别构造FutureTask提交道线程池并行执行。

## CountDownLatch用法

有一个对账需求，需要查询两种类型账单，并将其结果对比入库。想优化这个需求，可以将对账操作从串行改为并行。但问题在于主线程如何等待查询账单的线程执行完毕？单线程情况可以通过thread.join()来等待线程的退出，而线程池中线程不会退出，我们可以通过CountDownLatch的计数器功能来实现。

创建CountDownLatch的时候设置初始值，通过latch.countDown()对计数器减一操作，通过latch.await()实现对计数器等于0的等待。

```java

// 创建2个线程的线程池
Executor executor = 
  Executors.newFixedThreadPool(2);
while(存在未对账订单){
  // 计数器初始化为2
  CountDownLatch latch = 
    new CountDownLatch(2);
  // 查询未对账订单
  executor.execute(()-> {
    pos = getPOrders();
    latch.countDown();
  });
  // 查询派送单
  executor.execute(()-> {
    dos = getDOrders();
    latch.countDown();
  });
  
  // 等待两个查询操作结束
  latch.await();

  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
}
```

## CyclicBarrier用法

上面的对账需求，其实还可以进一步优化。使用countdownlatch后，查询操作变成并行了，但是diff入库操作与查询操作是串行的，这两者没直接关联，可以将其也并行化。这里有两个难点：

- 我们需要让两个查询线程相互等待
- 当线程T1，T2都生产完一条数据时，还要通知线程T3进行对账入库操作。

这个时候我们可以使用CyclicBarrier，初始计数为2，构造函数传入一个回调函数。查出订单时，调用barrier.await()让计数器减一，同时等待计数器变成0。当T1T2均调用完barrier.await()，计数器减到0，此时T1T2可以执行下一条语句了，同时当计数器值为0会调用回调函数执行对账操作。

注意：

- CyclicBarrier有自动重置的功能，减到0的时候会重置为初始值，所以可以实现循环查询订单。

- CyclicBarrier的回调函数，是同步的，会由最后一个执行await()方法的线程执行。所以此处用一个单线程线程池进行对账入库的操作，实现异步。
- CountDownLatch主要用于一个线程等待多个线程的问题，而CyclicBarrier是一组线程相互等待，并且有回调函数，可以循环利用。

```java
   // 订单队列
Vector<P> pos;
// 派送单队列
Vector<D> dos;

// 执行回调的线程池
ExecutorService executor = Executors.newFixedThreadPool(1);
final CyclicBarrier barrier = new CyclicBarrier(2, () -> {
    executor.execute(() -> check());
});

void check() {
    P p = pos.remove(0);
    D d = dos.remove(0);
    // 执行对账操作
    diff = check(p, d);
    // 差异写入差异库
    save(diff);
}

// 创建2个线程的线程池
ExecutorService workPool = Executors.newFixedThreadPool(2);
while (存在未对账订单) {
    // 查询未对账订单
    workPool.submit(() -> {
        // 查询订单库
        pos.add(getPOrders());
        // 等待
        barrier.await();
    });

    // 查询派送单
    workPool.submit(() -> {
        // 查询运单库
        dos.add(getDOrders());
        // 等待
        barrier.await();
    });
}
```



参考：

《Java并发编程实战》王宝令