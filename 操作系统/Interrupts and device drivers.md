# Interrupts and device drivers

[TOC]

A **driver** is the code in an operating system that manages a particular device: 

- it configures the device hardware
- tells the device to perform operations 
- handles the resulting interrupts
- interacts with processes that may be waiting for I/O from the device



Many device drivers execute code in two contexts: 

- a **top half** that runs in a process’s kernel thread
- a **bottom half** that executes at interrupt time. 



大致流程：进程内核线程通过read、write等系统调用执行驱动中的top half代码，该部分代码向I/O设备发送请求并等待操作的完成。一旦设备完成了操作，就会生成中断。内核代码识别是何种中断后，调用相应的bottom half代码（分发中断）。最后相应的进程被唤醒，继续执行top half代码。



## Console Input

在QEMU上，键盘以及屏幕都由串行端口UART来连接。UART通过memory-mapped control registers来暴露自己的接口（大部分设备也是这么做的）。每一个控制寄存器都是1 byte宽的。

在QEMU上，UART的寄存器从0x10000000开始。

- `LSR`：它的某些位指示是否有输入字符供软件读取

- `RHR`：存放可以读取的字符。设备内部维护着一个FIFO。如果FIFO为空，那么清除`LSR`中的ready位
- `THR`：软件向`THR`写一个字符，那么UART将传输该字符



初始化Console：配置完后，每当传输完或者接受一个字符时，UART都会生成中断信号。

~~~c
void consoleinit(void)
{
  initlock(&cons.lock, "cons");
  uartinit();
  // connect read and write system calls
  // to consoleread and consolewrite.
  devsw[CONSOLE].read = consoleread;			//注册read函数
  devsw[CONSOLE].write = consolewrite;			//注册write函数
}

//配置UART电路
void
uartinit(void)
{
  // disable interrupts.
  WriteReg(IER, 0x00);
  // special mode to set baud rate.
  WriteReg(LCR, LCR_BAUD_LATCH);
  // LSB for baud rate of 38.4K.
  WriteReg(0, 0x03);
  // MSB for baud rate of 38.4K.
  WriteReg(1, 0x00);
  // leave set-baud mode,
  // and set word length to 8 bits, no parity.
  WriteReg(LCR, LCR_EIGHT_BITS);
  // reset and enable FIFOs.
  WriteReg(FCR, FCR_FIFO_ENABLE | FCR_FIFO_CLEAR);
  // enable transmit and receive interrupts.
  WriteReg(IER, IER_TX_ENABLE | IER_RX_ENABLE);
  initlock(&uart_tx_lock, "uart");
}

#define ReadReg(reg) (*(Reg(reg)))
#define WriteReg(reg, v) (*(Reg(reg)) = (v))
#define Reg(reg) ((volatile unsigned char *)(UART0 + reg))
#define RHR 0                 // receive holding register (for input bytes)
#define THR 0                 // transmit holding register (for output bytes)
#define IER 1                 // interrupt enable register
#define IER_RX_ENABLE (1<<0)
#define IER_TX_ENABLE (1<<1)
~~~



~~~c
struct devsw devsw[NDEV];
struct devsw {
  int (*read)(int, uint64, int);
  int (*write)(int, uint64, int);
};
~~~



当用户代码调用`read(0, buf, n)`时，会执行内核中的`fileread`：

~~~c
// Read from file f.
// addr is a user virtual address.
int
fileread(struct file *f, uint64 addr, int n)
{
  int r = 0;

  if(f->readable == 0)
    return -1;

  if(f->type == FD_PIPE){
    r = piperead(f->pipe, addr, n);
  } else if(f->type == FD_DEVICE){									//如果文件描述符指向的是设备
    if(f->major < 0 || f->major >= NDEV || !devsw[f->major].read)	   //主设备号合法
      return -1;							
    r = devsw[f->major].read(1, addr, n);							 //调用相应设备驱动的top half代码
  } else if(f->type == FD_INODE){
    ilock(f->ip);
    if((r = readi(f->ip, 1, addr, f->off, n)) > 0)
      f->off += r;
    iunlock(f->ip);
  } else {
    panic("fileread");
  }
  return r;
}
~~~

