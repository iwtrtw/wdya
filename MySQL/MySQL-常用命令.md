#### MySQL

##### 常用命令

> mysql -h localhost  -P 3306 -u root -p

查看版本：select version();

查看当前所在的数据库：select database();

查看当前用户：select user();

查看数据库：show databases;

使用数据库sql_test：use sql_test;

查看表：show tables;

查看数据库sql_test的表：show tables from  sql_test;

建表：create table student(id int,name varchar(20));

查看表结构：desc student;

插入数据：insert into student  (id,name) values (1,'Tom');

更新数据：update student set name='Jerry' where id=1;

删除数据：delete from student where id=1;

##### 语法规范

+ 不区分大小写，建议关键字大写，表名、列名小写
+ 每条命令最好用分号结尾

##### SQL执行顺序

```
SELECT DISTINCT <select_list>
FROM <left_table>
<join_type> JOIN <right_table>
ON <join_condition>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```

**1. 数据查询语言DQL**
数据查询语言DQL基本结构是由SELECT子句，FROM子句，WHERE
子句组成的查询块：

> SELECT <字段名表>
> FROM <表或视图名>
> WHERE <查询条件>

**2 .数据操纵语言DML**
数据操纵语言DML主要有三种形式：

> 1) 插入：INSERT
> 2) 更新：UPDATE
> 3) 删除：DELETE

**3. 数据定义语言DDL**
数据定义语言DDL用来创建数据库中的各种对象-----表、视图、
索引、同义词、聚簇等如：
CREATE TABLE/VIEW/INDEX/SYN/CLUSTER
| | | | |
表 视图 索引 同义词 簇

DDL操作是隐性提交的！不能rollback 

**4. 数据控制语言DCL**
数据控制语言DCL用来授予或回收访问数据库的某种特权，并控制
数据库操纵事务发生的时间及效果，对数据库实行监视等。如：

> 1) GRANT：授权。

> 2) ROLLBACK [WORK] TO [SAVEPOINT]：回退到某一点。
> 回滚---ROLLBACK
> 回滚命令使数据库状态回到上次最后提交的状态。其格式为：
> SQL>ROLLBACK;

> 3) COMMIT [WORK]：提交。


  在数据库的插入、删除和修改操作时，只有当事务在提交到数据
库时才算完成。在事务提交前，只有操作数据库的这个人才能有权看
到所做的事情，别人只有在最后提交完成后才可以看到。
提交数据有三种类型：显式提交、隐式提交及自动提交。下面分
别说明这三种类型。

(1) 显式提交
用COMMIT命令直接完成的提交为显式提交。其格式为：

> SQL>COMMIT；

(2) 隐式提交
用SQL命令间接完成的提交为隐式提交。这些命令是：
ALTER，AUDIT，COMMENT，CONNECT，CREATE，DISCONNECT，DROP，
EXIT，GRANT，NOAUDIT，QUIT，REVOKE，RENAME。

(3) 自动提交
若把AUTOCOMMIT设置为ON，则在插入、修改、删除语句执行后，
系统将自动进行提交，这就是自动提交。其格式为：

> SQL>SET AUTOCOMMIT ON；



##### SELECT

+ 起别名（AS）：SELECT first_name AS 姓 FROM studnet;

+ 去重（DISTINCT）：SELECT DISTINCT class_name FROM student;

+ 字符串拼接（CONCAT）：SELECT CONCAT(last_name,first_name) AS `name` FROM student;

  > 若某字段为null，则拼接后的值为null

IFNULL(age,0)：如果字段age为空则显示为0，否则显示原数据

> SELECT IFNULL(age,0) AS 年龄 FROM student;

##### WHERE

```
条件查询：根据条件过滤原始表的数据，查询到想要的数据
语法：
	select 
		要查询的字段|表达式|常量值|函数
	from 
		表
	where 
		条件 ;
```

1. 条件表达式
   条件运算符：<   >    >=   <=   =   !=   <>

   > SELECT * FROM student WHERE salary>10000

