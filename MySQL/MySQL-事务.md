物理日志：存储数据被修改的值

逻辑日志：存储逻辑SQL修改语句

概念、特性、what、why、how

### 事务/隔离级别

#### 事务特性

​		事务是访问和更新数据库时的一个不可分割执行单元，在每一个事务中，所有修改操作要么全部都执行生效，要么都不执行失效。一个事务可由一条或多条SQL组成。事务的提出是为了保证数据库从一种一致状态转换到另一种一致状态，保证了数据的一致性。例如银行转账，从A转账到B，其中主要有两个操作：A出账和B入账；这两个操作要不都操作成功，要么都操作失败，否则会造成数据不一致的后果（假如A出账了，但B入账失败，此时如果不回滚，那么A出账的那部分金额就不翼而飞，B并没有收到）

​		事务的特性有四点ACID，但许多数据库厂商出于各种考虑(性能或其他因素)并没有严格按照事务的ACID标准去支持事务，这需要用户使用的时候去避免可能存在的潜在问题。InnoDB的默认隔离级别READ REPEATABLE完全遵循和满足事务的ACID

+ **原子性(atomicity)：事务是一个不可分割的执行单元，一个事务中所有操作，要么都成功，要么都失败；当有一个操作失败了，那么之前已经成功的操作也必须撤销，使得数据库状态退回到执行事务前的状态，不能出现部分成功部分失败的情况**

  ```mysql
  	要想实现原子性，那么就要做到在事务回滚时能够撤销所有已经成功执行的sql的语句。在InnoDB中，事务回滚是通过undolog来实现的，每当有事务对数据库进行修改时，都会产生相应的undolog，用于记录当前sql操作的相关信息（操作类型insert/update/delete，事务id、table_id、行记录主键、下一undolog指针）
  	当事务执行失败或调用了rollback，InnoDB会根据undolog中的内容做与之前相反的工作：对于每个insert，回滚时会执行delete；对于每个delete，回滚时会执行insert；对于每个update，回滚时会执行一个相反的update，把数据改回去...最终将数据回滚到事务开始之前的状态。
  	undolog是逻辑日志，undo操作只是将数据库逻辑地恢复到原来的样子，即所有修改操作被逻辑的取消了，但数据结构和页本身即使回滚了还是可能和事务开始前不同的
  ```

+ **一致性(consistency)：在事务发生前后，数据库的数据状态都是合法的，数据的完整性没有被破坏，仍然保持数据一致性状态**

  ```
  	数据库层面：数据库通过原子性、隔离性、持久性来保证一致性。ACID四大特性之中，C(一致性)是目的，A(原子性)、I(隔离性)、D(持久性)是手段，是为了保证一致性，数据库提供的手段。数据库必须要实现AID三大特性，才有可能实现一致性。例如，原子性无法保证，显然一致性也无法保证。
  	应用层面：如果你在事务里故意写出违反约束的代码，一致性还是无法保证的。例如，你在转账的例子中，你的代码里故意不给B账户加钱，那一致性还是无法保证。因此，还必须从应用层角度考虑。通过代码判断数据库数据是否有效，然后决定回滚还是提交数据！
  ```

