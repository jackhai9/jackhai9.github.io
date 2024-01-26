title: 常用JVM配置参数
date: 2015-03-01 15:12:52
categories: 技术
tags: [Java,JVM配置参数]
---

## 一、Trace跟踪参数：
```
-verbose:gc
-XX:+printGC
```

解释：如上，打印GC的简要信息。比如：
<!--more-->
[GC 4790K->374K(15872K), 0.0001606 secs]
[GC 4790K->374K(15872K), 0.0001474 secs]
[GC 4790K->374K(15872K), 0.0001563 secs]
[GC 4790K->374K(15872K), 0.0001682 secs]
上方日志的意思是说，GC之前，用了4M左右的内存，GC之后，用了374K内存，一共回收了将近4M。内存大小一共是16M左右。

```
-XX:+PrintGCDetails
```

解释：如上，打印GC详细信息。

```
-XX:+PrintGCTimeStamps
```

解释：如上，打印CG发生的时间戳。

理解GC日志的含义：
例如下面这段日志：
```
[GC[DefNew: 4416K->0K(4928K), 0.0001897 secs] 4790K->374K(15872K), 0.0002232 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
```

上面日志的意思是说：
这是一个新生代的GC。方括号内部的“4416K->0K(4928K)”含义是：“GC前该内存区域已使用容量->GC后该内存区域已使用容量（该内存区域总容量）”。
而在方括号之外的“4790K->374K(15872K)”表示“GC前Java堆已使用容量->GC后Java堆已使用容量（Java堆总容量）”。
再往后看，“0.0001897 secs”表示该内存区域GC所占用的时间，单位是秒。


再比如下面这段GC日志：
![](http://7qna9x.com1.z0.glb.clouddn.com/jvm1.png)
上图中，我们先看一下用红框标注的“[0x27e80000, 0x28d80000, 0x28d80000)”的含义，它表示新生代在内存当中的位置：第一个参数是申请到的起始位置，第二个参数是申请到的终点位置，第三个参数表示最多能申请到的位置。上图中的例子表示新生代申请到了15M的控件，而这个15M是等于：（eden space的12288K）+（from space的1536K）+（to space的1536K）。
疑问：分配到的新生代有15M，但是可用的只有13824K，为什么会有这个差异呢？


指定GC log的位置:
```
-Xloggc:log/gc.log
```

解释：如上，指定GC log的位置，以文件输出，当前项目目录下的log目录下。帮助开发人员分析问题。

```
-XX:+PrintHeapAtGC
```

解释：如上，每一次GC前和GC后，都打印堆信息。
例如：
![](http://7qna9x.com1.z0.glb.clouddn.com/jvm2.png)
上图中，红框部分正好是一次GC，红框部分的前面是GC之前的日志，红框部分的后面是GC之后的日志。

```
-XX:+TraceClassLoading
```

解释：如上，监控类的加载。
例如：
```
[Loaded java.lang.Object from shared objects file]
[Loaded java.io.Serializable from shared objects file]
[Loaded java.lang.Comparable from shared objects file]
[Loaded java.lang.CharSequence from shared objects file]
[Loaded java.lang.String from shared objects file]
[Loaded java.lang.reflect.GenericDeclaration from shared objects file]
[Loaded java.lang.reflect.Type from shared objects file]
```

```
-XX:+PrintClassHistogram
```
解释：如上，按下Ctrl+Break后，打印类的信息。
例如：
![](http://7qna9x.com1.z0.glb.clouddn.com/jvm3.png)

## 二、堆的分配参数：

1、**-Xmx –Xms：指定最大堆和最小堆**
举例：
```
-Xmx20m -Xms5m
```

2、-Xmn、-XX:NewRatio、-XX:SurvivorRatio
	-Xmn 设置新生代大小
	-XX:NewRatio  新生代（eden+2*s）和老年代（不包含永久区）的比值
    　　　　      例如：4，表示新生代:老年代=1:4，即新生代占整个堆的1/5
	-XX:SurvivorRatio（幸存代）设置两个Survivor区和eden的比值
    　　　　          例如：8，表示两个Survivor:eden=2:8，即一个Survivor占新生代的1/10
举例：
```
-Xmx20m -Xms20m -Xmn7m -XX:SurvivorRatio=2 -XX:+PrintGCDetails
```

3、-XX:+HeapDumpOnOutOfMemoryError、-XX:+HeapDumpPath
	-XX:+HeapDumpOnOutOfMemoryError  OOM时导出堆到文件，根据这个文件，我们可以看到系统dump时发生了什么
	-XX:+HeapDumpPath   导出OOM的路径
举例：
```
-Xmx20m -Xms5m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=d:/a.dump
```

如上意思是说，现在给堆内存最多分配20M的空间。如果发生了OOM异常，那就把dump信息导出到d:/a.dump文件中。

可以使用Java自带的Java VisualVM工具对dump文件进行分析。

4、-XX:OnOutOfMemoryError
在OOM时，执行一个脚本。（可以在OOM时，发送邮件，甚至是重启程序。）
举例：
```
-XX:OnOutOfMemoryError=D:/tools/jdk1.7_40/bin/printstack.bat %p //p代表的是当前进程的pid 
```

如上参数的意思是说，执行printstack.bat脚本，而这个脚本做的事情是：D:/tools/jdk1.7_40/bin/jstack -F %1 > D:/a.txt，即当程序OOM时，在D:/a.txt中将会生成线程的dump。

5、堆的分配参数总结：
*  根据实际事情调整新生代和幸存代的大小
*  官方推荐新生代占堆的1/3
*  幸存代占新生代的1/10
*  在OOM时，记得Dump出堆，确保可以排查现场问题

6、永久区分配参数：
-XX:PermSize  -XX:MaxPermSize
设置永久区的初始空间和最大空间。也就是说，jvm启动时，永久区一开始就占用了PermSize大小的空间，如果空间还不够，可以继续扩展，但是不能超过MaxPermSize，否则会OOM。

举例：
我们知道，使用CGLIB等库的时候，可能会产生大量的类，这些类，有可能撑爆永久区导致OOM。于是，我们运行下面这段代码：
```
for(int i=0;i<100000;i++){
　　CglibBean bean = new CglibBean("geym.jvm.ch3.perm.bean"+i,new HashMap());
}
```

上面这段代码会在永久区不断地产生新的类，导致OOM。
总结：**如果堆空间没有用完也抛出了OOM，有可能是永久区导致的。**堆空间实际占用非常少，但是永久区溢出一样抛出OOM。

## 三、栈的分配参数：
-Xss
```
设置栈空间的大小。通常只有几百K
　　决定了函数调用的深度
　　每个线程都有独立的栈空间
　　局部变量、参数 分配在栈上
```

注：栈空间是每个线程私有的区域。栈里面的主要内容是栈帧，而栈帧存放的是局部变量表，局部变量表的内容是：局部变量、参数。

如果-Xss设置的过小，函数调用的次数过多，如递归调用太深。可能会抛出StackOverflowError。
举例：
```
-Xss128K 
```



