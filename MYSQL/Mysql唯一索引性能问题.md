# Mysql唯一索引性能问题 - change buffer

Q：如果业务保证了唯一。从性能的角度考虑，选择唯一索引还是普通索引呢？

## 查询过程：

> 唯一索引：查找到第一个满足的条件就停止检索
> 普通索引：直到不满足条件时停止检索
> 性能差异：极低可忽略

InnoDB 的数据是按**数据页**为单位来读写的。也就是说，当需要读一条记录的时候，并不是将这个记录本身从磁盘读出来，而是以页为单位，将其整体读入内存。在 InnoDB 中，每个数据页的大小默认是 16KB。

因为引擎是按页读写的，所以说，当找到一条记录的时候，它所在的数据页就都在内存里了。那么，对于普通索引来说，要多做的那一次“查找和判断下一条记录”的操作，就只需要一次指针寻找和一次计算。如果刚好是该数据页最后一条记录，必须读下一数据页会稍微复杂。但其概率很小，如对于整型字段，一个数据页可以放仅千个key。

## 更新过程：

change buffer介绍：

> 当需要**更新一个数据页**时，如果数据页在内存中就直接更新，而**如果这个数据页还没有在内存中的话，在不影响数据一致性的前提下，InooDB 会将这些更新操作缓存在 change buffer 中，这样就不需要从磁盘中读入这个数据页了。**在下次查询需要访问这个数据页的时候，将数据页读入内存，然后执行 change buffer 中与这个页有关的操作。通过这种方式就能保证这个数据逻辑的正确性。
>

将 change buffer 中的操作应用到原数据页，得到最新结果的过程称为 merge。除了**访问这个数据页**会触发 merge 外，系统有后台线程会定期 merge。在数据库正常关闭（shutdown）的过程中，也会执行 merge 操作。

change buffer优点：

- **减少IO，提升执行速度**：如果能够将更新操作先记录在 change buffer，减少读磁盘IO，语句的执行速度会得到明显的提升。
- **提高内存利用率**：数据读入内存是需要占用 buffer pool 的，所以这种方式还能够避免占用内存，提高内存利用率。

**唯一索引更新会先判断唯一性约束，必须要将数据页读入内存才能判断。因此不会使用change buffer。**

change buffer 用的是 buffer pool 里的内存，因此不能无限增大。change buffer 的大小，可以通过参数 `innodb_change_buffer_max_size` 来动态设置。这个参数设置为 50 的时候，表示 change buffer 的大小最多只能占用 buffer pool 的 50%。

执行更新插入操作时，普通索引和唯一索引区别

- 如果记录要更新的目标页在内存中，影响不大
- 如果记录要更新的目标页不在内存中
  - 唯一索引：需要**将数据页读入内存**。判断到没有冲突，插入这个值。
  - 普通索引：将更新记录在 change buffer，语句执行就结束了。

将数据从磁盘读入内存涉及随机 IO 的访问，是数据库里面成本最高的操作之一。change buffer 因为减少了随机磁盘访问，所以对更新性能的提升是会很明显的。

## change buffer 的使用场景

Q：普通索引的所有场景，使用 change buffer 都可以起到加速作用吗？

merge 的时候是真正进行数据更新的时刻，而 change buffer 的主要目的就是将记录的变更动作缓存下来，所以在一个数据页做 merge 之前，change buffer 记录的变更越多（也就是这个页面上要更新的次数越多），收益就越大。

- 对于**写多读少**的业务来说，页面在写完以后马上被访问到的概率比较小，此时 change buffer 的使用效果最好。这种业务模型常见的就是**账单类、日志类**的系统。
- 对于**更新后马上会查询**的业务来说，随机访问 IO 的次数不会减少，反而增加了 change buffer 的维护代价。

## change buffer 与redo log

Mysql中WAL技术，即`Write-aheading Logging`，关键点就是先写日志再写磁盘。如果每条更新都要写磁盘，磁盘找到记录再更新整个IO、查找成本都很高。所以更新操作时，先把记录写到redo log，并更新内存就算更新完成。在适当的时候将记录更新到磁盘。

如何理解change buffer 和 redo log的顺序？

假设语句`insert into t(id,k) values(id1,k1),(id2,k2);` 其中k1所在数据页在内存中，k2数据页不在更新操作：

1. k1对应Page 1在内存中直接更新内存
2. k2对应Page 2 没有在内存中，就在内存的 change buffer 区域，记录下“我要往 Page 2 插入一行”这个信息
3. 将上述两个动作记入 redo log 中

上述做完事务完成，更新的成本很低，写了两处内存，一处磁盘（两次操作合在一起写，还是顺序写）

如果后续读操作，假设此时内存中数据页还在：

1. 读page1直接从内存中返回。此处可能会疑惑，wal后是否一定要读盘，是否一定要从redo log里将数据更新后才返回。其实不需要，直接从内存返回结果
2. 读 Page 2 的时候，需要把 Page 2 从磁盘读入内存中，然后应用 change buffer 里面的操作日志，生成一个正确的版本并返回结果。

Q:如果某次写入使用了 change buffer 机制，之后主机异常重启，是否会丢失 change buffer 和数据?

不会丢失。虽然是只更新内存，但是在事务提交的时候，我们把 change buffer 的操作也记录到 redo log 里了，所以崩溃恢复的时候，change buffer 也能找回来。

## 总结

普通索引和唯一索引查询能力没差别，主要在**更新性能**影响。**如果所有的更新后面马上伴随了查询，应该关闭change buffer**。**change buffer主要适用于写多读少的场景。**如果碰上了大量插入数据慢、内存命中率低的时候，可以考虑唯一索引方向的排查思路。历史数据的归档库，可以考虑唯一索引改成普通索引。



参考：
《mysql实战45讲》丁奇