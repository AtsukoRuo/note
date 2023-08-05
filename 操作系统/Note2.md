Interrupts 相比 System Call有以下不同点：

- Asynchronous
- Concurrency



Where do Interrupts come from？


![image-20230801083719548](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20230801083719548.png)

![image-20230801083847673](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20230801083847673.png)



OS can program PLIC，giving different interrupts priority

Route Interrupts

![image-20230801083900556](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20230801083900556.png)





Programming device

- memory mapped I/O
- ld/st read/write control register

device通过中断机制来与内核通信保持同步。比如内核发送一个字节后，在接受到device的中断信号之后，再发送一个字节



SIE is one bit for External Device、Software Interrupts、 Timer Interrupts

SSTATUS has one bit for enable/disable interrupt

SIP interrupt pending

SCAUSE

STVEC



配置PLIC

![image-20230801090802465](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230801090802465.png)



疑问？多核之间如何同步设备中断



when PLIC Route some interrupt to particular CPU Core. If SIE bit set：

- Clear SIE bit
- sepc <- PC
- save current mode
- mode <- supervisor
- pc <- stvec



UART connect keyboard and display on QEMU





- device and CPU run in paralled . producer/consumer parallelism. using buffer

- kernel itself is interrupted
- top of device + bottom device may run in parallel on different CPU. using lock can avoid this problem