+ **隔离性(isolation)：并发执行的各事务之间互不干扰，即一个事务内部的操作及使用的数据与其他正在进行的事务隔离的。**在数据库里面会有很多的事务同时去操作我们的同一张表或者同一行数据，必然会产生一些并发或者干扰的操作， 那么我们对隔离性的定义，就是这些很多个的事务，对表或者行的并发操作，应该是透明的，互相不干扰的。通过这种方式，我们最终也是保证业务数据的一致性。事务允许多个用户对同一个数据进行并发访问，而不破坏数据的正确性和完整性。同时，并行事务的修改必须与其他并行事务的修改相互独立。

  ```mysql
  	严格的隔离性，对应了事务隔离级别中的Serializable (可串行化)，但实际应用中出于性能方面的考虑很少会使用可串行化。
  	在InnoDB中，隔离性是通过锁机制和MVCC来实现的：锁机制实现了写操作与写操作之间的隔离、MVCC实现了写操作与读操作之间的隔离。
  	锁机制：事务在修改数据之前，需要先获得相应的锁；获得锁之后，事务便可以修改数据；该事务操作期间，这部分数据是锁定的，其他事务如果需要修改数据，需要等待当前事务提交或回滚后释放锁
  	MVCC：Multi-Version Concurrency Control，即多版本的并发控制协议；在同一时刻，不同的事务读取到的数据可能是不同的(即多版本)。MVCC的优点是读不加锁，因此读写不冲突，并发性能好。InnoDB通过MVCC实现一致性非锁定读，当读取的行正在执行DELETE或UPDATE操作时，这时的读操作不会等待该行上锁释放，而是去读取该行的快照数据。MVCC机制在可重复读(Repeateable Read)和读已提交(Read Commited)的MVCC表现形式不同。在事务隔离级别为读已提交(Read Commited)时，一个事务能够读到另一个事务已经提交的数据，是不满足隔离性的。但是当事务隔离级别为可重复读(Repeateable Read)中，是满足隔离性的。
  ```

+ **持久性(durability)：事务一旦被提交，那么它对数据库中数据的改变就是永久性的。即使发生宕机等故障，数据库也能将数据恢复，保证已提交的数据不会丢失。持久性是从事务本身角度来保证结果的永久性的，如果不是数据库本身发生崩溃，而是一些外部因素（RAID卡损坏、自然灾害等）导致数据库发生问题，那么提交的数据还是可能丢失的。持久性保证的是高可靠性而不是高可用性**

  ```mysql
  	InnoDB存储引擎是基于磁盘存储的，但CPU与磁盘之间的读写速度存在着巨大鸿沟，如果每次读写数据都需要进行磁盘IO，效率会很低。为了解决这个问题，InnoDB使用缓冲池(Buffer Pool)来提高数据库的整体性能，在进行读页操作时，首先将从磁盘读到的页存放在缓冲池中，下一次再读相同页时，首先判断该页是否存在缓冲池中，若在缓冲池中，则直接读取该页，否则读取磁盘上的页。在进行修改操作时，首先修改缓冲池中的页，然后再以一定的频率刷新到磁盘中(以一种Checkpoint的机制刷新回磁盘)...缓冲池的使用大大提高了读写数据的效率，这样会有一个问题：假如某个事务在缓冲池中已完成数据的修改并提交了，但还没有刷新回磁盘时数据库发生宕机，那么内存中的数据将会丢失，那么事务持久性就无法保证了。
  	为此，InnoDB引入redo log来解决这一问题，redo log分为两部分：内存中的redo log buffer和磁盘中的redo log。当对数据进行修改时，除了修改缓冲池中的数据，还会在redo log buffer记录在这次操作(数据库会以一定的频率刷新到磁盘中的redo log);当事务提交时，会调用fsync操作强制将redo log buffer中数据刷新回磁盘中redo log，确保即使缓冲池已提交的事务的数据没有刷回磁盘时数据库突然宕机，重启时也可以通过读取redo log中的数据，完成数据恢复。
  ```

#### 四大隔离级别

当事务并发操作同一份数据时，**读操作**可能会产生脏读、不可重复读、幻读等问题

+ 脏读：一个事务可以读到另一个事务中未提交的数据。对于两个事务T1、T2，T1读到了已经被T2**更新但还没有被提交**的数据。假如T2回滚，那么T1读到的内容就是临时且无效的，这违反了事务的隔离性。

+ 不可重复读：在一个事务内两次同样的读取操作读取到的数据是不一样的。对于两个事务T1、T2，T1第一次读取了一个字段，然后T2**更新(UPDATE)**了该字段并结束事务(T1此时还没结束)，T1再次读取该字段时，值不同了（即一个事务范围内两个相同的查询却返回了不同数据）这违反了事务的隔离性、一致性

