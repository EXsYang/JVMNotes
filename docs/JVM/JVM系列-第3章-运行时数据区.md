---
title: JVM系列-第3章-运行时数据区
tags:
  - JVM
  - 虚拟机
categories:
  - JVM
  - 1.内存与垃圾回收篇
keywords: JVM，虚拟机。
description: JVM系列-第3章-运行时数据区。
cover: 'https://npm.elemecdn.com/lql_static@latest/logo/jvm.png'
abbrlink: a7ad3cab
date: 2020-11-09 15:38:42
---





> 此章把运行时数据区里比较少的地方讲一下。虚拟机栈，堆，方法区这些地方后续再讲。

运行时数据区概述及线程
=================

前言
----

本节主要讲的是运行时数据区，也就是下图这部分，它是在类加载完成后的阶段

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0001.png">

当我们通过前面的：类的加载 --> 验证 --> 准备 --> 解析 --\> 初始化，这几个阶段完成后，就会用到执行引擎对我们的类进行使用，同时执行引擎将会使用到我们运行时数据区

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0002.png">

类比一下也就是大厨做饭，我们把大厨后面的东西（切好的菜，刀，调料），比作是运行时数据区。而厨师可以类比于执行引擎，将通过准备的东西进行制作成精美的菜品。

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0003.png">



运行时数据区结构
----------

### 运行时数据区与内存

1. 内存是非常重要的系统资源，是硬盘和CPU的中间仓库及桥梁，承载着操作系统和应用程序的实时运行。JVM内存布局规定了Java在运行过程中内存申请、分配、管理的策略，保证了JVM的高效稳定运行。**不同的JVM对于内存的划分方式和管理机制存在着部分差异**。结合JVM虚拟机规范，来探讨一下经典的JVM内存布局。

2. 我们通过磁盘或者网络IO得到的数据，都需要先加载到内存中，然后CPU从内存中获取数据进行读取，也就是说内存充当了CPU和磁盘之间的桥梁

> 下图来自阿里巴巴手册JDK8

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0004.jpg">



### 线程的内存空间

1. Java虚拟机定义了若干种程序运行期间会使用到的运行时数据区：其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁**【一个虚拟机对应一个进程,堆和方法区是一个进程只有一份，或者说每一个JVM只有一份堆和方法区】**。另外一些则是与线程一一对应的，这些与线程对应的数据区域会随着线程开始和结束而创建和销毁。
2. 灰色的为单独线程私有的，红色的为多个线程共享的。即：
   - 线程独有：独立包括程序计数器、栈、本地方法栈
   - 线程间共享：堆、堆外内存（永久代或元空间、代码缓存）

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0005.png">



### Runtime类

**每个JVM只有一个Runtime实例**。即为运行时环境，相当于内存结构的中间的那个框框：运行时数据区。

Runtime实例对象是单例的,在一个虚拟机当中运行时数据区只有一份

