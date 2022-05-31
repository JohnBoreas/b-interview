#### 线上频繁FullGC优化方案有哪些？

1. 线上频繁FullGC一般会有这么几个特征：
   1. 线上多个线程的CPU都超过了100%，通过jstack命令可以看到这些线程主要是垃圾回收线程
   1. 通过jstat命令监控GC情况，可以看到Full GC次数非常多，并且次数在不断增加
2. 排查流程：
   1. top找到cpu占用最高的一个 **进程id**
   1. 然后 【top -Hp 进程id】，找到cpu占用最高的 **线程id**
   1. 【printf "%x\n" **线程id 】**，假设16进制结果为 a
   1. jstack 线程id | grep '0xa' -A 50 --color
   1. 如果是正常的用户线程， 则通过该线程的堆栈信息查看其具体是在哪处用户代码处运行比较消耗CPU
   1. 如果该线程是 VMThread，则通过 jstat-gcutil命令监控当前系统的GC状况，然后通过 jmapdump:format=b,file=导出系统当前的内存数据。导出之后将内存情况放到eclipse的mat工具中进行分析即可得出内存中主要是什么对象比较消耗内存，进而可以处理相关代码；正常情况下会发现VM Thread指的就是垃圾回收的线程
   1. 再执行【jstat -gcutil  **进程id】, **看到结果，如果FGC的数量很高，且在不断增长，那么可以定位是由于内存溢出导致FullGC频繁，系统缓慢
   1. 然后就可以Dump出内存日志，然后使用MAT的工具分析哪些对象占用内存较大，然后找到对象的创建位置，处理即可
3. 参考案例：[https://mp.weixin.qq.com/s/g8KJhOtiBHWb6wNFrCcLVg](https://mp.weixin.qq.com/s/g8KJhOtiBHWb6wNFrCcLVg)
   