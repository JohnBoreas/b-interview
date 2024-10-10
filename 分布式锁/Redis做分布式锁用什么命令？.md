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



1、Redission代码

```java
//rLock.lock()
// 1. 看门狗机制：锁的自动续期，如果业务超长，运行期间自动给锁续上新的30s，不用担心业务时间长，锁自动过期 被删掉
// 2. 加锁的业务只要运行完成，就不会给当前锁续期，即使不手动解锁，锁默认在30s以后自动删除
// rLock.lock(5, TimeUnit.SECONDS);
// 10s自动解锁，自动解锁时间一定要大于业务的执行时间，没有看门狗机 制
RLock rLock = client.getLock(code);
rLock.lock(5, TimeUnit.SECONDS);
GoodsStore goodsStore = getGoodsStore(code);
if (goodsStore != null) {
    // 业务代码
    //解锁
    rLock.unlock();
    return "恭喜您，购买成功！";
} else {
    //解锁
    rLock.unlock();
    return "获取库存失败。";
}
```

2、Redis代码

```java
//上锁 加锁的时候，值指定为UUID，每个⼈匹配⾃⼰的锁才能删除。
UUID uuid = UUID.randomUUID();
if(!redisLockUtil.lock(code,String.valueOf(uuid))){
	return "排队人数太多，请稍后";
}
try {
	业务代码
} finally {
	//释放锁 值指定为UUID，每个⼈匹配⾃⼰的锁才能删除。
	redisLockUtil.release(code,String.valueOf(uuid));
}

public boolean lock(String code,String uuid) {
    try {
       do {
// 对应setnx命令，可以成功设置,也就是key不存在，获得锁成功，设置过期时间，防止业务逻辑中锁释放失败，导致死锁
//注意占位和过期时间设置要保持原子性
           if (redisTemplate.opsForValue().setIfAbsent(code, uuid,5, TimeUnit.SECONDS)) {
                return true;
            }
        } while (true);
    } catch (Exception e) {
        LOGGER.error(e.getMessage(), e);
    }
    return Boolean.FALSE;
}
public void release(String code, String uuid) {
    try {
        /* String value = (String) redisTemplate.opsForValue().get(code);
         // 高并发场景下，如果由于业务时间⻓，锁⾃⼰过期了，我们直接删除，有可能把别⼈正在持有的锁删除了。
         // 所以我们要验证是否为自己的锁
            if(value != null && uuid !=null && value.equals(uuid)){
                // 删除锁状态
                redisTemplate.opsForValue().getOperations().delete(code);
            }*/
        // 或者我们使用lua脚本来执行，反正就一句话，保证原子性
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] " +
            "then " +
            "return redis.call('del', KEYS[1]) " +
            "else " +
            "return 0 end";
        redisTemplate.execute( new DefaultRedisScript<Integer>(script, Integer.class), Arrays.asList(code),uuid);
    } catch (Exception e) {
        System.out.println("解锁异常");
    }
}
```





