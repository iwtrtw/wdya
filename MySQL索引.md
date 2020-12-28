## mysql index

+ 查看所有数据库的列数

  > select count(*) from information_schema.columns;

+ 查看某个数据库的列数

  > select count(*) from information_schema.columns where table_schema='库名';

+ 查看某个数据库某个表的列数

  > select count(*) from information_schema.columns where table_schema='库名' and table_name='表名';

+ 查看mysql中哪些表没有建索引

  > select  *  from information_schema.tables as t left join (select distinct table_schema, table_name from information_schema.KEY_COLUMN_USAGE) as kt on kt.table_schema=t.table_schema and kt.table_name= t.table_name where t.table_schema not in('mysql','information_schema','performance_schema','test','sys')  and  kt.table_name is null ;

数据结构：B-树（所有节点都存储数据）、B+树（只有叶子节点存储数据，非叶子节点只存储key）

MyISAM主(唯一)索引：叶子节点储存数据的地址（key唯一性，根据地址只找到一条数据）

MyISAM辅助索引：叶子节点存储数据的地址(key可重复，根据地址可能找到多条数据)

InnoDB主(唯一)索引：叶子节点直接储存数据记录（占用空间较大）

InnoDB二级索引：叶子节点存储主键，然后根据主键再去找主索引的数据

> show index from student
>
> alter table student add index  age_index(age)  //为age字段建立索引
>
> ALTER TABLE student ADD UNIQUE(id)   //为id建立唯一索引  
>
> create index name_index on student(name)  //为name字段建立普通索引
>
> create unique index name_index on student(name)  //为name建立唯一索引
>
> drop index name_index on student  //删除name_index索引
>
> alter table student drop primary key  //去除主键（前提主键无自增）
>
> alter table student change id in int //修改id的类型(非自增)再删除主键

### EXPLAIN

#### id

标识select语句的编号，有多select就会有多个id(有时id是一样的)，编号从1开始递增；代表执行顺序，值越大则优先级越高，值相同则从上而下执行

#### select_type

+ simple：简单查询，不包含子查询
+ primary：主查询，即最外层的select查询
+ subquery：子查询的第一个select查询，不依赖外部查询的结果集

> explain select * from item where id=(select id from item where id=2)

| id   | select_type | table |
| ---- | ----------- | ----- |
| 1    | PRIMARY     | item  |
| 2    | SUBQUERY    | item  |

+ union  && union result

> EXPLAIN select * from item where id=1 union select * from item where id=2

| id   | select_type  | table      |
| ---- | ------------ | ---------- |
| 1    | PRIMARY      | item       |
| 2    | UNION        | item       |
| NULL | UNION RESULT | (union1,2) |

+ dependent union && dependent subquery

> explain select * from item where id in (select id from item where id=2 union select id from item where id=3)

| id   | select_type       | table      |
| ---- | ----------------- | ---------- |
| 1    | PRIMARY           | item       |
| 2    | DEPENDENT SUQUERY | item       |
| 3    | DEPENDENT UNION   | item       |
| NULL | UNION RESULT      | (union2,3) |

#### table

查询所使用到的表名，可以是实际表名，也可以是别名

#### type

显示连接使用的类型，可以决定查询速度

+ null

+ system：表仅有一行
+ const：用于常数值比较PRIMARY KEY（当查询的表仅有一行时，使用system）

> explain select name from item **where id=1**  //id建立了唯一索引

+ eq_ref：相等引用查询，并且只能找到一行数据。常见于主键或唯一索引扫描的多表连接操作中

+ ref：引用查询，快速查找到多个符合条件的行(非唯一索引)
+ ref_or_nul：在初次查找的结果里找出null条目，然后进行二次查找
+ index_merge：索引合并优化被使用
+ unique_subquery：在某些IN查询中使用此类型，查询的是唯一索引
+ index_subquery：用于in形式子查询使用到了辅助索引或者in常数列表，子查询可能返回重复值，查询的是非唯一索引
+ range：索引范围扫描，检索给定范围的索引行，常见于betwenn、<、>等查询

> explain select sales from item **where id>1**  //where条件处的id有建索引

+ index：全表扫描索引树的节点，只遍历索引树（即查询结果是在索引树上的）

> explain select **id** from item  //id建立了索引

+ all：从头到尾全表扫描，扫描全表数据（查询结果不是在索引树上找到的）

> explain select * from item
>
> explain select name from item  //name没有建索引
>
> explain select name from item  where age>20  //age没有建索引

#### possible_keys

可能用到的索引，如果为空说明没有可用的索引

#### key

实际用到的索引，只能使用一个索引；如果为null，则没有使用索引

#### key_len

用到的索引长度，在不损失精确性的情况下，长度越短越好 

#### rows

扫描的行数

#### Extra

附加信息，查看索引使用情况

+ Using index：使用索引，没有查询数据表（即没有使用硬盘的数据表），只用索引表完成查询，也叫覆盖索引；如果同时出现Using where,说明使用索引查找记录，但需要查询到数据表

+ Using where：条件查询，不用读取表的所有数据或者通过索引就找到所需数据
+ Using filesort：文件排序（order by）
+ Using temporary：为了得到结果，使用了临时表

### 索引

#### 索引类型

+ B树索引：innodb默认的索引时B+数索引
+ hash索引：检索效率高，一次定位，减少IO访问；仅仅使用于"="、“in”，不能使用范围索引（只有Memory引擎才支持hash索引）
+ FULLTEXT索引：只有char、varchar、text列才可以创建全文索引。MyISAM和InnoDB引擎都支持

