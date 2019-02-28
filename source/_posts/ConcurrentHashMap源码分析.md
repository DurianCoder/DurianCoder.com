---
title: ConcurrentHashMap源码分析
date: 2019-02-27 23:27:49
tags:
	- ConcurrentHashMap
	- 多线程并发
---

在学习`ConcurrentHashMap`之前需要了解一些基础知识，如`synchronize`、`volatile`、`cas`、以及红黑树等

# 0x01、基础

## 1、原子性，指令有序性，线程可见性
**原子性**：和事务原子性一样，对于一个操作或者多个操作，要么都执行，要么都不执行。
<!-- more -->
**指令有序性**：保证上下不关联的语句不会被指令重排序。指令重排序是指处理器为了优化性能，改变代码的执行顺序。
**线程可见性**：指一个线程修改了某个变量，其他线程马上能够知道变化。

## 2、内存屏障
内存屏障前面的指令进行重排序但是不会排到内存屏障的后面去，后面的指令不会排到内存屏障的前面来；执行内存屏障指令时，它前面的操作必须全部完成；它会强制将对缓存的修改操作立马写入主存，它会导致其他cpu中对应的缓存行无效。使用volatile修饰的变量会产生内存屏障。

## 3、volatile
`volatile`有三个特性：
- **可见性**和**有序性**：`volatile`修饰的变量一旦变化会通知其他线程；产生内存屏障，防止了指令重排序。
- **原子性**：`volatile`具有原子性，但volatile修饰的变量必须执行原子操作才能保证原子性，如`a = 1;`具有原子性，而`a++`不是原子操作。
```
public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            new Thread(new Runnable() {
                public void run() {
                    increase();
                }
            }
            ).start();
        }
        Thread.sleep(5000);
        System.out.println(c);
    }
// 运行结果均小于1000
```
## 4、CAS

