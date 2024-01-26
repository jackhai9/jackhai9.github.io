title: Java垃圾回收机制
date: 2015-02-27 20:25:11
categories: 技术
tags: [Java,垃圾回收,gc]
---

垃圾回收（Garbage Collection，GC），在Java中，程序员不需要去关心内存动态分配和垃圾回收的问题，这一切都交给了JVM来处理。顾名思义，垃圾回收就是释放垃圾占用的空间，那么在Java中，什么样的对象会被认定为“垃圾”？那么当一些对象被确定为垃圾之后，采用什么样的策略来进行回收（释放空间）？在目前的商业虚拟机中，有哪些典型的垃圾收集器？
一.垃圾回收要解决的问题（哪些内存需要回收？什么时候回收？如何回收？）
二.典型的垃圾回收算法
三.典型的垃圾回收器
四.垃圾收集器参数总结

<!--more-->

## 准备知识
解释执行和即时编译器
JVM有两种方式去执行编译器编译出来的.class字节码文件——解释器（Interpreter）和即时编译器（Just In Time Complier）。
解释器就像一个老实本分的翻译家，逐字逐句的翻译，每遇到一个指令，就将它编译成本机的机器语言（Native Machine Code），然后执行，下次再遇到这一条指令，还会再编译一次；
而即时编译器，则像一个善于将外文翻译成地道的中文的翻译家，会对指令编译出来的机器语言进行执行效率的优化，并且把这个优化后的机器语言保存下来，下次遇到再这条的指令，就不需要编译，直接执行。
JVM运行时，默认是采用“混合模式”（Mixed Mode）,也就是解释器和即使编译器搭配使用的方式。当需要程序迅速启动和快速执行时，解释器可以首先发挥作用，省去耗时的优化时间；而随着时间的推移，编译器会逐渐发挥作用，把越来越多的字节码编译并优化成本地代码，提高执行效率。

