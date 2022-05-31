#### 如何进行GC日志分析？

为了方便分析GC日志信息，可以指定启动参数 【-Xloggc: app-gc.log  -XX:+PrintGCDetails -XX:+PrintGCDateStamps】,方便详细地查看GC日志信息

1. 使用 【jinfo pid】查看当前JVM堆的相关参数
1. 继续使用 【jstat -gcutil 2315 1s 10】查看10s内当前堆的占用情况
1. 也可以使用【jmap -heap pid】查看当前JVM堆的情况
1. 我们可以继续使用 【jmap -F -histo pid | head -n 20】，查看前20行打印，即查看当前top20的大对象，一般从这里可以发现一些异常的大对象，如果没有，那么可以继续排名前50的大对象，分析
1. 最后使用【jmap -F -dump:file=a.bin pid】，如果dump文件很大，可以压缩一下【tar -czvf a.tar.gz a.bin】
1. 再之后，就是对dump文件进行分析了，使用MAT分析内存泄露
1. 参考案例： [https://www.lagou.com/lgeduarticle/142372.html](https://www.lagou.com/lgeduarticle/142372.html)