**CAS**即`compare and swap`(比较与交换)，它涉及到三个操作数：内存值、预期值、新值。当且仅当预期值和内存值相等时才将内存值修改为新值 。
![](https://pic4.zhimg.com/80/v2-cf24054c2f010329073cd786ba392671_hd.png)

Java并发包中很多地方使用到了`CAS`算法，有效的避免了并发，像`AtomicInteger`、`Semaphore`、`ReentrantLock`等底层都采用了`CAS`算法。

**CAS自旋**原理：

```
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
    	//得到此时var2这个偏移量在内存中的值，即期望值var5
    	var5 = this.getIntVolatile(var1, var2);
    	//compareAndSwapInt通过var1, var2得出实际值，然后和var5进行    对比，如果相同则把var5 + var4（新值写入内存），如果不相同不断do，while循环（自旋）直到相同。
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```

# 0x02、JDK1.7中的ConcurrentHashMap

jdk1.7中采用了`Segment`+`HashEntry`的方式来实现，结构如下：
![](https://pic1.zhimg.com/80/v2-e3ffa2f184d520f7e00f573577b63480_hd.png)
`Segment`在实现上继承了`ReentrantLock`,`Segment`数组将一个大的table分割成多个小的table来进行加锁，也就是实现了细粒度锁分离；每一个`Segment`元素存储的是一个`HashEntry`数组+链表，和`HashMap`的数据存储结构一样。
- Put实现：
  当执行put方法插入数据时，根据key的hash值，在Segment数组中找到相应的位置，如果相应位置的Segment还未初始化，则通过CAS进行赋值，接着执行Segment对象的put方法通过加锁机制插入数据，实现如下：
  场景：线程A和线程B同时执行相同Segment对象的put方法
  - 1、线程A执行tryLock()方法成功获取锁，则把HashEntry对象插入到相应的位置；
  - 2、线程B获取锁失败，则执行scanAndLockForPut()方法，在scanAndLockForPut方法中，会通过重复执行tryLock()方法尝试获取锁，在多处理器环境下，重复次数为64，单处理器重复次数为1，当执行tryLock()方法的次数超过上限时，则执行lock()方法挂起线程B；
  - 3、当线程A执行完插入操作时，会通过unlock()方法释放锁，接着唤醒线程B继续执行；

- get实现：
  ConcurrentHashMap的get操作跟HashMap类似，只是ConcurrentHashMap第一次需要经过一次hash定位到Segment的位置，然后再hash定位到指定的HashEntry，遍历该HashEntry下的链表进行对比，成功就返回，不成功就返回null。

- size实现：
  先采用不加锁的方式，连续计算元素的个数，最多计算3次：
  - 1、如果前后两次计算结果相同，则说明计算出来的元素个数是准确的；
  - 2、如果前后两次计算结果都不同，则给每个`Segment`进行加锁，再计算一次元素的个数；

# 0x03、JDK1.8中的ConcurrentHashMap

1.8中放弃了`Segment`臃肿的设计，取而代之的是采用`Node` + `CAS` + `Synchronized`来保证并发安全进行实现，结构如下：

![](https://pic1.zhimg.com/80/v2-052fc729f0e4634484d875887ba40c22_hd.png)

- put实现
  当执行put方法插入数据时，根据key的hashcode再哈希，在Node数组中找到对应的位置，实现如下：
  1、当相应位置的Node还未初始化，则使用CAS插入相应的数据；
  ```
  else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
      if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
          break; // no lock when adding to empty bin
  }
  ```
  2、如果相应位置的`Node`不为空，且当前该节点不处于移动状态，则对该节点加`synchronized`锁，如果该节点的`hash`不小于0，则遍历链表更新节点或插入新节点；
  3、如果该节点是`TreeBin`类型的节点，说明是红黑树结构，则通过`putTreeVal`方法往红黑树中插入节点；
  4、如果`binCount`不为0，说明`put`操作对数据产生了影响，如果当前链表的个数达到8个，则通过`treeifyBin`方法转化为红黑树，如果`oldVal`不为空，说明是一次更新操作，没有对元素个数产生影响，则直接返回旧值；
  5、如果插入的是一个新节点，则执行`addCount()`方法尝试更新元素个数`baseCount`；

- get操作
  1. 计算hash值，定位到该table索引位置，如果是首节点符合就返回
  2. 如果遇到扩容的时候，会调用标志正在扩容节点ForwardingNode的find方法，查找该节点，匹配就返回
  3. 以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null

- size实现 
  1.8中使用一个`volatile`类型的变量`baseCount`记录元素的个数，当插入新数据或则删除数据时，会通过`addCount()`方法更新`baseCount`，实现如下：
  ```
  if ((as = counterCells) != null ||
      !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
      CounterCell a; long v; int m;
      boolean uncontended = true;
      if (as == null || (m = as.length - 1) < 0 ||
          (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
          !(uncontended =
            U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
          fullAddCount(x, uncontended);
          return;
      }
      if (check <= 1)
          return;
      s = sumCount();
  }
  ```

  在1.8中的`size`实现比1.7简单多，因为元素个数保存`baseCount`中，部分元素的变化个数保存在`CounterCell`数组中，通过累加`baseCount`和`CounterCell`数组中的数量，即可得到元素的总个数；

# 0x04、ConcurrentHashMap思考
其实可以看出JDK1.8版本的ConcurrentHashMap的数据结构已经接近HashMap，相对而言ConcurrentHashMap只是增加了同步的操作来控制并发，从JDK1.7版本的ReentrantLock+Segment+HashEntry，到JDK1.8版本中synchronized+CAS+HashEntry+红黑树,相对而言，总结如下思考：
1. JDK1.8的实现降低锁的粒度，JDK1.7版本锁的粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度就是HashEntry（首节点）
2. JDK1.8版本的数据结构变得更加简单，使得操作也更加清晰流畅，因为已经使用synchronized来进行同步，所以不需要分段锁的概念，也就不需要Segment这种数据结构了，由于粒度的降低，实现的复杂度也增加了
3. JDK1.8使用红黑树来优化链表（当链表的长度超过8将链表转化为红黑树），基于长度很长的链表的遍历是一个很漫长的过程，而红黑树的遍历效率是很快的，代替一定阈值的链表，这样形成一个最佳拍档；
4. JDK1.8为什么使用内置锁synchronized来代替重入锁ReentrantLock，我觉得有以下几点：
- 因为粒度降低了，在相对而言的低粒度加锁方式，synchronized并不比ReentrantLock差，在粗粒度加锁中ReentrantLock可能通过Condition来控制各个低粒度的边界，更加的灵活，而在低粒度中，Condition的优势就没有了
- JVM的开发团队从来都没有放弃synchronized，而且基于JVM的synchronized优化空间更大，使用内嵌的关键字比使用API更加自然
- 在大量的数据操作下，对于JVM的内存压力，基于API的ReentrantLock会开销更多的内存，虽然不是瓶颈，但是也是一个选择依据

5.为什么加载因子默认为0.75?
```
     * Ideally, thefrequency of nodes in bins follows a Poisson distribution
     * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
     * parameter of about 0.5 on average, given the resizing threshold
     * of 0.75, although with a large variance because of resizing
     * granularity. Ignoring variance, the expected occurrences of
     * list size k are (exp(-0.5) * pow(0.5, k) / factorial(k)). The
     * first values are:
     *
     * 0:    0.60653066
     * 1:    0.30326533
     * 2:    0.07581633
     * 3:    0.01263606
     * 4:    0.00157952
     * 5:    0.00015795
     * 6:    0.00001316
     * 7:    0.00000094
     * 8:    0.00000006
     * more: less than 1 in ten million
在理想情况下,使用随机哈希码,节点出现的频率在hash桶中遵循泊松分布，同时给出了桶中元素个数和概率的对照表。从上面的表中可以看到当桶中元素到达8个的时候，概率已经变得非常小，也就是说用0.75作为加载因子，每个碰撞位置的链表长度超过８个是几乎不可能的。
```
加载因子越大，hash冲突的概率越大，空间利用率越高，查询效率越低，加载因子的值需要在hash冲突和空间利用率中间做平衡。

# 0x05、参考链接

- [为什么加载因子默认为0.75?](https://blog.csdn.net/hcmony/article/details/56494527)

- [谈谈ConcurrentHashMap1.7和1.8的不同实现](<https://www.jianshu.com/p/e694f1e868ec>)

- [深入并发包](<https://duriancoder.github.io/2018/04/07/%E6%B7%B1%E5%85%A5%E5%B9%B6%E5%8F%91%E5%8C%85ConcurrentHashMap/>)

