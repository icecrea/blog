## JVM问题排查指令总结

#### 常用分析指令

1. 查看各个类的实例大小与个数
   `sudo -u tomcat jmap -histo pid`

2. DUMP内存，获取对应的内存快照
   `sudo -u tomcat jmap -dump:format=b,file=dumpxx.bin pid`

3. CPU飙高排查线程信息
   `top -Hp pid` 查看Java进程信息 p : 根据CPU使用百分比大小进行排序
   `printf "%x\n" 21742` 将CPU高的线程ID转换成十六进制
   `sudo -u tomcat jstack -l 21711 | grep 54ee` 查看线程栈中对应CPU高的线程信息

4. 统计进程状态个数
   `sudo -utomcat jstack 5020 |grep 'java.lang.Thread.State' | awk '{print $2,$3,$4,$5}' | sort | uniq -c`

   ```
   36 RUNNABLE
   3 TIMED_WAITING (on object monitor)
   13 TIMED_WAITING (parking)
   6 TIMED_WAITING (sleeping)
   3 WAITING (on object monitor)
   47 WAITING (parking)
   ```

------

#### MAT 使用简介

Shallow Heap ：一个对象内存的消耗大小，不包含对其他对象的引用；
Retained Heap ：是shallow Heap的总和，也就是该对象被GC之后所能回收的内存大小
在某一项上右键打开菜单选择 list objects ：
with incoming references 将列出哪些类引入该类；
with outgoing references 列出该类引用了哪些类