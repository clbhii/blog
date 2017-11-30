---
title: java锁机制简述
tags: lock
grammar_cjkRuby: true
---

# 简述
> &ensp;&ensp;在多线程开发中，我们经常会用到java锁，接下来做一下简单的介绍
我们知道java的并发包有很多锁，如下图

![1][1]

> &ensp;&ensp;深入其中的源码你会发现，基本所有的锁获取都是通过AbstractQueuedSynchronizer的compareAndSetState方法实现的，

![2][2]

> &ensp;&ensp;看方法注释，我们知道获取锁的方式：
> &ensp;&ensp;1.  比较当前值（stateOffset）和预期值（expect）是否一致
> &ensp;&ensp;2.  一致的话，用最新值（update）更新
> &ensp;&ensp;3.  更新成功，获取锁
> &ensp;&ensp;4.  同时必须是原子的

> &ensp;&ensp;举个例子，当前stateOffset的值是0，一个线程调用compareAndSetState（0,1）,获取锁，stateOffset的值更新为1，其他线程再调用compareAndSetState（0,1），因为现在stateOffset的值为1，和expect不一致获取锁失败，直到获取锁的线程释放锁，重新置当前stateOffset的值是0。如果同时有多个线程调用compareAndSetState（0,1），因为是原子的，所以也只会有一个成功。这里的关键是这个方法是如何保证原子的？

> &ensp;&ensp;继续往下看compareAndSetState方法的实现，是通过sun.misc.Unsafe的compareAndSwapInt来实现，继续深入发现compareAndSwapInt是本地方法，找到对应的c++源码

![3][3]

> &ensp;&ensp;调用的是Atomic的cmpxchg方法，这个类的实现是跟操作系统有关，我们选择x86的

![4][4]

> &ensp;&ensp;发现cmpxchg方法内部嵌入了汇编指令，通过LOCK_IF_MP来判断是否在cmpxchgl前加lock

![5][5]

> &ensp;&ensp;到这里，我们明白了java的锁是通过汇编指令lock实现的

# Lock
> &ensp;&ensp;在早期的单处理系统中，单条指令的操作都可以认为是原子的，但在多处理系统中，情况就不一样了，即使单条指令的操作可能也会被干扰，所以出现了lock指令，lock指令的作用就是保证当前指令的原子性.


> &ensp;&ensp;当使用lock 指令前缀时，它会使 CPU 宣告一个 lock# 信号，这样就能确保在多处理器系统或多线程竞争的环境下互斥地使用这个内存地址。当指令执行完毕，这个锁定动作也就会消失。


> &ensp;&ensp;能够和lock指令一起使用的指令
ADD, ADC, AND, BTC, BTR, BTS, CMPXCHG, CMPXCH8B,CMPXCHG16B, DEC, INC, NEG, NOT, OR, SBB, SUB, XOR, XADD, XCHG
>
> &ensp;&ensp;XCHG默认带有 lock

	
###  intel的手册对lock前缀的说明如下：

> &ensp;&ensp;1.	确保对内存的读-改-写操作原子执行。在Pentium及Pentium之前的处理器中，带有lock前缀的指令在执行期间会锁住总线，使得其他处理器暂时无法通过总线访问内存。很显然，这会带来昂贵的开销。从Pentium 4，Intel Xeon及P6处理器开始，intel在原有总线锁的基础上做了一个很有意义的优化：如果要访问的内存区域（area of memory）在lock前缀指令执行期间已经在处理器内部的缓存中被锁定（即包含该内存区域的缓存行当前处于独占或以修改状态），并且该内存区域被完全包含在单个缓存行（cache line）中，那么处理器将直接执行该指令。由于在指令执行期间该缓存行会一直被锁定，其它处理器无法读/写该指令要访问的内存区域，因此能保证指令执行的原子性。这个操作过程叫做缓存锁定（cache locking），缓存锁定将大大降低lock前缀指令的执行开销，但是当多处理器之间的竞争程度很高或者指令访问的内存地址未对齐时，仍然会锁住总线。

> &ensp;&ensp;2.	禁止该指令与之前和之后的读和写指令重排序。

> &ensp;&ensp;3.	把写缓冲区中的所有数据刷新到内存中。


  [1]: ./images/1511939770516.jpg
  [2]: ./images/1511938828714.jpg
  [3]: ./images/1512024013085.jpg
  [4]: ./images/1512024076224.jpg
  [5]: ./images/1512024107063.jpg