在console中，对应的read top half代码为

~~~c
struct {
  struct spinlock lock;
  
  // input
#define INPUT_BUF_SIZE 128
  char buf[INPUT_BUF_SIZE];
  uint r;  // Read index
  uint w;  // Write index
  uint e;  // Edit index
} cons;


// user read()s from the console go here.
// copy (up to) a whole input line to dst.
// user_dist indicates whether dst is a user
// or kernel address.
//
int
consoleread(int user_dst, uint64 dst, int n)
{
  uint target;
  int c;
  char cbuf;

  target = n;
  acquire(&cons.lock);
  while(n > 0){									//如果期望读出的字符数未达标
    // wait until interrupt handler has put some
    // input into cons.buffer.
    while(cons.r == cons.w){					 //已经没有数据可以读出
      if(killed(myproc())){
        release(&cons.lock);
        return -1;
      }
      sleep(&cons.r, &cons.lock);			     //休眠
    }

    c = cons.buf[cons.r++ % INPUT_BUF_SIZE];	  //从读缓冲队列中取出一个字符

    if(c == C('D')){  // end-of-file			 // ctrl + D
      if(n < target){
        // Save ^D for next time, to make sure
        // caller gets a 0-byte result.
        cons.r--;
      }
      break;
    }

    // 将字符复制到用户的缓冲区中， 设备 -> 驱动的读缓冲队列 -> 用户的缓冲区
    cbuf = c;
    if(either_copyout(user_dst, dst, &cbuf, 1) == -1)
      break;

    dst++;
    --n;

    if(c == '\n'){
      // a whole line has arrived, return to
      // the user-level read().
      break;
    }
  }
  release(&cons.lock);
	
    
  // 返回实际读取的字符数
  return target - n;
}

~~~



当进程由于读缓冲队列中没有可读取的字符而休眠后，如果此时用户键入一个字符，那么UART就会生成一个中断信号，并开始执行trap handler代码。handler会调用`devintr`来处理设备中断

~~~c
void kerneltrap()
{
  //省略了其他代码
  int which_dev = 0;
  if((which_dev = devintr()) == 0){
    //错误处理代码
  }
}
void usertrap(void)
{
  //省略了其他代码
  if((which_dev = devintr()) != 0)
}
~~~



~~~c
// check if it's an external interrupt or software interrupt,
// and handle it.
// returns 2 if timer interrupt,
// 1 if other device,
// 0 if not recognized.
int
devintr()
{
  //省略了其他无关代码
  uint64 scause = r_scause();

  if((scause & 0x8000000000000000L) &&
     (scause & 0xff) == 9){
    // this is a supervisor external interrupt, via PLIC.
    // irq indicates which device interrupted.
    int irq = plic_claim();
    if(irq == UART0_IRQ){					//如果是来自UART设备的中断
      uartintr();
    }
  }

}
~~~



### bottom half：

~~~c
// handle a uart interrupt, raised because input has
// arrived, or the uart is ready for more output, or
// both. called from devintr().
void
uartintr(void)
{
  // read and process incoming characters. 这里是处理input
  while(1){
    int c = uartgetc();
    if(c == -1)
      break;
    consoleintr(c);					//处理从设备中读取出来的字符
  }

  // send buffered characters.     这里是处理output
  acquire(&uart_tx_lock);
  uartstart();
  release(&uart_tx_lock);
}

// read one input character from the UART.
// return -1 if none is waiting.
int
uartgetc(void)
{
  if(ReadReg(LSR) & 0x01){
    // input data is ready.
    return ReadReg(RHR);
  } else {
    return -1;
  }
}

