---
title: java锁机制简述
tags: cpu,lock
grammar_cjkRuby: true
---

# 简述
很多同学应该在开发中，经常会用到java的锁，多线程开发中，会用到同步队列，但很多同学不知道为什么这些锁类能保证共享资源的安全性。
![enter description here][1]
上面这些就是我们java中常见的锁类








#  cpu锁

### 1.  处理器自动保证基本内存操作的原子性
> &ensp;&ensp;首先处理器会自动保证基本的内存操作的原子性。处理器保证从系统内存当中读取或者写入一个字节是原子的，意思是当一个处理器读取一个字节时，其他处理器不能访问这个字节的内存地址。奔腾6和最新的处理器能自动保证单处理器对同一个缓存行里进行16/32/64位的操作是原子的，但是复杂的内存操作处理器不能自动保证其原子性，比如跨总线宽度，跨多个缓存行，跨页表的访问。但是处理器提供总线锁定和缓存锁定两个机制来保证复杂内存操作的原子性。 


![enter description here][2]
![enter description here][3]


  java中很多的锁类都是通过AbstractQueuedSynchronizer的compareAndSetState方法实现的
  
  Atomically sets synchronization state to the given updated
     * value if the current state value equals the expected value.
     * This operation has memory semantics of a {@code volatile} read
     * and write.


  原子访问是sun.misc.Unsafe被广泛应用的特性之一


  [1]: ./images/1511939770516.jpg
  [2]: ./images/1511938607669.jpg
  [3]: ./images/1511938828714.jpg