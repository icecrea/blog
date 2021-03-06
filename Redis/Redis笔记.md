# Redis

## 数据结构

String——字符串
Hash——字典
List——列表
Set——集合
Sorted Set——有序集合

## 过期键删除策略

如果一个键过期了，那么它什么时候会被删除呢？三种不同的删除策略：

- 定时删除：在设置键的过期时间的同时，创建一个定时器（timer），让定时器在键的过期时间来临时，立即执行对键的删除操作。即从设置key的Expire开始，就启动一个定时器，到时间就删除该key；这样会**对内存比较友好，但浪费CPU资源**
- 惰性删除：放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；如果没有过期，就返回该键。对**CPU友好，但是浪费内存资源**。
- 定期删除：每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。