+ 幻读：在同一事务下，两次相同的读取操作发现数据变多了。对于两个事务T1、T2，T1先从一个表中读取一些数据，然后T2在该表中**插入(INSERT)**了新的数据并提交，之后如果T1再次进行读取操作时，发现数据多了原来不存在的一些数据。这违反了事务的隔离性

  ```
  不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据
  幻读和不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）不可重复读重点在于update和delete，而幻读的重点在于insert
  ```

**写操作**会出现数据丢失等问题：A和B一起执行了一个数据，然后B事务先提交，后来A事务回滚或提交，那么B事务的操作会因为A事务回滚或提交而丢失；

为了有效保证并发事务读取数据的正确性，提出了事务隔离级别，InnoDB中事务隔离级别是依靠锁机制和MVCC来实现的，SQL标准定义了四种隔离级别：

> show variables like 'transaction_isolation';   //查看当前事务隔离状态
>
> select * from information_schema.innodb_trx;   //查看当前运行的所有事务
>
> show variables like 'autocommit';  //查看提交模式
>
> set autocommit=0;   //关闭自动提交
>
> set SESSION/GLOBAL transaction_isolation=0;  //设置隔离级别为READ-UNCOMMITTED
>
> set SESSION/GLOBAL transaction_isolation=1;  //设置隔离级别为READ-COMMITTED
>
> set SESSION/GLOBAL transaction_isolation=2; // 设置隔离级别为REPEATABLE-READ
>
> set SESSION/GLOBAL transaction_isolation=3; // 设置隔离级别为SERIALIZABLE

+ READ UNCOMMITTED：读未提交，即一个事务可以读取另一个未提交事务的数据。脏读、不可重复读和幻读的问题都会出现

  ```
  不加锁，不同事务之间的操作时彼此可见且互相影响的
  ```

+ READ COMMIITTED：读提交，即一个事务要等另一个事务提交后才能读取最新数据。只允许事务读取其它事务已提交的变更，可以避免脏读，但不可重复读和幻读问题仍然可能出现

  ```
  RC隔离级别下，普通的select都是快照读，使用MVCC实现
  
  加锁的select都使用Record Lock，因为没有Gap Lock
  
  外键约束检查(foreign-key constraint checking)以及重复键检查(duplicate-key checking)时会使用间隙锁封锁区间；所以 RC 会出现幻读的问题。
  ```

+ REPEATABLE READ：重复读，确保事务可以多次从一个字段中读取相同的值(但可能不是最新值)，即事务不会读到其他事务对已有数据的修改，即使其他事务已提交，也就是说，事务开始时读到的已有数据是什么，在事务提交前的任意时刻，这些数据的值都是一样的。可以避免脏读和不可重复读，但幻读问题仍存在。在SQL标准中，RR是无法避免幻读问题的，但是InnoDB实现的RR避免了幻读问题。

  ```
  RR隔离级别下，普通的select使用快照读(snapshot read)，底层使用MVCC来实现
  
  加锁的select(select ... in share mode / select ... for update)以及更新操作update, delete 等语句使用当前读（current read），均使用next-key lock进行加锁。Next-Key Lock是行锁和间隙锁的组合，当InnoDB扫描索引记录的时候，会首先对索引记录加上行锁（Record Lock），再对索引记录两边的间隙加上间隙锁（Gap Lock）。加上间隙锁之后，其他事务就不能在这个间隙修改或者插入记录。
  
  当查询的索引含有唯一属性（唯一索引，主键索引）时，Innodb存储引擎会对next-key lock进行优化，将其降为record lock,即仅锁住索引本身，而不是范围。
  delete from user where id=15 and id=20; //record lock
  delete from user where id in(15,20);  //record lock
  delete from user where id between 5 and 7; //next-key
  ```