void
consoleintr(int c)
{
  acquire(&cons.lock);

  switch(c){
  case C('P'):  // Print process list.
    procdump();
    break;
  case C('U'):  // Kill line.
    while(cons.e != cons.w &&
          cons.buf[(cons.e-1) % INPUT_BUF_SIZE] != '\n'){
      cons.e--;
      consputc(BACKSPACE);
    }
    break;
  case C('H'): // Backspace
  case '\x7f': // Delete key
    if(cons.e != cons.w){
      cons.e--;
      consputc(BACKSPACE);
    }
    break;
  default:
    if(c != 0 && cons.e-cons.r < INPUT_BUF_SIZE){
      c = (c == '\r') ? '\n' : c;

      // echo back to the user.
      consputc(c);

      // store for consumption by consoleread().
      cons.buf[cons.e++ % INPUT_BUF_SIZE] = c;
	
      //如果队列已经满了 或者读取到了\n，那么就唤醒相应的进程
      //如果读取到普通的字符，那么就仅仅存放到缓冲队列中，并不唤醒进程
      if(c == '\n' || c == C('D') || cons.e-cons.r == INPUT_BUF_SIZE){
        // wake up consoleread() if a whole line (or end-of-file)
        // has arrived.
        cons.w = cons.e;
        wakeup(&cons.r);
      }
    }
    break;
  }
  
  release(&cons.lock);
}
~~~



## Console Output

当用户进程调用`write(1, buf, n)`后，会执行内核中的`filewrite`

~~~c

// the transmit output buffer.
struct spinlock uart_tx_lock;
#define UART_TX_BUF_SIZE 32
char uart_tx_buf[UART_TX_BUF_SIZE];
uint64 uart_tx_w; // write next to uart_tx_buf[uart_tx_w % UART_TX_BUF_SIZE]
uint64 uart_tx_r; // read next from uart_tx_buf[uart_tx_r % UART_TX_BUF_SIZE]


// Write to file f.
// addr is a user virtual address.
int
filewrite(struct file *f, uint64 addr, int n)
{
    //省略了其他无关代码
  if(f->type == FD_DEVICE){
    if(f->major < 0 || f->major >= NDEV || !devsw[f->major].write)
      return -1;
    ret = devsw[f->major].write(1, addr, n);
  }
}


int
consolewrite(int user_src, uint64 src, int n)
{
  int i;

  for(i = 0; i < n; i++){
    char c;
    if(either_copyin(&c, user_src, src+i, 1) == -1)		//从用户缓冲区中读取一个字符
      break;
    uartputc(c);
  }

  return i;
}

void
uartputc(int c)
{
  acquire(&uart_tx_lock);

  if(panicked){
    for(;;)
      ;
  }
  while(uart_tx_w == uart_tx_r + UART_TX_BUF_SIZE){				//如果写缓冲区已经满了	
    // buffer is full.
    // wait for uartstart() to open up space in the buffer.
    sleep(&uart_tx_r, &uart_tx_lock);						    //休眠
  }
  uart_tx_buf[uart_tx_w % UART_TX_BUF_SIZE] = c;				//写入设备的写缓冲区
  uart_tx_w += 1;
  uartstart();
  release(&uart_tx_lock);
}

// 有两个地方调用该函数
// 一个是bottom half，也就是中断来的时候会调用
// 另一个是top half，当发送字符时也会调用该函数
void uartstart()
{
  while(1){
    if(uart_tx_w == uart_tx_r){
      // transmit buffer is empty.
      return;
    }
    if((ReadReg(LSR) & LSR_TX_IDLE) == 0){
      //如果UART还没有完成之前一个字符的传输，
      //那么就直接退出，等待下一次中断来处理即可
      //当前字符已经存入到写缓冲区中了
      return;
    }
    int c = uart_tx_buf[uart_tx_r % UART_TX_BUF_SIZE];
    uart_tx_r += 1;
    // 这里是在bottom half中唤醒在top half中由于写缓冲区已满的进程。
    wakeup(&uart_tx_r);
    WriteReg(THR, c);
  }
}
~~~





## Timer interrupts

RISC-V requires that timer interrupts be taken in machine mode, not supervisor mode.

在`start.c`中配置时钟电路

~~~c
// core local interruptor (CLINT), which contains the timer.
#define CLINT 0x2000000L
// 每一个CPU自启动后到当前时钟中断所经历的时钟数，用在trap handler中
#define CLINT_MTIMECMP(hartid) (CLINT + 0x4000 + 8*(hartid)) 
#define CLINT_MTIME (CLINT + 0xBFF8) // cycles since boot.

