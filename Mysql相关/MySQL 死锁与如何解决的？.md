#### MySQL 遇到过死锁问题吗，你是如何解决的？



死锁的四个条件：

1、互斥条件：线程(进程)对于所分配到的资源具有排它性，即一个资源只能被一个线程(进程)占用，直到被该线程(进程)释放
2、请求与保持条件：一个线程(进程)因请求被占用资源而发生阻塞时，对已获得的资源保持不放。
3、不剥夺条件：线程(进程)已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4、循环等待条件：当发生死锁时，所等待的线程(进程)必定会形成一个环路（类似于死循环），造成永久阻塞



产生death lock原理

insert ... on duplicate key 在执行时，innodb引擎会先判断插入的行是否产生重复key错误，如果存在，在对该现有的行加上S（共享锁）锁，如果返回该行数据给mysql,然后mysql执行完duplicate后的update操作，然后对该记录加上X（排他锁），最后进行update写入。



批处理死锁：

更新数据的时候，其中有部分数据锁加不上去，只能等待，就发生了阻塞

如果A进程阻塞了B，B又阻塞了A，就变成了死锁，估计有至少2个进程，同时在更新同一批数据，才会发生这种情况

同一批数据里要是2条相同的数据也会出现这种情况，因为会去更新同一条数据，一般来说会等待，超时就被牺牲掉了，或者运气不好升级为死锁，也可能被牺牲掉



避免死锁

1. 对**索引加锁顺序的不一致**很可能会导致死锁，所以如果可以，尽量以相同的顺序来访问索引记录和表。在程序以批量方式处理数据的时候，如果事**先对数据排序**，保证每个线程按固定的顺序来处理记录，也可以大大降低出现死锁的可能；
2. **Gap 锁**往往是程序中导致死锁的真凶，由于默认情况下 MySQL 的隔离级别是 RR，所以如果能确定幻读和不可重复读对应用的影响不大，可以考虑将**隔离级别改成 RC**，可以避免 Gap 锁导致的死锁；
3. 为表添加**合理的索引**，如果不走索引将会为表的每一行记录加锁，死锁的概率就会大大增大；
4. 我们知道 MyISAM 只支持表锁，它采用一次封锁技术来保证事务之间不会发生死锁，所以，我们也可以使用同样的思想，在事务中**一次锁定所需要的所有资源**，减少死锁概率；
5. **避免大事务**，尽量将大事务拆成多个小事务来处理；因为大事务占用资源多，耗时长，与其他事务冲突的概率也会变高；
6. 避**免在同一时间点运行多个对同一表进行读写的脚本**，特别注意加锁且操作数据量比较大的语句；我们经常会有一些定时脚本，避免它们在同一时间点运行；
7. 设置锁等待超时参数：`innodb_lock_wait_timeout`，这个参数并不是只用来解决死锁问题，在并发访问比较高的情况下，如果大量事务因无法立即获得所需的锁而挂起，会占用大量计算机资源，造成严重性能问题，甚至拖跨数据库。我们通过设置合适的锁等待超时阈值，可以避免这种情况发生。



### mysql文档对死锁的一个处理：

`InnoDB`使用自动行级锁定。即使在仅插入或删除单行的事务的情况下，您也可能会出现死锁。那是因为这些操作并不是真正的“原子”；它们会自动对插入或删除的行的（可能是多个）索引记录设置锁定。



使用以下技术应对死锁并降低其发生的可能性：

- 随时发出问题 [`SHOW ENGINE INNODB STATUS`](https://dev.mysql.com/doc/refman/5.6/en/show-engine.html)以确定最近死锁的原因。这可以帮助您调整应用程序以避免死锁。

- [`innodb_print_all_deadlocks`](https://dev.mysql.com/doc/refman/5.6/en/innodb-parameters.html#sysvar_innodb_print_all_deadlocks) 如果频繁的死锁警告引起关注，请通过启用该变量 来收集更广泛的调试信息 。有关每个死锁的信息，而不仅仅是最新的，都记录在 MySQL [错误日志](https://dev.mysql.com/doc/refman/5.6/en/glossary.html#glos_error_log)中。完成调试后禁用此选项。

- 如果由于死锁而失败，请始终准备好重新发出事务。死锁并不危险。再试一次。

- **保持交易小且持续时间短**，以减少它们发生冲突的可能性。

- 在进行一组相关**更改后立即提交事务**，以降低它们发生冲突的可能性。特别是，不要让交互式 [**mysql**](https://dev.mysql.com/doc/refman/5.6/en/mysql.html)会话在未提交事务的情况下长时间打开。

- 如果您使用[锁定读取](https://dev.mysql.com/doc/refman/5.6/en/glossary.html#glos_locking_read)（[`SELECT ... FOR UPDATE`](https://dev.mysql.com/doc/refman/5.6/en/select.html)或 [`SELECT ... LOCK IN SHARE MODE`](https://dev.mysql.com/doc/refman/5.6/en/select.html)），请尝试使用较低的隔离级别，例如[`READ COMMITTED`](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-isolation-levels.html#isolevel_read-committed).

- 当修改一个事务中的多个表，或同一个表中的不同行集时，每次都**以一致的顺序执行**这些操作。然后事务形成定义明确的队列并且不会死锁。例如，将数据库操作组织成应用程序中的函数，或调用存储的例程，而不是在不同的地方编写多个相似的 `INSERT`、`UPDATE`和 `DELETE`语句序列。

- 为您的表添加精心挑选的**索引**，以便您的查询**扫描更少的索引记录并设置更少的锁**。用于 [`EXPLAIN SELECT`](https://dev.mysql.com/doc/refman/5.6/en/explain.html)确定 MySQL 服务器认为哪些索引最适合您的查询。

- **使用较少的锁定**。如果您有能力允许 a [`SELECT`](https://dev.mysql.com/doc/refman/5.6/en/select.html)从旧快照返回数据，请不要在其中添加`FOR UPDATE`or `LOCK IN SHARE MODE`子句。在这里使用[`READ COMMITTED`](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-isolation-levels.html#isolevel_read-committed) 隔离级别很好，因为同一事务中的每个一致读取都从其自己的新快照中读取。

- 如果没有其他帮助，请使用表级锁序列化您的事务。[`LOCK TABLES`](https://dev.mysql.com/doc/refman/5.6/en/lock-tables.html)使用事务表（例如表）的正确方法是使用 (not ) 后跟`InnoDB` 开始事务，并且在显式提交事务之前不要调用 。例如，如果您需要写入 table 并从 table 读取 ，您可以这样做： `SET autocommit = 0`[`START TRANSACTION`](https://dev.mysql.com/doc/refman/5.6/en/commit.html)[`LOCK TABLES`](https://dev.mysql.com/doc/refman/5.6/en/lock-tables.html)[`UNLOCK TABLES`](https://dev.mysql.com/doc/refman/5.6/en/lock-tables.html)`t1``t2`

  ```sql
  SET autocommit=0;
  LOCK TABLES t1 WRITE, t2 READ, ...;
  ... do something with tables t1 and t2 here ...
  COMMIT;
  UNLOCK TABLES;
  ```

  表级锁可防止对表的并发更新，避免死锁，但会降低对繁忙系统的响应速度。

- 序列化事务的另一种方法是创建一个仅包含一行的辅助“信号量”表。让每个事务在访问其他表之前更新该行。这样，所有交易都以串行方式发生。请注意，`InnoDB` 即时死锁检测算法也适用于这种情况，因为序列化锁是行级锁。使用 MySQL 表级锁，必须使用 timeout 方法来解决死锁。