+ SERIALIZABLE：序列化，事务串行化顺序执行。确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其它事务对该表的插入、更行和删除操作。可以避免脏读、不可重复读、幻读问题但性能低下

  ```
  Serializable所有的select语句都会被隐式的转化为select ... in share mode，会和update、delete 互斥。
  ```

| 级别     | symbol           | 值   | 描述                                                         |
| -------- | ---------------- | ---- | ------------------------------------------------------------ |
| 读未提交 | READ-UNCOMMITTED | 0    | 存在脏读、不可重复读、幻读的问题                             |
| 读已提交 | READ-COMMITTED   | 1    | 解决脏读的问题，存在不可重复读、幻读的问题                   |
| 可重复读 | REPEATABLE-READ  | 2    | mysql 默认级别，解决脏读、不可重复读的问题，存在幻读的问题。使用 MMVC机制 实现可重复读 |
| 序列化   | SERIALIZABLE     | 3    | 解决脏读、不可重复读、幻读，可保证事务安全，但完全串行执行，性能最低 |

##### REPEATABLE-READ问题

+ B事务在A事务进行第一次select前修改某个数据(id=p)并提交，A事务是查询(select)到修改后的数据

| Session A                                                    | Session B                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| START TRANSACTION;                                           | START TRANSACTION;                                           |
|                                                              | update student set title='CR' where id=12; (在此之前title值不为CR) |
|                                                              | COMMIT;                                                      |
| select * from student where id=12;    （可以看到B修改的title值）ReadView第一次生成 |                                                              |

+ B事务在A事务进行过程中插入新数据(id=p)并提交，虽然A事务是查询(select)不到新数据，但A事务想插入相同新数据(id=p)时却失败了（明明查询显示没有，但却插入失败，报错提示记录id重复）

  >  select * from student **LOCK IN SHARE MODE**; 
  >
  >  select * from student **FOR UPDATE**;
  >
  >  InnoDB提供了这样的机制，在默认的可重复读的隔离级别里，可以使用加锁读去查询最新的数据，如果使用加锁读，是可以读到其他事务中已提交的新增insert数据或delete掉的数据。

| Session A                                                    | Session B                            |
| ------------------------------------------------------------ | ------------------------------------ |
| START TRANSACTION;                                           | START TRANSACTION;                   |
| SELECT * FROM student;  （>>empty set）                      |                                      |
|                                                              | INSERT INTO student VALUES (1, 'a'); |
| SELECT * FROM student;    (>>empty set)                      |                                      |
| INSERT INTO student VALUES (1, 'a');  (>>ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction)  阻塞被锁住，等待一段时间后报错 |                                      |
|                                                              | COMMIT;                              |
| SELECT * FROM student;    (>>empty set)                      |                                      |
| INSERT INTO student VALUES (1, 'a');  (>>Duplicate entry '1' for key 'PRIMARY')  插入失败 |                                      |

+ 事务B事务中A进行过程中插入新数据(id=p)并提交，虽然A事务是查询(select)不到新数据，但A事务进行更新操作时(update/delete)是可以对看不到的新数据(id=p)产生影响的，当A事务对(id=p)新数据进行update时(必须某个值和原来的记录不一样，否则再次查询还是没有多出数据的)，再次查询数据发现多出了一条数据；当A事务对(id=p)新数据进行delete，是会把新数据(id=p)删掉的(A事务提交后)

| Session A                                                    | Session B                            |
| ------------------------------------------------------------ | ------------------------------------ |
| START TRANSACTION;                                           | START TRANSACTION;                   |
| SELECT * FROM student;  （>>empty set）                      |                                      |
|                                                              | INSERT INTO student VALUES (1, 'a'); |
| SELECT * FROM student;    (>>empty set)                      |                                      |
| UPDATE student SET value='b' ;   (>>ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction)  阻塞被锁住，等待一段时间后报错 |                                      |
|                                                              | COMMIT;                              |
| SELECT * FROM student;    (>>empty set)                      |                                      |
| UPDATE student SET value='b'  ;   (>>Rows matched: 1 Changed: 1 Warnings: 0)    **将所有行值替换为b,这会影响到id=1的数据，虽然当前事务并不知道id=1数据的存在；由于value=b和原来value=a不同，则更新成功** |                                      |
| SELECT * FROM student;    (>>多出了更新后的记录，但之前并没插入过) |                                      |

