		多版本控制（Multiversion Concurrency Control）是一种提高并发的技术，一般情况下，只有读读之间可以并发，读写，写写都会阻塞，但是引入多版本可以实现只有写写之间相互阻塞，其他两种操作都可以并行，这样提高了InnoDB的并发度。在内部实现中，InnoDB通过undo log保存每条数据的多个版本，并且能够找回数据历史版本提供给用户读，每个事务读到的数据版本可能是不一样的。在同一个事务中，用户只能看到该事务创建快照之前已经提交的修改和该事务本身做的修改。

​		MVCC只在 Read Committed 和 Repeatable Read两个隔离级别下工作。其他两个隔离级别和MVCC不兼容，Read Uncommitted总是读取最新的记录行，而不是符合当前事务版本的记录行；Serializable 则会对所有读取的记录行都加锁。

​		MySQL的InnoDB存储引擎默认事务隔离级别是RR(可重复读)，是通过 "行级锁+MVCC"一起实现的，正常读的时候不加锁，写的时候加锁。而 MVCC 的实现依赖：隐藏字段、Read View、Undo log。

#### 隐藏字段

 InnoDB存储引擎在每行数据的后面添加了三个隐藏字段：

        1. DB_TRX_ID(6字节)：表示最近一次对本记录行作修改（insert | update）的事务ID。至于delete操作，InnoDB认为是一个update操作，不过会更新一个另外的删除位，将行表示为deleted。并非真正删除。
    
        2. DB_ROLL_PTR(7字节)：回滚指针，指向当前记录行的undo log信息
    
        3. DB_ROW_ID(6字节)：随着新行插入而单调递增的行ID。理解：当表没有主键或唯一非空索引时，innodb就会使用这个行ID自动产生聚簇索引。如果表有主键或唯一非空索引，聚簇索引就不会包含这个行ID了。这个DB_ROW_ID跟MVCC关系不大。
#### Read View

Read View主要是用来做可见性判断的, 里面保存了“对本事务不可见的其他活跃事务”。

```c++
struct read_view_t{
	ulint		type;	/*!< VIEW_NORMAL, VIEW_HIGH_GRANULARITY */
	undo_no_t	undo_no;/*!< 0 or if type is
				VIEW_HIGH_GRANULARITY
				transaction undo_no when this high-granularity
				consistent read view was created */
	trx_id_t	low_limit_no;
				/*!< The view does not need to see the undo
				logs for transactions whose transaction number
				is strictly smaller (<) than this value: they
				can be removed in purge if not needed by other
				views */
	trx_id_t	low_limit_id;
				/*!< The read should not see any transaction
				with trx id >= this value. In other words,
				this is the "high water mark". */
	trx_id_t	up_limit_id;
				/*!< The read should see all trx ids which
				are strictly smaller (<) than this value.
				In other words,
				this is the "low water mark". */
	ulint		n_trx_ids;
				/*!< Number of cells in the trx_ids array */
	trx_id_t*	trx_ids;/*!< Additional trx ids which the read should
				not see: typically, these are the read-write
				active transactions at the time when the read
				is serialized, except the reading transaction
				itself; the trx ids in this array are in a
				descending order. These trx_ids should be
				between the "low" and "high" water marks,
				that is, up_limit_id and low_limit_id. */
	trx_id_t	creator_trx_id;
				/*!< trx id of creating transaction, or
				0 used in purge */
	UT_LIST_NODE_T(read_view_t) view_list;
				/*!< List of read views in trx_sys */
};
```

+ low_limit_id：目前出现过的最大事务id+1，即下一个将被分配的事务ID（low_limit_id 并不是活跃事务列表中最大的事务ID）

```c++
	trx_id_t	max_trx_id;	/*!< The smallest number not yet
					assigned as a transaction id or
					transaction number */
	view->low_limit_no = trx_sys->max_trx_id;
	view->low_limit_id = view->low_limit_no;
```

+ up_limit_id：活跃事务列表trx_ids中的最小事务id，如果trx_ids为空，则up_limit_id 为 low_limit_id（trx_ids中的活跃事务号是逆序的，所以最后一个为最小活跃事务ID)

```c++
	if (view->n_trx_ids > 0) {
		/* The last active transaction has the smallest id: */
		view->up_limit_id = view->trx_ids[view->n_trx_ids - 1];
	} else {
		view->up_limit_id = view->low_limit_id;
	}
```

+ trx_ids：Read View创建时其他未提交的活跃事务ID列表。意思就是创建Read View时，将当前未提交事务ID记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。（Read View中trx_ids的活跃事务，不包括当前事务自己和已提交的事务（正在内存中））