2. 逻辑表达式

   逻辑运算符：

   + and（&&）:两个条件如果同时成立，结果为true，否则为false
   + or(||)：两个条件只要有一个成立，结果为true，否则为false
   + not(!)：如果条件成立，则not后为false，否则为true

   > SELECT * FROM student WHERE salary>10000 AND salary<20000

3. 模糊查询（LIKE、IN、BETWEEN AND、IS NULL、IS NOT NULL）

   + LIKE  % 任意多个字符   _ 单个字符 （通配符不能匹配null）

     > SELECT * FROM student WHERE last_name LIKE 'a%'

     > SELECT * FROM student WHERE last_name LIKE '__a_e%'（last_name第三个字符为a，第5个字符为e）

   （当需查询字段包含_可用转义字符\，或者指定ESCAPE）查询第二个字符

   > SELECT * FROM student WHERE last_name LIKE '___ \\___%'

   > SELECT * FROM student WHERE last_name LIKE '_$%' ESCAPE '$'

   + BETWEEN AND（等价于>= AND <= 包含临界值）

   > SELECT * FROM student WHERE salary BETWEEN 100 AND 200

   大小顺序不要颠倒，因为等价于 salary >= 100 AND salary <=200；若BETWEEN 200 AND 100，则查询区间变为salary >= 200 AND salary <=100（结果为空）

   + IN（等价于 = OR = OR = 列表中的数据类型需一致，不支持通配符）

     > SELECT * FROM student WHERE las_name IN('Tom','Jack','Jane')

   + IS NULL（在mysql中无法使用=null、<>null来判断是否为空）

     > SELECT * FROM student WHERE las_name IS NULL

   + IS NOT NULL

     > SELECT * FROM student WHERE las_name IS NOT NULL

##### ORDER BY

```
排序查询(order by一般放在最后，但在limit前)
语法：
select
	要查询的东西
from
	表
where 
	条件
group by 分组列表
having 
order by 排序的字段|表达式|函数|别名 【asc|desc】
```

+ 多个字段排序（先按salary升序排，若salary相同则按stu_id降序排）

  > SELECT * FROM sutdent ORDER BY salary ASC, stu_id DESC;

+ 函数排序（别名）

  > SELECT * ,salary* 12 *(1+IFNULL(com_pct,0)) AS 年薪 FROM student ORDER BY 年薪

  > SELECT * ,salary* 12 *(1+IFNULL(com_pct,0))  AS 年薪 FROM student ORDER BY salary* * 12 *(1+IFNULL(com_pct,0))

+ 字符长度（LENGTH）(查询邮箱中包含e的员工信息，并先按邮箱字节数降序)

  > SELECT * FROM student WHERE email LIKE '%e%' ORDER BY LENGTH(email) DESC

##### 常见函数

函数调用：SELECT 函数名(实参列表) [FROM table]

**utf-8：一个中文汉字3字节，一个英文字符一个字节**

**MySQL索引是从1开始的**

+ 字符函数

  + LENGTH 获取参数值的字节个数

    > SELECT LENGTH('john')

  + CONCAT 拼接字符串

    > SELECT CONCAT( last_name , '_' , first_name )  AS 姓名 FROM student ;

  + UPPER、LOWER 改变字符大小写
    
    > SELECT CONCAT( UPPER(last_name) , LOWER(first_name) )  AS 姓名 FROM student；  
    
  + SUBSTR(str,start,length) 截取字符 
  
    > SELECT SUNSTR("ABCDEFG",3);     //CDEFG
    >
    > SELECT SUNSTR("ABCDEFG",2,3);     //BCD
  
  + INSTR(str,substr) 返回子串substr在字符串str中的第一次出现起始索引（若无则返回0）
  
    > SELECT INSTR('ABCDEFG','CDE');     //3
  
  + TRIM 去除前后空格（可指定去除字符）
  
    > SELECT TRIM('       ABC     ');     //ABC
    >
    > SELECT TRIM('a' FROM 'aaaaaABCaaaaa' );     //ABC
  
  + LPAD 用指定字符实现左填充指定长度
  
    > SELECT LOAD('abc',10,'X');     //XXXXXXXabc
    >
    > SELECT LOAD('abc',2,'X');     //ab
  
  + RPAD 用指定字符实现右填充指定长度
  
  + REPLACE 替换
  
    > SELECT REPLACE('后端选择后端',‘后端’,'前端');     //前端选择前端
  
