高级语言并不直接提供system call。因为这种行为直接与特定操作系统绑定，而这与编程语言跨平台的设计目标相违背。相反，它们会提供一个中间层封装各种不同的system call，并为用户抽象出一个统一的接口来访问它们


Isolation
	- Memory Isolation - Virtual Address
	- CPU Isolation(Time Sharing) - Schedule Not Cooperation
Kernel/User Mode
Systemcall

Before loading OS, the code in BOOT must be executed by CPU, cause the code in BOOT is trust by hardware, so its privilege is the most highest!

Kernel = Trusted Computing Base(TCB)
Kernel must have no bugs
Kernel must treat processes as malicious

tight integration on sub modules for performance 宏核

amount of code is small for fewer bugs 微核




thread相比process，它的通信、上下文切换的代价小
satp寄存器 

物理地址空间大小的上限由硬件设计者来决定（地址线的根数可以减少），对应地，页表中的PA字段具有相应的位数。

而虚拟地址空间大小的上限也是由硬件与定义的，因为翻译工作是由MMU硬件来自动完成的，软件（操作系统）方面仅仅涉及到修改页表的内容，也就是定义映射关系而已。如果MMU需要三个阶段来翻译地址，那么对应的OS要管理三级页表。
而用户虚拟空间的布局是由OS来决定的，随机化、段的排列等

satp与页表的表项都是存储PA。要是存储VA，那么就要递归查询了，这就没有任何意义了。

从地址的翻译过程中我们可以做出这种推断：页表以4kb对齐

Translation Lookaside Buffer（TLB）加速翻译过程。
CPU首先查询TLB，若查询失败再启动翻译过程

switch page table =》 Flush TLB 

注意：硬件要将物理地址空间要映射到不同的设备上，不要误以为物理地址空间仅仅是内存专用的。

为了方便修改用户页表（主要还是方便直接修改内存），OS采用direct mapping（identify mapping ）。当关闭地址翻译的方式来修改内存时，还要考虑内存地址的偏移量。

TLB刷新，会冲洗掉进程的缓存状态

.global XXXX， 就是为了XXXX函数可以在别的文件中使用。







如何debug
在一个终端里make qemu-gdb 在另一个终端里gdb-multiarch

b 函数名

delete

watch

b XXXX if 

![image-20230724123247487](assets\image-20230724123247487.png)





tui enable
layout asm
layout reg
layout split
focus reg



step 就是单步执行，遇到子函数就进入并且继续单步执行
next 是在单步执行时，在函数内遇到子函数时不会进入子函数内单步执行



info breakpoints 
info reg

info frame

info args

info locals



p *add  打印指定地址的内容

p *add@{num}

![image-20230724122812111](assets\image-20230724122812111.png)

`x /nfu <addr>`

n表示要显示的内存单元的个数

f表示显示方式, 可取如下值
x 按十六进制格式显示变量。
d 按十进制格式显示变量。
u 按十进制格式显示无符号[整型](https://so.csdn.net/so/search?q=整型&spm=1001.2101.3001.7020)。
o 按八进制格式显示变量。
t 按[二进制](https://so.csdn.net/so/search?q=二进制&spm=1001.2101.3001.7020)格式显示变量。
a 按十六进制格式显示变量。
i 指令地址格式
c 按字符格式显示变量。
f 按[浮点](https://so.csdn.net/so/search?q=浮点&spm=1001.2101.3001.7020)数格式显示变量。



u表示一个地址单元的长度
b表示单字节，
h表示双字节，
w表示四字节，
g表示八字节



![image-20230724115750986](assets\image-20230724115750986.png)

ReturnAddress是有关指令的，而FP是有关数据的。不能用简单的算术运算代替FP，因为Caller的栈帧大小是不确定的，所以Callee要用FP返回到正确的数据区中。

栈帧：当前活跃函数所使用的数据集，由编译器自动管理。栈帧包括以下部分（有顺序性）

- Saved Register

  - Return address。如果当前函数callee要调用其他函数，那么必须把它的返回地址放在ra中，但是ra保存着caller的返回地址，所以要放在栈中。一旦子函数执行完毕，ra就立即恢复到原先的状态。

    一般叶子节点可以不用在栈中保存Return address，因为可以直接通过ra来访问到。

  - FP：FP与Return address类似，一个是返回caller的指令区，另一个返回callee的数据区

  - Other Saved Register

- Local Variable

- Param + Return value （大小不确定）



Process State ：  PC + Generic Register + Control Register









`ctrl+a  c` qemu打印页表

`x/2c $a1`	打印a1寄存器中指定内存地址的内容

`x/6i 0x30000` 从0x3000处打印六条指令

`csrrw a0, sscratch, a0`交换a0与sscratch内容



在xv6初始化时，会将sscatch指向trapframe。

trapframe会一直维护的

~~~c
  uint64 fn = TRAMPOLINE + (userret - trampoline);
  ((void (*)(uint64,uint64))fn)(TRAPFRAME, satp);
~~~

~~~c
globl userret
userret:
        # userret(TRAPFRAME, pagetable)
        # switch from kernel to user.
        # usertrapret() calls here.

        # a0: TRAPFRAME, in user page table. ******
        # a1: user page table, for satp.

        # switch to the user page table.
        csrw satp, a1
        sfence.vma zero, zero
~~~



为什么不使用user stack来保存。因为每一个高级语言的栈结构可能是不同的，甚至是没有的。



SSTATUS_SPIE与SIE不一样，SIE是立即打开中断，而SPIE是在sret时打开中断





Virtual Memory Benefits：

- Isolation

- Level of Indirection（VA -> PA）

  Features using page faluts

  - Lazy allocation
  
  - COW Fork
  
    ![image-20230726175946141](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230726175946141.png)
  
  - demand Paging
  
    在初始化时，将valid置于0
  
    ![image-20230726181859662](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230726181859662.png)
  
    第二种形式：在运行中动态调换页面。
  
    采用LRU-family算法来决定驱逐哪一个页面，可以借助A位，并定期清除它
  
  - Memory Map File（MMAP） 
  
    ![image-20230726183340322](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230726183340322.png)
  
    当多个进程映射同一个页面时，默认是没有同步机制的，除非上锁。
  
    映射可以是shared也可以是private，具体见linux手册
  
  - zero fill on demand
  
    - exec may execute faster
    - optimize memory for global array
  
    ![image-20230726174235272](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230726174235272.png)



Information needed:

- the faulting VA (staval register) （当前指令所访问的内存地址）

- the type of page fault(scause register)

- the VA of instruction that caused the fault（当前指令的地址）（sepc）

  

sbrk() -> Eager allocation : the kernel will immediately allocate the physical memory

sbrk() Lazy allocation: 

- p->size += n 
- when deference pointer that cause page fault 
- va < p->size
  - allocate page
  - zero page
  - map the page
  - restart the  instruction



RISC-V中的指令设计为简单的，是为了更好的DATAPATH，充分利用Memory bandwidth



不仅可以用RSW位记录是否为COW页，也可以在进程中维护这些信息，例如记录每一段的范围，只要写入该段时发生错误，那么一定是COW页



> 疑问：如何建立硬盘块与虚拟页面的之间的映射？
>
> 这对demand paging以及MMAP很关键

