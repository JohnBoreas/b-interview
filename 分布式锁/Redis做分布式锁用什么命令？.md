##### Redis做分布式锁用什么命令？



答：SETNX，key不存在才设置为value，key存在不做操作





详解：

```shell
## 将 key 的值设为 value ，当且仅当 key 不存在。
setnx key value
```

若给定的 key 已经存在，则 SETNX 不做任何动作，操作失败。

SETNX 是『SET if Not exists』（如果不存在，则 SET）的简写。



加锁：set key value nx ex 10s

释放锁：delete key