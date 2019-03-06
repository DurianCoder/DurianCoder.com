---
title: 原子操作类AtomicInteger详解
date: 2019-03-06 19:38:59
tags:
	- 并发
	- AtomicInteger
---

# 0x01、为什么需要AtomicInteger原子操作类？

对于Java中的运算操作，例如自增或自减，若没有进行额外的同步操作，在多线程环境下就是线程不安全的。num++解析为num=num+1，明显，这个操作不具备原子性，多线程并发共享这个变量时必然会出现问题。测试代码如下:<!-- more -->

```
public class AtomicIntegerTest {
 
    private static final int THREADS_CONUT = 20;
    public static int count = 0;
 
    public static void increase() {
        count++;
    }
 
    public static void main(String[] args) {
        Thread[] threads = new Thread[THREADS_CONUT];
        for (int i = 0; i < THREADS_CONUT; i++) {
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 1000; i++) {
                        increase();
                    }
                }
            });
            threads[i].start();
        }
 
        while (Thread.activeCount() > 1) {
            Thread.yield();
        }
        System.out.println(count);
    }
}
```

这里运行了20个线程，每个线程对count变量进行1000此自增操作，如果上面这段代码能够正常并发的话，最后的结果应该是20000才对，但实际结果却发现每次运行的结果都不相同，都是一个小于20000的数字。这是为什么呢？

0x02、使用volatile修饰count变量？

```
public class AtomicIntegerTest {
 
    private static final int THREADS_CONUT = 20;
    public static volatile int count = 0;
 
    public static void increase() {
        count++;
    }
 
    public static void main(String[] args) {
        Thread[] threads = new Thread[THREADS_CONUT];
        for (int i = 0; i < THREADS_CONUT; i++) {
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 1000; i++) {
                        increase();
                    }
                }
            });
            threads[i].start();
        }
 
        while (Thread.activeCount() > 1) {
            Thread.yield();
        }
        System.out.println(count);
    }
}

```

结果似乎又失望了，测试结果和上面的一致，每次都是输出小于20000的数字。这又是为什么么？ 上面的论据是正确的，也就是上面标红的内容，但是这个论据并不能得出"基于volatile变量的运算在并发下是安全的"这个结论，因为核心点在于java里的运算（比如自增）并不是原子性的。

0x03、用了AtomicInteger类后会变成什么样子呢？

把上面的代码改造成AtomicInteger原子类型，先看看效果

```
package com.durian;

import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicTest {
    private static AtomicInteger count = new AtomicInteger(0);
    private static int num = 0;


    private static void increase() {
        count.incrementAndGet();
    }

    private static void inc() {
        num ++;
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("111");

        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(5, 10, 2, TimeUnit.SECONDS, new LinkedBlockingDeque<Runnable>());

        for (int i = 0; i < 1000; i++) {
            threadPoolExecutor.submit(new Runnable() {
                @Override
                public void run() {
                    increase();
                    inc();
                }
            });
        }

        while (threadPoolExecutor.getActiveCount() > 0) {
        }

        System.out.println(count);
        System.out.println(num);
    }

}

```
结果每次都输出20000，程序输出了正确的结果，这都归功于AtomicInteger.incrementAndGet()方法的原子性。

# 0X04、非阻塞同步

同步：多线程并发访问共享数据时，保证共享数据再同一时刻只被一个或一些线程使用。
我们知道，阻塞同步和非阻塞同步都是实现线程安全的两个保障手段，非阻塞同步对于阻塞同步而言主要解决了阻塞同步中线程阻塞和唤醒带来的性能问题，那什么叫做非阻塞同步呢？在并发环境下，某个线程对共享变量先进行操作，如果没有其他线程争用共享数据那操作就成功；如果存在数据的争用冲突，那就才去补偿措施，比如不断的重试机制，直到成功为止，因为这种乐观的并发策略不需要把线程挂起，也就把这种同步操作称为非阻塞同步（操作和冲突检测具备原子性）。在硬件指令集的发展驱动下，使得 "操作和冲突检测" 这种看起来需要多次操作的行为只需要一条处理器指令便可以完成，这些指令中就包括非常著名的CAS指令（Compare-And-Swap比较并交换）。《深入理解Java虚拟机第二版.周志明》第十三章中这样描述关于CAS机制:

![](https://pic3.zhimg.com/80/v2-a786369d2fd2feade31705654ead2fb1_hd.png)

所以再返回来看AtomicInteger.incrementAndGet()方法，它的时间也比较简单

```
 /**
     * Atomically increments by one the current value.
     *
     * @return the updated value
     */
    public final int incrementAndGet() {
        for (;;) {
            int current = get();
            int next = current + 1;
            if (compareAndSet(current, next))
                return next;
        }

```
incrementAndGet()方法在一个无限循环体内，不断尝试将一个比当前值大1的新值赋给自己，如果失败则说明在执行"获取-设置"操作的时已经被其它线程修改过了，于是便再次进入循环下一次操作，直到成功为止。这个便是AtomicInteger原子性的"诀窍"了，继续进源码看它的compareAndSet方法:

```
  /**
     * Atomically sets the value to the given updated value
     * if the current value {@code ==} the expected value.
     *
     * @param expect the expected value
     * @param update the new value
     * @return true if successful. False return indicates that
     * the actual value was not equal to the expected value.
     */
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
	}
```

可以看到，compareAndSet()调用的就是Unsafe.compareAndSwapInt()方法，即Unsafe类的CAS操作。

0X05、参考链接

[原子操作类AtomicInteger详解](<https://blog.csdn.net/fanrenxiang/article/details/80623884>)