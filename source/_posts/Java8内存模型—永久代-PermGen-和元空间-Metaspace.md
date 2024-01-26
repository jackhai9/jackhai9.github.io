title: Java8内存模型—永久代(PermGen)和元空间(Metaspace)
date: 2017-08-02 16:21:17
categories: 技术
tags: [Java,内存模型]
---

## 一、JVM运行时数据区
**根据 JVM 规范**，JVM 内存共分为虚拟机栈、堆、方法区、程序计数器、本地方法栈五个部分。

![](http://7qna9x.com1.z0.glb.clouddn.com/jvm-1.png)
<!--more-->
JVM运行时数据区：

![](http://7qna9x.com1.z0.glb.clouddn.com/jvm-3.png)

1、虚拟机栈：每个线程有一个私有的栈，随着线程的创建而创建。栈里面存着的是一种叫“栈帧”的东西，每个方法会创建一个栈帧，栈帧中存放了局部变量表（基本数据类型和对象引用）、操作数栈、方法出口等信息。栈的大小可以固定也可以动态扩展。当栈调用深度大于JVM所允许的范围，会抛出 StackOverflowError 的错误，不过这个深度范围不是一个恒定的值，我们通过下面这段程序可以测试一下这个结果：

栈溢出测试源码： java.lang.StackOverflowError

```java
public class StackErrorMock {
    private static int index = 1;
 
    public void call(){
        index++;
        call();
    }
 
    public static void main(String[] args) {
        StackErrorMock mock = new StackErrorMock();
        try {
            mock.call();
        }catch (Throwable e){
            System.out.println("Stack deep : "+index);
            e.printStackTrace();
        }
    }
}
```

虚拟机栈除了上述 Error 外，还有另一种 Error ，那就是当申请不到空间时，会抛出 OutOfMemoryError 。这里有一个小细节需要注意，catch 捕获的是 Throwable 。Error 是可以 catch 的，而且也可以向常规 Exception 一样被处理，而且就算不捕捉的话也只是导致当前线程挂掉，其他线程还是可以正常运行，如果有需要的话捕捉 Error 之后也可以做些其他处理。但是 Error 是一种系统内部的错误，这种错误不像 Exception 一样是可能是程序和业务上的错误是可以恢复的。

2、本地方法栈：
　　这部分主要与虚拟机用到的 Native 方法相关，一般情况下， Java 应用程序员并不需要关心这部分的内容。

3、PC 寄存器：
　　PC 寄存器，也叫程序计数器。JVM支持多个线程同时运行，每个线程都有自己的程序计数器。倘若当前执行的是 JVM 的方法，则该寄存器中保存当前执行指令的地址；倘若执行的是native 方法，则PC寄存器中为空。

4、堆：
　　堆内存是 JVM 所有线程共享的部分，在虚拟机启动的时候就已经创建。所有的对象和数组都在堆上进行分配。这部分空间可通过 GC 进行回收。当申请不到空间时会抛出 OutOfMemoryError 。下面我们简单的模拟一个堆内存溢出的情况：java.lang.OutOfMemoryError: Java heap space

```java
import java.util.ArrayList;
import java.util.List;
 
public class HeapOomMock {
    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<byte[]>();
        int i = 0;
        boolean flag = true;
        while (flag){
            try {
                i++;
                list.add(new byte[1024 * 1024]);//每次增加一个1M大小的数组对象
            }catch (Throwable e){
                e.printStackTrace();
                flag = false;
                System.out.println("count="+i);//记录运行的次数
            }
        }
    }
}
```

5、方法区：
　　方法区也是所有线程共享。主要用于存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。 关于方法区内存溢出的问题会在下文中详细探讨。

## 二、PermGen（永久代）
　　绝大部分 Java 程序员应该都见过 "java.lang.OutOfMemoryError: PermGen space "这个异常。**这里的 “PermGen space”其实指的就是方法区。不过方法区和“PermGen space”又有着本质的区别。前者是 JVM 的规范，而后者则是 JVM 规范的一种实现**，并且只有 HotSpot 才有 “PermGen space”，而对于其他类型的虚拟机，如 JRockit（Oracle）、J9（IBM） 并没有“PermGen space”。由于方法区主要存储类的相关信息，所以对于动态生成类的情况比较容易出现永久代的内存溢出。最典型的场景就是，在 jsp 页面比较多的情况，容易出现永久代内存溢出。我们现在通过动态生成类来模拟 “PermGen space”的内存溢出，如下代码段1：
```java
import java.io.File;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.ArrayList;
import java.util.List;
 
public class PermGenOomMock{
    public static void main(String[] args) {
        URL url = null;
        List<ClassLoader> classLoaderList = new ArrayList<ClassLoader>();
        try {
            url = new File("/tmp").toURI().toURL();
            URL[] urls = {url};
            while (true){
                ClassLoader loader = new URLClassLoader(urls);
                classLoaderList.add(loader);
                loader.loadClass("com.paddx.test.memory.Test");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

本例中使用的 JDK 版本是 1.7，指定的 PermGen 区的大小为 8M。通过每次生成不同URLClassLoader对象来加载Test类，从而生成不同的类对象，这样就能看到我们熟悉的 "java.lang.OutOfMemoryError: PermGen space " 异常了。这里之所以采用 JDK 1.7，是因为在 JDK 1.8 中， HotSpot 已经没有 “PermGen space”这个区间了，取而代之是一个叫做 Metaspace（元空间） 的东西。下面我们就来看看 Metaspace 与 PermGen space 的区别。

## 三、Metaspace（元空间）
　　其实，移除永久代的工作从JDK1.7就开始了。JDK1.7中，存储在永久代的部分数据就已经转移到了Java Heap或者是 Native Heap。但永久代仍存在于JDK1.7中，并没完全移除，譬如符号引用(Symbols)转移到了native heap；字面量(interned strings)转移到了java heap；类的静态变量(class statics)转移到了java heap。我们可以通过一段程序来比较 JDK 1.6 与 JDK 1.7及 JDK 1.8 的区别，以字符串常量为例：
```java
import java.util.ArrayList;
import java.util.List;
 
public class StringOomMock {
    static String  base = "string";
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        for (int i=0;i< Integer.MAX_VALUE;i++){
            String str = base + base;
            base = str;
            list.add(str.intern());
        }
    }
}
```

这段程序以2的指数级不断的生成新的字符串，这样可以比较快速的消耗内存。我们通过 JDK 1.6、JDK 1.7 和 JDK 1.8 分别运行：
JDK 1.6 的运行结果：
```java
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
```

JDK 1.7 的运行结果：
```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

JDK 1.8 的运行结果：
```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

从上述结果可以看出，JDK 1.6下，会出现“PermGen Space”的内存溢出，而在 JDK 1.7和 JDK 1.8 中，会出现堆内存溢出，并且 JDK 1.8中 PermSize 和 MaxPermGen 已经无效。因此，可以大致验证 JDK 1.7 和 1.8 将字符串常量由永久代转移到堆中，并且 JDK 1.8 中已经不存在永久代的结论。现在我们看看元空间到底是一个什么东西？
　　**元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。**因此，默认情况下，元空间的大小仅受本地内存限制，但可以通过以下参数来指定元空间的大小：
　　-XX:MetaspaceSize，初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。
　　-XX:MaxMetaspaceSize，最大空间，默认是没有限制的。
　　除了上面两个指定大小的选项以外，还有两个与 GC 相关的属性：
　　-XX:MinMetaspaceFreeRatio，在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集
　　-XX:MaxMetaspaceFreeRatio，在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集
　　
　　现在我们在 JDK 8下重新运行一下代码段1，不过这次不再指定 PermSize 和 MaxPermSize。而是指定 MetaSpaceSize 和 MaxMetaSpaceSize的大小。输出结果如下：
```java
Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
```

从输出结果，我们可以看出，这次不再出现永久代溢出，而是出现了元空间的溢出。

## 四、总结
通过上面分析，大家应该大致了解了 JVM 的内存划分，也清楚了 JDK 8 中永久代向元空间的转换。不过大家应该都有一个疑问，就是为什么要做这个转换？所以，最后给大家总结以下几点原因：
    
    1、字符串存在永久代中，容易出现性能问题和内存溢出。
    2、类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
    3、永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
    4、Oracle 可能会将 HotSpot 与 JRockit 合二为一。

## Java内存模型
多任务和高并发是衡量一台计算机处理器的能力重要指标之一。一般衡量一个服务器性能的高低好坏，使用每秒事务处理数（Transactions Per Second，**TPS**）这个指标比较能说明问题，它代表着一秒内服务器平均能响应的请求数，而TPS值与程序的并发能力有着非常密切的关系。在讨论Java内存模型和线程之前，先简单介绍一下硬件的效率与一致性。
### 1. 硬件的效率与一致性
   由于计算机的存储设备与处理器的运算能力之间有几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的高速缓存（cache）来作为内存与处理器之间的缓冲：将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中没这样处理器就无需等待缓慢的内存读写了。
　　基于高速缓存的存储交互很好地解决了处理器与内存的速度矛盾，但是引入了一个新的问题：**缓存一致性（Cache Coherence）**。在多处理器系统中，每个处理器都有自己的高速缓存，而他们又共享同一主存，如下图所示：多个处理器运算任务都涉及同一块主存，需要一种协议可以保障数据的一致性，这类协议有MSI、MESI、MOSI及Dragon Protocol等。Java虚拟机内存模型中定义的内存访问操作与硬件的缓存访问操作是具有可比性的，后续将介绍Java内存模型。
　　![](http://7qna9x.com1.z0.glb.clouddn.com/jmm1.jpg)
　　除此之外，为了使得处理器内部的运算单元能竟可能被充分利用，处理器可能会对输入代码进行乱起执行（Out-Of-Order Execution）优化，处理器会在计算之后将对乱序执行的代码进行结果重组，保证结果准确性。与处理器的乱序执行优化类似，**Java虚拟机的即时编译器(JIT)中也有类似的指令重排序（Instruction Recorder）优化**。
### 2. Java内存模型
   Java内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样底层细节。此处的变量与Java编程时所说的变量不一样，指包括了实例字段、静态字段和构成数组对象的元素，但是不包括局部变量与方法参数，后者是线程私有的，不会被共享。

   Java内存模型中规定了**所有的变量都存储在主内存中，每条线程还有自己的工作内存**（可以与前面将的处理器的高速缓存类比），**线程的工作内存中保存了该线程使用到的变量到主内存副本拷贝，线程对变量的所有操作（读取、赋值）都必须在工作内存中进行，而不能直接读写主内存中的变量。**不同线程之间无法直接访问对方工作内存中的变量，线程间变量值的传递均需要在主内存来完成，线程、主内存和工作内存的交互关系如下图所示，和上图很类似。
   ![](http://7qna9x.com1.z0.glb.clouddn.com/jmm2.jpg)
这里的主内存、工作内存与Java内存区域的Java堆、栈、方法区不是同一层次内存划分。

![](http://7qna9x.com1.z0.glb.clouddn.com/jmm5.jpg)
对于普通变量，一个线程中更新的值，不能马上反应在其他变量中。如果需要在其他线程中立即可见，需要使用volatile关键字作为标识。
		
		1. 原子性：如下的八种基本操作都具有原子性。Java内存模型只保证了基本读取和赋值是原子性操作，如果要实现更大范围操作的原子性，可以通过synchronized和Lock来实现。由于synchronized和Lock能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。
		2. 可见性：一个线程修改了变量，其他线程可以立即知道：
		    保证可见性的方法：
		    volatile
		    synchronized （unlock之前，写变量值回主存）
		    final(一旦初始化完成，其他线程就可见)
		3. 有序性：在本线程内，操作都是有序的；在线程外观察，操作都是无序的。在Java内存模型中，允许编译器和处理器对指令进行重排序，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。在Java里面，可以通过volatile关键字来保证一定的“有序性”。另外可以通过synchronized和Lock来保证有序性，很显然，synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。对于volatile，JMM内存屏障插入策略：
		    在每个volatile写操作的前面插入一个StoreStore屏障。
		    在每个volatile写操作的后面插入一个StoreLoad屏障。
		    在每个volatile读操作的后面插入一个LoadLoad屏障。
		    在每个volatile读操作的后面插入一个LoadStore屏障。


内存间交互操作：
关于主内存与工作内存之间的具体交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步到主内存之间的实现细节，Java内存模型定义了以下八种操作来完成：
	•	lock（锁定）：作用于主内存的变量，把一个变量标识为一条线程独占状态。
	•	unlock（解锁）：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
	•	read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
	•	load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
	•	use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
	•	assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
	•	store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
	•	write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。
	![](http://7qna9x.com1.z0.glb.clouddn.com/jmm4.png)
   如果要把一个变量从主内存中复制到工作内存，就需要按顺寻地执行read和load操作，如果把变量从工作内存中同步回主内存中，就要按顺序地执行store和write操作。每一个操作都是原子的，即执行期间不会被中断。**Java内存模型只要求上述操作必须按顺序执行，而没有保证必须是连续执行。**也就是read和load之间，store和write之间是可以插入其他指令的，如对主内存中的变量a、b进行访问时，可能的顺序是read a，read b，load b， load a。Java内存模型还规定了在执行上述八种基本操作时，必须满足如下规则：
	•	不允许read和load、store和write操作之一单独出现
	•	不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中。
	•	不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。
	•	一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。
	•	一个变量在同一时刻只允许一条线程对其进行lock操作，lock和unlock必须成对出现
	•	如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值
	•	如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。
	•	对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）。

### 3. 重排序
在执行程序时为了提高性能，编译器和处理器经常会对指令进行重排序。重排序分成三种类型：
	1.	编译器优化的重排序。编译器在不改变单线程程序语义放入前提下，可以重新安排语句的执行顺序。
	2.	指令级并行的重排序。现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
	3.	内存系统的重排序。由于处理器使用缓存和读写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从Java源代码到最终实际执行的指令序列，会经过下面三种重排序：
![](http://7qna9x.com1.z0.glb.clouddn.com/jmm3.png)
为了保证内存的可见性，Java编译器在生成指令序列的适当位置会插入**内存屏障指令**来禁止特定类型的处理器重排序。

### 参考
  1.[Java8内存模型](http://www.cnblogs.com/paddix/p/5309550.html)
  2.[浅析java内存模型--JMM(Java Memory Model)](http://www.cnblogs.com/lewis0077/p/5143268.html)


