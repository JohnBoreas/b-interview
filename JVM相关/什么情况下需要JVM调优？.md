#### 什么情况下需要JVM调优？


- Heap内存（老年代）持续上涨达到设置的最大内存值
- Full GC 次数频繁
- GC 停顿（Stop World）时间过长（超过1秒，具体值按应用场景而定）
- 应用出现OutOfMemory 等内存异常
- 应用出现OutOfDirectMemoryError等内存异常（ failed to allocate 16777216 byte(s) of direct memory (used: 1056964615, max: 1073741824)）
- 应用中有使用本地缓存且占用大量内存空间
- 系统吞吐量与响应性能不高或下降
- 应用的CPU占用过高不下或内存占用过高不下

