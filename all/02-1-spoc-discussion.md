# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
- **BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？**

第一个扇区是bootloader。因为BIOS的能力没有那么强。

> BIOS完成硬件初始化和自检后，会根据CMOS中设置的启动顺序启动相应的设备，这里假定按顺序系统要启动硬盘。但此时，文件系统并没有建立，BIOS也不知道硬盘里存放的是什么，所以BIOS是无法直接启动操作系统的。另外一个硬盘可以有多个分区，每个分区都有可能包括一个不同的操作系统，BIOS也无从判断应该从哪个分区启动，所以对待硬盘，所有的BIOS都是读取硬盘的0磁头、0柱面、1扇区的内容，然后把控制权交给这里面的MBR(Main Boot Record）。
> MBR由两个部分组成：即主引导记录MBR和硬盘分区表DPT。在总共512字节的主引导分区里其中MBR占446个字节（偏移0--偏移1BDH)，一般是一段引导程序，其主要是用来在系统硬件自检完后引导具有激活标志的分区上的操作系统。DPT占64个字节（偏移1BEH--偏移1FDH),一般可放4个16字节的分区信息表。最后两个字节“55，AA”（偏移1FEH，偏移1FFH)是分区的结束标志。

- **比较UEFI和BIOS的区别。**

UEFI依赖于文件系统，而BIOS是直接读取主引导扇区内容并跳转的。