+ 数学函数

  + ROUND 四舍五入

    > SELECT ROUND(-1.57)		//-2
    >
    > SELECT ROUND(1.57)		//2
    >
    > SELECT ROUND(1.5678,2)		//1.57

  + CEIL 向上取整（返回大于等于该参数的最小整数）

    > SELECT CEIL(1.01)   //2
    >
    > SELECT CEIL(1.00)   //1
    >
    > SELECT CEIL(-1.02)   //-1

  + FLOOR 向下取整（返回小于等于该参数的最大整数）

    > SELECT FLOOR(9.99)   //9
    >
    > SELECT FLOOR(-9.99)   //-10

  + TRUNCATE 截断

    > SELECT TRUNCATE(1.349,2)	//1.34
    >
    > SELECT TRUNCATE(1.343453,3)	//1.343

  + MOD 取余 （mod(a,b)  <==> a-a/b*b）

    > SELECT MOD(10,3)		//1
    >
    > SELECT 10%3		//1
    >
    > SELECT MOD(-10,3)		//-1  看被除数符号
    >
    > SELECT MOD(10,-3)		//1

+ 日期函数

  + NOW  返回当前系统日期+时间

  + CURDATE  返回当前系统日期（不包括时间）

  + CURTIME  返回当前系统时间（不包括日期）

  + 获取指定部分（年、月、日）

    > SELECT YEAR('1998-1-1')  		//1998
    >
    > SELECT MONTH('1998-6-1')  		//6
    >
    > SELECT MONTHNAME('1998-6-1')  		//June

  + STR_TO_DATE  将日期格式的字符转换成指定格式的日期

    > SELECT STR_TO_DATE('9-13-1999' , '%m-%d-%Y')		//1999-09-13

  + DATE_FORMAT  将日期转换为字符

    > SELECT DATE_FORMAT('2018/6/6' , '%Y年%m月%d日')		//2018年06月06日

**AVG不包括null，COUNT(*)统计所有行数包括null，COUNT(name)统计函数但不包括null，COUNT(1)统计所有行数**

