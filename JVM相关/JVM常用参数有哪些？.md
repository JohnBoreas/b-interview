#### JVM常用参数有哪些？

1. **Xms** 是指设定程序启动时占用内存大小。一般来讲，大点，程序会启动的快一点，但是也可能会导致机器暂时间变慢
2. **Xmx** 是指设定程序运行期间最大可占用的内存大小。如果程序运行需要占用更多的内存，超出了这个设置值，就会抛出OutOfMemory异常
3. **Xss** 是指设定每个线程的堆栈大小。这个就要依据你的程序，看一个线程大约需要占用多少内存，可能会有多少线程同时运行等
4. **-Xmn、-XX:NewSize/-XX:MaxNewSize、-XX:NewRatio **
   1. 高优先级：-XX:NewSize/-XX:MaxNewSize 
   1. 中优先级：-Xmn（默认等效 -Xmn=-XX:NewSize=-XX:MaxNewSize=?） 
   1. 低优先级：-XX:NewRatio 
5. 如果想在日志中追踪类加载与类卸载的情况，可以使用启动参数  **-XX:TraceClassLoading -XX:TraceClassUnloading **