BIOS启动流程：
* 系统开机 - 上电自检（Power On Self Test 或 POST）。
* POST过后初始化用于启动的硬件（磁盘、键盘控制器等）。
* BIOS会运行BIOS磁盘启动顺序中第一个磁盘的首440bytes（MBR启动代码区域）内的代码。
* 启动引导代码从BIOS获得控制权，然后引导启动下一阶段的代码（如果有的话）（一般是系统的启动引导代码）。
* 再次被启动的代码（二阶段代码）（即启动引导）会查阅支持和配置文件。
* 根据配置文件中的信息，启动引导程序会将内核和initramfs文件载入系统的RAM中，然后开始启动内核
。
UEFI启动流程：
* 系统开机 - 上电自检（Power On Self Test 或 POST）。
* UEFI 固件被加载，并由它初始化启动要用的硬件。
* 固件读取其引导管理器以确定从何处（比如，从哪个硬盘及分区）加载哪个 UEFI 应用。
* 固件按照引导管理器中的启动项目，加载UEFI 应用。
* 已启动的 UEFI 应用还可以启动其他应用（对应于 UEFI shell 或 rEFInd 之类的引导管理器的情况）或者启动内核及initramfs（对应于GRUB之类引导器的情况），这取决于 UEFI 应用的配置。
（来源：[知乎](https://www.zhihu.com/question/21672895)）

> UEFI，全称Unified Extensible Firmware Interface，即“统一的可扩展固件接口”, 是适用于电脑的标准固件接口，旨在代替BIOS（基本输入/输出系统）。此标准由UEFI联盟中的140多个技术公司共同创建，其中包括微软公司。UEFI旨在提高软件互操作性和解决BIOS的局限性。
> UEFI启动对比BIOS启动的优势有三点：
> 1. 安全性更强：UEFI启动需要一个独立的分区，它将系统启动文件和操作系统本身隔离，可以更好的保护系统的启动。
> 2. 启动配置更灵活：EFI启动和GRUB启动类似，在启动的时候可以调用EFIShell，在此可以加载指定硬件驱动，选择启动文件。比如默认启动失败，在EFIShell加载U盘上的启动文件继续启动系统。
> 3. 支持容量更大:传统的BIOS启动由于MBR的限制，默认是无法引导超过2TB以上的硬盘的。随着硬盘价格的不断走低，2TB以上的硬盘会逐渐普及，因此UEFI启动也是今后主流的启动方式。

## 3.2 系统启动流程

- **分区引导扇区的结束标志是什么？**

结束标志是`0x55AA`。

- **在UEFI中的可信启动有什么作用？**

UEFI安全引导（Secure Boot）的核心职能是利用数字签名来确认EFI驱动程序或者应用程序是否是受信任的。（[知乎](https://zhuanlan.zhihu.com/p/25279889)）

> 通过启动前的数字签名检查来保证启动介质的安全性。

## 3.3 中断、异常和系统调用比较
- **什么是中断、异常和系统调用？**
* 系统调用（system call）：应用程序主动向操作系统发出的服务请求
* 异常（exception）：非法指令或其他原因导致当前指令执行失败（如：内存出错）后的处理请求
* 中断（hardware interrupt）：来自硬件设备的处理请求

> 中断：外部意外的响应；
> 异常：指令执行意外的响应；
> 系统调用：系统调用指令的响应；

- **中断、异常和系统调用的处理流程有什么异同？**
中断处理的流程：
* 硬件处理：
  - 在CPU初始化时设置中断使能标志
  - 依据内部或外部事件设置中断标志
  - 依据中断向量调用对应的中断服务例程
* 软件处理：
  - 现场保存（编译器）
  - 中断服务处理（服务例程）
  - 清除中断标记（服务例程）（系统调用只占用一个中断向量，另有系统调用表）
  - 现场恢复（编译器）

系统调用的处理流程：
* 每个系统调用对应一个系统调用号
  - 系统调用接口根据系统调用号来维护表的索引
* 系统调用接口调用内核态中的系统调用功能实现，并返回系统调用的状态和结果

相同点：
* 都需要进入内核态进行处理
* 都需要查找中断向量表

不同点：
* 中断的来源是硬件，异常的来源是应用程序或内核，系统调用的来源是应用程序
* 中断是异步的，异常是同步的，系统调用是异步或同步的
* 中断可以通过硬件开启
* 中断通过设备驱动程序处理，异常通过异常处理程序处理，系统调用通过系统调用表查找系统调用的具体实现处理

> 源头、响应方式、处理机制

- **以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？**

以下代码摘录自`kern/syscall/syscall.c`:
```c
static int (*syscalls[])(uint32_t arg[]) = {
    [SYS_exit]              sys_exit,
    [SYS_fork]              sys_fork,
    [SYS_wait]              sys_wait,
    [SYS_exec]              sys_exec,
    [SYS_yield]             sys_yield,
    [SYS_kill]              sys_kill,
    [SYS_getpid]            sys_getpid,
    [SYS_putc]              sys_putc,
    [SYS_pgdir]             sys_pgdir,
    [SYS_gettime]           sys_gettime,
    [SYS_lab6_set_priority] sys_lab6_set_priority,
    [SYS_sleep]             sys_sleep,
    [SYS_open]              sys_open,
    [SYS_close]             sys_close,
    [SYS_read]              sys_read,
    [SYS_write]             sys_write,
    [SYS_seek]              sys_seek,
    [SYS_fstat]             sys_fstat,
    [SYS_fsync]             sys_fsync,
    [SYS_getcwd]            sys_getcwd,
    [SYS_getdirentry]       sys_getdirentry,
    [SYS_dup]               sys_dup,
};
```

* SYS_exit：退出系统
* SYS_fork：创建一个与原来进程几乎完全相同的进程
* SYS_wait：等待另一个进程
* SYS_exec：根据指定的文件名找到可执行文件，并用它来取代调用进程的内容
* SYS_yield：当前CPU所处理的进程提前退出
* SYS_kill：杀死进程
* SYS_getpid：获得进程的pid
* SYS_putc：输出字符
* SYS_pgdir：？
* SYS_gettime：获得系统时间
* SYS_lab6_set_priority：设置优先级（？）
* SYS_sleep：暂停
* SYS_open：打开文件
* SYS_close：关闭文件
* SYS_read：读取文件内容
* SYS_write：向文件写内容
* SYS_seek：改变文件中指针的位置
* SYS_fstat：文件状态
* SYS_fsync：文件同步
* SYS_getcwd：获得当前工作目录
* SYS_getdirentry：将目录项读入
* SYS_dup：复制一个文件的描述符，经常用来重定向进程的stdin、stdout和stderr

## 3.4 linux系统调用分析
-  通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
- 通过分析`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore系统调用过程，及调用参数和返回值的传递方法。
