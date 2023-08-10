# Memory & Trap

[TOC]



## Page tables

操作系统实现了页表机制有以下好处：

- 权限控制：可读、可写、可执行
- 私有地址空间：每个进程只能访问自己的页表
- 多个进程复用单一物理内存

### Paging hardware

在Sv39 RISC-V上，虚拟地址仅启用了低39位，其中12位**偏移量（offset**），27位**索引（index）**。所以在一级页表中，共有$2^{27}(134,217,728)$个**page table entries（PTE）**。每一个PTE包含44位的物理页面号**（Physical Page Number， PPN）**以及一些权限位：

- V：页表是否有效，
- R/W：读写位
- X：页面中的数据解释为指令
- U：用户态是否可以访问此页面。如果U未设置，那么该PTE只能被特权级访问
- A：由软件清理，可以实现一些置换算法

![image-20230810202753050](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230810202753050.png)



也就是说paging hardware将虚拟地址的top 27 bits of the 39 bits映射到物理地址的top 44 bits of the 56 bits。而偏移量直接映射到低12位上。如下图所示：

![image-20230810202434477](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230810202434477.png)

> 注意：这里物理地址比虚拟地址大得多，但是在实际中，物理内存一般仅有16GB，而虚拟内存可高达128TB。



### Multi-Page

我们发现直接分配一个一级页表给进程的话，可能有很多PTE用不到，而造成页表的利用率并不高。这里我们采用多级页表的方案**按需分配**，其基本工作原理如下图所示：

![image-20230810202422281](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230810202422281.png)

计算可得，若使用全部的虚拟内存，则三级页表占用的空间为 $2^9 * 2^9 * 2^9 * 54 + 2^9 * 2^9 * 54 +  2^9 * 54$。这要比一级页表所占用的空间$ 2^27 * 54$大，但是记住这里是**按需分配页表的**！实际上多级页表可以高效利用内存空间。



这里有`satp`寄存器，用于存储第一级页表的基地址。



### Virtual Space

一般来说，内核的页表采用直接映射的方案，这样方便内存直接修改物理内存的内容。注意页表本身也是一段内存，采用直接映射的方式，内核可以方便、正确地修改页表的内容。所以内核的虚拟地址空间如下：

![image-20230810203650445](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230810203650445.png)

注意：在内核的虚拟地址中，Trampoline这段代码不仅在MAXVA处，还在Kernel text中，也就是说这两处虚拟内存映射到同一处物理地址上，这样做的原因是为了方便在用户态与内核态之间的切换。同理还有Kstack在MAXVA处，也在Kernel data处。



用户进程的虚拟地址空间如下：

![image-20230810204523307](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230810204523307.png)

### Page Falut

RISC-V区分以下三种page fault：

- load page fault： when a load instruction cannot translate its virtual address
- store page fault： when a store instruction cannot translate its virtual address
- instruction page fault： when the address in the program counter doesn’t translate

## Trap

在RISC-V中，每个CPU核都有自己的控制寄存器。下面列出比较重要的寄存器：

- stvec：中断处理程序的地址
- sepc：保存中断发生时正在执行的指令地址。sret指令将sepc中的内容复制到pc中
- seval：保存无法翻译的地址。注意要和sepc区分开来，sepc是指令本身的地址，而seval可能是指令中的根据操作数计算出的地址。
- scause：一个整数，表示中断的原因
- sscratch：中断处理程序利用该寄存器完成寄存器保存工作
- sstatus：
  - SIE位决定是否开启设备中断；
  - SPP位指示从用户模式来，还是从特权模式来，在sret时，CPU根据SPP设置返回后的执行模式。



When it needs to force a trap, the RISC-V hardware does the following for all trap types (other

than timer interrupts):

1. If the trap is a device interrupt, and the sstatus SIE bit is clear, don’t do any of the following.

2. Disable interrupts by clearing the SIE bit in sstatus。**此时禁用了设备中断！！！**
3. Copy the pc to sepc.
4. Save the current mode (user or supervisor) in the SPP bit in sstatus.
5. Set scause to reflect the trap’s cause.
6. Set the mode to supervisor.
7. Copy stvec to the pc.
8. Start executing at the new pc.

注意：当中断发生时，CPU并不切换到内核页表，不切换到内核栈，不保存任何通用寄存器。这样做的原因是 CPU does minimal work during a trap is to provide **flexibility** to software;

