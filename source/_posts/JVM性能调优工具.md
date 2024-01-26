title: JVM性能调优工具
date: 2015-11-27 22:00:01
categories: 技术
tags: [Java,性能调优]
---

JDK内置了很多有用的工具，可以对程序进行问题排查、性能调优。如jps，jstat，jinfo，jmap等，可以通过查手册学习相关使用，今天主要介绍下面这些：

## jstack

jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息，如果是在64位机器上，需要指定选项”-J-d64”，Windows的jstack使用方式只支持以下的这种方式：
```java
jstack [-l] pid
```

<!--more-->
如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。


## jconsole

一个java GUI监视工具，可以以图表化的形式显示各种数据。并可通过远程连接监视远程的服务器VM。用java写的GUI程序，用来监控VM，并可监控远程的VM，非常易用，而且功能非常强。命令行里打 jconsole，选则进程就可以了。

需要注意的就是在运行jconsole之前，必须要先设置环境变量DISPLAY，否则会报错误，Linux下设置环境变量如下：
```java
export DISPLAY=:0.0
```

## jvisualvm

VisualVM 是一款免费的\集成了多个JDK 命令行工具的可视化工具，它能为您提供强大的分析能力，对 Java 应用程序做性能分析和调优。这些功能包括生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析，同时它还支持在 MBeans 上进行浏览和操作。

在内存分析上，Java VisualVM的最大好处是可通过安装Visual GC插件来分析 GC（Gabage Collection）趋势、内存消耗详细状况。

1、Visual GC(监控垃圾回收器)

Java VisualVM默认没有安装Visual GC插件，需要手动安装，JDK的安装目录的bin目露下双击jvisualvm.exe，即可打开Java VisualVM，点击菜单栏 工具->插件 安装Visual GC

安装完成后重启Java VisualVM，Visual GC界面自动打开，即可看到JVM中堆内存的分代情况

堆(Heap) ：JVM管理的内存叫堆

分代：根据对象的生命周期长短，把堆分为3个代：Young，Old 和 Permanent，根据不同代的特点采用不同的收集算法，扬长避短也。
	1. Young（年轻代）年轻代分三个区。一个 Eden 区，两个Survivor 区。大部分对象在 Eden 区中生成。当 Eden 区满时，还存活的对象将被复制到 Survivor 区（两个中的一个），当这个 Survivor 区满时，此区的存活对象将被复制到另外一个 Survivor 区，当这个 Survivor 去也满了的时候，从第一个 Survivor 区复制过来的并且此时还存活的对象，将被复制到 “年老区(Old)” 。需要注意，Survivor 的两个区是对称的，没先后关系，所以同一个区中可能同时存在从 Eden 复制过来对象，和从前一个 Survivor 复制过来的对象，而复制到年老区的只有从第一个 Survivor 复制过来的对象。而且，Survivor 区总有一个是空的。
	2. Old（年老代）年老代存放从年轻代存活的对象。一般来说年老代存放的都是生命期较长的对象。如果 Old 区也满了，将会触发 Full GC ，回收整个堆内存。
	3. Perm（持久代，Java8中已经移除了，用Metaspace来代替，使用本地内存来存储类元数据信息）用于存放的主要是类的 Class 对象。持久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些 class，例如 Hibernate 等，在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类。持久代大小通过 -XX:MaxPermSize= 进行设置。如果一个类被频繁的加载，也可能会导致 Perm 区满，Perm 区的垃圾回收也是由 Full GC 触发的。

GC的基本概念
gc 分为 full gc 和 minor gc，当每一块区满的时候都会引发 gc。
	•	Scavenge GC
一般情况下，当新对象生成，并且在 Eden 申请空间失败时，就触发了 Scavenge GC，堆 Eden 区域进行 GC，清除非存活对象，并且把尚且存活的对象移动到 Survivor 区。然后整理 Survivor 的两个区。
	•	Full GC
对整个堆进行整理，包括 Young、Tenured 和 Perm。Full GC 比 Scavenge GC 要慢，因此应该尽可能减少 Full GC。有如下原因可能导致 Full GC：
上一次 GC 之后 Heap 的各域分配策略动态变化
	•	System.gc() 被显示调用
	•	Perm 域被写满
	•	Old 被写满
内存溢出 out of memory，是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory；比如申请了一个integer,但给它存了long才能存下的数，那就是内存溢出。
内存泄露 memory leak，是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存,迟早会被占光。其实说白了就是该内存空间使用完毕之后未回收。

2、VisualVM 的其他功能 监视界面（cpu，类，堆，线程）

堆dump和线程dump操作：
Dump文件是进程的内存镜像，可以把程序的执行状态通过调试器保存到dump文件中。

dump文件可以使用[MAT](http://www.eclipse.org/mat/)(Memory Analyzer Tool)内存分析工具以图形界面的方式展现结果。



## 参考：
[使用 VisualVM 进行性能分析及调优](https://www.ibm.com/developerworks/cn/java/j-lo-visualvm/)

