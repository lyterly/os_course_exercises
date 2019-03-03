# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

ucore的进程切换需要硬件支持时钟中断；虚存管理需要地址映射，故需要MMU等硬件；文件系统需要保证操作系统的持久性，故需要硬件中的稳定的存储介质。
至少应当提供的特权指令：中断使能指令，触发软中断等中断相关的指令，设置内存寻址模式，设置页表等内存管理相关，执行I/O操作等文件系统相关的

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

区别在于：
实模式中，整个物理内存被看作分段的区域，程序代码和数据位于不同区域，系统程序和用户程序没有区分，而且每个指针都指向实际物理地址。在此模式下，一个指针如果被错误地指向程序区域并改变了值，将产生严重的后果。
保护模式下，物理内存地址不能直接访问，要由操作系统来将虚拟地址转换为物理地址来访问。

物理地址：处理器提交到总线上用于访问内存和外设的地址
逻辑地址：有地址变换功能的计算机中，访存指令给出的地址
线性地址：逻辑地址到物理地址变换间的中间层，是处理器通过段机制控制下形成的地址空间

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

risc-v有三种特权模式：机器模式，监督模式和用户模式
机器模式：拥有最高特权，可不受限制地访问整个机器。可低层次访问机器的实现
监督模式：当操作系统需要处理异常或中断时，机器的控制权将交给监督模式。提供虚拟存储系统。
用户模式：用于传统应用程序，访存空间受限，需要通过相应接口来请求所需的系统服务。

在地址访问方面：机器模式可不受限制地访问整个机器；用户模式和监督模式由于虚拟存储系统的存在，可使用的逻辑地址空间大于实际可用的绝对地址空间

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

这些数字代表每个域在结构体中所占的位数。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

假设机器为小端序，对intr中的每个域进行填充：
gd_off_15_0 = 3 & 0xffff = 0x0003
gd_ss = sel = 0x0002
gd_args = b00000
gd_rsv1 = b000
gd_type = STS_TG32 = 0xf
gd_s = b0
gd_dpl = b00
gd_p = b1
gd_off_31_16 = 0x0000

填充结束后，intr取前4个字节，由于是小端序，故intr的值为0x20003

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
