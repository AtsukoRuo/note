# Architecture RISC-V

[TOC]

## Basic Knowledge

The ***architecture* **is the programmer’s view of a computer. It is defined by the a **instruction set **(language) and **operand locations** (registers and memory)



Computer **instructions** indicate both the operation to perform and the operands to use. The operands may come from memory, registers, or the instruction itself. instructions are encoded as binary numbers in a format called **machine language.** represent the instructions in a symbolic format called **assembly language**.



A computer architecture does not define the underlying hardware implementation. Often, many different hardware implementations of a single architecture exist. The specific arrangement of registers, memories, arithmetic/logical units (ALUs), and other building blocks to form a microprocessor is called the **microarchitecture**.



RISC-V is a **reduced instruction set computer** (RISC) architecture. Architectures with many complex instructions, such as Intel’s x86 architecture, are **complex instruction set  computers** (CISC)



**the design principle of the RISC-V architecture **

- regularity supports simplicity
  - 每种类型的指令格式都类似，降低了硬件的复杂性。
- make the common case fast; 
  - 只设计简单而又常用的指令，更复杂的指令可以用多个简单指令来代替
  - 常数0硬编码到寄存器x0中
- smaller is faster; 
  - The fewer the registers, the faster they can be accessed
- good design demands good compromises.



~~~risc-v
add a, b, c
~~~

The first part of the assembly instruction, add, is called the **mnemonic（助记符）** and indicates what operation to perform. The operation is performed on b and c, the **source operands**, and the result is written to a, the **destination operand**.