Client模式和Server模式
这是Java的两种运行模式，顾名思义，一个用在客户端，一个用在服务器端。
Client模式和Server模式不同点在于即时编译器的优化程度，当采用Client模式时，JVM只启用一个叫C1的即时编译器（Client Complier）,这个即时编译器会对指令进行一些简单、可靠的优化；而当采用Server模式时，则会再启用一个叫C2的重量级即时编译器（Server Complier），会进行一些比较耗时的优化，甚至会做一些不可靠的激进优化。
HotSpot会根据自身版本和宿主机器的硬件性能自动选择运行模式，用户也可以使用“-client”或者“-server”来强制指定虚拟机运行在client模式或者server模式。
命令行上直接敲入“java -version”, 可以看到是在serer模式下运行，并且采用mixed mode：
敲入“java -client -version”，结果还是显示运行在Server模式：
![](http://7qna9x.com1.z0.glb.clouddn.com/version.png)
根据Oracle上的[资料](http://www.oracle.com/technetwork/java/hotspotfaq-138619.html#64bit_compilers)，结论是：64位的操作系统上只能采用server模式。看来Oracle认为server模式的激进优化在64位的操作系统上是优于client模式的简单优化的。

## 一.垃圾回收要解决的问题
　　根据 JVM 规范，JVM 内存共分为虚拟机栈、堆、方法区、程序计数器、本地方法栈五个部分。其中，**程序计数器、虚拟机栈、本地方法栈**3个区域随线程而生，随线程而灭；**栈中的栈帧随着方法的进入和退出有条不紊地执行着出栈和入栈操作**。每一个栈帧中分配多少内存基本上是在类结构确定下来时就已知的，因此这几个区域的内存分配和回收都具备确定性，故**这几个区域就不需要过多考虑回收的问题**，因为方法结束或者线程结束时，内存自然就跟着回收了。

### 1.1哪些内存需要回收？  
　　对于java堆和方法区则不一样，java堆是存放实例对象的地方，我们只有在程序运行期间才能知道会创建哪些对象，这部分内存的分配和回收是动态的，因此，垃圾收集器所关注的就是这一部分。
　　对于方法区（或者说HotSpot虚拟机中的永久代），垃圾回收主要是回收这两部分内容：**废弃常量和无用的类**。对于废弃常量，主要是判断当前系统中有没有对象引用这个常量；对于无用类则比较严格，需要满足下面三个条件：
（1）该类的所有实例都已经被回收，即堆中不存在该类任何实例对象；
（2）加载该类的ClassLoader已经被回收；
（3）该类对应的java.lang.Class对象没有在任何地方被引用，无法再任何地方通过反射访问该类的方法；
满足了上面三个条件也仅仅是“可以”进行回收了，还要根据HotSpot的一些配置参数综合考虑。

### 1.2什么时候回收？
垃圾收集器在对堆进行回收前，第一件事就是要确定这些对象之中哪些还“存活”着，哪些已经“死去”，对于这些已经“死去”的对象我们需要进行回收。判断对象是否存活的算法：

（1）引用计数算法
算法过程如下：【给对象添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能再被使用的】。
引用计数算法实现简单，判定效率也很高，大部分情况下是一个不错的算法。但有一个比较重要的缺点：**很难解决对象之间相互循环引用的问题**。比如：j假设变量objA、objB为某个类的对象实例，objA中持有一个指向objB的成员，此时objB的引用计数为1；在objB中持有一个指向objA的成员，此时objA的引用计数值也为1；此时，即使把objA、objB都置为null，此时两个对象都不能被回收，因为这两个对象虽然为null了，但是它们的引用计数值都还为1。

（2）可达性分析算法(根搜索算法)
**目前主流的虚拟机，如java默认虚拟机HotSpot就是用的这种方式**。算法基本思路为：【通过一系列的称为“**GC Roots**”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为**引用链**，当一个对象到GC Roots没有任何引用链相连时（或者说从GC Roots到这个对象不可达），则证明此对象是不可用的】。
可作为GC Roots的对象包括：
1）虚拟机栈（栈帧中的本地变量表）中引用的对象；
2）方法区中类静态static属性引用的对象；
3）方法区中常量final引用的对象；
4）本地方法栈中JNI（即一般说的Native方法）引用的对象；
![](http://7qna9x.com1.z0.glb.clouddn.com/gcroot.jpg)
    需要注意的是，即使在可达性分析算法中不可达的对象，也并非是“非死不可”的，要真正宣告一个对象死亡，至少要经历**两次标记**过程：如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链，那它将会被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行finalize()方法。当对象没有覆盖finalize()方法，或者finalize()方法已经被虚拟机调用过（也就是说对象的finalize()方法只能被调用一次），虚拟机将这两种情况都视为“没有必要执行”。
    如果这个对象被判定为有必要执行finalize()方法，那么这个对象将会放置在一个叫做F-Queue的队列中，并在稍后由一个由虚拟机自动建立的、低优先级的Finalizer线程去执行它（即去执行对象的finalize()方法，这里所谓的“执行”是值虚拟机会触发这个方法，但并不承若会等待它运行结束，主要是为了防止对象的finalize方法执行缓慢或发生死循环，导致其他对象不能被执行的，从而引起内存回收系统崩溃）。
    finalize()方法是对象逃脱死亡命运的最后一次机会，稍后GC将对F-Queue中的对象进行第二次小规模的标记，如果对象要在finalize()中成功拯救自己——只需要重新与引用链上的任何一个对象建立关联即可，譬如把自己（this）赋值给某个类变量或者对象的成员变量，那在第二次标记时它将被移除出“即将回收”的集合；如果对象这时候还没有逃脱，那基本上它就真的被回收了。

因此对于**不可达对象判定真正死亡的过程**小结如下：
（1）GC进行第一次标记并进行一次筛选（筛选那些覆盖了finalize方法并且finalize方法是第一次调用的对象）；--> （2）另一个低优先级的线程去调用那些被筛选出来的对象的finalize方法；--> （3）GC进行第二次标记，如果在前一步中那些筛选出来的对象没有在finalize拯救自己，此时，那些未被筛选到的和这些这些筛选到的但是没有拯救自己的对象都将会回收。

### 1.3如何回收？
如何高效地进行垃圾回收？由于Java虚拟机规范并没有对如何实现垃圾收集器做出明确的规定，因此各个厂商的虚拟机可以采用不同的方式来实现垃圾收集器，所以在此只讨论几种常见的垃圾收集算法的核心思想。

## 二.典型的垃圾回收算法

垃圾回收算法主要有三种，分别是标记-清除算法、复制算法、标记-整理算法。
所谓的分代收集算法，也是针对新生代、老年代配合使用这3种。

### 2.1 Mark-Sweep（标记-清除）算法
是最基础的垃圾回收算法，之所以说它是最基础的是因为它最容易实现，思想也是最简单的。分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象所占用的空间。标记过程就是上面可达性分析算法中所讲的二次标记过程。
缺点：
（1）效率问题：标记和清除的两个过程效率都不高；
（2）空间问题：标记清除后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后需要分配较大对象时，无法找到足够的连续内存而不得不提前出发另一次垃圾收集动作；

### 2.2 Copying（复制）算法
为了解决Mark-Sweep算法的缺陷，Copying算法就被提了出来。它将可用内存按容量划分为**大小相等**的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉，这样一来就不容易出现内存碎片的问题。
优点：运行高效且不容易产生内存碎片；
缺点：能够使用的内存缩减到原来的一半。

### 2.3 Mark-Compact（标记-整理）算法
为了解决Copying算法的缺陷，充分利用内存空间，提出了Mark-Compact算法。该算法标记阶段和Mark-Sweep一样，但是在完成标记之后，它不是直接清理可回收对象，而是**将存活对象都向一端移动**，然后清理掉端边界以外的内存。

### 2.4 Generational Collection（分代收集）算法
**分代收集算法是目前大部分JVM的垃圾收集器采用的算法**。它是基于这样一个事实：**不同对象的生命周期是不一样的**。因此，不同生命周期的对象可以采取不同的回收算法，以便提高回收效率。它的核心思想是根据对象存活的生命周期**将内存划分为若干个不同的区域**。一般情况下将堆区划分为**老年代**（Tenured Generation）和**新生代**（Young Generation），**老年代内存比新生代也大很多(大概比例是2:1)**，老年代的特点是每次垃圾收集时只有少量对象需要被回收，而新生代的特点是每次垃圾回收时都有大量的对象需要被回收，那么就可以根据不同代的特点采取最适合的收集算法。

**新生代**
**目前大部分垃圾收集器对于新生代都采取Copying算法**，因为新生代中每次垃圾回收都要回收大部分对象，也就是说需要复制的操作次数较少，但是实际中并不是按照1：1的比例来划分新生代的空间，一般来说是按照**8:1:1**的比例划**分为一个Eden区和两个Survivor(Survivor0,Survivor1)区**。每次使用Eden空间和其中的一块Survivor空间，当进行回收时，将Eden和Survivor0中还存活的对象复制到另一块Survivor1空间中，然后清理掉Eden和Survivor0空间。此时survivor0区是空的，然后将survivor0区和survivor1区交换，即保持survivor1区为空，如此往复。当对象在Survivor区躲过一次GC的话，其对象年龄便会加1，默认情况下，如果对象年龄达到15岁，就会移动到老年代中。
当survivor1区不足以存放Eden和survivor0的存活对象时，就将存活对象直接存放到老年代。【若是老年代也满了就会触发一次Full GC，也就是新生代、老年代都进行回收。】
新生代发生的GC也叫做Minor GC，MinorGC发生频率比较高(不一定等Eden区满了才触发)。

**老年代**
而由于老年代的特点是每次回收都只回收少量对象，**一般使用的是Mark-Compact算法**。
在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。
当老年代内存满时触发Major GC即Full GC，Full GC发生频率比较低，老年代对象存活时间比较长，存活率标记高。
一般来说，大对象会被直接分配到老年代，所谓的大对象是指需要大量连续存储空间的对象，最常见的一种大对象就是大数组，比如：
byte[] data = new byte[4*1024*1024]
这种一般会直接在老年代分配存储空间。当然分配的规则并不是百分之百固定的，这要取决于当前使用的是哪种垃圾收集器组合和JVM的相关参数。


## 三.典型的垃圾回收器
如果说上面介绍的收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现，按照上面的介绍，目前垃圾收集器基本都采用分代收集。**没有最好的垃圾收集器，只有最好的收集器组合**。一款收集器的好坏，主要有两个指标：**停顿时间和吞吐量**。不同的虚拟机提供的垃圾收集器也有很大差异，如下是HotSpot虚拟机基于JDK1.7版本所包含的所有垃圾收集器，如果两个收集器之间有连线，则说明它们可以搭配使用:
![](http://7qna9x.com1.z0.glb.clouddn.com/gc.jpg)
并发和并行
这两个名词都是并发编程中的概念，在谈论垃圾收集器的上下文语境中，它们可以解释如下：
     并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。
     并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），用户程序在继续运行，而垃圾收集程序运行于另一个CPU上。
     
Minor GC 和 Full GC
     新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
     老年代GC（Major GC / Full GC）：指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会比Minor GC慢10倍以上。一般情况下，对整个堆进行整理，包括Young、Old和Perm。Full GC因为需要对整个堆进行回收，所以比Minor GC要慢，因此应该尽可能减少Full GC的次数。在对JVM调优的过程中，很大一部分工作就是对于Full GC的调节。有如下原因可能导致Full GC：
1.年老代（Old）被写满
2.持久代（Perm）被写满
3.System.gc()被显示调用
4.上一次GC之后Heap的各域分配策略动态变化
     
吞吐量
吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即
吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间）
虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。

### 3.1 Serial（串行GC）收集器
最基本、发展历史最悠久的一种收集器。是针对新生代的收集器，采用的是Copying算法。（也有针对老年代的Serial Old，采用的是Mark-Compact算法）。看名字就知道，这个收集器是一个单线程的收集器，只使用一个CPU或一条收集线程去完成垃圾收集工作，最重要的是，在它进行垃圾收集的时候，**必须暂停其他所有的工作线程，在用户不可见的情况下把用户正常的线程全部停掉，也就是”Stop the world“，会给用户带来停顿**，直到它收集结束。虽然有这个缺点，但是Serial收集器的优点：**简单而高效**，对于限定单个CPU的环境来说，Serial收集器由于**没有线程交互**的开销，专心做垃圾收集自然可以获得较高的收集效率。到目前为止，**Serial收集器依然是虚拟机运行在Client模式下的默认的新生代垃圾收集器**。Serial、Serial Old收集器的工作示意图如下：
![](http://7qna9x.com1.z0.glb.clouddn.com/serial.jpg)

### 3.2 ParNew（并行GC）收集器
ParNew收集器是Serial收集器的多线程版本，使用多个线程进行垃圾收集，其他行为和Serial收集器一样。
![](http://7qna9x.com1.z0.glb.clouddn.com/ParNew.jpg)
**ParNew收集器是许多运行在Server模式下的虚拟机中首选的新生代收集器。除去性能因素，很重要的原因是除了Serial收集器外，目前只有它能与CMS收集器配合工作。**
但是，在单CPU环境中，ParNew收集器绝对不会有比Serial收集器更好的效果，甚至由于存在线程交互的开销，该收集器在通过超线程技术实现的两个CPU的环境中都不能百分之百地保证可以超越Serial收集器。然而，随着可以使用的CPU的数量的增加，它对于GC时系统资源的有效利用还是很有好处的。

### 3.3 Parallel Scavenge（并行GC）收集器
Parallel Scavenge收集器也是一个新生代收集器，它也是使用复制算法的收集器，又是并行多线程收集器。和ParNew相似，但是Parallel Scavenge的关注点不同，而**Parallel Scavenge收集器的目标是达到一个可控制的吞吐量**。

### 上面三种都是新生代收集器，下面介绍四种老年代收集器。

### 3.4 Serial Old（串行GC）收集器
Serial Old是Serial收集器的老年代版本，它同样使用一个单线程执行收集，使用“标记-整理”算法。**主要使用在Client模式下的虚拟机的老年代**。
当运行在Server模式下时，它主要有两大用途：
1.在JDK 1.5以及之前的版本中与Parallel Scavenge收集器搭配使用，因为那时还没有Parallel  Old老年代收集器搭配；
2.另一种就是作为CMS收集器的后备预案，在并发收集发生Concurrent Model Failure时使用。

### 3.5 Parallel Old（并行GC）收集器
Parallel Old是新生代收集器Prarallel Scavenge的老年代版本，**使用多线程和“标记-整理”算法**。
在Parallel Old出现之前，如果新生代选择了Parallel Scavenge，老年代除了Serial Old就没有别的选择，而由于受到单线程的Serial Old在服务器端表现的拖累，使用Parallel Scavenge也未必可以获得吞吐量最大化的效果。
**Parallel Old 收集器的出现让“吞吐量优先”收集器终于有了合适的应用组合**。
![](http://7qna9x.com1.z0.glb.clouddn.com/ParallelOld.jpg)


### 3.6 CMS（并发GC）收集器
CMS（Concurrent Mark Sweep）收集器是一种以**获取最短回收停顿时间**为目标的收集器。对于互联网站或者B/S系统的这种注重响应速度的服务端来说，CMS是很好的选择。CMS收集器是HotSpot虚拟机中的一款真正意义上的并发收集器，第一次实现了让垃圾回收线程和用户线程（基本上）同时工作。用CMS收集老年代的时候，新生代只能选择Serial或者ParNew收集器。
　　**CMS收集器是基于“标记-清除”算法实现的**，整个收集过程大致分为4个步骤：
①.初始标记(CMS initial mark)
②.并发标记(CMS concurrent mark)
③.重新标记(CMS remark)
④.并发清除(CMS concurrent sweep)
     其中初始标记、重新标记这两个步骤任然需要停顿其他用户线程（Stop The World）。初始标记仅仅只是标记出GC ROOTS能直接关联到的对象，速度很快，并发标记阶段是进行GC ROOTS 根搜索算法阶段，会判定对象是否存活。而重新标记阶段则是为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间会被初始标记阶段稍长，但比并发标记阶段要短。
     由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以整体来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。
![](http://7qna9x.com1.z0.glb.clouddn.com/CMS.jpg)
CMS收集器的优点：并发收集、低停顿，但是CMS还远远达不到完美，主要有三个显著缺点：
　　1.CMS收集器对CPU资源非常敏感。在并发（并发标记、并发清除）阶段，虽然不会导致用户线程停顿，但是会占用CPU资源而导致应用程序变慢，总吞吐量下降。CMS默认启动的回收线程数是：(CPU数量+3) / 4。收集器线程所占用的CPU数量为：（CPU+3）/4=0.25+3/（4*CPU）。因此这时垃圾收集器始终不会占用少于25%的CPU，因此当进行并发阶段时，虽然用户线程可以跑，但是很缓慢，特别是双核CPU的时候，已经占用了5/8的CPU，吞吐量会很低。为了解决这种情况，产生了“增量式并发收集器”（Incremental Concurrent Mark Sweep/i-CMS）。就是采用抢占方式来模拟多任务机制，就是在并发（并发标记、并发清除）阶段，让GC线程、用户线程交替执行，尽量减少GC线程独占CPU，这样垃圾收集过程更长，但是对用户程序影响小一些。实际上i-CMS效果很一般，目前已经被声明为“deprecated”。
　　2.CMS收集器无法处理浮动垃圾，可能出现“Concurrent Mode Failure“，失败后而导致另一次Full  GC的产生。由于CMS并发清理阶段用户线程还在运行，伴随程序的运行自热会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法在本次收集中处理它们，只好留待下一次GC时将其清理掉。这一部分垃圾称为“浮动垃圾”。也是由于在垃圾收集阶段用户线程还需要运行，即需要预留足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分内存空间提供并发收集时的程序运作使用。在默认设置下，CMS收集器在老年代使用了68%的空间时就会被激活，也可以通过参数-XX:CMSInitiatingOccupancyFraction的值来提高触发百分比，以降低内存回收次数提高性能。JDK1.6中，CMS收集器的启动阈值已经提升到92%。要是CMS运行期间预留的内存无法满足程序其他线程需要，就会出现“Concurrent Mode Failure”失败，这时候虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。所以说参数-XX:CMSInitiatingOccupancyFraction设置的过高将会很容易导致“Concurrent Mode Failure”失败，性能反而降低。
　　3.最后一个缺点，CMS是基于“标记-清除”算法实现的收集器，使用“标记-清除”算法收集后，会产生大量碎片。空间碎片太多时，将会给对象分配带来很多麻烦，比如说大对象，内存空间找不到连续的空间来分配不得不提前触发一次Full  GC。为了解决这个问题，CMS收集器提供了一个-XX:UseCMSCompactAtFullCollection开关参数，用于在Full  GC之后增加一个内存碎片的合并整理过程，但是内存整理过程是无法并发的，因此解决了空间碎片问题，却使停顿时间变长。还可通过-XX:CMSFullGCBeforeCompaction参数设置执行多少次不压缩的Full  GC之后，跟着来一次碎片整理过程（默认值是0，表示每次进入Full GC时都进行碎片整理）。


### 3.7 G1收集器
![](http://7qna9x.com1.z0.glb.clouddn.com/g1.jpg)
　G1(Garbage First)收集器是JDK1.7提供的一个新的面向服务端应用的垃圾收集器，**其目标就是替换掉JDK1.5发布的CMS收集器**。其优点有：
　　1.并发与并行：G1能充分利用多CPU、多核环境下的硬件优势，使用多个CPU（CPU或CPU核心）来缩短停顿（Stop The World）时间。
　　2.分代收集：G1不需要与其他收集器配合就能独立管理整个GC堆，但他能够采用不同方式去处理新建对象和已经存活了一段时间、熬过多次GC的老年代对象以获取更好收集效果。
　　3.空间整合：从整体来看是基于“标记-整理”算法实现，从局部（两个Region之间）来看是基于“复制”算法实现的，但是都意味着G1运行期间不会产生内存碎片空间，更健康，遇到大对象时，不会因为没有连续空间而进行下一次GC，甚至一次Full GC。
　　4.可预测的停顿：降低停顿是G1和CMS共同关注点，但G1除了追求低停顿，还能建立可预测的停顿模型，可以明确地指定在一个长度为M的时间片内，消耗在垃圾收集的时间不超过N毫秒
　　5.跨代特性：**之前的收集器进行收集的范围都是整个新生代或老年代，而G1扩展到整个Java堆(包括新生代，老年代)。**
那么是怎么实现的呢？
　　1.如何实现新生代和老年代全范围收集：其实它的Java堆布局就不同于其余收集器，它将整个Java堆划分为多个大小相等的独立区域（Region），仍然保留新生代和老年代的概念，可是不是物理隔离的，都是一部分Region（不需要连续）的集合。
　　2.如何建立可预测的停顿时间模型：是因为有了独立区域Region的存在，就避免在Java堆中进行全区域的垃圾收集，G1跟踪各个Region里面的垃圾堆积的价值大小（回收可以获得的空间大小和回收所需要的时间的经验值），后台维护一个优先队列，根据每次允许的收集时间，优先回收价值最大的Region（Garbage-First理念）。因此使用Region划分内存空间以及有优先级的区域回收方式，保证了有限时间获得尽可能高的收集效率。
　　3.如何保证垃圾回收真的在Region区域进行而不会扩散到全局：由于Region并不是孤立的，一个Region的对象可以被整个Java堆的任意其余Region的对象所引用，在做可达性判定确定对象是否存活时，仍然会关联到Java堆的任意对象，G1中这种情况特别明显。而以前在别的分代收集里面，新生代规模要比老年代小许多，新生代收集也频繁得多，也会涉及到扫描新生代时也会扫描老年代的情况，相反亦然。解决：G1收集器Region之间的对象引用以及新生代和老年代之间的对象引用，虚拟机都是使用Remembered Set来避免全堆扫描。G1中每个Region都有一个与之对应的Remembered Set,虚拟机发现程序在对Reference类型的数据进行写操作时，会产生一个Write Barrier暂时中断写操作，检查Reference引用的对象是否处于不同的Region之中(分代的例子中就检查是否老年代对象引用了新生代的对象)，如果是则通过CardTable把相关引用信息记录到被引用对象所属的Region的Remembered Set之中，当进行内存回收时，在GC根节点的枚举范围中加入Remembered Set即可避免全堆扫描。
忽略Remembered Set的维护，G1的运行步骤可简单描述为：
①.初始标记(Initial Marking)
②.并发标记(Concurrenr Marking)
③.最终标记(Final Marking)
④.筛选回收(Live Data Counting And Evacution)
1.初始标记：初始标记仅仅标记GC Roots能直接关联到的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的Region中创建新的对象。这阶段需要停顿线程，不可并行执行，但是时间很短。
2.并发标记：此阶段是从GC Roots开始对堆中对象进行可达性分析，找出存活对象，此阶段时间较长可与用户程序并发执行。
3.最终标记：此阶段是为了修正在并发标记期间因为用户线程继续运行而导致标记产生变动的那一份标记记录，虚拟机将这段时间对象变化记录在线程Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这段时间需要停顿线程，但是可并行执行。
4.筛选回收：对各个Region的回收价值和成本进行排序，根据用户期望的GC停顿时间来制定回收计划。
　　**如果现有的垃圾收集器没有出现任何问题，没有任何理由去选择G1，如果应用追求低停顿，G1可选择，如果追求吞吐量，和Parallel Scavenge/Parallel Old组合相比G1并没有特别的优势。**

### 总结：
本文介绍了HotSpot虚拟机中的七种垃圾收集器，最后用一张图总结一下：
![](http://7qna9x.com1.z0.glb.clouddn.com/gc-final.jpg)

## 四.垃圾收集器参数总结
	-XX:+<option> 启用选项
	-XX:-<option> 不启用选项
	-XX:<option>=<number> 
	-XX:<option>=<string>
<table border="1" width="100%" cellpadding="0" cellspacing="0"><tr align="center"><th>参数</th><th>描述</th></tr><tr align="left"><td>-XX:+UseSerialGC</td><td>Jvm运行在Client模式下的默认值，打开此开关后，使用Serial + Serial Old的收集器组合进行内存回收</td></tr><tr align="left"><td>-XX:+UseParNewGC</td><td>打开此开关后，使用ParNew + Serial Old的收集器进行垃圾回收</td></tr><tr align="left"><td>-XX:+UseConcMarkSweepGC</td><td>使用ParNew + CMS +  Serial Old的收集器组合进行内存回收，Serial Old作为CMS出现“Concurrent Mode Failure”失败后的后备收集器使用。</td></tr><tr align="left"><td>-XX:+UseParallelGC</td><td>Jvm运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge +  Serial Old的收集器组合进行回收</td></tr><tr align="left"><td>-XX:+UseParallelOldGC</td><td>使用Parallel Scavenge +  Parallel Old的收集器组合进行回收</td></tr><tr align="left"><td>-XX:SurvivorRatio</td><td>新生代中Eden区域与Survivor区域的容量比值，默认为8，代表Eden:Subrvivor = 8:1</td></tr><tr align="left"><td>-XX:MaxTenuringThreshold</td><td>晋升到老年代的对象年龄，每次Minor GC之后，年龄就加1，当超过这个参数的值时进入老年代</td></tr><tr align="left"><td>-XX:UseAdaptiveSizePolicy</td><td>动态调整java堆中各个区域的大小以及进入老年代的年龄</td></tr><tr align="left"><td>-XX:+HandlePromotionFailure</td><td>是否允许新生代收集担保，进行一次minor gc后, 另一块Survivor空间不足时，将直接会在老年代中保留</td></tr><tr align="left"><td>-XX:ParallelGCThreads</td><td>设置并行GC进行内存回收的线程数</td></tr><tr align="left"><td>-XX:GCTimeRatio</td><td>GC时间占总时间的比列，默认值为99，即允许1%的GC时间，仅在使用Parallel Scavenge 收集器时有效</td></tr><tr align="left"><td>-XX:MaxGCPauseMillis</td><td>设置GC的最大停顿时间，在Parallel Scavenge 收集器下有效</td></tr><tr align="left"><td>-XX:CMSInitiatingOccupancyFraction</td><td>设置CMS收集器在老年代空间被使用多少后出发垃圾收集，默认值为68%，仅在CMS收集器时有效，-XX:CMSInitiatingOccupancyFraction=70</td></tr><tr align="left"><td>-XX:+UseCMSCompactAtFullCollection</td><td>由于CMS收集器会产生碎片，此参数设置在垃圾收集器后是否需要一次内存碎片整理过程，仅在CMS收集器时有效</td></tr><tr align="left"><td>-XX:+CMSFullGCBeforeCompaction</td><td>设置CMS收集器在进行若干次垃圾收集后再进行一次内存碎片整理过程，通常与UseCMSCompactAtFullCollection参数一起使用</td></tr><tr align="left"><td>-XX:+UseFastAccessorMethods</td><td>原始类型优化</td></tr><tr align="left"><td>-XX:+DisableExplicitGC</td><td>是否关闭手动System.gc</td></tr><tr align="left"><td>-XX:+CMSParallelRemarkEnabled</td><td>降低标记停顿</td></tr><tr align="left"><td>-XX:LargePageSizeInBytes</td><td>内存页的大小不可设置过大，会影响Perm的大小，-XX:LargePageSizeInBytes=128m</td></tr></table>


Client、Server模式默认GC:
<table border="1" width="100%" cellpadding="10" cellspacing="0"><tr align="center"><th></th><th>新生代GC方式</th><th>老年代和持久代GC方式</th></tr><tr align="left"><td>Client</td><td>Serial 串行GC</td><td>Serial Old 串行GC</td></tr><tr align="left"><td>Server</td><td>Parallel Scavenge  并行回收GC</td><td>Parallel Old 并行GC</td></tr></table>


Sun/Oracle JDK GC组合方式:
<table border="1" width="100%" cellpadding="0" cellspacing="0"><tr align="center"><th></th><th>新生代GC方式</th><th>老年代和持久代GC方式</th></tr><tr align="left"><td>-XX:+UseSerialGC</td><td>Serial 串行GC</td><td>Serial Old 串行GC</td></tr><tr align="left"><td>-XX:+UseParallelGC</td><td>Parallel Scavenge  并行回收GC</td><td>Parallel Old 并行GC</td></tr><tr align="left"><td>-XX:+UseConcMarkSweepGC</td><td>ParNew 并行GC</td><td>CMS 并发GC ; 当出现“Concurrent Mode Failure”时采用Serial Old 串行GC</td></tr><tr align="left"><td>-XX:+UseParNewGC</td><td>ParNew 并行GC</td><td>Serial Old 串行GC</td></tr><tr align="left"><td>-XX:+UseParallelOldGC</td><td>Parallel Scavenge  并行回收GC</td><td>Parallel Old 并行GC</td></tr><tr align="left"><td>-XX:+UseConcMarkSweepGC -XX:+UseParNewGC</td><td>Serial 串行GC</td><td>CMS 并发GC ; 当出现“Concurrent Mode Failure”时采用Serial Old 串行GC</td></tr></table>


## 参考
周志明的《深入理解java虚拟机》
[Oracle官网上的，权威、原汁原味的介绍了垃圾收集器](http://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/)


