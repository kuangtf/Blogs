# volatile详解

## 常见面试题

- volatile关键字的作用是什么?
- volatile能保证原子性吗?
- 之前32位机器上共享的long和double变量的为什么要用volatile? 
- i++为什么不能保证原子性?
- volatile是如何实现可见性的?
- volatile是如何实现有序性的?

## volatile的作用

#### 1、volatile保证可见性

- 概念：volatile修饰的共享变量对所有线程总数可见的，也就是当一个线程修改了一个被volatile修饰共享变量的值，新值总是可以被其他线程立即得知。  

- 产生可见性的原因：为了提高处理速度，每个线程拥有自己的一个高速缓存区——线程工作内存（JMM层面），对共享变量操作之后不知道会何时写入内存。（高速缓存区对应操作系统里的L1、L2高速缓存，不包含L3、因为L3是线程共享的）

- volatile如何保证可见性？

    1、JMM内存交互层面：volatile修饰的变量的read、load、use操作和assign、store、write必须是连续的，即修改后必须立即同步回主内存，使用时必须从主内存刷新，由此保证volatile变量的可见性。

    2、底层实现：如果对声明了 volatile 的变量进行写操作，JVM 就会向处理器发送一条 **lock 前缀**的指令，将这个变量所在缓存行的数据写回到系统内存。（缓存行是缓存的最小单元）

    - 这个lock前缀指令的作用，lock 前缀的指令在多核处理器下会引发两件事情：

        1、将当前处理器缓存行的数据写回到系统内存。

        2、写回内存的操作会使在其他 CPU 里缓存了该内存地址的数据无效。

    - 在 Pentium 和早期的 IA-32 处理器中，lock 前缀会使处理器执行当前指令时产生一个 LOCK# 信号，会对总线进行锁定，其它 CPU 对内存的读写请求都会被阻塞，直到锁释放。 后来的处理器，加锁操作是由高速缓存锁代替总线锁来处理。 因为锁总线的开销比较大，锁总线期间其他 CPU 没法访问内存。 

    3、内存里变量的值变化了，那其他的处理器缓存怎么知道这个变量的值变化了呢？

    - 为了保证各个处理器的缓存是一致的，实现了**缓存一致性协议(MESI)**，每个处理器通过**嗅探**在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里，这样每个线程就可以获取到该变量最新的值了。

    4、那什么是缓存一致性协议呢？

    - 缓存一致性协议有多种，如**嗅探(snooping)协议**和**MESI协议**。
    - 嗅探(snooping)协议：每个处理器通过**嗅探**在总线上传播的数据来检查自己缓存的值是不是过期了，以此来更新数据。但是该协议效率很低，总线的负载很大。
    - MESI协议：Modified，已修改； Exclusive，独占；Shared，共享，Invalidated，已失效。（状态机模型）

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/MESI.png" style="zoom: 75%;" />

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/MESI2.png" style="zoom: 90%;" />

#### 2、volatile保证有序性

- 概念：volatile 变量的内存有序性是基于内存屏障(Memory Barrier)实现。

- 为什么会有有序性问题呢？
    - 编译器和处理器对指令序列进行重排序：即只要程序的最终结果与它顺序化情况的结果相等，那么指令的执行顺序可以与代码顺序不一致。
    - 指令重排的意义：JVM能根据处理器特性（CPU多级缓存系统、多核处理器等）适当的对机器指令进行重排序，使机器指令能更符合CPU的执行特性，最大限度的发挥机器性能。

- 内存屏障是什么东西呢？

    - 内存屏障，又称内存栅栏，是一个 CPU 指令。
    - 在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM 为了保证在不同的编译器和 CPU 上有相同的结果，通过插入特定类型的内存屏障来禁止+ 特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序。

- 那么volatile是怎么使用内存屏障来进行指令重排的呢？

    ​    1、JMM针对编译器制定的volatile重排序规则表  ：" NO " 表示禁止重排序。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/volatile3.jpg" style="zoom:80%;" />

​             2、表的解释：I：当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。这个规则确保volatile写之前的操					作不会被编译器重排序到volatile写之后 。II：当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。这				   个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前 。III：当第一个操作是volatile写，第二个操作是					volatile读时，不能重排序  。

​			 3、上面语义的实现：编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。对于编译			器来说，发现一个最优布置来最小化插入屏障的总数几乎不可能。为此，JMM采取保守策略，可以保证在任意处理器平			台，任意的程序中都能得到正确的volatile内存语义 。下面是基于保守策略的JMM内存屏障插入策略  ：

​				1）：在每个volatile写操作的前面插入一个StoreStore屏障  。

​				2）：在每个volatile写操作的后面插入一个StoreLoad屏障   。

​				3）：在每个volatile读操作的后面插入一个LoadLoad屏障  。

​				4）：在每个volatile读操作的后面插入一个LoadStore屏障  。

​			4、图解：volatile写插入内存屏障后生成的指令序列示意图  ：

​					为啥屏障会有重排序的作用呢：以StoreStore为例：StoreStore屏障将保障上面所有的普通写在volatile写之前刷新到主

​					内存  。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/volatile4.jpg" style="zoom:67%;" />

​			5、图解：volatile读插入内存屏障后生成的指令序列示意图  ：

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/5.jpg" style="zoom:67%;" />

> 注意：上述volatile写和volatile读的内存屏障插入策略非常保守。在实际执行时，只要不改变 volatile写-读的内存语义，编译器可以根据具体情况省略不必要的屏障。  

#### 3、volatile能保证原子性吗

- volatile不能保证完全的原子性，只能保证单次的读/写操作具有原子性。
- 对volatile变量的单次读/写操作可以保证原子性的，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上i++是读、加一、写三次操作，volatile并不能保证这三次操作的原子性。

- 共享的long和double变量的为什么要用volatile?
    - 因为long和double两种数据类型的操作可分为高32位和低32位两部分，因此普通的long或double类型读/写可能不是原子的。在32位的机器上，对long和double变量的操作是分为两次操作，不是原子性的。但是在64位的机器上是原子性的操作。





> 参考链接链接：
>
> [Java全栈](https://www.pdai.tech/md/java/thread/java-thread-x-key-volatile.html)
>
> [小林coding](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247486479&idx=1&sn=433a551c37a445d068ffbf8ac85f0346&chksm=f98e48a5cef9c1b3fadb691fee5ebe99eb29d83fd448595239ac8a2f755fa75cacaf8e4e8576&scene=178&cur_album_id=1408057986861416450#rd)