In RISC-V assembly language, **only single-line comments are used**. They begin with a hash symbol (#) and continue until the end of the line. 



The RISC-V architecture has 32 registers, called the **register set**, stored in a small multiported memory called a **register file**. A register file is typically built from a small SRAM array 





RISC-V instructions can use constant or **immediate** operands. These constants are called *immediates* because their values are immediately available from the instruction and do not require a register or memory access. In assembly code, the immediate can be written in decimal, hexadecimal, or binary. Hexadecimal constants in RISC-V assembly language start with 0x and binary constants start with 0b.

**Immediates are 12-bit two’s complement numbers**, so they are sign-extended to 32 bits. 



对于CPU来说，它并不知道oprand是signed还是unsigned的。操作数的符号性是由汇编语言提供的特性，程序语义的正确性由程序员来保证，即源操作数的符号性必须满足指令的要求，否则结果的正确性得不到保证。



The RV32I RISC-V architecture uses 32-bit memory addresses and 32-bit data words（机器字长并不是存储字长）. RISC-V uses a *byte-addressable* memory.

![image-20230719095151333](assets\image-20230719095151333.png)



Assembly code uses **labels** to indicate instruction locations in the program. A label refers to the instruction just after the label. When the assembly code is translated into machine code, these labels correspond to instruction addresses **offset**

## assembly language

### Instructions

推荐直接查阅指令手册，这里仅仅笔记一些有代表性的指令。

#### Multiply Instructions

These instructions are not part of RV32I but are included in the RVM (RISC-V multiply/divide) extension.



The *multiply* instruction (mul) multiplies two 32-bit numbers and produces a 32-bit product. mul s1,s2,s3 multiplies the values in s2 and s3 and places the least significant 32 bits of the product in s1; the most significant 32 bits of the product are discarded.



The bottom 32 bits of the product are the same whether the operands are viewed as signed or unsigned.

Three versions of the “multiply high” operation exist: mulh, mulhsu, and mulhu. These instructions put the high 32 bits of the multiplication result in the destination register. 

- **mulh (*multiply high signed signed*) **treats both operands as signed. 

- **mulhsu (*multiply high signed unsigned*) **treats the first operand as signed and the second as unsigned

- **mulhu(*multiply high unsigned unsigned*) **treats both operands as unsigned. 

#### Branch

**Conditional branch** instructions perform a test and branch only if the test is TRUE. **Unconditional branch** instructions, called **jumps**, always branch.



- **beq (*branch if equal*)** branches when the values in the two source registers are equal. 

- **bne (*branch if not equal*) **branches when they are unequal.

- **blt (*branch if less than*) **branches when the value in the first source register is less than the value in the second

- **bge (*branch if* *greater than or equal to*)** branches when the first is greater than or equal to the second. 

blt and bge treat the operands as signed numbers, while bltu and bgeu treat the operands as unsigned.

注意：branch指令的立即数部分是13位偏移量（12位表示，bit0总是为0不表示），因此偏移范围为$[-4048, 4047]$B



- **jump and link (jal)**：`jal rd offset`：`[rd] = PC + 4` then `PC = PC + offset`

  `jal offset`is equivalent to `jal ra offset`，这里立即数为21位有符号数（但是用20位表示，bit0总是为0不表示），可跳转范围为$[-1, 1)$MB

- **jump register (jalr)**：`jalr rd rs offset` ：``[rd] = PC + 4` and `PC = [rs] + offset`。注意可以理解为两个子操作是同时发生的，这也就意味着rd与rs可以是同一个寄存器。立即数是12位符号数。



#### auipc

`auipc rd imm` ：`[rd] = PC + imm[31:12] << 12`

 For example, the instruction auipc s3,0xABCDE places PC + 0xABCDE000 in s3.

auipc一般与jalr指令搭配使用

#### Logical Instructions

RISC-V *logical operations* include and, or, and xor. These each operate bitwise on two source registers and write the result to a destination register,

![image-20230719095855608](assets\image-20230719095855608.png)

Immediate versions of these logical operations, andi, ori, and xori, use one source register and a 12-bit sign-extended immediate.

通过这些指令可以执行位运算，例如设置掩码、清零、标记位

#### Shift Instructions

*Shift instructions* shift the value in a register left or right, dropping bits off the end. RISC-V shift operations are **sll (shift left logical)**, **srl (shift right logical)**, and **sra (shift right arithmetic)**. However, right shifts can be either *logical* (zeros shift into the most significant bits) or *arithmetic* (the sign bit shifts into the most significant bits)

![image-20230719100155320](assets\image-20230719100155320.png)

Immediate versions of each instruction are also available (**slli, srli, and srai**), where the amount to shift is specified by a **5-bit unsigned immediate**.

shifting a value left by *N* is equivalent to multiplying it by 2*N*. Likewise, shifting a value right by *N* is equivalent to dividing it by 2*N*. Arithmetic right shifts divide two’s complement numbers, while logical right shifts divide unsigned numbers.



#### lw、sw

*load word* instruction, lw, reads a data word from memory into a register.

![image-20230719095417653](assets\image-20230719095417653.png)

The *store word* instruction, sw, writes a data word from a register into memory

![image-20230719095423666](assets\image-20230719095423666.png)

Many RISC-V implementations require **word-aligned* addresses**—that is, a word address that is divisible by four—for lw and sw. Some architectures, such as x86, allow non-word-aligned data reads and writes. **we will assume strict alignment in RISC-V**



The *load byte* (lb), *load byte unsigned* (lbu), and *store byte* (sb) instructions access individual bytes in memory. 

- **lb** sign-extends the byte, 

- **lbu** zero-extends the byte to fill the entire 32-bit register. 

- **sb** stores the least significant byte of the 32-bit register into the specified byte address in memory.



![image-20230719103710383](assets\image-20230719103710383.png)

#### lui

**load upper immediate instruction（lui）** loads a 20-bit immediate into the most significant 20 bits of the instruction and places zeros in the least significant bits.

To create larger constants, use a *load upper immediate* instruction (lui) followed by an add immediate instruction (addi)

When creating large immediates, if the 12-bit immediate in addiis negative (i.e., bit 11 is 1), the upper immediate in the lui must be incremented by one.Remember that addi *sign*-extends the 12-bit immediate, so a negative immediate will have all 1’s in its upper 20 bits.

![image-20230719095226487](assets\image-20230719095226487.png)

### Register Set

![image-20230719092626882](assets\image-20230719092626882.png)

寄存器`x0`硬编码为常数0，如何对它的写操作都会被忽略。



Specifically, the callee must leave the **saved registers (s0−s11)** 

RISC-V divides registers into **preserved** and **nonpreserved** categories. Preserved Register are ：`ra` `sp` `s0-s11`, and Nonpreserved Register are：`t0 - t6`、`a0 - a7`。其中并未对`gp`、`tp`做出规定

![image-20230719110329104](assets\image-20230719110329104.png)

> note : The convention of which registers are preserved or not preserved is part of the standard calling convention for the RISC-V Architecture（也就是Application的ABI规范）, instead of being part of the architecture itself.





The stack pointer, **sp (register 2)**, is an ordinary RISC-V register that, by convention, *points* to the **top of the stack**

![image-20230719104848717](assets\image-20230719104848717.png)



### Function Calls

 In RISC-V programs, the caller conventionally places up to **eight arguments in registers a0 to a7** and stores the **return address in the ra(return address register, x1)**  before making the function call.the callee places the **return value in register a0** before finishing. 



Preserved and Nonpreserved require that

- **Caller save rule**: Before a function call, the caller must save any nonpreserved registers (t0–t6 and a0–a7) that it needs after the call. After the call, it must restore these registers before using them.

- **Callee save rule**: Before a callee disturbs any of the preserved registers (s0–s11 and ra), it must save the registers. Before it returns, it must restore these registers.



函数调用过程通常分为 6 个阶段

- Caller将参数存储到函数能够访问到的位置
- 跳转到函数开始位置（使用 RV32I 的 jal 指令）
- Callee获取函数需要的局部存储资源，按需保存寄存器；
- 执行函数中的指令；
- 将返回值存储到调用者能够访问到的位置，恢复寄存器，释放局部存储资源
- 返回调用函数的位置（使用 ret 指令）

![image-20230719115308629](assets\image-20230719115308629.png)



### Conditional Statements

#### if-elseif-else

~~~
if (condition_result)
	statementA
statementB


branch to L1 when !condition_result
	statementA
L1:
	statementB
~~~



~~~
if (condition_resultA)
	statementA
else if (condition_resultB)
	statementB
else 
	statementC
statementD


branch to L1 when !condition_resultA
	statementA
	jump END
L1:
branch to L2 when !condition_resultB
	statementB
	jump END
L2:
	statementC
END:
	statementD
~~~



#### switch

实际上switch等价与if-elseif语句

~~~
switch(num) 
	case 1:
		statementA
		break;
	case 2:
		statementB
		breal;
	default :
		statementC


branch to L2 when num != 1
	statementA;
	jump END:
L2:
branch to L3 when num != 2
	statementB;
	jump END;
L3:
	statementC;
statementD;
~~~



#### for/while

~~~
while(condition) {
	statement;
}

while :
	branch to DONE when !condition
	statement
	jump while;
DONE:
~~~



~~~
for (init; condition; step) {
	statement
}



init
FOR:
	branch to DONE when !condition
	statement
	step
	jump FOR
DONE:
~~~



#### do-while

~~~
do {
	statement
} while (condition)

while:
	statement
	branch while when condition
~~~



### Pseudoinstructions

![image-20230719115654570](assets\image-20230719115654570.png)

这里的call伪指令描述有错，应该是

~~~
auipc ra, (offset[31:12] << 12 + offset[11])
jalr ra, ra, offset[11:0]
~~~

补充la伪指令：`la rd, symbol`

~~~risc-v
auipc rd, offset[31:12] + offset[11]
addi rd, rd delta[11:0]
~~~

### Assembler Directives & Memory Map

![image-20230719125120553](assets\image-20230719125120553.png)

-  **text segment** stores the machine language user program. In addition to code, it may include literals (constants) and read-only data.

- **global data segment** stores global variables that, in contrast to local variables, can be accessed by all functions in a program. they are typically accessed using the **global pointer register gp (register x3)** that points to the middle of the global data segment.

- **dynamic data segment** holds the stack and the heap. The data in this segment is not known at start-up but is dynamically allocated and deallocated throughout the execution of the program.

  The stack includes temporary storage and local variables. whereas data in heap can be used and discarded in any order. 

  If the stack and heap ever grow into each other, the program’s data can become corrupted. The memory allocator ensures that this never happens by returning an out-of-memory error if there is insufficient space to allocate more dynamic data.

- The lowest part of the example RISC-V memory map is reserved for the exception handlers and boot code that is run at start-up. The highest part of the memory map is reserved for the operating system and memory-mapped I/O



**Assembler directives** guide the assembler in allocating and initializing global variables, defining constants, and differentiating between code and data.



![image-20230719130118229](assets\image-20230719130118229.png)

![image-20230719130335185](assets\image-20230719130335185.png)





### Endianness

 In *big-endian* machines, bytes are numbered starting with 0 at the big (most significant) end. In *little-endian* machines, bytes are numbered starting with 0 at the little (least significant) end.



由于字节顺序仅在同时以按字访问和按字节访问同一份数据时才会有影响，字节序只会影响很少一部分的程序 员。

![image-20230719130429120](assets\image-20230719130429120.png)

RISC-V is typically little-endian

### Exceptions

Such a hardware exception triggered by an input/output (I/O) device such as a keyboard is often called an **interrupt**. Alternatively, the program may encounter an error condition caused by the software, such as an undefined instruction. Software exceptions are sometimes called **traps**. 

The three main RISC-V privilege levels are

- **user mode**
-  **supervisor mode**
- **machine mode**: is the highest privilege level; a program running in this mode can access all registers and memory locations. used in processors without an operating system (OS), including many embedded systems or in start-up time of OS



Exception handlers use four special-purpose registers, called **control and status registers (CSRs)**, to handle an exception: **mtvec**, **mcause**, **mepc**, and **mscratch**.

- mtvec，中断处理程序的地址
- mcause，中断的原因
- mepc，中断时，正在执行的指令的地址
- mscratch，用于帮助中断程序保存所有的寄存器



- When an exception occurs, the processor records the cause of an exception in mcause stores the **PC of the excepting instruction** in **mepc**, the machine exception PC register, 

- and **jumps to the exception handler at the address** preconfigured in **mtvec**.

- After jumping to the address in mtvec, the exception handler reads the**mcause** register to examine **what caused the exception** and responds appropriately

- Exception handlers must use program registers (x1−x31) to handle exceptions, so they use the memory pointed to by **mscratch** to store and restore these registers.
- It then either aborts the program or returns to the program by executing the mret, machine exception return instruction, that jumps to the address in mepc

![image-20230719131251348](assets\image-20230719131251348.png)

> https://five-embeddev.com/riscv-isa-manual/latest/supervisor.html#sec:scause

The mepc and mcause registers are not part of the RISC-V program registers (x1−x31), so the exception handler must move these special-purpose (CSR) registers into the program registers to read and operate on them. RISC-V uses three instructions to read, write, or both read and write CSRs: **csrr **(read CSR), **csrw** (write CSR), and **csrrw**(read/write CSR). 

 `csrr t1, mcause` reads the value in **mcause into t1**; 

`csrw mepc, t2` writes the value in **t2 into mepc**; 

`csrrw t1, mscratch,t0` simultaneously reads the value in **mscratch into t1** and writes the value in **t0 into mscratch**.



### Floating-Point Instructions

略

### Compressed Instructions

略




## Machine Language
### R-Type

![image-20230719121446065](assets\image-20230719121446065.png)

![image-20230719121456627](assets\image-20230719121456627.png)

![image-20230719121534545](assets\image-20230719121534545.png)

R-type operation is determined by the**opcode（op）** and the **function fields（funct3、funct7）**. These bits together are called the *control bits* because they control what operation to perform. 

### I-Type

![image-20230719121551787](assets\image-20230719121551787.png)

![image-20230719121600646](assets\image-20230719121600646.png)

The immediate field represents a 12-bit signed (two’s complement) number for all I-type instructions except immediate shift instructions (slli, srli, and srai) .For these shift instructions, **imm4:0 is the 5-bit unsigned shift amount**; **the upper seven imm bits are 0 for srli and slli, but srai puts a 1 in imm10 (i.e., instruction bit 30)**

注意**`jalr`是I-Type指令**

### S/B-Type

![image-20230719121633224](assets\image-20230719121633224.png)

![image-20230719121644488](assets\image-20230719121644488.png)



![image-20230719123805115](assets\image-20230719123805115.png)

B-type instructions encode a 13-bit signed immediate representing the *branch offset*, but only 12 of the bits are encoded in the instruction. The least significant bit is always 0, because **branch amounts are always an even number of bytes**, 因为指令长度要么是32位的，要么是16位的



下面给出一个例子。实际上，我们很少直接使用立即数，而是使用标签，由汇编器计算相应的偏移量。

![image-20230719123119283](assets\image-20230719123119283.png)

注意：branch指令的立即数部分是13位偏移量（12位表示，bit0总是为0不表示），因此偏移范围为$[-4048, 4047]$Byte

### U/J-Type

![image-20230719121746394](assets\image-20230719121746394.png)

![image-20230719121754544](assets\image-20230719121754544.png)





![image-20230719124412784](assets\image-20230719124412784.png)

In U-type instructions, the remaining bits specify the most significant 20 bits of a 32-bit immediate. **In J-type instructions, the remaining 20 bits specify the most significant 20 bits of a 21-bit immediate jump offset. As with B-type instructions, the least significant bit of the immediate is always 0** and is not encoded in the J-type instruction.

可跳转范围为$[-1, 1)$MB

### Immediate Encodings

![image-20230719121956786](assets\image-20230719121956786.png)

### Address Modes

- Register-Only Addressing
- Immediate Addressing：**slli, srli, and srai**
- Base Addressing：lw、bw
- PC-Relative Addressing：jal

### Stored Program

Like other binary numbers, these instructions can be stored in memory. This is called the **stored program** concept, and it is a key reason why computers are so powerful. **Running a different program does not require large amounts of time and effort to reconfigure or rewire hardware; it only requires writing a new program to memory**. 




## Evolution of the RISC-V architecture

RISC-V has 32-, 64-, and 128-bit base instruction sets: RV32I/E, RV64I, and RV128I. RV32E is embedded version with only 16 registers, intended for very low-cost processors.

All RISC-V processors must support one of the base architectures—RV32/64/128I or RV32E—and may optionally support extensions, such as the compressed or floating-point extensions.

![image-20230719090716467](assets\image-20230719090716467.png)

