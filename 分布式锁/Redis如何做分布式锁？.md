

##### Redis如何做分布式锁？

答：



假设有两个服务A、B都希望获得锁，执行过程大致如下:

Step1： 服务A为了获得锁，向Redis发起如下命令: SET productId:lock 0xx9p03001 NX EX 30000 其中，"productId"由自己定义，可以是与本次业务有关的id，"0xx9p03001"是一串随机值，必须保证全局唯一，“NX"指的是当且仅当key(也就是案例中的"productId:lock”)在Redis中不存在时，返回执行成功，否则执行失败。"EX 30000"指的是在30秒后，key将被自动删除。执行命令后返回成功，表明服务成功的获得了锁。

Step2: 服务B为了获得锁，向Redis发起同样的命令: SET productId:lock 0000111 NX  EX 30000
由于Redis内已经存在同名key，且并未过期，因此命令执行失败，服务B未能获得锁。服务B进入循环请求状态，比如每隔1秒钟(自行设置)向Redis发送请求，直到执行成功并获得锁。

Step3: 服务A的业务代码执行时长超过了30秒，导致key超时，因此Redis自动删除了key。此时服务B再次发送命令执行成功，假设本次请求中设置的value值为0000222。此时需要在服务A中对key进行续期，watch dog。

Step4: 服务A执行完毕，为了释放锁，服务A会主动向Redis发起删除key的请求。注意: 在删除key之前，一定要判断服务A持有的value与Redis内存储的value是否一致。比如当前场景下，Redis中的锁早就不是服务A持有的那一把了，而是由服务2创建，如果贸然使用服务A持有的key来删除锁，则会误将服务2的锁释放掉。此外，由于删除锁时涉及到一系列判断逻辑，因此一般使用lua脚本，具体如下:

```sql
if redis.call("get", KEYS[1])==ARGV[1] then
	return redis.call("del", KEYS[1])
else
	return 0
end
```

