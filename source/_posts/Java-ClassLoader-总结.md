title: Java ClassLoader 总结
date: 2014-11-07 18:35:56
categories: 技术
tags: [Java,ClassLoader]
---

## 一.介绍

程序在启动时，并不会一次性加载所要用的所有class文件，而是根据程序的需要，通过Java的类加载机制（ClassLoader）来动态加载某个 class 文件到内存中，从而只有 class 文件被载入到了内存之后，才能被其它 class 所引用。所以 ClassLoader 就是用来动态加载 class 文件到内存当中的。具体作用就是：

```
 负责将 Class 加载到 JVM 中
 审查每个类由谁加载（父优先的等级加载机制） 
 将 Class 字节码重新解析成 JVM 统一要求的对象格式
```

![](http://7qna9x.com1.z0.glb.clouddn.com/14.png)

<!--more-->

**类加载的动态性体现：**
一个应用程序总是由n多个类组成，Java程序启动时，并不是一次把所有的类全部加载后再运行，它总是先把保证程序运行的基础类一次性加载到jvm中，其它类等到jvm用到的时候再加载，这样的好处是节省了内存的开销，因为java最早就是为嵌入式系统而设计的，内存宝贵，这是一种可以理解的机制，而用到时再加载这也是java动态性的一种体现。

JVM加载Class文件的两种方法(都是使用ClassLoader来加载):

```
 隐式加载， 程序在运行过程中当碰到通过new 等方式生成对象时，隐式调用类装载器加载对应的类到jvm中。
 显式加载， 通过Class.forName()、this.getClass.getClassLoader().loadClass()等方法显式加载需要的类，或者我们自己实现的 ClassLoader 的 findClass() 方法。
```

Class.forName是一个静态方法，同样可以用来加载类。
该方法有两种形式：Class.forName(String name,boolean initialize, ClassLoader loader)和Class.forName(String className)。第一种形式的参数 name 表示的是类的全名；initialize表示是否初始化类；loader表示加载时使用的类加载器。第二种形式则相当于设置了参数 initialize 的值为 true，loader的值为当前类的类加载器。Class.forName的一个很常见的用法是在加载数据库驱动的时候。如Class.forName("org.apache.derby.jdbc.EmbeddedDriver")用来加载 Apache Derby 数据库的驱动。


![](http://7qna9x.com1.z0.glb.clouddn.com/10.png?imageView2/2/w/850/h/750)

![](http://7qna9x.com1.z0.glb.clouddn.com/12.png?imageView2/2/w/850/h/750)

![](http://7qna9x.com1.z0.glb.clouddn.com/11.png?imageView2/2/w/850/h/750)


Java中有5种创建对象的方式：

| 创建对象的方式 | 是否调用构造函数 | 
| ------- | :-------: | 
| 使用new关键字 | 调用构造函数 | 
| 使用Class类的newInstance方法 | 调用构造函数 | 
| 使用Constructor类的newInstance方法 | 调用构造函数 | 
| 使用clone方法 | 不调用构造函数 | 
| 使用反序列化 | 不调用构造函数 | 

![](http://7qna9x.com1.z0.glb.clouddn.com/332.png)

既然使用newInstance()构造对象的地方通过new关键字也可以创建对象，为什么又会使用newInstance()来创建对象呢？
    假设定义了一个接口Door，开始的时候是用木门的，定义为一个类WoodenDoor，在程序里就要这样写 Door door = new WoodenDoor() 。假设后来生活条件提高，换为自动门了，定义一个类AutoDoor，这时程序就要改写为 Door door = new AutoDoor() 。虽然只是改个标识符，如果这样的语句特别多，改动还是挺大的。于是出现了工厂模式，所有Door的实例都由DoorFactory提供，这时换一种门的时候，只需要把工厂的生产模式改一下，还是要改一点代码。
    而如果使用newInstance()，则可以在不改变代码的情况下，换为另外一种Door。具体方法是把Door的具体实现类的类名放到配置文件中，通过newInstance()生成实例。这样，改变另外一种Door的时候，只改配置文件就可以了。示例代码如下：
    String className = 从配置文件读取Door的具体实现类的类名; 
    Door door = (Door) Class.forName(className).newInstance();
    再配合依赖注入的方法，就提高了软件的可伸缩性、可扩展性。

## 二.具体原理


```java
public abstract class ClassLoader
```
ClassLoader类是一个抽象类

### Java默认提供的三个ClassLoader：

```
 BootStrap ClassLoader：称为启动类加载器，是Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等
 Extension ClassLoader：称为扩展类加载器，负责加载Java的扩展类库，Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。默认加载JAVA_HOME/jre/lib/ext/目下的所有jar。
 Application ClassLoader：称为系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。
```

### ClassLoader加载类的原理：

  1. ClassLoader使用的是**双亲委托模型**来搜索类的，每个ClassLoader实例都有一个父类加载器的引用（不是继承的关系，是一个包含的关系），虚拟机内置的类加载器（Bootstrap ClassLoader）本身没有父类加载器，但可以用作其它ClassLoader实例的的父类加载器。当一个ClassLoader实例需要加载某个类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是由上至下依次检查的，首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。

  2. **为什么要使用双亲委托这种模型呢？**因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要 ClassLoader再加载一次。考虑到安全因素，我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义的类型，这样会存在非常大的安全隐患，而双亲委托的方式，就可以避免这种情况，因为String已经在启动时就被引导类加载器（Bootstrcp ClassLoader）加载，所以用户自定义的ClassLoader永远也无法加载一个自己写的String，除非你改变JDK中ClassLoader搜索类的默认算法。

  3. **JVM在搜索类的时候，又是如何判定两个class是相同的呢？** JVM在判定两个class是否相同时，不仅要判断两个类名是否相同，而且要判断是否由同一个类加载器实例加载的。只有两者同时满足的情况下，JVM才认为这两个class是相同的。就算两个class是同一份class字节码，如果被两个不同的ClassLoader实例所加载，JVM也会认为它们是两个不同class。比如网络上的一个Java类org.classloader.simple.NetClassLoaderSimple，javac编译之后生成字节码文件NetClassLoaderSimple.class，ClassLoaderA和ClassLoaderB这两个类加载器并读取了NetClassLoaderSimple.class文件，并分别定义出了java.lang.Class实例来表示这个类，对于JVM来说，它们是两个不同的实例对象，但它们确实是同一份字节码文件，如果试图将这个Class实例生成具体的对象进行转换时，就会抛运行时异常java.lang.ClassCaseException，提示这是两个不同的类型。

### JVM通过ClassLoader加载class文件的过程：

![](http://7qna9x.com1.z0.glb.clouddn.com/334.png)

* 	第一阶段找到 .class 文件并把这个文件包含的字节码加载到内存中。
* 	第二阶段分三步，字节码验证；class 类数据结构分析及相应的内存分配；最后的符号表的链接。
   1. 验证，ClassLoader对类的字节码要做很多检测，以确保格式正确，行为正确。
   2. 准备，ClassLoader准备每个类中定义的字段、方法和实现接口所必须的数据结构。
   3. 解析，ClassLoader装入类所引用的其他所有类。
* 	第三阶段是类中静态属性和初始化赋值，以及静态块的执行等。


### ClassLoader的体系架构：

![](http://7qna9x.com1.z0.glb.clouddn.com/333.png)


### 实现自定义的 ClassLoader：

ClassLoader 能够完成的事情有以下情况：

* 	在自定义路径下查找自定义的class类文件。
* 	对我们自己要加载的类做特殊处理。
* 	可以定义类的实现机制。

虽然在绝大多数情况下，系统默认提供的类加载器实现已经可以满足需求。但是在某些情况下，您还是需要为应用开发出自己的类加载器。比如您的应用通过网络来传输 Java 类的字节代码，为了保证安全性，这些字节代码经过了加密处理。这个时候您就需要自己的类加载器来从某个网络地址上读取加密后的字节代码，接着进行解密和验证，最后定义出要在 Java 虚拟机中运行的类来。

**定义自己的类加载器分两步：**
```
继承java.lang.ClassLoader
重写父类的findClass方法
```

#### 1. 自定义文件系统类加载器：加载存储在文件系统上的 Java 字节代码。

```java
public class FileSystemClassLoader extends ClassLoader
{
    private String rootDir;

    public FileSystemClassLoader(String rootDir){
        this.rootDir = rootDir;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null){
            throw new ClassNotFoundException();
        }
        else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream os = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead = 0;
            while ((bytesNumRead = ins.read(buffer)) != -1){
                os.write(buffer, 0, bytesNumRead);
            }
            return os.toByteArray();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        return rootDir + File.separatorChar + className.replace('.', File.separatorChar) + ".class";
    }
}
```

类 FileSystemClassLoader继承自类java.lang.ClassLoader。java.lang.ClassLoader类的方法loadClass()封装了前面提到的双亲委托机制（代理模式）的实现。该方法会首先调用findLoadedClass()方法来检查该类是否已经被加载过；如果没有加载过的话，会调用父类加载器的loadClass()方法来尝试加载该类；如果父类加载器无法加载该类的话，就调用 findClass()方法来查找该类。因此，为了保证类加载器都正确实现代理模式，在开发自己的类加载器时，最好不要覆写 loadClass()方法，而是覆写findClass()方法。
类 FileSystemClassLoader的 findClass()方法首先根据类的全名在硬盘上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 defineClass()方法来把这些字节代码转换成java.lang.Class类的实例。

#### 2. 自定义网络类加载器

通过网络类加载器来实现组件的动态更新。基本的场景是：Java 字节代码（.class）文件存放在服务器上，客户端通过网络的方式获取字节代码并执行。当有版本更新的时候，只需要替换掉服务器上保存的文件即可。通过类加载器可以比较简单的实现这种需求。
类 NetworkClassLoader 负责通过网络下载 Java 类字节代码并定义出 Java 类。它的实现与FileSystemClassLoader 类似。在通过 NetworkClassLoader 加载了某个版本的类之后，一般有两种做法来使用它。第一种做法是使用 Java 反射 API。另外一种做法是使用接口。需要注意的是，并不能直接在客户端代码中引用从服务器上下载的类，因为客户端代码的类加载器找不到这些类。使用 Java 反射 API 可以直接调用 Java 类的方法。而使用接口的做法则是把接口的类放在客户端中，从服务器上加载实现此接口的不同版本的类。在客户端通过相同的接口来使用这些实现类。


### WEB应用中的ClassLoader：

对于运行在 Java EE™ 容器中的 Web 应用来说，类加载器的实现方式与一般的 Java 应用有所不同。不同的 Web 容器的实现方式也会有所不同。以 Apache Tomcat 来说，每个Web 应用都有一个对应的类加载器实例。该类加载器也使用代理模式，所不同的是它是首先尝试去加载某个类，如果找不到再代理给父类加载器。这与一般类加载器的顺序是相反的。这是 Java Servlet 规范中的推荐做法，其目的是使得Web 应用自己的类的优先级高于 Web 容器提供的类。这种代理模式的一个例外是：Java 核心库的类是不在查找范围之内的。这也是为了保证 Java 核心库的类型安全。

绝大多数情况下，Web 应用的开发人员不需要考虑与类加载器相关的细节。下面给出几条简单的原则：

* 每个 Web 应用自己的 Java 类文件和使用的库的 jar 包，分别放在 WEB-INF/classes和 WEB-INF/lib目录下面。
* 多个应用共享的 Java 类文件和 jar 包，分别放在 Web 容器指定的由所有 Web 应用共享的目录下面。
* 当出现找不到类的错误时，检查当前类的类加载器和当前线程的上下文类加载器是否正确。

### 参考
1.[深度分析 Java 的 ClassLoader 机制（源码级别）](http://blog.jobbole.com/96145/)
2.[深入浅出ClassLoader](http://ifeve.com/classloader/)
3.[深入探讨 Java 类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/)
4.[深入分析Java ClassLoader原理](http://blog.csdn.net/xyang81/article/details/7292380)
5.[详细深入分析 Java ClassLoader 工作机制](http://www.54tianzhisheng.cn/2017/03/26/%E8%AF%A6%E7%BB%86%E6%B7%B1%E5%85%A5%E5%88%86%E6%9E%90%20Java%20ClassLoader%20%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6/)



