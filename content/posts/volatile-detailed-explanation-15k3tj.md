---
title: volatile详解
slug: volatile-detailed-explanation-15k3tj
url: /post/volatile-detailed-explanation-15k3tj.html
date: '2023-02-19 14:15:28+08:00'
lastmod: '2024-05-01 15:42:02+08:00'
toc: true
isCJKLanguage: true
---

# volatile详解

# 简介

volatile是JVM提供的轻量级的同步机制

1. 保证可见性
2. 保证有序性，防止指令重排
3. 不保证原子性(需要借助synchronized 或者 CAS)

# volatile怎么保证多个线程之间可见性的？—— lock前缀指令

再来看看JMM内存模型

所谓可见性，就是当一条线程修改了共享变量的值，新值对于其他线程来说是可以立即得知的

​![image](https://raw.githubusercontent.com/Parksosaurus/ParksoBlog/main/images/image-20230219144519-c6kw6k2.png)​

每个线程都有自己独立的工作空间，为了匹配CPU的运行速度，他们不会直接从内存中读取数据，而是将数据拷贝一份到cpu缓存中(即每个线程自己的工作内存)，他们之间的相互交互，是通过内存来完成的。

根据JMM内存模型的8大原子操作，每个线程在将数据操作完`store`回主存前，会加`lock`指令来锁定区域的缓存(缓存行锁定)，根据MESI缓存一致性协议，总线通过侦听器发现数据被修改，会立即让其他线程工作内存中不一致的副本立即失效。

等到当前线程将更改后的数据`write`​回主存后，立即执行`unlock`​指令

​![image](https://raw.githubusercontent.com/Parksosaurus/ParksoBlog/main/images/image-20230219145313-hocadwf.png)​

此时，其他线程再重新读取更新后的数据，再拷贝到自己的工作内存。总线侦听机制会在总线上检测线程的数据，发现有线程做了更改时准备`store`​​​回主内存时，它就会立刻将其他线程工作内存中的副本置位无效，然后从新到主存获取更新后的值。

> 如果对声明了 volatile 的变量进行写操作，JVM 就会向处理器发送一条 lock 前缀的指令，将这个变量所在缓存行的数据写回到系统内存。
>
> 为了保证各个处理器的缓存是一致的，实现了缓存一致性协议(MESI)，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。
>
> 所有多核处理器下还会完成：当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。
>
> volatile 变量通过这样的机制就使得每个线程都能获得该变量的最新值。

所以

> lock 前缀的指令在多核处理器下会引发两件事情:
>
> * **将当前处理器缓存行的数据写回到系统内存。**
> * **写回内存的操作会使在其他 CPU 里缓存了该内存地址的数据无效。**

除了使用 volatile 关键字来保证内存可见性之外，使用`synchronized`或`Lock`锁也能保证变量的内存可见性。只是相比而言使用 `volatile`​关键字开销更小，是轻量级的锁。

这就是内存可见性的原理。

# volatile关键字是怎么保证有序性的？ —— 内存屏障

使用volatile关键字修饰共享变量便可以​**禁止指令重排序**​。若用volatile修饰共享变量，在JVM底层volatile是采用“​**内存屏障**​”来实现禁止特定类型的处理器重排序。加入volatile关键字时，会多出一个lock前缀指令，lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：

1. 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
2. 它会强制将对缓存的修改操作立即写入主存；
3. 如果是写操作，它会导致其他CPU中对应的缓存行无效。

JMM具备一些先天的有序性，通过Happens-Before原则就可以保证的一定的有序性。

## volatile的happens-before 关系

happens-before 规则中有一条是 volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。

```java
//假设线程A执行writer方法，线程B执行reader方法
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;
  
    public void writer() {
        a = 1;              // 1 线程A修改共享变量
        flag = true;        // 2 线程A写volatile变量
    } 
  
    public void reader() {
        if (flag) {         // 3 线程B读同一个volatile变量
        int i = a;          // 4 线程B读共享变量
        ……
        }
    }
}
```

根据 happens-before 规则，上面过程会建立 3 类 happens-before 关系。

* 根据程序次序规则：1 happens-before 2 且 3 happens-before 4。
* 根据 volatile 规则：2 happens-before 3。
* 根据 happens-before 的传递性规则：1 happens-before 4。

​![image](https://raw.githubusercontent.com/Parksosaurus/ParksoBlog/main/images/image-20230219153214-7z7qerl.png)​

因为以上规则，当线程 A 将 volatile 变量 flag 更改为 true 后，线程 B 能够迅速感知。

## volatile禁止重排序

为了性能优化，JMM 在不改变正确语义的前提下，会允许编译器和处理器对指令序列进行重排序。JMM 提供了**内存屏障**阻止这种重排序。

Java 编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。

​![image](https://raw.githubusercontent.com/Parksosaurus/ParksoBlog/main/images/image-20230219153454-fhnqeyx.png)​

NO  表示禁止重排序。

为了实现 volatile 内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎是不可能的，为此，JMM 采取了保守的策略。

* 在每个 volatile 写操作的前面插入一个 StoreStore 屏障。
* 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障。
* 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障。
* 在每个 volatile 读操作的后面插入一个 LoadStore 屏障。

volatile 写是在前面和后面分别插入内存屏障，而 volatile 读操作是在后面插入两个内存屏障。

|内存屏障|说明|
| -----------------| -------------------------------------------------------------|
|StoreStore 屏障|禁止上面的普通写和下面的 volatile 写重排序。|
|StoreLoad 屏障|防止上面的 volatile 写与下面可能有的 volatile 读/写重排序。|
|LoadLoad 屏障|禁止下面所有的普通读操作和上面的 volatile 读重排序。|
|LoadStore 屏障|禁止下面所有的普通写操作和上面的 volatile 读重排序。|

​![image](https://raw.githubusercontent.com/Parksosaurus/ParksoBlog/main/images/image-20230219153854-vy9auid.png)​

​![image](https://raw.githubusercontent.com/Parksosaurus/ParksoBlog/main/images/image-20230219153858-sdx0l06.png)​

‍
