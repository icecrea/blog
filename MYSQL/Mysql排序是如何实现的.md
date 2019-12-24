## Mysql排序是如何实现的？

### 背景

```
表定义
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
```

执行语句`select city,name,age from t where city='杭州' order by name limit 1000 ;` 里面用到了排序，它是如何执行的呢？

### 全字段排序策略

通过explain分析，看到Extra 这个字段有“Using filesort”，表示的就是需要排序，MySQL 会给每个线程分配一块内存用于排序，称为 `sort_buffer`。

通常情况下，执行流程为：

- 初始化sort_buffer，放入city,name,age三个字段，
- 从索引 city 找到第一个满足 city=’杭州’条件的主键 id，回表取出整行，取 name、city、age 三个字段的值，存入 sort_buffer 中。重复该步骤取下一条记录直到不满足查询条件
- 对sort_buffer中数据按name做快速排序
- 按排序结果取前1000行返回客户端

“按 name 排序”这个动作，如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。否则会使用磁盘临时文件辅助排序。

### rowid排序策略

对全字段排序来说，如果查询字段太多，sort_buffer不足，分成临时文件排序会导致性能很差。

如果 MySQL 认为排序的单行长度太大会怎么做呢？可以通过修改参数，让mysql采取另一种算法

```sql
SET max_length_for_sort_data = 16;
```

`max_length_for_sort_data`，是 MySQL 中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，MySQL 就认为单行太大，要换一个算法。city、name、age 这三个字段的定义总长度是 36，可以设置为16来验证变化。

执行流程为：

- 初始化sort_buffer，放入name，id两个字段，
- 从索引 city 找到第一个满足 city=’杭州’条件的主键 id，回表取出整行，取 name、id 的值，存入 sort_buffer 中。重复该步骤取下一条记录直到不满足查询条件
- 对sort_buffer中数据按name做快速排序
- 按排序结果取前1000行，并按照id的值回到原表取city,name,age三个字段给客户端

可以看出，比全字段排序多了一次回表，从sort_buffer排序后，还需要再回主键索引取其它需要的值。

### 全字段排序与rowid排序比较

MySQL 的一个设计思想：**如果内存够，就要多利用内存，尽量减少磁盘访问。**

对于 **InnoDB 表**来说，rowid 排序会要求回表多造成磁盘读，因此不会被优先选择。
可以想到，对于内存表，回表过程只是简单地根据数据行的位置，直接访问内存得到数据，不会导致多访问磁盘。

当然，并不是所有order by 都需要排序操作，MySQL 之所以需要生成临时表，并且在临时表上做排序操作，其原因是原来的数据都是无序的。对本文例子来说，如果从 city 这个索引上取出来的行，天然就是按照 name 递增排序的话还需要排序呢？

假如有(city, name)联合索引，在这个索引里面，我们依然可以用树搜索的方式定位到第一个满足 city=’杭州’的记录，并且额外确保了，接下来按顺序取“下一条记录”的遍历过程中，只要 city 的值是杭州，name 的值就一定是有序的。查询过程不需要临时表，也不需要排序。Extra 字段中没有 Using filesort 了，也就是不需要排序了。



参考：
《mysql实战45讲》丁奇