```c++
		if (trx->id != m_view->creator_trx_id
		    && !trx_state_eq(trx, TRX_STATE_COMMITTED_IN_MEMORY)) {

			ut_ad(m_n_trx > m_view->n_trx_ids);

			m_view->trx_ids[m_view->n_trx_ids++] = trx->id;

			/* NOTE that a transaction whose trx number is <
			trx_sys->max_trx_id can still be active, if it is
			in the middle of its commit! Note that when a
			transaction starts, we initialize trx->no to
			TRX_ID_MAX. */

			/* trx->no is protected by trx_sys->mutex, which
			we are holding. It is assigned by trx_commit()
			before lock_trx_release_locks() assigns
			trx->state = TRX_STATE_COMMITTED_IN_MEMORY. */

			if (m_view->low_limit_no > trx->no) {
				m_view->low_limit_no = trx->no;
			}
		}
```

+ creator_trx_id：当前创建事务的ID，是一个递增的编号

```c++
	trx_id_t	cr_trx_id,	/*!< in: trx_id of creating
					transaction, or 0 used in purge */
	view->creator_trx_id = cr_trx_id;
```

#### Undo log

Undo log中存储的是老版本数据，当一个事务需要读取记录行时，如果当前记录行不可见，可以顺着undo log链找到满足其可见性条件的记录行版本。大多数对数据的变更操作包括 insert/update/delete，在InnoDB里，undo log分为如下两类：
    ①insert undo log : 事务对insert新记录时产生的undo log, 只在事务回滚时需要, 并且在事务提交后就可以立即丢弃。
    ②update undo log : 事务对记录进行delete和update操作时产生的undo log，不仅在事务回滚时需要，快照读也需要，只有当数据库所使用的快照中不涉及该日志记录，对应的回滚日志才会被purge线程删除。

```
	Purge线程：为了实现InnoDB的MVCC机制，更新或者删除操作都只是设置一下旧记录的deleted_bit，并不真正将旧记录删除。
    为了节省磁盘空间，InnoDB有专门的purge线程来清理deleted_bit为true的记录。purge线程自己也维护了一个read view，如果某个记录的deleted_bit为true，并且DB_TRX_ID相对于purge线程的read view可见，那么这条记录一定是可以被安全清除的。
```

#### read view可见性

​		在innodb中，创建一个新事务后，执行第一个select语句的时候，innodb会创建一个快照（read view），快照中会保存系统当前不应该被本事务看到的其他活跃事务id列表（即trx_ids）。当用户在这个事务中要读取某个记录行的时候，innodb会将该记录行的DB_TRX_ID与该Read View中的一些变量进行比较，判断是否满足可见性条件。

​		假设当前事务要读取某一个记录行，该记录行的DB_TRX_ID（即最新修改该行的事务ID）为trx_id，Read View的活跃事务列表trx_ids中最早的事务ID为up_limit_id，将在生成这个Read Vew时系统出现过的最大的事务ID+1记为low_limit_id（即还未分配的事务ID）

**(low_limit_id，up_limit_d]**

+ trx_id < up_limit_id：表明“最新修改该行的事务”在“当前事务”创建快照之前就提交了，所以该记录行的值对当前事务是可见的
+ trx_id >= low_limit_id：表明“最新修改该行的事务”在“当前事务”创建快照之后才修改该行，所以该记录行的值对当前事务不可见，在该记录行的 DB_ROLL_PTR 指针所指向的undo log回滚段中，寻找下一个更久远的旧事务号DB_TRX_ID
+ low_limit_id > trx_id >= up_limit_id：表明“最新修改该行的事务”在“当前事务”创建快照的时候可能处于“活动状态”或者“已提交状态”；则需对活跃事务列表trx_ids进行查找（源码中是用的二分查找，因为是有序的）
  + trx_id∈trx_ids：表明在“当前事务”创建快照前，“该记录行的值”被“id为trx_id的事务”修改了，但没有提交；或者在“当前事务”创建快照后，“该记录行的值”被“id为trx_id的事务”修改了（不管有无提交）；这些情况下，这个记录行的值对当前事务都是不可见的
  + trx_id∉trx_ids：表明“id为trx_id的事务”在修改“该记录行的值”后，在“当前事务”创建快照前就已经提交了，所以记录行对当前事务可见





​    **快照读(snapshot read)**：普通的 select 语句(不包括 select ... lock in share mode, select ... for update)

​    **当前读(current read)** ：select ... lock in share mode，select ... for update，insert，update，delete 语句（这些语句获取的是数据库中的**最新数据**）

​    **只靠 MVCC 实现RR隔离级别，可以保证可重复读，还能防止部分幻读，但并不是完全防止**

​    比如事务A开始后，执行普通select语句，创建了快照；之后事务B执行insert语句；然后事务A再执行普通select语句，得到的还是之前B没有insert过的数据，因为这时候A读的数据是符合快照可见性条件的数据。这就防止了**部分**幻读，此时事务A是**快照读**。