+ 事务B事务中A进行第一次select后过程中删除掉某个数据(id=p)并提交，A事务是可以查询(select)到数据(id=p)，但A事务对数据(id=p)进行update/delete操作时完全不起作用，使用select依旧可以查询到数据(id=p)

事物隔离级别不是设置得越高越好，事物隔离级别设置得越高，意味着势必要花手段去加锁用以保证事物的正确性，那么效率就要降低，因此实际开发中往往要在效率和并发正确性之间做一个取舍，一般情况下会设置为READ_COMMITED，此时避免了脏读，并发性也还不错，之后再通过一些别的手段去解决不可重复读和幻读的问题就好了。

一般来说，隔离级别越低，系统开销越低，可支持的并发越高，但隔离性也越差

InnoDB 存储引擎在分布式事务的情况下一般会用到SERIALIZABLE(可串行化)隔离级别。

#### redo log

通常会在一下三种情况下将重做日志缓冲的内容刷新到外部磁盘的重做日志中：Master Thread每一秒将重做日志缓冲刷新到重做日志文件、每个事务提交时将重做日志缓冲刷新到重做日志文件、当重做日志缓冲池剩余空间小于1/2时，重做日志缓冲刷新到重做日志文件
	redo log采用的是WAL（Write-ahead logging，预写式日志），所有修改先写入日志，再更新到Buffer Pool，保证了数据不会因MySQL宕机而丢失，从而满足了持久性要求。
	既然redo log也需要在事务提交时将日志写入磁盘，为什么它比直接将Buffer Pool中修改的数据写入磁盘(即刷脏)要快呢？主要有以下两方面的原因：

（1）刷脏是随机IO，因为每次修改的数据位置随机，但写redo log是追加操作，属于顺序IO。

（2）刷脏是以数据页（Page）为单位的，MySQL默认页大小是16KB，一个Page上一个小修改都要整页写入；而redo log中只包含真正需要写入的部分，无效IO大大减少。
从重做日志缓冲往磁盘写入时，是按512个字节，也就是一个扇区的大小，因为扇区是写入的最小单位，因此可以保证写入必定是成功的。

#### undo log





### 锁/LBCC/MVCC



> Deadlock found when trying to get lock; try restarting transaction //死锁，事务回滚
>
> Lock wait timeout exceeded; try restarting transaction //等待锁超时，不会回滚

死锁导致InnoDB回滚整个事务，若单是发生了 lock wait timeout则InnoDB仅会回滚事务中等待锁并发生超时的SQL语句。若想在此种情况下回滚整个事务，可通过同时开启 innodb_rollback_on_timeout选项。若是通过START TRANSACTION或BEGIN明确开启事务，则死锁或者锁等待超时导致的回滚并不会关闭当前事务，后续的SQL语句仍会成为当前事务的一部分，除非使用COMMIT，ROLLBACK或者其他隐式提交事务的语句。InnoDB中死锁自动被检测出，并选择代价较小的事务进行回滚以打破死锁。事务完全回滚后其保持的锁被全部释放，若是仅有单条SQL由于错误发生了回滚则语句保持的锁可能不会被释放，因为InnoDB中不保存哪条语句持有哪些锁的信息。若事务中的select调用了存储函数，函数中的SQL执行失败，则该语句被回滚。

