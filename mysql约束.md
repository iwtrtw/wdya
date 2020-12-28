### 约束

主要是防止非法数据进入到表中，确保数据的正确性和一致性（统称数据完整性），约束可以防止一个表被删除

+ NOT NULL：非空约束；当插入数据时，必须为列提供数据

+ UNIQUE：唯一约束；列值不能重复，但可以为null

+ Primary Key：主键约束；列值不能重复，且不可以为null

  一张表最多只能有一个主键，但可以有多个unique约束

+ Foreign Key：外键约束

+ CHECK：检查约束(mysql只检查而不约束)





+ 表注释 comment

> create table student(id int) comment '学生表'
>
> select * from information_schema.TABLES where table_name = 'student'  //查看表信息

+ 列注释

> create table student(
> id int not null primary key comment '学号' ，
> name varchar(20) not null comment '姓名'
> )；
>
> select * from information_schema.colums where table_name='student'   //查看某个表的列信息





### 视图

视图就是一条select语句执行后返回的结果集，视图是对若干张基本表的引用，是一张虚表，查询语句执行的结果，不存储具体的数据。视图可以方便操作，另外在权限控制时，可以合理限制访问的内容。对视图修改数据时会修改源表的数据

> CREATE VIEW  view_name AS SELECT * FROM student;
>
> SELECT * FROM view_name  //从视图中查询数据
>
> SHOW CREATE VIEW  view_name   //查看视图

### 存储过程

存储过程就是为以后使用而保存的一条或多条SQL语句，类似增强版的批处理文件。存储过程把处理封装在一个易用的单元中，可以简化复杂的操作，存储过程通常以编译过的形式存储的（大并发的应用不建议使用存储过程，逻辑运算在DB端会导致数据库压力很大）

> DELIMITER $$
> CREATE PROCEDURE 过程名()
> BEGIN
> ​	......
> END$$
> DELIMITER;

> CALL  过程名();       //存储过程调用

> DECLARE name varchar(20)   //定义变量

> show procedure status like '%过程名%';   //查看存储过程

> show create procedure  过程名  //查看存储过程具体内容



+ 返回员工工资的最大值、最小值、平均值

```mysql
delimiter $$
create procedure sp_sal(
    out salmax decimal(8,2),
    out salmin decimal(8,3),
    out salavg decimal(8,2)
)
begin
	select MAX(salary) into salmax from employee;
	select MIN(salary) into salmin from employee;
	select AVG(salary) into salavg from employee;
end $$
delimiter;
```

