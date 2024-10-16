##### 基于 ZooKeeper 的分布式锁实现原理是什么?

顺序节点特性：

使用 ZooKeeper 的顺序节点特性，假如我们在/lock/目录下创建3个节点，ZK集群会按照发起创建的顺序来创建节点，节点分别为/lock/0000000001、/lock/0000000002、/lock/0000000003，最后一位数是依次递增的，节点名由zk来完成。



临时节点特性：

ZK中还有一种名为临时节点的节点，临时节点由某个客户端创建，当客户端与ZK集群断开连接，则该节点自动被删除。EPHEMERAL_SEQUENTIAL为临时顺序节点。

根据ZK中节点是否存在，可以作为分布式锁的锁状态，以此来实现一个分布式锁，下面是分布式锁的基本逻辑：

1. 客户端1调用create()方法创建名为“/业务ID/lock-”的临时顺序节点。
2. 客户端1调用getChildren(“业务ID”)方法来获取所有已经创建的子节点。
3. 客户端获取到所有子节点path之后，如果发现自己在步骤1中创建的节点是所有节点中序号最小的，就是看自己创建的序列号是否排第一，如果是第一，那么就认为这个客户端1获得了锁，在它前面没有别的客户端拿到锁。
4. 如果创建的节点不是所有节点中需要最小的，那么则监视比自己创建节点的序列号小的最大的节点，进入等待。直到下次监视的子节点变更的时候，再进行子节点的获取，判断是否获取锁。



优点：具备高可用、可重入、阻塞锁特性，可解决失效死锁问题。

缺点：因为需要频繁的创建和删除节点，性能上不如Redis方式。



代码：

```java
// Curator zookeeper客户端
@Autowired
private CuratorFramework client;

InterProcessMutex lock = new InterProcessMutex(client, "/"+code);
try {
    // 创建znode节点，可重入锁
    // lock.acquire(5,TimeUnit.MINUTES) --->类似于redisson设置过期时间，时间一到不可重入
    // lock.acquire();----》类似于Redisson不设置过期时间，一直等待
    lock.acquire();
    /*if(lock.acquire(60,TimeUnit.MINUTES)){
                Stat stat = client.checkExists().forPath("/"+code);
                if (null == stat){
                    return "排队人数太多，请稍后";
                }
            }*/
} catch (Exception e) {
    e.printStackTrace();
}
try {
    // 业务代码
} catch (Exception ex) {
    ex.printStackTrace();
} finally {
    // 锁的线程存在，则释放删除znode，判断一下避免删除其他线程的znode
    if (lock.isAcquiredInThisProcess()) {
        try {
            lock.release();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