// a scratch area per CPU for machine-mode timer interrupts.
uint64 timer_scratch[NCPU][5];

void timerinit()
{
  // each CPU has a separate source of timer interrupts.
  int id = r_mhartid();

  // ask the CLINT for a timer interrupt.
  int interval = 1000000; // cycles; about 1/10th second in qemu.
  //CLINT_MTIME 获取自从启动到现在所经历的时钟周期数
  //要求PILC在 1/10th second后生成时钟中断
  *(uint64*)CLINT_MTIMECMP(id) = *(uint64*)CLINT_MTIME + interval;

  // prepare information in scratch[] for timervec.
  // scratch[0..2] : space for timervec to save registers.
  // scratch[3] : address of CLINT MTIMECMP register.
  // scratch[4] : desired interval (in cycles) between timer interrupts.
  // 类似trapframe，临时保存一些寄存器的值
  uint64 *scratch = &timer_scratch[id][0];
  scratch[3] = CLINT_MTIMECMP(id);
  scratch[4] = interval;
  w_mscratch((uint64)scratch);

  // set the machine-mode trap handler.
  w_mtvec((uint64)timervec);

  // enable machine-mode interrupts.
  w_mstatus(r_mstatus() | MSTATUS_MIE);

  // enable machine-mode timer interrupts.
  w_mie(r_mie() | MIE_MTIE);
}
~~~



**A timer interrupt can occur at any point when user or kernel code is executing**; there’s no way for the kernel to disable timer interrupts during critical operations. 



 Thus the timer interrupt handler must do its job in a way guaranteed **not to disturb interrupted kernel code**. The basic strategy is for the handler to ask the RISC-V to raise a “software interrupt” and immediately return. The RISC-V delivers software interrupts to the kernel with the ordinary trap mechanism, and allows the kernel to disable them

~~~asm
.globl timervec
.align 4
timervec:
        # start.c has set up the memory that mscratch points to:
        # scratch[0,8,16] : register save area.
        # scratch[24] : address of CLINT's MTIMECMP register.
        # scratch[32] : desired interval between interrupts.
        csrrw a0, mscratch, a0
        sd a1, 0(a0)
        sd a2, 8(a0)
        sd a3, 16(a0)

        # schedule the next timer interrupt
        # by adding interval to mtimecmp.
        ld a1, 24(a0) # CLINT_MTIMECMP(hart)
        ld a2, 32(a0) # interval
        ld a3, 0(a1)
        add a3, a3, a2
        sd a3, 0(a1)				

        # arrange for a supervisor software interrupt
        # after this handler returns.
        li a1, 2
        csrw sip, a1
		
	    # 恢复寄存器
        ld a3, 16(a0)
        ld a2, 8(a0)
        ld a1, 0(a0)
        csrrw a0, mscratch, a0

        mret
~~~



## Concurrency

通过 缓冲 + 中断 + sleep 来完成IO设备与进程之间的解耦。这样可以提升性能，尤其是在I/O设备很慢时，因为这使得I/O设备可以与进程并发执行。

基本上就是消费者-生成者模型，基本上是由bottom唤醒top



## Lab Net

- ~~~c
  struct mbuf {
    struct mbuf  *next; // the next mbuf in the chain
    char         *head; // the current start position of the buffer
    unsigned int len;   // the length of the buffer
    char         buf[MBUF_SIZE]; // the backing store
  };
  ~~~

  一开始我还以为这里next是长度大于MBUF_SUZE的数据部分

  后来看看tx_desc，发现mbuf的链表式存储不起作用。

  很好奇这个next字段到底有什么用？

- 在e1000_recv中，一次中断应该把所有的recv读出来，否则在multi-process的测试中过不去。因为e1000只会在这次测试中发送一次中断，这个在文档中有说明：

  > When the E1000 receives each packet from the ethernet, it DMAs the packet to the memory pointed to by `addr` in the next RX (receive) ring descriptor. If an E1000 interrupt is not already pending, the E1000 asks the PLIC to deliver one as soon as interrupts are enabled. Your `e1000_recv()` code must scan the RX ring and deliver each new packet's mbuf to the network stack (in `net.c`) by calling `net_rx()`
