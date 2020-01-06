# HashMap与ConcurrentHashMap

## 1. HashMap的put、get操作

### put操作

判断数组如果为空，初始化。key如果为空，put空值

根据Key的哈希值，通过扰动函数计算hash，定位到Entry数组下标桶。

如果对应数组下标已经存在元素，key相同且哈希值相同，（即key已经存在）替换新值，返回旧值。

如果对应数组下标不存在元素，或者数组下标存在key不相同的元素，（即Key不存在）：插入新元素

插入新元素注意：判断数组个数是否达到阈值，扩容。

扩容过程：声明两倍大小数组，遍历原数组所有元素，重新哈希定位，采用头插法插入新位置。

```java
  void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```

### get操作

根据Key计算出hash值，定位到数组桶中，判断该位置是否链表。不是链表就根据key和hash值是否相等判断，是则遍历链表找到Key和hash值相等元素返回。

## 2.HashMap多线程会可能出现什么问题？能讲下为什么会出现死循环么？

https://juejin.im/post/5a66a08d5188253dc3321da0

https://coolshell.cn/articles/9606.html

Jdk1.8在链表的阶段，头插法改为了尾插法，没有了死循环问题。

## 3. HashMap 1.7和1.8有什么区别

1. 1.7链表，1.8链表或红黑树
2.  链表部分时，1.7头插法，1.8尾插法
3. 计算hash值的扰动函数实现变化



## 4. ConcurrentHashMap的put操作

核心是segments数组，内部类Segment继承ReentrantLock，里面存放HashEntry数组。

```java
/**
 * Segment 数组，存放数据时首先需要定位到具体的 Segment 中。
 */
final Segment<K,V>[] segments;

static final class Segment<K,V> extends ReentrantLock implements Serializable {
      
       // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
       transient volatile HashEntry<K,V>[] table;
      
}
```

通过 key 定位到 Segment





