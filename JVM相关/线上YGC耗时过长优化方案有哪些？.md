#### 线上YGC耗时过长优化方案有哪些？

1. 如果生命周期过长的对象越来越多（比如全局变量或者静态变量等），会导致标注和复制过程的耗时增加
1. 对存活对象标注时间过长：比如重载了Object类的Finalize方法，导致标注Final Reference耗时过长；或者String.intern方法使用不当，导致YGC扫描StringTable时间过长。可以通过以下参数显示GC处理Reference的耗时-XX:+PrintReferenceGC
1. 长周期对象积累过多：比如本地缓存使用不当，积累了太多存活对象；或者锁竞争严重导致线程阻塞，局部变量的生命周期变长
1. 案例参考： [https://my.oschina.net/lishangzhi/blog/4703942](https://my.oschina.net/lishangzhi/blog/4703942)