![image-20240607005427744](https://raw.githubusercontent.com/EXsYang/PicGo-images-hosting/main/images/image-20240607005427744.png)

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0006.png">



线程
----

### JVM 线程



1.  线程是一个程序里的运行单元。JVM允许一个应用有多个线程并行的执行
2.  **在Hotspot JVM里，每个线程都与操作系统的本地线程直接映射**
    - 当一个Java线程准备好执行以后，此时一个操作系统的本地线程也同时创建。Java线程执行终止后，本地线程也会回收
3.  操作系统负责将线程安排调度到任何一个可用的CPU上。一旦本地线程初始化成功，它就会调用Java线程中的run()方法

4. 关于JVM虚拟机在Java线程终止后是否可以终止的问题。如果在进程中剩下的都是守护进程或者当前线程是最后一个非守护线程，那么JVM虚拟机也就可以终止了



> 关于线程，并发可以看笔者的Java并发系列



### JVM 系统线程



- 如果你使用jconsole或者是任何一个调试工具，都能看到在后台有许多线程在运行。这些后台线程不包括调用`public static void main(String[])`的main线程以及所有这个main线程自己创建的线程。

- 这些主要的后台系统线程在Hotspot JVM里主要是以下几个：

1.  **虚拟机线程**：这种线程的操作是需要JVM达到安全点才会出现。这些操作必须在不同的线程中发生的原因是他们都需要JVM达到安全点，这样堆才不会变化。这种线程的执行类型括"stop-the-world"的垃圾收集，线程栈收集，线程挂起以及偏向锁撤销
2.  **周期任务线程**：这种线程是时间周期事件的体现（比如中断），他们一般用于周期性操作的调度执行
3.  **GC线程**：这种线程对在JVM里不同种类的垃圾收集行为提供了支持
4.  **编译线程**：这种线程在运行时会将字节码编译成到本地代码
5.  **信号调度线程**：这种线程接收信号并发送给JVM，在它内部通过调用适当的方法进行处理



程序计数器(PC寄存器)
===========

PC寄存器介绍
----------

> 官方文档网址：https://docs.oracle.com/javase/specs/jvms/se8/html/index.html

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0007.png">

1.  JVM中的程序计数寄存器（Program Counter Register）中，Register的命名源于CPU的寄存器，**寄存器存储指令相关的现场信息**。CPU只有把数据装载到寄存器才能够运行。
2.  这里，并非是广义上所指的物理寄存器，或许将其翻译为PC计数器（或指令计数器）会更加贴切（也称为程序钩子），并且也不容易引起一些不必要的误会。**JVM中的PC寄存器是对物理PC寄存器的一种抽象模拟**。
3.  它是一块很小的内存空间，几乎可以忽略不记。也是运行速度最快的存储区域。
4.  在JVM规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致。
5.  任何时间一个线程都只有一个方法在执行，也就是所谓的**当前方法**。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址；或者，如果是在执行native方法，则是未指定值（undefned）。
6.  它是**程序控制流**的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
7.  字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。
8.  它是**唯一一个**在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。

![image-20240607114733573](https://raw.githubusercontent.com/EXsYang/PicGo-images-hosting/main/images/image-20240607114733573.png)


## PC寄存器的作用

PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码。由执行引擎读取下一条指令，并执行该指令。

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0008.png">



举例
------

```java
public class PCRegisterTest {

    public static void main(String[] args) {
        int i = 10;
        int j = 20;
        int k = i + j;

        String s = "abc";
        System.out.println(i);
        System.out.println(k);

    }
}
```

查看字节码

> 看字节码的方法：https://blog.csdn.net/21aspnet/article/details/88351875

```java
Classfile /F:/IDEAWorkSpaceSourceCode/JVMDemo/out/production/chapter04/com/atguigu/java/PCRegisterTest.class
  Last modified 2020-11-2; size 675 bytes
  MD5 checksum 53b3ef104479ec9e9b7ce5319e5881d3
  Compiled from "PCRegisterTest.java"
public class com.atguigu.java.PCRegisterTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#26         // java/lang/Object."<init>":()V
   #2 = String             #27            // abc
   #3 = Fieldref           #28.#29        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #30.#31        // java/io/PrintStream.println:(I)V
   #5 = Class              #32            // com/atguigu/java/PCRegisterTest
   #6 = Class              #33            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcom/atguigu/java/PCRegisterTest;
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               args
  #17 = Utf8               [Ljava/lang/String;
  #18 = Utf8               i
  #19 = Utf8               I
  #20 = Utf8               j
  #21 = Utf8               k
  #22 = Utf8               s
  #23 = Utf8               Ljava/lang/String;
  #24 = Utf8               SourceFile
  #25 = Utf8               PCRegisterTest.java
  #26 = NameAndType        #7:#8          // "<init>":()V
  #27 = Utf8               abc
  #28 = Class              #34            // java/lang/System
  #29 = NameAndType        #35:#36        // out:Ljava/io/PrintStream;
  #30 = Class              #37            // java/io/PrintStream
  #31 = NameAndType        #38:#39        // println:(I)V
  #32 = Utf8               com/atguigu/java/PCRegisterTest
  #33 = Utf8               java/lang/Object
  #34 = Utf8               java/lang/System
  #35 = Utf8               out
  #36 = Utf8               Ljava/io/PrintStream;
  #37 = Utf8               java/io/PrintStream
  #38 = Utf8               println
  #39 = Utf8               (I)V
{
  public com.atguigu.java.PCRegisterTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/atguigu/java/PCRegisterTest;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=5, args_size=1
         0: bipush        10
         2: istore_1
         3: bipush        20
         5: istore_2
         6: iload_1
         7: iload_2
         8: iadd
         9: istore_3
        10: ldc           #2                  // String abc
        12: astore        4
        14: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        17: iload_1
        18: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
        21: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        24: iload_3
        25: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
        28: return
      LineNumberTable:
        line 10: 0
        line 11: 3
        line 12: 6
        line 14: 10
        line 15: 14
        line 16: 21
        line 18: 28
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      29     0  args   [Ljava/lang/String;
            3      26     1     i   I
            6      23     2     j   I
           10      19     3     k   I
           14      15     4     s   Ljava/lang/String;
}
SourceFile: "PCRegisterTest.java"
```



*   左边的数字代表**指令地址（指令偏移）**，即 PC 寄存器中可能存储的值，然后执行引擎读取 PC 寄存器中的值，并执行该指令

<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0009.png">



两个面试题
-------

**使用PC寄存器存储字节码指令地址有什么用呢？**或者问**为什么使用 PC 寄存器来记录当前线程的执行地址呢？**

1.  因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行
  
2.  JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令


<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0010.png">



**PC寄存器为什么被设定为私有的？**

1.  我们都知道所谓的多线程在一个特定的时间段内只会执行其中某一个线程的方法，CPU会不停地做任务切换，这样必然导致经常中断或恢复，如何保证分毫无差呢？<mark>**为了能够准确地记录各个线程正在执行的当前字节码指令地址，最好的办法自然是为每一个线程都分配一个PC寄存器**</mark>，这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况。
2.  由于CPU时间片轮限制，众多线程在并发执行过程中，任何一个确定的时刻，一个处理器或者多核处理器中的一个内核，只会执行某个线程中的一条指令。
3.  这样必然导致经常中断或恢复，如何保证分毫无差呢？每个线程在创建后，都会产生自己的程序计数器和栈帧，程序计数器在各个线程之间互不影响。

在讨论计算机科学和软件开发的上下文中，**并行**和**并发**这两个术语经常被提及，它们描述了程序执行多任务的方式，但各有侧重点。

### 并发 (Concurrency)
并发是指程序的设计上允许多个任务在重叠的时间段内执行。在并发模型中，单个核心的处理器可以同时处理多个任务的执行。重点在于任务启动和完成的时间上：这些任务可以交错进行，但在任何具体的时刻，可能只有一个任务在执行。并发更多地关注如何有效地管理多个同时发生的任务。

例如，在单核处理器上运行的操作系统可以允许**用户同时运行多个应用程序**（如文字处理器、浏览器等），通过时间片轮转（time slicing）的方式，使得用户感觉到所有应用都在同时运行，尽管在任何给定的微秒内，实际上只有一个任务在处理器上执行。

### 并行 (Parallelism)
并行处理指的是同时使用多个计算资源来执行多个任务或一个任务的不同部分。这通常涉及多核或多处理器的系统，其中不同的处理器核心可以同时执行不同的任务。并行的重点是同时执行，以减少完成任务所需的总时间。

在并行计算的例子中，例如，一个视频渲染软件可能会将一个大任务（如视频文件的渲染）分割成多个小部分，然后这些小部分分别在不同的处理器核心上同时处理。这样可以显著减少完成整个渲染过程所需的时间。

### 总结
- **并发**：**一个处理器多任务交替执行**，强调任务管理和资源共享。
- **并行**：**多个处理器或多个核心同时执行多个任务或一个任务的多个部分**，强调性能和效率提升。

理解这两者的区别有助于更好地设计和优化软件应用程序，特别是在处理性能瓶颈和响应时间方面。

CPU 时间片
---------

1.  CPU时间片即CPU分配给各个程序的时间，每个线程被分配一个时间段，称作它的时间片。
  
2.  在宏观上：我们可以同时打开多个应用程序，每个程序并行不悖，同时运行。
  
3.  但在微观上：由于只有一个CPU，一次只能处理程序要求的一部分，如何处理公平，一种方法就是引入时间片，**每个程序轮流执行**。


<img src="https://npm.elemecdn.com/youthlql@1.0.8/JVM/chapter_003/0011.png">



