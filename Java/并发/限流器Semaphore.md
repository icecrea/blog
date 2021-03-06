# 限流器Semaphore

## 概念
Semaphore信号量模型，是一种通过维护计数器数值来控制并发数量的模型，Lock实现的互斥锁只允许一个线程访问临界区，而Semaphore允许有限多个线程访问临界区。常见于池化资源，连接池、线程池、对象池等。构造时传入初始值表示可以获取锁的线程数。

## 信号量模型
可以简单的概括为：一个计数器、一个等待队列、三个方法。 在信号量模型里，计数器和等待队列对外是透明的，只能通过信号量模型提供的三个方法访问它们，init()、acquire()、release()。

 -  init(): 设置计数器的初始值，初始化凭证数量。
 - acquire()：计数器的值减 1 ；如果**此时**计数器的值小于 0，则当前线程将被阻塞，放到等待队列之中，否则当前线程可以继续执行。
 - release()：计数器值加 1；如果此时计数器的值小于或者等于 0，则唤醒等待队列中的一个线程，并将其从等待队列中移除。

## 信号量基础使用

线程1/2同时执行方法，acquire() 是一个原子操作，只有一个线程访问。线程1信号量减1变成0，线程2再减1变成-1，线程2阻塞。线程1release()操作后计数为0，唤醒线程2，线程2得到执行的机会。

```java
static int count;
//初始化信号量
static final Semaphore s = new Semaphore(1);
//用信号量保证互斥    
static void addOne() {
  s.acquire();
  try {
    count+=1;
  } finally {
    s.release();
  }
}
```



## 信号量应用：N个线程顺序循环打印从0至100

> 通过N个线程顺序循环打印从0至100，如给定N=3。则输出: thread0: 0 thread1: 1 thread2: 2 thread0: 3 thread1: 4

这道题很考察多线程基础。主要在于N个线程循环打印。关键在于如何控制线程的执行顺序，即唤醒特定的线程。之前通过notify()唤醒一个，notifyAll()唤醒全部，但都无法选择特定的线程执行。那如何控制控制线程有序的打印呢？

这里可以用semaphore信号量实现。设置semaphore[]数组，每个线程持有本身的信号量以及上个线程的信号量。对于三个线程abc来说，他们的last信号量对应关系为：

a.last - > semaphores[2]

b.last - > semaphores[0]

c.last - > semaphores[1]

每个线程执行时，会抢占last信号量，如a抢占 semaphores[2]，同时释放当前信号量semaphores[0]，通知等待队列中的线程b，这时线程b才能抢占通过它的last信号量：semaphores[0]，同时再释放semaphores[1]，然后唤醒c线程。。依此循环执行。

```java
  int count = 0;
    Thread[] threads;
    Semaphore[] semaphores;

    public void print(int len) throws Exception {
        threads = new Thread[len];
        semaphores = new Semaphore[len];
        for (int i = 0; i < len; i++) {
            semaphores[i] = new Semaphore(1);
            if (i != len - 1) {
                semaphores[i].acquire();
            }
        }
        for (int i = 0; i < len; i++) {
            Semaphore lastSemphore = i == 0 ? semaphores[len - 1] : semaphores[i - 1];
            Semaphore cur = semaphores[i];
            threads[i] = new orderThread(lastSemphore, cur, "线程" + i);
            threads[i].start();
        }
    }


    class orderThread extends Thread {

        /**
         * 每一个线程都保存上一个线程对应的信号量
         */
        Semaphore last;

        /**
         * 保存当前线程的信号量
         */
        Semaphore cur;

        orderThread(Semaphore last, Semaphore cur, String threadName) {
            super(threadName);
            this.last = last;
            this.cur = cur;
        }

        /**
         * 争夺上个线程的信号量，同时释放当前线程的信号量。这样唤醒下一个线程去争夺当前线程的信号量。
         */
        @Override
        public void run() {
            while (count <= 100) {
                try {
                    last.acquire();
                    //需要判断，防止线程被唤醒的时候，此时已经超过了100
                    if (count <= 100) {
                        System.out.println(Thread.currentThread().getName() + " : " + count++);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    cur.release();
                }
            }
        }
    }

    @Test
    public void test() throws Exception {
        print(3);
        Thread.sleep(3000);
    }
```



## 信号量和管程有什么区别？

信号量可以用于限流，同时允许多个线程进入临界区，但不能同时唤醒多个线程起争抢锁，只能唤醒一个阻塞中的线程。并且没有condition概念，阻塞线程唤醒直接运行，不会检查临界条件是否仍满足。也是因为此考虑信号量模型只让一个线程被唤醒，防止线程安全问题。同时信号量实现阻塞队列很麻烦，需要实现condition逻辑。





参考：

《Java并发编程实战》

