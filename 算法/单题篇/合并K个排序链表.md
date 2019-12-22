# 合并K个排序链表 

[git代码地址](https://github.com/icecrea/leetcode/blob/master/src/main/java/com/example/leetcode/linkedlist/LeetCode23_MergeKSortedLists.java)

## 题目描述

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/leetcode23.png)

## 基本思路

这道题属于双链表合并的进阶。理解这道题首先需要了解有序双链表合并的解法。

已知链表有序，使用两个指针指向两个链表，逐一比较大小移动指针。代码很简单如下所示。

```java
  public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(Integer.MIN_VALUE);
        ListNode cur = dummy;
        while (l1 != null && l2 != null) {
            if (l1.val < l2.val) {
                cur.next = l1;
                l1 = l1.next;
            } else {
                cur.next = l2;
                l2 = l2.next;
            }
            cur = cur.next;
        }
        cur.next = l1 == null ? l2 : l1;
        return dummy.next;
    }
```

两个有序链表合并，我们用到了两个指针。那么N个链表，显然无法保存N个指针去遍历和移动。所以需要换种思路，可以对链表逐一进行两两合并。但是这样会导致链表的重复遍历。此时可以用到归并排序中的思路：分治法。将链表对半拆分到单链表，再进行两两合并，这样就不会有重复遍历，详情见方法三。

同时，如果题目不要求空间复杂度，可以使用数组或集合来快速实现整体的排序，容易理解，见方法一。

扩展思路，双链表合并，每次需要比较计算两个指针大小。那么k个链表每次需要计算K个指针对应的节点大小。此时可以想到使用优先级队列，具体见方法二。



## 方法一：数组集合

不要求空间复杂度情况，可以借助数组集合，遍历所有的链表上的节点，放入集合中，进行排序。再从集合中将其取出，构造新链表

```java
   /**
     * 如果不限制空间复杂度，可以放到数组中排序，再拿出
     * 时间复杂度O(NlogN) = 遍历所有值 O(N) + 稳定排序算法O(NlongN) + 遍历创建新链表O(N)
     */
    public ListNode mergeKLists1(ListNode[] lists) {
        List<ListNode> allList = new ArrayList<>();
        for (ListNode node : lists) {
            while (node != null) {
                allList.add(node);
                node = node.next;
            }
        }
        allList.sort(new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                return o1.val - o2.val;
            }
        });
        ListNode dummy = new ListNode(0);
        ListNode cur = dummy;
        for (ListNode node : allList) {
            cur.next = node;
            cur = cur.next;
        }
        cur.next = null;
        return dummy.next;
    }
```



## 方法二：优先级队列

使用大小为链表长度的优先级队列，可以将优先级队列看成大小为k的小根堆。将k个链表的头节点全部加入小根堆中。当堆不为空，从堆中弹出堆顶，即最小值，并将当前节点的next指针指向该值，当前节点后移。此时当前节点指向了k个链表中最小的节点，判断其下一个节点是否为空，不为空则将下一个节点加入小根堆中。

```java
/**
     * 如果不限制空间复杂度，也可以使用优先级队列
     */
    public ListNode mergeKLists2(ListNode[] lists) {
        //空集合初始化优先级队列会抛出异常， 此处需要先判空
        if (lists == null || lists.length == 0) {
            return null;
        }
        PriorityQueue<ListNode> queue = new PriorityQueue<>(lists.length, new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                return o1.val - o2.val;
            }
        });
        ListNode dummy = new ListNode(0);
        ListNode cur = dummy;
        for (ListNode node : lists) {
            if (node != null) {
                queue.add(node);
            }
        }
        while (!queue.isEmpty()) {
            cur.next = queue.poll();
            cur = cur.next;
            if (cur.next != null) {
                queue.add(cur.next);
            }
        }
        return dummy.next;
    }
```

## 方法三：分治法

思路类似链表的归并排序，使用分治法，递归将链表拆分，直到每个都为单独链表，再将其两两合并。

这样避免了链表的重复遍历。

```java
/**
     * 分治法：递归拆分链表变成单独链表，再两两合并 思路类似链表的归并排序
     * 时间复杂度：O(Nlogk)，其中k 链表的数目
     */
    public ListNode mergeKLists(ListNode[] lists) {
        return partion(lists, 0, lists.length - 1);
    }

    public ListNode partion(ListNode[] lists, int start, int end) {
        //终止条件
        if (start == end) {
            return lists[start];
        }
        int mid = start + ((end - start) >> 1);
        ListNode l1 = partion(lists, start, mid);
        ListNode l2 = partion(lists, mid + 1, end);
        return mergeTwoLists(l1, l2);
    }

    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(Integer.MIN_VALUE);
        ListNode cur = dummy;
        while (l1 != null && l2 != null) {
            if (l1.val < l2.val) {
                cur.next = l1;
                l1 = l1.next;
            } else {
                cur.next = l2;
                l2 = l2.next;
            }
            cur = cur.next;
        }
        cur.next = l1 == null ? l2 : l1;
        return dummy.next;
    }
```