​    但是，如果事务A执行的不是普通select语句，而是select ... for update等语句，这时候，事务A是**当前读**，每次语句执行的时候都是获取的最新数据。也就是说，**在只有MVCC时**，A先执行 select ... where nid between 1 and 10 … for update；然后事务B再执行 insert … nid = 5 …；然后 A 再执行 select ... where nid between 1 and 10 … for update，就会发现，多了一条B insert进去的记录。这就产生幻读了，所以单独靠MVCC并不能完全防止幻读。

​    因此，InnoDB在实现RR隔离级别时，不仅使用了MVCC，还会对“当前读语句”读取的记录行加记录锁（record lock）和间隙锁（gap lock），禁止其他事务在间隙间插入记录行，来防止幻读。也就是前文说的"行级锁+MVCC"。





#### RR和RC的Read View产生区别

​    ①在innodb中的**Repeatable Read**级别, 只有事务在begin之后，执行**第一条**select（读操作）时, 才会创建一个快照(read view)，将当前系统中活跃的其他事务记录起来；并且事务之后都是使用的这个快照，不会重新创建，直到事务结束。

​    ②在innodb中的**Read Committed**级别, 事务在begin之后，执行**每条**select（读操作）语句时，快照会被重置，即会重新创建一个快照(read view)

 快照读也就是普通select语句，它们不会对访问的数据加锁。   只有普通select语句才会创建快照，select ... lock in share mode，select ... for update不会，update、delete、insert语句也不会，因为它们都是当前读，会对访问的数据加锁。



#### 锁

​		锁（Locking）是为了支持对共享资源进行并发访问时保证数据一致性和完整性的主要机制。在InnoDB中有latch和lock，latch是一种轻量级锁，锁定的对象是线程，通常要求锁定的时间非常短，latch又可以分为mutex(互斥量)和rwlock(读写锁)，目的是用来保证并发线程操作临界资源的正确性，并且通常没有死锁检测的机制。lock的对象是事务，用来锁定数据库中的行、表、页等对象，一般在事务commit或rollback后进行释放，通常有死锁检测机制与处理。

> show engine innodb mutex

MySQL 中的锁可以按照粒度分为**锁定整个表的表级锁**（table-level locking）和**锁定数据行的行级锁**（row-level locking）：

- 表级锁具有开销小、加锁快的特性；表级锁的锁定粒度较大，发生锁冲突的概率高，支持的并发度低；
- 行级锁具有开销大，加锁慢的特性；行级锁的锁定粒度较小，发生锁冲突的概率低，支持的并发度高。

InnoDB 存储引擎同时支持行级锁（row-level locking）和表级锁（table-level locking），默认情况下采用行级锁。

> 表级锁适用于并发较低、以查询为主的应用，例如中小型的网站；MyISAM 和 MEMORY 存储引擎采用表级锁。
> 行级锁适用于按索引条件高并发更新少量不同数据，同时又有并发查询的应用，例如 OLTP 系统；InnoDB 和 NDB 存储引擎实现了行级锁。

#### 行锁

InnoDB 实现了以下两种类型的**行锁**：

- **共享锁**（S）：允许获得该锁的事务读取数据行（读锁），同时允许其他事务获得该数据行上的共享锁，并且阻止其他事务获得数据行上的排他锁

  > select ... lock in share mode

- **排他锁**（X）：允许获得该锁的事务更新或删除数据行（写锁），同时阻止其他事务取得该数据行上的共享锁和排他锁。

  > select ... for update

**共享锁和共享锁可以兼容，排他锁和其它锁都不兼容**。例如，事务 A 获取了一行数据的共享锁，事务 B 可以立即获得该数据行的共享锁，也就是锁兼容；但是此时事务 B 如果想获得该数据行的排他锁，则必须等待事务 A 释数据行上的共享锁，此种情况存在锁冲突。

InnoDB 通过给索引上的索引记录加锁的方式实现行级锁。具体来说，InnoDB 实现了三种行锁的算法：记录锁（Record Lock）、间隙锁（Gap Lock）和 Next-key 锁（Next-key Lock）。

+ Record Lock：单个行记录上锁。记录锁是针对索引记录（index record）的锁定。例如，SELECT * FROM t WHERE id = 1 FOR UPDATE;会阻止其他事务对表 t 中 id = 1 的数据执行插入、更新，以及删除操作。

  ```mysql
  记录锁总是去锁定索引记录，锁定非聚集索引会先锁定聚集索引。如果表中没有定义索引，InnoDB 默认为表创建一个隐藏的聚簇索引，并且使用该索引锁定记录。
  ```

