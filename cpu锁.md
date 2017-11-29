---
title: java锁机制简述
tags: cpu,lock
grammar_cjkRuby: true
---

# 简述
很多同学应该在开发中，经常会用到java的锁，多线程开发中，会用到同步队列，但很多同学不知道为什么这些锁类能保证共享资源的安全性。
![enter description here][1]
上面这些就是我们java中常见的锁类
深入其中的源码你会发现，基本通过AbstractQueuedSynchronizer的compareAndSetState方法实现的

![enter description here][3]

看compareAndSetState方法的实现，是通过sun.misc.Unsafe的compareAndSwapInt来实现，这个方法可以保证原则操作
我们继续深入发现compareAndSwapInt是本地方法，找到对应的c++源码





# Lock
> &ensp;&ensp;在早期的单处理系统中，单条指令的操作都可以认为是原子的，但在多处理系统中，情况就不一样了，即使单条指令的操作可能也会被干扰，所以出现了Lock指令，lock指令的作用就是保证当前指令的原子性


> 当cpu执行 Lock指令时，会告诉总线，访问的内存是排他性的，其他cpu就不能操作这个内存了。说白了，lock保证当前指令执行期间，访问的内存是排他性的


	
###  intel的手册对lock前缀的说明如下：

> &ensp;&ensp;1.	确保对内存的读-改-写操作原子执行。在Pentium及Pentium之前的处理器中，带有lock前缀的指令在执行期间会锁住总线，使得其他处理器暂时无法通过总线访问内存。很显然，这会带来昂贵的开销。从Pentium 4，Intel Xeon及P6处理器开始，intel在原有总线锁的基础上做了一个很有意义的优化：如果要访问的内存区域（area of memory）在lock前缀指令执行期间已经在处理器内部的缓存中被锁定（即包含该内存区域的缓存行当前处于独占或以修改状态），并且该内存区域被完全包含在单个缓存行（cache line）中，那么处理器将直接执行该指令。由于在指令执行期间该缓存行会一直被锁定，其它处理器无法读/写该指令要访问的内存区域，因此能保证指令执行的原子性。这个操作过程叫做缓存锁定（cache locking），缓存锁定将大大降低lock前缀指令的执行开销，但是当多处理器之间的竞争程度很高或者指令访问的内存地址未对齐时，仍然会锁住总线。


> &ensp;&ensp;2.	禁止该指令与之前和之后的读和写指令重排序。


> &ensp;&ensp;3.	把写缓冲区中的所有数据刷新到内存中。


#  cpu锁

### 1.  处理器自动保证基本内存操作的原子性
> &ensp;&ensp;首先处理器会自动保证基本的内存操作的原子性。处理器保证从系统内存当中读取或者写入一个字节是原子的，意思是当一个处理器读取一个字节时，其他处理器不能访问这个字节的内存地址。奔腾6和最新的处理器能自动保证单处理器对同一个缓存行里进行16/32/64位的操作是原子的，但是复杂的内存操作处理器不能自动保证其原子性，比如跨总线宽度，跨多个缓存行，跨页表的访问。但是处理器提供总线锁定和缓存锁定两个机制来保证复杂内存操作的原子性。 



  java中很多的锁类都是通过AbstractQueuedSynchronizer的compareAndSetState方法实现的
  
  Atomically sets synchronization state to the given updated
     * value if the current state value equals the expected value.
     * This operation has memory semantics of a {@code volatile} read
     * and write.


  原子访问是sun.misc.Unsafe被广泛应用的特性之一
  
  


  [1]: ./images/1511939770516.jpg
  [3]: ./images/1511938828714.jpg