---
title: Iterator迭代器删除集合元素
date: 2018-08-26 21:37:01
tags:
	- jase
---

## 0x01、使用Iterator删除元素实注意点

##### 1、Iteartor机制
从API中可以看到List等Collection的实现并没有同步化，如果在多 线程应用程序中出现同时访问，而且出现修改操作的时候都要求外部操作同步化；调用Iterator操作获得的Iterator对象在多线程修改Set的时 候也自动失效，并抛出<!-- more --> java.util.ConcurrentModificationException。这种实现机制是fail-fast，对外部 的修改并不能提供任何保证。
网上查找的关于Iterator的工作机制。Iterator是工作在一个独立的线程中，并且拥有一个 mutex锁，就是说Iterator在工作的时候，是不允许被迭代的对象被改变的。Iterator被创建的时候，建立了一个内存索引表（单链表），这 个索引表指向原来的对象，当原来的对象数量改变的时候，这个索引表的内容没有同步改变，所以当索引指针往下移动的时候，便找不到要迭代的对象，于是产生错 误。List、Set等是动态的，可变对象数量的数据结构，但是Iterator则是单向不可变，只能顺序读取，不能逆序操作的数据结构，当 Iterator指向的原始数据发生变化时，Iterator自己就迷失了方向 

##### 2、在使用迭代器循环直接从Set中删除元素会报错

```
import java.util.*;
public class Client{
    public static void main(String []args){
        Set set = new HashSet();
        set.add(3);
        set.add(2);
        set.add(1);
        for(Iterator it = set.iterator();it.hasNext();){
            Object obj = it.next();
            set.remove(obj);//试图删除迭代出来的元素
        }
    }
}
```

##### Exception:

```
Exception in thread "main" java.util.ConcurrentModificationException
        at java.util.HashMap$HashIterator.nextEntry(HashMap.java:793)
        at java.util.HashMap$KeyIterator.next(HashMap.java:828)
        at Client.main(Client.java:9)
```

##### 3、使用Iterator提供的remove方法

```
import java.util.*;
public class Client{
    public static void main(String []args){
        Set set = new HashSet();
        set.add(3);
        set.add(2);
        set.add(1);
        for(Iterator it = set.iterator();it.hasNext();){
            Object obj = it.next();
            it.remove();//试图删除迭代出来的元素
        }
        System.out.println(set.isEmpty());//判断集合是否为空
    }
}
```