> show status like 'innodb_row_lock_%'； //查看锁信息
>
> show variables like 'innodb_lock_wait_timeout';  //查看innodb锁等待时间（默认为50秒）
>
>  show variables like 'innodb_rollback_on_timeout';  //锁超时默认事务不回滚

+ 查看查看等待锁的事务

> select * from sys.innodb_lock_waits

+ 查询进程

> show processlist;

+ 查询是否锁表

> show open tables where in_use>0;

+ 等待锁的语句

> SELECT * FROM performance_schema.events_statements _history WHERE thread_id IN(SELECT b.`THREAD_ID` FROM sys.`innodb_lock_waits` AS a , performance_schema. threads AS b WHERE a.waiting_pid = b.`PROCESSLIST_ID`)
>
> ORDER BY timer_start ASC;

+ 查看持有锁的语句

> SELECT * FROM performance_schema.events_statements_history WHERE thread_id IN(
>
> SELECT b.`THREAD_ID` FROM sys.`innodb_lock_waits` AS a , performance_schema.threads AS b
>
> WHERE a.`blocking_pid` = b.`PROCESSLIST_ID`)
>
> ORDER BY timer_start ASC;

+ 查看当前执行了什么语句

> select * from performance_schema. events_statements_ current\G; 

+ 查看死锁保护机制

> show variables like '%deadlock%'



...表锁在操作数据时会锁定整张表，并发性能较差；行锁则只锁定需要操作的数据，并发性能好。但是由于加锁本身需要消耗资源(获得锁、检查锁、释放锁等都需要消耗资源)，因此在锁定数据较多情况下使用表锁可以节省大量资源。MySQL中不同的存储引擎支持的锁是不一样的，例如MyIsam只支持表锁，而InnoDB同时支持表锁和行锁，且出于性能考虑，绝大多数情况下使用的都是行锁，使用表锁的情况: 当执行的sql(delete、update)中没有使用到索引信息，需要全表扫描才能确认sql执行的行数，此时就会使用表锁。


#### LBCC

​		基于锁的并发控制（Lock Based Concurrency Control）；使用锁的机制，在当前事务需要对数据修改时，将当前事务加上锁，同一个时间只允许一条事务修改当前数据，其他事务必须等待锁释放之后才可以操作。

#### MVCC

​		多版本的并发控制（Multi-Version Concurrency Control）；使用版本来控制并发情况下的数据问题，MVCC使得数据库读不会对数据加锁，普通的SELECT请求不会加锁，提高了数据库的并发处理能力。 借助MVCC，数据库可以实现READ COMMITTED，REPEATABLE READ等隔离级别，用户可以查看当前数据的前一个或者前几个历史版本，保证了ACID中的I特性（隔离性)。

​		InnoDB的MVCC是通过在每行记录后面保存两个隐藏的列来实现的，一个保存了行的事务ID(**DB_TRX_ID**)，一个保存了行的回滚指针(**DB_ROLL_PT**)。每开始一个新的事务，都会自动递增产 生一个新的事务id。事务开始时刻的会把事务id放到当前事务影响的行事务id中，当查询时需要用当前事务id和每行记录的事务id进行比较。在REPEATABLE READ隔离级别下，MVCC的操作如下：

+ select

```
1.InnoDB只查找版本早于当前事务版本的数据行（也就是，行的事务编号小于或等于当前事务的事务编号），这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。
2.删除的行要事务ID判断，读取到事务开始之前状态的版本，只有符合上述两个条件的记录，才能返回作为查询结果。
```

+ insert：InnoDB为新插入的每一行保存当前事务编号作为行版本号
+ delete：InnoDB为删除的每一行保存当前事务编号作为行删除标识
+ update：InnoDB为插入一行新记录，保存当前事务编号作为行版本号，同时保存当前事务编号到原来的行作为行删除标识

