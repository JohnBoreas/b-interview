#### volatile的可见性和禁止指令重排序怎么实现的？

- 可见性：
  volatile的功能就是被修饰的变量在被修改后可以立即同步到主内存，被修饰的变量在每次是用之前都从主内存刷新。本质也是通过内存屏障来实现可见性
  写内存屏障（Store Memory Barrier）可以促使处理器将当前store buffer（存储缓存）的值写回主存。读内存屏障（Load Memory Barrier）可以促使处理器处理invalidate queue（失效队列）。进而避免由于Store Buffer和Invalidate Queue的非实时性带来的问题。
- 禁止指令重排序：
  volatile是通过**内存屏障**来禁止指令重排序
  JMM内存屏障的策略
   - 在每个 volatile 写操作的前面插入一个 StoreStore 屏障。
   - 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障。
   - 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障。
   - 在每个 volatile 读操作的后面插入一个 LoadStore 屏障。

# 