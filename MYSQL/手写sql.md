# 手写sql

### 学生表、课程表、成绩表。1）查每个学生的平均成绩和学号；2）查询总分大于500分的学生的姓名和总分。

1. select 学生表.sno,avg(grade) 
       from 学生表,成绩表
       where 学生表.sno=成绩表.sno
       group by 学生表.Sno

2. select sname,sum(grade) from 学生表，成绩表 where 

   学生表.sno=成绩表.sno group by sno having sum(grade)>500