​		MVCC使大多数普通读操作都可以不用加锁。这样设计使得读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行。不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作，以及一些额外的维护工作（MVCC只在REPEATABLE READ和READ COMMITIED两个隔离级别下工作。其他两个隔离级别都和 MVCC不兼容 ，因为READ UNCOMMITIED总是读取最新的数据行，而不是符合当前事务版本的数据行。而SERIALIZABLE则会对所有读取的行都加锁）

所谓的MVCC（Multi-Version Concurrency Control ，多版本并发控制）指的就是在使用 **READ COMMITTD** 、**REPEATABLE READ** 这两种隔离级别的事务在执行普通的 SEELCT 操作时访问记录的版本链的过程，这样子可以使不同事务的 `读-写` 、 `写-读` 操作并发执行，从而提升系统性能。READ COMMITTED和REPEATABLE READ隔离级别的的一个区别是它们生成 ReadView 的时机不同。在 READ COMMITTED 中每次查询都会生成一个实时的 ReadView，做到保证每次提交后的数据是处于当前的可见状态。而 REPEATABLE READ 中，在当前事务第一次查询时生成当前的 ReadView，并且当前的 ReadView 会一直沿用到当前事务提交，以此来保证可重复读（REPEATABLE READ）

​		在mysql中MVCC是通过undo log和read view来实现的，undo log分为insert undo log和update undo log

+ insert undo log：insert操作记录只对当前事务本身可见，对于其他事务此记录不可见，所以 insert undo log 可以在事务提交后直接删除而不需要进行purge操作（purge的主要任务是将数据库中已经 mark del 的数据删除，另外也会批量回收undo pages）

  | DB_TRX_ID  | DB_ROOLL_PT | ...  |
  | ---------- | ----------- | ---- |
  | 当前事务id | null        | ...  |

+ update undo log：update 或 delete 操作会对已经存在的记录产生影响，为了提供 MVCC机制，因此update undo log 不能在事务提交时就进行删除，而是将事务提交时放到入 history list 上，等待 purge 线程进行最后的删除操作。

#### Read View

查看死锁的方法有两种： 
通过show engine innodb status命令可以查看最后一个死锁的情况 
通过innodb_print_all_deadlocks参数配置可以将所有死锁的信息都打印到MySQL的错误日志中



减少死锁发生的方法： 

1、尽可能的保持事务小型化，减少事务执行的时间可以减少发生影响的概率 
2、及时执行commit或者rollback，来尽快的释放锁 
3、当要访问多个表数据或者要访问相同表的不同行集合时，尽可能的保证每次访问的顺序是相同的。比如可以将多个语句封装在存储过程中，通过调用同一个存储过程的方法可以减少死锁的发生 
4、增加合适的索引以便语句执行所扫描的数据范围足够小 
5、尽可能的少使用锁，比如如果可以承担幻读的情况，则直接使用select语句，而不要使用select…for update语句 
6、如果没有其他更好的选择，则可以通过施加表级锁将事务执行串行化，最大限度的限制死锁发生

### TPS

计算数据库的TPS

> show global status like '%com_rollback%';
>
> show global status like '%com_commit%';

计算 TPS 的方法时( com_commit + com_rollback）/time 。但是利用这种方法进行计算的前提是：所有的事务必须都是显示提交的，如果存在隐式提交和回滚（默认autocommit =1 ），不会计算到com_commit 和 com_rollback 变量中

mysql 数据库中另外还有两个参数handler_commit 和 handler_rollback 用于事务的统计操作。可以很好的用来统计InnoDB 存储引擎显式和隐式的事务提交操作。

如果用户的程序都是显示控制事务的提交和回滚，那么可以通过com_commit 和 com_rollback 进行统计。







为什么删除掉的记录的id被占用?但是可以通过显示指定插入id重新生成一条记录？自增是通过什么决定的

为什么要使用redolog，而不是在事务提交前直接把数据写入磁盘呢？

CPU速度与磁盘读写速度差距有多大？？？

如何计算数据的TPS

高可靠性、高可用性？？？

完整性约束？？？

information_schema ？？？