# JVM参数调优

[TOC]

## 一.OS不同情况的设置

一般情况下，64位的OS性能比32位的好。但是根据Sun官方说明，32位的JVM反而比64位的JVM性能要好。这里并不是建议大家使用32位JVM，只是说可以在64位OS系统环境下，调优效果不大可以尝试换成32位的JVM。

具体启动命令加入```-d32```,可以启用32位JVM（如果可用的话）：

```shell
java -d32 -jar jarfile [args...]
```

## 二.JVM内存模型

JVM内存并不是设置越大越好。太大会造成寻址计算和垃圾回收开销过大，太小资源又不够用。先讲一下JVM每个区的作用。

![1543818665506](C:\Users\lemoncome\AppData\Roaming\Typora\typora-user-images\1543818665506.png)



可以从上图看到，JVM内存结构大分类上可以分为Non-Heap和Heap。

### 1.Non-Heap

官方文档定义：

> Non-heap memory includes a method area shared among all threads and memory required for the internal processing or optimization for the Java VM. 
>
> 简单翻译：Non-heap(非堆区)主要是由被线程共享的方法区和Java虚拟机内部处理所需内存组成。

具体说明：

| name                       | 说明                                                         | 相关JVM参数                                                  |
| :------------------------- | ------------------------------------------------------------ | :----------------------------------------------------------- |
| CodeCache                  | 代码缓冲区，用于JIT编译和保存本地代码（Native code）所需的内存。 | -XX:ReservedCodeCacheSize <br />java7默认值48M，java8默认值250M。<br />默认值已经够用，一般不会去修改。出现下列情况，可以根据情况修改这个参数值：<br />1.出现Hotspot编译失败。(增加)<br />2.为了减少JVM内存。(减少)<br />需要注意的是，CodeCache分配的内存，GC是不会去回收的。 |
| Permanent Generation space |                                                              |                                                              |
| Direct Memory              |                                                              |                                                              |