+ Gap Lock：锁定一个范围，但不包含记录本身。GAP锁的目的，是为了防止同一事务的两次当前读，出现幻读的情况。间隙锁（Gap Lock）锁定的是索引记录之间的间隙、第一个索引之前的间隙或者最后一个索引之后的间隙。例如，SELECT * FROM t WHERE c1 BETWEEN 1 and 10 FOR UPDATE;会阻止其他事务将 1 到 10 之间的任何值插入到 c1 字段中，即使该列不存在这样的数据；因为这些值都会被锁定。

  ```
  索引有1,5,10,15,20,25
  select * from user where id between 10 and 15 lock in share mode
  GAP LOCK锁定的范围是(5,10)(10,15)(15,20)，那么以下语句是可以插入成功的
  insert into user(id) values(4);
  insert into user(id) values(22);
  ```

+ Next-Key Lock：gap lock+recode lock，锁定一个范围并且锁定记录本身。当查询的索引含有唯一属性时，InnoDB存储会对Next-Key Lock进行优化，将其降级为Record Lock，即仅锁住索引本身，而不是范围

  ```
  索引有1,5,10,15,20,25
  select * from user where id between 10 and 15 lock in share mode
  以上sql锁定的范围是(5,10](10,15](15,20]，那么以下语句是可以插入成功的
  insert into user(id) values(4);
  insert into user(id) values(22);
  ```

1. 只使用唯一索引查询，并且只锁定一条记录时，innoDB会使用行锁。
2. 只使用唯一索引查询，但是检索条件是范围检索，或者是唯一检索然而检索结果不存在（试图锁住不存在的数据）时，会产生 Next-Key Lock。
3. 使用普通索引检索时，不管是何种查询，只要加锁，都会产生间隙锁。
4. 同时使用唯一索引和普通索引时，由于数据行是优先根据普通索引排序，再根据唯一索引排序，所以也会产生间隙锁。

全局变量 innodb_status_output 和 innodb_status_output_locks 用于控制 InnoDB 标准监控和锁监控，我们利用监控查看锁的使用情况。

> SET GLOBAL innodb_status_output=ON
>
> SET GLOBAL innodb_status_output_locks=ON
>
> SHOW ENGINE INNODB STATUS  //查看 InnoDB 监控中关于锁的事务数据

#### 表锁

​		InnoDB 除了支持行级锁，还支持表锁：由 MySQL 服务层实现的表级锁（`LOCK TABLES ... WRITE`在指定的表加上表级排他锁）和自己实现的意向锁。当行、表两种锁同时存在时，可能导致冲突。例如，事务 A 获取了表中一行数据的读锁；然后事务 B 申请该表的写锁（例如修改表的结构）。如果事务 B 加锁成功，那么它就应该能修改表中的任意数据行，但是 A 持有的行锁不允许修改锁定的数据行。显然数据库需要避免这种问题，B 的加锁申请需要等待 A 释放行锁。那么如何判断事务 B 是否应该获取表级锁呢？首先需要看该表是否已经被其他事务加上了表级锁，然后依次查看该表中的每一行是否已经被其他事务加上了行级锁。这种方式需要遍历整个表中的记录，效率很低。为此，InnoDB 引入了另外一种锁：意向锁（Intention Lock），意向锁的主要作用是表明某个事务正在或者即将想要锁定表中的数据行。意向锁属于表级锁，由 InnoDB 自动添加，不需要用户干预。意向锁也分为共享和排他两种方式：

+ 意向共享锁（IS）：事务在给数据行加行级共享锁之前，必须先取得该表的 IS 锁。

+ 意向排他锁（IX）：事务在给数据行加行级排他锁之前，必须先取得该表的 IX 锁。

此时，事务 A 必须先申请该表的意向共享锁，成功后再申请数据行的行锁。事务 B 申请表锁时，数据库查看该表是否已经被其他事务加上了表级锁；如果发现该表上存在意向共享锁，说明表中某些数据行上存在共享锁，事务 B 申请的写锁会被阻塞。因此，意向锁是为了使得行锁和表锁能够共存，从而实现多粒度的锁机制。

|              | 共享锁S  | 排他锁X  | 意向共享锁IS | 意向排他锁IX |
| ------------ | -------- | -------- | ------------ | ------------ |
| 共享锁S      | 兼容     | **冲突** | 兼容         | **冲突**     |
| 排他锁X      | **冲突** | **冲突** | **冲突**     | **冲突**     |
| 意向共享锁IS | 兼容     | **冲突** | 兼容         | 兼容         |
| 意向排他锁IX | **冲突** | **冲突** | 兼容         | 兼容         |

> InnoDB 表存在两种表级锁，一种是`LOCK TABLES`语句手动指定的锁，另一种是由 InnoDB 自动添加的意向锁。意向锁和表锁之间只有共享锁兼容，意向锁和意向锁之间都可以兼容

> LOCK TABLES student READ;  //为student表加上共享锁
>
> LOCK TABLES student WRITE;  //为student表加上排他锁
>
> UNLOCK TABLES;  //主动释放表锁