ThreadPoolExecutor

#### 一、ThreadPoolExecutor有6个参数：

corePoolSize：核心线程数

maximumPoolSize：最大线程数

keepAliveTime：存活时间

unit：keepAliveTime的单位。

workQueue：任务队列

threadFactory：线程工厂

RejectedExecutionHandler：拒绝策略

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
          TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory,
          RejectedExecutionHandler handler) {}
```



**（1）四种workQueue：**（任务队列）

```shell
# SynchronousQueue：没有容量

# ArrayBlockingQueue：有界的任务队列

# LinkedBlockingQueue：无界的任务队列

# PriorityBlockingQueue：优先任务队列
```

**（2）四种RejectedExecutionHandler：**（拒绝策略）

```shell
## ThreadPoolExecutor.AbortPolicy：
处理器会在拒绝后抛出一个运行异常RejectedExecutionException。

## ThreadPoolExecutor.CallerRunsPolicy：
线程会调用它直接的execute来运行这个任务。这种方式提供简单的反馈控制机制来减缓新任务提交的速度。

## ThreadPoolExecutor.DiscardPolicy：
不予任何处理，无法执行的任务将被简单得删除掉。

## ThreadPoolExecutor.DiscardOldestPolicy：
如果executor没有处于终止状态，在工作队列头的任务将被删除，然后会重新执行（可能会再次失败，这会导致重复这个过程）。
```





#### 二、corePoolSize（核心线程数）与 maximumPoolSize（最大线程数）区别

1、当 corePoolSize > 活跃线程数时，任务数量的增加，会增加活跃的线程数

2、当 corePoolSize = 活跃线程数时，看workQueue的类型，

​			SynchronousQueue：无界就创建新线程

​			ArrayBlockingQueue：放队列，队列满，创建新线程

​			LinkedBlockingQueue：maximumPoolSize参数无效，最大线程数量就是corePoolSize

3、当 corePoolSize = 最大线程数时，不再增加线程，执行拒绝策略