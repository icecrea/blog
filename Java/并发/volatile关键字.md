# volatile关键字

并发编程中的三个概念：**原子性，可见性，有序性**

## volatile含义

1. 保证了变量的**可见性**。通过"**内存屏障**"实现部分“**有序性**”。
2. java5版本增强语义，修复缓存导致的可见性问题，通过**volatile的happens-before规则**

它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。

## volatile汇编指令lock

加入volatile关键字时，汇编代码会多出一个lock前缀指令。lock前缀指令实际上相当于一个**内存屏障**（也成内存栅栏），内存屏障会提供3个功能：

1. 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
2. 它会强制**将对缓存的修改操作立即写入内存**；
3. 如果是**写操作，它会导致其他CPU中对应的缓存行无效**。

## Happens-Before规则

Java使用happens-before的概念来阐述操作之间的**内存可见性**。在JMM中，如果**一个操作执行的结果需要对另一个操作可见**，那么这两个操作之间必须要存在happens-before关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。

- 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C

强调一个非常重要的概念：

**如果A happens-before B，JMM并不要求A一定要在B之前执行，仅仅要求前一个操作（执行的结果）对后一个操作可见**。如下代码为例

```java
int a = 1 ;      //A
int b = 2 ;      //B
int c = a + b;   //C
```

在单线程中，得出如下结论：

> 1. A happens-before B
> 2. B happens-before C
> 3. A happens-before C

其中1，2根据程序顺序性规则，而3根据传递性规则。但实际执行上B却可能在A之前执行。

该代码中操作A的执行结果不需要对操作B可见；而且重排序先B后A的执行结果，与先A后B按happens-before顺序执行的结果一致。在这种情况下，JMM会认为这种重排序并不非法（not illegal），允许这种重排序。



## volatile在JDK5如何增强语义

如下代码，线程A执行writer方法，线程B执行reader()。在Jdk1.5之前，x可能是42也可能是0。在1.5以上版本x=42。1.5之前存在cpu缓存导致的可见性问题。

```java
class VolatileExample {
  int x = 0;
  volatile boolean v = false;
  public void writer() {
    x = 42; // a 
    v = true; // b
  }
  public void reader() {
    if (v == true) { //c
      // 这里x会是多少呢？  // d
    }
  }
}
```

根据程序顺序性原则推出，a happens before b , c happens before d。

Java在单线程不影响执行结果的情况下，jvm可能对指令重排序。但是volatile修饰了v变量，它的“内存屏障”特性保证了执行到该行，前面的操作已经完成。所以在程序执行顺序中，x的赋值一定先于v的的赋值，a是先于b执行的。

jdk1.5增强了volatile的语义，增加了happens-before原则，对一个volatile变量的写，happens-before于任意后续对这个volatile变量的读。所以b happens before c。结合传递性不难得出：a happens before d。

再强调一下happens-before的语义，前一个操作的结果对后一个操作可见。所以此时a的结果对于d是可见的，x = 42。

而在jdk1.5之前，则没有b happens before c的规则，自然无法推出a happens before d。a的赋值操作对d无法保证是可见的。存在x写入到cpu缓存，未刷新到内存中的情况，此时则d处读取的值为0。jdk1.5后通过a appens before d保证了x的可见性，即x = 42。



## volatile的应用场景

### 1. 双重检查的单例模式

这里的问题重点在于new()操作，这是一个复合操作。

我们理解的new()操作：

1.分配一块内存 M； 2.在内存 M 上初始化 Singleton 对象； 3.然后 M 的地址赋值给 instance 变量。 

然而发生了重排序后，实际的new()操作：

1.分配一块内存 M；2.将 M 的地址赋值给 instance 变量；3.最后在内存 M 上初始化 Singleton 对象。

这是一个**先将地址赋值给变量，后在内存上初始化对象的过程。**假设线程A执行到c处，给instance变量赋值完但还没在内存上初始化对象，此时发生cpu切换，线程B执行，执行到a处判断instance != null，直接返回了instance，而此时实际上instance内存上还未初始化对象，产生异常。

而通过volatile的**内存屏障**功能，可以使上述过程正确执行，达到一个**禁止重排序**的效果。

```java
class Singleton{
    private volatile static Singleton instance = null;
     
    private Singleton() {}
     
    public static Singleton getInstance() {
        if(instance == null) { // a
            synchronized (Singleton.class) {
                if(instance == null) {
                    instance = new Singleton(); // c
                }
            }
        }
        return instance;
    }
}
```



参考：

《Java并发编程的艺术》

《Java并发编程实战》