> MYISAM存储引擎下，COUNT(*）效率高
>
> INNODB存储引擎下，COUNT(*）和COUNT(1)的效率差不多，但比COUNT(字段)要高一些

count(*)&count(1)&count(列名)执行效率比较

> （1）如果列为主键，count(列名)效率优于count(1)
>
> （2）如果列不为主键，count(1)效率优于count(列名)
>
> （3）如果表中存在主键，count(主键列名)效率最优
>
> （4）如果表中只有一列，则count(*)效率最优
>
> （5）如果表有多列，且不存在主键，则count(1)效率优于count(*)

##### 分组函数（group by）

​    可以按单个字段分组；和分组函数一同查询的字段最好是分组后的字段；可以按多个字段分组，字段之间用逗号隔开；可以支持排序

|            | 针对表         | 位置           | 关键字 |
| ---------- | -------------- | -------------- | ------ |
| 分组前筛选 | 原始表         | group by的前面 | where  |
| 分组后筛选 | 分组后的结果集 | group by的后面 | having |

+ 查询每个班级的最高分

> SELECT MAX(grade),class_id  FROM class GROUP BY class_id

+ 查询每个年级的班级个数

> SELECT COUNT(*)，nianji_id  FROM school GROUP BY nianji_id

+ 查询邮箱中包含a字符的每个部门的平均工资(email在employee表)

> SELECT AVG(salary),department_id  FROM employee WHERE   email LIKE '%a%' GROUP BY department_id 

+ 查询哪个部门的员工个数>2

> SELECT COUNT(*),department_id FROM employee GROUP BY department_id HAVING COUNT(\*)>2

+ 查询每个部门有奖金(bonus)的员工的最高工资>12000的部门编号和最高工资

> SELECT MAX(salary), department_id  FROM employee
>
> WHERE bonus  IS NOT NULL
>
> GROUP BY department_id 
>
> HAVING MAX(salary)>12000

+ 查询领导编号>102的每个领导手下最低工资>5000的领导编号是哪个，以及其最低工资

> SELECT MIN(salary),manger_id FROM employee
>
> WHERE manger_id>102
>
> GROUP BY manger_id
>
> HAVING MIN(salary)>5000

+ 按员工姓名的长度分组，查询每一组的员工个数，筛选员工个数>5的有哪些

> SELECT COUNT(*) c,LENGTH(name)   FROM employee
>
> GROUP BY LENGTH(name)
>
> HAVING c>5 

+ 查询每个部门每个工种(job_id)的员工的平均工资

> SELECT AVG(salary), department_id ,job_id  FROM employee
>
> GROUP BY job_id,department_id

##### 分页查询

limit  start_index  size

页数page，每页的条目数size

select 查询列表 from 表 limit (page-1)*size,size

+ 查询第11条到第25条

> SELECT * FROM employee LIMIT 10,15

##### 联合查询

union 将多条查询语句的结果合并成一个结果（默认去重，union all不去重），多条查询语句的查询列数一致，每一列的类型和顺序最好一致

##### 联表查询

+ JOIN

> select * from  A  inner join  B  on A.key=B.key   //A∩B

> select * from  A  left join   B  on  A.key=B.key   //A

> select * from  A   right join  B  on  A.key=B.key   //B

> select * from A left join B on  A.key=B.key where B.key is null   //A-B

> select * from A right join B on  A.key=B.key where A.key is null   //B-A

> select * from  A  left join   B  on  A.key=B.key 
> union      //A∪B
> select * from  A   right join  B  on  A.key=B.key  

>select * from A left join B on  A.key=B.key where B.key is null 
>union    //A∪B-A∩B
>select * from A right join B on  A.key=B.key where A.key is null   

### DML

##### insert

​      插入列数和值的个数必须一致；插入的值的类型要与列的类型一致或兼容；不可为null的列必须插入值

> INSERT INTO 表名(列名...)  values(值...)

##### update

> UPDATE 表名 SET  列=新值,列=新值  WHERE+筛选条件
>
> UPDATE 表1 别名  LEFT JOIN 表2 别名 ON 连接条件 SET 列=值... WHERE+筛选条件

##### delete

> DELETE FROM 表名 WHERE 筛选条件

### DDL

##### 库管理

+ 建库

> CREATE DATABASE IF NOT EXISTS 库名

+ 改库

> ALTER DATABASE  库名  CHARACTER SET  gbk   //修改库的字符集

+ 删库

> DROP DATABASE IF EXISTS 库名

##### 表管理

+ 建表

> CREATE TABLE  IF NOT EXISTS 表名(列名  列类型 [长度/约束]，...）

> CREATE TABLE 新表 LIKE 原表  //复制表结构(没有具体数据) 
>
> CREATE TABLE 新表 SELECT * FROM 原表   //复制整个表(含数据)

+ 改表

> ALTER TABLE 表名  CHANGE COLUMN 原列名  新列名  //修改列名
>
> ALTER TABLE 表名  MODIFY COLUMN 列名 新类型   //修改列类型
>
> ALTER TABLE 表名 ADD COLUMN 新列名 列类型      //添加新列
>
> ALTER TABLE 表名 DROP COLUMN 列名     //删除某列
>
> ALTER TABLE 原表名 RENAME TO 新表名    //修改表名

+ 删表

> DROP TABLE IF EXISTS 表名

##### 数据类型

+ 整型

INT默认是有符号，想要设置无符号可以用INT UNSIGNED ；如果插入的数值超过了整型的范围，会报out of range异常，并且会插入临界值

+ 浮点型：float(M,D)、double(M,D)   //M表示整数与小数合起来的位数   D表示小数点后位数

> float(5,2)   //当插入1523.45时实际插入的数据为999.99

+ 定点型：decimal(10,0)的M默认为10，D默认为0

##### 常见约束

NOT NULL：字端值不能为空

DEFAULT  value：字段值的默认值为value

PRIMARY KEY：字段值具有唯一性且非空

UNIQUE：字段值具有唯一性但可为空(只能有一个为null)

FOREGIN KEY：字段值必须来自另一个主表的字段值

> CREATE TABLE 表名(
>
> ​		字段名  字段类型  列级约束（外键无效）
>
> ​		字段名   字段类型
>
> ​		表级约束
>
> ​        PRIMARY KEY(id)
>
> ​        FOREIGN KEY(id) REFERENCE 表名(字段)
> )

​      当含有外键时，要先删除从表，可以通过级联删除或级联置空删除主表

> //级联删除
>
> ALTER TABLE student  ADD CONSTRAINT fk_stu_major FOREIGN KEY(majorid)   REFERENCES major(id)  ON DELETE CASCADE
>
> //级联置空
>
> ALTER TABLE student ADD CONSTRAINT fk_stu_major FOREIGN KEY(majorid)  REFERENCES major(id)  ON DELETE SET NULL

##### 标识列

AUTO INCREMENT：自增

> CREATE TABLE 表名(
>
> ​		id  IN  AUTO INCREMENT
>
> )

> 一个表中至多有一个字段属于自增
>
> 自增的字段只能是数值型
>
> 自增字段必须是一个key，但不一定是主键
>
> SEE AUTO_INCREMENT_INCREMENT = 3 设置自增步长

### DCL

​      事务由单独单元的一个或多个SQL语句组成，在这个单元中，每个MySQL语句是相互依赖的，而整个单独单元作为一个不可分割的整体，如果单元中某个SQL语句一旦执行失败或产生错误，整个单元将会回滚。所有受到的影响数据将返回到事物开始以前的状态；如果单元中所有的SQL语句均执行成功，则事务被顺利执行（innodb支持事务，myisam不支持事务）

+ 原子性(Atomicity)：原子性是指事务是一个不可分割的工作单位，事务中的操作要么发生，要么都不发生
+ 一致性(Consistency)：事务必须是数据库从一个一致性状态变换到另一个一致性状态
+ 隔离型(Isolation)：事务的隔离型是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务隔离的，并发执行的各个事务之间不能互相干扰
+ 持久性(Durability)：持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响

隐式事务：没有明显的开启和结束的标记（单条insert、update、delete）

显式事务：具有明显的开启和结束标记，前提是关闭自动提交

> SET AUTOCOMMIT=0    //禁用自动提交
>
> START TRANSACTION  //可选，禁用自动提交时相当于开启事务了
>
> 多条sql语句(通常指select、insert、update、delete)
>
> 结束事务（提交或回滚）
>
> COMMIT   //提交事务
>
> ROLLBACK  //回滚事务，回到最初状态(即原数据没变)

##### 隔离级别

​      对于同时运行的多个事务，当这些事务访问数据库中相同的数据时，如果没有采取必要的隔离机制，就回导致各种并发问题：

+ 脏读：对于两个事务T1、T2，T1读取了已经被T2**更新**但还没有被提交的字段，若T2回滚，T1读到的内容就是临时且无效的
+ 不可重复读：对于两个事务T1、T2，T1第一次读取了一个字段，然后T2**更新(UPDATE)**了该字段并结束事务(T1此时还没结束)，T1再次读取该字段时，值不同了（即一个事务范围内两个相同的查询却返回了不同数据）

> 不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据。

+ 幻读：对于两个事务T1、T2，T1从一个表中读取了一个字段，然后T2在该表中**插入(INSERT)/删除(DELETE)**了新的行并结束，之后如果T1再次读取同一表时，就会多出几行。

> 幻读和不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）

​      MySQL支持4中事务隔离级别，其默认隔离级别为REPEATABLE READ

+ READ UNCOMMITTED：读未提交，即一个事务可以读取另一个未提交事务的数据。允许事务读取为被其它事务提交的变更，脏读、不可重复读和幻读的问题都会出现
+ READ COMMIITTED：读提交，即一个事务要等另一个事务提交后才能读取最新数据。只允许事务读取已经被其它事务提交的变更，可以避免脏读，但不可重复读和幻读问题仍然可能出现
+ REPEATABLE READ：重复读，确保事务可以多次从一个字段中读取相同的值(但可能不是最新值)，即事务不会读到其他事务对已有数据的修改，即使其他事务已提交，也就是说，事务开始时读到的已有数据是什么，在事务提交前的任意时刻，这些数据的值都是一样的。可以避免脏读和不可重复读，但幻读问题仍存在
+ SERIALIZABLE：序列化，事务串行化顺序执行。确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其它事务对该表的插入、更行和删除操作。可以避免脏读、不可重复读、幻读问题但性能低下

> SET SESSION TRANSACTION ISOLATION LEVEL xxx

​      隔离级别的设置只对当前链接有效。对于使用MySQL命令窗口而言，一个窗口就相当于一个链接，当前窗口设置的隔离级别只对当前窗口中的事务有效；对于JDBC操作数据库来说，一个Connection对象相当于一个链接，而对于Connection对象设置的隔离级别只对该Connection对象有效，与其他链接Connection对象无关。

##### 视图

​      视图本身是一虚拟表，它的数据来自于表，通过执行时动态获取真实数据。视图就是一条select语句执行后返回的结果集，视图是对若干张基本表的引用，是一张虚表，查询语句执行的结果，不存储具体的数据。视图一般用于查询(视图一般情况下是不允许更改的)视图可以方便操作，另外在权限控制时，可以合理限制访问的内容。对视图修改数据时会修改源表的数据使用视图可以重用sql语句、简化复杂的sql操作、保护数据的安全性，在数据库中视图并不存放真实数据(只是保存了sql逻辑，可以指向真实数据)，

+ 创建视图

> CREATE VIEW 视图名
>
> AS  查询语句

+ 修改视图

> ALTER VIEW 原视图  AS 新查询语句

+ 删除视图

> DROP VIEW 视图名,视图名...

+ 查看视图

> DESC 视图名



> CREATE VIEW  view_name AS SELECT * FROM student;
>
> SELECT * FROM view_name  //从视图中查询数据
>
> SHOW CREATE VIEW  view_name   //查看视图

### 系统变量

+ 查看系统变量

> SHOW GLOBAL|SESSION VARIABLES  LIKE 'xx' 

+ 查看指定的系统变量的值

> select @@global|session.变量名

+ 为系统变量赋值

> SET  GLOBAL|SESSION  变量名=值

### 存储过程

​     存储过程就是为以后使用而保存的一条或多条SQL语句，类似增强版的批处理文件。 一组预先编译好的SQL语句的集合，可以理解成批处理语句，使用存储过程可以提高代码的可重用性、简化操作、减少编译次数、减少和数据库服务器的连接次数。存储过程把处理封装在一个易用的单元中，可以简化复杂的操作，存储过程通常以编译过的形式存储的（大并发的应用不建议使用存储过程，逻辑运算在DB端会导致数据库压力很大）

+ 创建存储过程

> DELIMITER $$
>
> CREATE PROCEDURE  存储过程名(参数模式 参数名 参数类型)
>
> BEGIN
>
> ​		方法体  //一组合法有效的SQL语句
>
> END$$
>
> DELIMITER;

+ 参数模式

>IN：该参数可以作为输入（即该参数需要调用方法传入值）
>
>OUT：该参数可以作为输出（该参数可以作为返回值）
>
>INOUT：该参数既可以作为输入也可以作为输出

+ 定义变量

> DECLARE name varchar(20) 

+ 调用存储过程

> CALL  存储过程名(实参列表)

+ 删除存储过程

> DROP PROCEDURE 存储过程名

+ 查看存储过程信息

> SHOW CREATE PROCEDURE 存储过程名

返回员工工资的最大值、最小值、平均值

```SQL
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



### 函数

​      存储过程可以有0个返回，也可以有多个返回，适合做批量插入、批量更新；而函数有且只有1个返回，适合做处理数据后返回一个结果

+ 创建函数

> CREATE FUNCTION 函数名(参数名 参数类型）RETURNS 返回类型
>
> BEGIN
>
> ​			函数体
>
> ​			return  值
>
> END

+ 查看函数

> SHOW CREATE FUNCTION 函数名

+ 删除函数

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



