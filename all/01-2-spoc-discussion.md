# 操作系统概述

---

## **提前准备**

（请在上课前完成）

* 完成lec1的视频学习和提交对应的在线练习
* git pull ucore\_os\_lab, ucore\_os\_docs, os\_tutorial\_lab, os\_course\_exercises in github repos。这样可以在本机上完成课堂练习。
* 知道OS课程的入口网址，会使用在线视频平台，在线练习/实验平台，在线提问平台\(piazza\)
  * [http://os.cs.tsinghua.edu.cn/oscourse/OS2017spring](http://os.cs.tsinghua.edu.cn/oscourse/OS2017spring)
* 了解学习方法

```
  for (i=1; i<=13; i++) {
    1. 完成第i周课前视频学习
    2. 完成第i周OpenEdx上的在线基本练习(直接在OpenEdx上提交)
    3. 上SPOC课并完成课上练习（课堂上完成并提交到piazza上的老师指定的特定帖下）
    4. 在deadline前，按序完成ucore_lab实验
    5. if (i==7) 参加期中考试;
    6. if (i==13) 参加期末考试;
    7. if (碰到问题)　到piazza的论坛上提问;
   }
```

* 会使用linux shell命令，如ls, rm, mkdir, cat, less, more, gcc等，也会使用linux系统的基本操作。
* 了解v9-computer会使用v9-computer的dis,xc, xem命令（包括启动参数）,阅读v9-computer中的v9_computer.md文档，了解汇编指令的类型和含义等，了解v9-computer的细节。
* [v9 computer](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)

* 在piazza上就学习中不理解问题进行提问。

* 观看[Peter Denning’s talk video, “Perspectives on OS Foundations”, presented at the SOSP History Day Workshop (SOSP‘15).](https://www.youtube.com/watch?v=xs6cDaXbsUA)


# 操作系统概述思考题

---

## 个人思考题

---
* 当前常见的操作系统包括**redhat, Centos, Debian, Fedora, Ubuntu, Mac OS X, iOS, Andriod, Windows**。
* 当前常见的操作系统主要用**C和汇编**编程语言编写。
* 操作系统是**对程序进行控制，对资源进行管理**的一种系统软件。
* "Operating system"这个单词起源于**Operator**（大型机的操作员）。
* 在计算机系统中，控制和管理各种资源、有效地组织多道程序运行的系统软件称作**操作系统**。
* 允许多用户将若干个作业提交给计算机系统集中处理的操作系统称为**批处理**操作系统。（还有单用户和多道程序系统）
* 你了解的当前世界上使用最多的32bit CPU是{%s%} ARM {%ends%}，其上运行最多的操作系统是{%s%} Android{%ends%}。（不了解，虽然看了答案，但还没有在网上找到来源数据）
* 应用程序通过**系统调用**接口获得操作系统的服务。
* 推动操作系统发展的因素有**硬件的升级以及新硬件类型、新的服务要求、对错误的修正**。（[计算机操作系统学习指导与习题解答](https://books.google.co.jp/books?isbn=7302120986)）
* 现代操作系统的特征包括**并发、共享、虚拟、异步**。
* 操作系统内核的架构设计包括**简单结构**（macro/uni kernel）、**分层结构**、**微内核结构**（Microkernel）、**外核结构**（Exokernel）。

---

请总结你认为操作系统应该具有的特征有什么？并对其特征进行简要阐述。

操作系统具有并发、共享、虚拟、同步的特点：
* 并发：计算机系统中同时存在多个运行的程序，需要OS管理和调度
* 共享：计算机系统中的程序对系统资源进行分时共享
* 虚拟：利用多道程序设计技术，让每个用户都觉得有一个计算机专门为他服务
* 异步

<!--
从总体上看，操作系统具有五个方面的特征：虚拟性（Virtualization）、并发性（concurrency）、异步性、共享性和持久性（persistency）。在虚拟性方面，可以从操作系统对内存，CPU的抽象和处理上有更好的理解；对于并发性和共享性方面，可以从操作系统支持多个应用程序“同时”运行的情况来理解；对于异步性，可以从操作系统调度，中断处理对应用程序执行造成的影响等几个放马来理解；对于持久性方面，可以从操作系统中的文件系统支持把数据方便地从磁盘等存储介质上存入和取出来理解。
-->



为什么没有人用python，java来实现操作系统？

事实上，python和java都是可以用来实现操作系统的（没有看到python的例子，但Sun Microsystems真的出过一个跑在JVM上的[JavaOS](https://en.wikipedia.org/wiki/JavaOS)）。然而，它们各自都有一些问题。其中最主要的是效率和生态问题：既然已经有了Linux和Windows了，我们难道需要一个JavaLinux吗？

单纯就实现而言，python的问题是很难接触到底层，仍然需要用C或汇编来写中断代码之类的；Java的问题是它需要在JVM中运行，而JVM需要内核。JavaOS的实现方式就是在硬件的微内核上跑JVM，然后再在JVM上跑Java。

[stackoverflow上对python操作系统的讨论](https://stackoverflow.com/questions/10904721/is-it-possible-to-create-an-operating-system-using-python)
[stackoverflow上对Java操作系统的讨论](https://stackoverflow.com/questions/29580170/why-operating-systems-are-not-written-in-java)

---

## v9-computer相关题目


- 请分析模拟v9\-computer的em.c。理解：在v9\-computer中如何实现时钟中断的；v9 computer的CPU指令，关键变量描述有误或不全的情况；在v9\-computer中的跳转相关操作是如何实现的；在v9\-computer中如何设计相应指令，可有效实现函数调用与返回；OS程序被加载到内存的哪个位置,其堆栈是如何设置的；在v9\-computer中如何完成一次内存地址的读写的；在v9\-computer中如何实现分页机制；

{%s%}

https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/tools/em.c       



https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md

{%ends%}

- 请编写一个小程序，在v9-cpu下，能够输出字符

{%s%}

https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_helloworld

{%ends%}

- 请编写一个小程序，在v9-cpu下，能够接收你输入的字符并输出你输入的字符

{%s%}

{%ends%}

- 请编写一个小程序，在v9-cpu下，能够产生各种异常/中断

{%s%}

https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_bad_phys_addr  

https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_divid_by_zero   

https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_invalid_intruction

https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_page_fault  

https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_timer_interrupt  

{%ends%}

- 请编写一个小程序，在v9-cpu下，能够统计并显示内存大小

{%s%}

{%ends%}
