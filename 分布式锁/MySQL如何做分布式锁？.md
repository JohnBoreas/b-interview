##### MySQL如何做分布式锁？





在Mysql中创建一张表，设置一个 主键或者UNIQUE KEY 这个 KEY 就是要锁的 KEY（商品ID），所以同一个 KEY 在mysql表里只能插入一次了，这样对锁的竞争就交给了数据库，处理同一个 KEY 数据库保证了只有一个节点能插入成功，其他节点都会插入失败。

DB分布式锁的实现：通过主键id 或者 唯一索性 的唯一性进行加锁，说白了就是加锁的形式是向一张表中插入一条数据，该条数据的id就是一把分布式锁，例如当一次请求插入了一条id为1的数据，其他想要进行插入数据的并发请求必须等第一次请求执行完成后删除这条id为1的数据才能继续插入，实现了分布式锁的功能。

这样 lock 和 unlock 的思路就很简单了，伪代码：

```
def lock ：
    exec sql: insert into locked—table (xxx) values (xxx)
    if result == true :
        return true
    else :
        return false

def unlock ：
    exec sql: delete from lockedOrder where order_id='order_id'
```



问题：

1、数据库的可用性和性能将直接影响分布式锁的可用性及性能，

所以，数据库需要双机部署、数据同步、主备切换；

2、**不具备可重入的特性**

因为同一个线程在释放锁之前，行数据一直存在，无法再次成功插入数据，所以，需要在表中新增一列，用于记录当前获取到锁的机器和线程信息，在再次获取锁的时候，先查询表中机器和线程信息是否和当前机器和线程相同，若相同则直接获取锁；

3、**没有锁失效机制**，因为有可能出现成功插入数据后，服务器宕机了，对应的数据没有被删除，当服务恢复后一直获取不到锁，

所以，需要在表中新增一列，用于记录失效时间，并且需要有定时任务清除这些失效的数据；

4、**不具备阻塞锁特性**，获取不到锁直接返回失败，所以需要优化获取逻辑，循环多次去获取。

5、在实施的过程中会遇到各种不同的问题，为了解决这些问题，实现方式将会越来越复杂；依赖数据库需要一定的资源开销，性能问题需要考虑。