> show variables like '%word%';   //查看fulltext参数
>
> show variables like '%ngram%';  //查看分词参数

>ALTER TABLE student ADD FULLTEXT INDEX  索引名(列名1，列名2) WITH PARSER NGRAM   //创建全文索引

> set global innodb_ft_aux_table='demo/student';    //显式指定全文检索的表源

> select * from information_schema.INNODB_FT_INDEX_TABLE;  //查看分词后的内容

> select count(*) FROM student WHERE MATCH(title,content) AGINST('学习' IN NATURAL LANGUAGE MODE)；//统计有多少行数据的（title，content）字段值包含‘学习’

> select * FROM student WHERE MATCH(title,content) AGINST('+学习 +设计' IN BOOLEAN MODE)   //查找同时包含‘学习’和‘设计’的记录

#### 索引失效

+ 不等于：<> 、!=
+ 前导模糊查询：%xx%
+ 比较不匹配的数据类型
+ or连接条件包含非索引列，即使全是索引列也可能没有使用索引
+ 条件表达式前使用函数
+ 隐式类型转换
+ 联合索引违背最左前缀

#### 索引利弊



### 性能优化

#### 查询慢

+ SQL语句过于复杂，使用了多表关联查询（MySQL数据多表关联查询使用的笛卡尔积的形式，n*m）；如果查询时使用全表扫描会非常慢，特别是数据量大时查询慢效果很明显
+ 一次查询数据量大，返回不必要的字段数据，会导致查询慢，可能会提高死锁发生的概率(对于无用的大字段，如 varchar、blob、text，会增加 io 操作)

+ SQL没有建立合适的索引
+ SQL特定类型语句(COUNT、MAX、GROUP BY、LIMIT、UNION)使用不合理
+ 数据库表结构设计不合理，对表字段类型设计不合理
+ 数据库系统配置不合理、服务器硬件性能不足、网络慢

#### 慢查询日志

> show variables like 'slow_query%';  //查看慢查询是否开启以及慢查询日志存放路径
>
> set global slow_query_log='ON';   //开启慢查询
>
> show variables like 'long_query_time';  //查看多长时间为慢查询
>
> set profiling=1 //开启SQL语句执行时间统计
>
> show profiles  //查看SQL语句执行时间

#### 索引优化

尽量避免全表扫描（万级以上数据）；虽然索引提高查询速度，但维护的成本也高

+ 合适字段添加索引（where字段经常使用的字段）
  + 表之间的join键值上
+ 选择离散度大的字段建立联合索引，并把离散度最大的字段放在最前(值区分明显的字段)

> select  count(distinct id)/count(id)  from student;   //计算id字段的值离散度(越接近1离散度越高)

+ 合适长度的索引：综合考虑索引建立维护以及区分度

> ALTER TABLE student ADD INDEX  name5_index(name(4))  //指定使用name的前4建立索引

+ **使用索引覆盖**：如果查询范围是少量字段，可以考虑覆盖索引；避免从二级索引回表到主键索引数据表上查询数据
+ 前缀不易区分建立索引
+ 删除重复和冗余索引

#### 特定类型优化

> count(*)包括了所有的列，相当于行数，在统计结果的时候， 不会忽略列值为NULL 
> count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候， 不会忽略列值为NULL 
> count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数， 即某个字段值为NULL时，不统计

> 列名为主键，count(列名)会比count(1)快 
> 列名不为主键，count(1)会比count(列名)快 
> 如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*） 
> 如果有主键，则 select count（主键）的执行效率是最优的 
> 如果表只有一个字段，则 select count（*）最优。

+ count：如果某一字段经常需要统计，考虑给该字段添加索引，优先使用count(*)>count(1)>
+ max、min：如果某一字段经常需要获取最大值、最小值，考虑给该字段添加索引
+ group by：如果某一字段经常需要分组，考虑给该字段添加索引
+ limit：通过有索引的字段定位所查数据的位置

> select * from student limit 1000000,5   //获取1000000后5条记录，实际查询了10000005条记录

优化如下：先通过查询id分页找到数据所在的起始位置，然后通过id定位获取5条数据

> SELECT * FROM student  s1 INNER JOIN (select id from student limit 1000000,5) as s2 ON s1.id = s2.id

#### 数据库表优化

+ 字段数据类型选择：整型>date、time>emum、char>varchar>text > blob；整型数据是定长的，没有字符集差异；date、time日期类型数据定长、运算快、节省空间；enum枚举类型能起到约束作用，内部用整型存储；varchar字符串是变长数据类型，同时要考虑字符集和校对集；text、blob类型不能使用索引，也不能使用内存临时表。
+ 不常用字段分离：将不常使用到字段并且占用空间大的字段(text、blob)从原表中分离出来，独立建立另一个表
+ 适当添加冗余字段：在一个表中添加经常需要关联查询另一个表的字段；减少关联查询

#### 数据库配置优化

+ 内存使用参数
  + innodb_buffer_pool_size：Innodb存储引擎的缓冲池大小
  + innodb_buffer_pool_instances：缓冲池的个数
+ 日志控制参数
  + innodb_log_buffer_size：日志缓冲的大小
  + innodb_flush_log_at_trx_commit：0/1/2
  + innodb_log_file_size：指定事务日志的大小
  + innodb_log_files_in_group：指定日志组的个数
+ IO分配参数优化
  + innodb_file_per_table：独立表空间设置
  + innodb_file_io_threads：文件读取的线程个数
  + innodb_open_files：打开的文件个数
+ 其他参数
  + default-character-set：设置mysql客户端的字符集
  + character-set-server：设置mysql数据库的字符集

