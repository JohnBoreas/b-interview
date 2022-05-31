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



