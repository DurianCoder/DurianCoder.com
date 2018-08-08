---
title: IO中的设计模式
date: 2018-04-07 22:11:38
tags:
	- java
	- 设计模式
---

涉及到的类主要有FileInputStream ，InputStreamReader ，BufferedReader 。涉及到的设计模式主要有适配器模式以及装饰者模式。下面分别展开介绍。<!--more-->

### 一、装饰者模式以及适配器模式的介绍

装饰者模式：动态地将责任附加到对象上，若要扩展功能，装饰者模提供了比继承更有弹性的替代方案。 
通俗的解释：装饰模式就是给一个对象增加一些新的功能，而且是动态的，要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例。

![这里写图片描述](https://img-blog.csdn.net/20170305220853799?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTF9rYW5nbGlu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



适配器模式：将一个类的接口，转换成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。 
适配器模式有三种：类的适配器模式、对象的适配器模式、接口的适配器模式。 
通俗的说法：适配器模式将某个类的接口转换成客户端期望的另一个接口表示，目的是消除由于接口不匹配所造成的类的兼容性问题。

下面以类的适配器模式举例： 
![这里写图片描述](https://img-blog.csdn.net/20170305220910059?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTF9rYW5nbGlu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
有一个Source类，拥有一个方法，待适配，目标接口时Targetable，通过Adapter类，将Source的功能扩展到Targetable里。

### 二、在Java IO中的应用

举例如下： 
1、适配器模式 
//file 为已定义好的文件流 
FileInputStream fileInput = new FileInputStream(file); 
InputStreamReader inputStreamReader = new InputStreamReader(fileInput);

以上就是适配器模式的体现，FileInputStream是字节流，而并没有字符流读取字符的一些api，因此通过InputStreamReader将其转为Reader子类，因此有了可以操作文本的文件方法。换句话说，就是将FileInputStream读取一个字节流的方法扩展转换为InputStreamReader读取一个字符流的功能。
2、装饰者模式

BufferedReader bufferedReader=new BufferedReader(inputStreamReader);

构造了缓冲字符流，将FileInputStream字节流包装为BufferedReader过程就是装饰的过程，刚开始的字节流FileInputStream只有read一个字节的方法，包装为inputStreamReader后，就有了读取一个字符的功能，在包装为BufferedReader后，就拥有了read一行字符的功能。