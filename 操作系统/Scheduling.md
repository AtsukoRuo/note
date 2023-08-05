# Scheduling

[TOC]

Any operating system runs with more processes than the computer has CPUs. A common approach is to provide each process with the illusion that it has its own virtual CPU by **multiplexing**（**time-share**） the processes onto the hardware CPUs. 





switching each CPU from one process to another in two situations：

- **sleep and wakeup** mechanism switches,  waiting for device or pipe I/O to complete
- **time interrupt**（is sort of preemptive scheduling）

普通的设备中断以及大部分系统调用（不会主动放弃CPU的）都不会将线程切换到其他CPU上



xv6不支持内核级线程，也就 是说它的调度基本单位是进程（一个user thread，一个kernel thread，不过它们的切换是通过trapframe来完成的）。





## Context Switching



![image-20230802100951990](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230802100951990.png)

**The xv6 scheduler has a dedicated scheduler thread (saved registers and stack) per CPU **.  it is not safe for the scheduler to execute on the old process’s kernel stack. 因为在执行scheduler的时候，可能其他Core唤醒并执行old process，那么有两个Core同时对一个stack进行操作，这是绝对的灾难。

~~~c
// Saved registers for kernel context switches.
struct context {
  uint64 ra;		//ra记录了我们是从哪里来的，PC无需记录，因为它不能提供给我们这条信息
  uint64 sp;

  // callee-saved
  uint64 s0;
  uint64 s1;
  uint64 s2;
  uint64 s3;
  uint64 s4;
  uint64 s5;
  uint64 s6;
  uint64 s7;
  uint64 s8;
  uint64 s9;
  uint64 s10;
  uint64 s11;
};

// Per-CPU state.
struct cpu {
  struct proc *proc;          // The process running on this cpu, or null.
  struct context context;     // swtch() here to enter scheduler().
  int noff;                   // Depth of push_off() nesting.
  int intena;                 // Were interrupts enabled before push_off()?
};
~~~



`swtch (kernel/swtch.S:3) `**saves only callee-saved registers**; the C compiler generates code in the caller to save caller-saved registers on the stack



不同于中断需要保存所有的寄存器，这种类似通过函数调用主动放弃CPU的执行流可以只保存`callee-saved register`

~~~asm
.globl swtch
swtch:
        sd ra, 0(a0)				#切换返回地址
        sd sp, 8(a0)				#切换栈
        sd s0, 16(a0)
        sd s1, 24(a0)
        sd s2, 32(a0)
        sd s3, 40(a0)
        sd s4, 48(a0)
        sd s5, 56(a0)
        sd s6, 64(a0)
        sd s7, 72(a0)
        sd s8, 80(a0)
        sd s9, 88(a0)
        sd s10, 96(a0)
        sd s11, 104(a0)

        ld ra, 0(a1)
        ld sp, 8(a1)
        ld s0, 16(a1)
        ld s1, 24(a1)
        ld s2, 32(a1)
        ld s3, 40(a1)
        ld s4, 48(a1)
        ld s5, 56(a1)
        ld s6, 64(a1)
        ld s7, 72(a1)
        ld s8, 80(a1)
        ld s9, 88(a1)
        ld s10, 96(a1)
        ld s11, 104(a1)
        
        ret
~~~



## Scheduling

![1690981848854-screenshot](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\1690981848854-screenshot.png)

无论是time interrupt（yield）还是sleep（sleep、wait、pipe...），它们都执行以下序列

~~~c
void
yield(void)
{
  struct proc *p = myproc();
  //获取锁，确保进程状态的变更是原子的
  acquire(&p->lock);		
  //release other lock if has
  //因为假设P1持有该锁L，然后它调用sched，CPU调度到P2，而P2想要获取锁L，但是P1此时还未释放，于是P2一直空悬等待。这里有一个很重要的点，就是xv6在持有任何一把锁时会关闭中断，因此P2不可能被抢占调度，一直死循环。也就是说CPU死锁了
    
  p->state = RUNNABLE;			     //更改状态
  sched();						   //开始调度
  release(&p->lock);
}
~~~



~~~c
// Switch to scheduler.  Must hold only p->lock
// and have changed proc->state. Saves and restores
// intena because intena is a property of this
// kernel thread, not this CPU. It should
// be proc->intena and proc->noff, but that would
// break in the few places where a lock is held but
// there's no process.
void
sched(void)
{
  int intena;
  struct proc *p = myproc();

  if(!holding(&p->lock))
    panic("sched p->lock");
  if(mycpu()->noff != 1)
    panic("sched locks");
  if(p->state == RUNNING)
    panic("sched running");
  if(intr_get())
    panic("sched interruptible");

  intena = mycpu()->intena;
  swtch(&p->context, &mycpu()->context);   
   //这里p->context就是kernel thread中sched当前这条调用swtch所处的状态
  //这个返回地址是什么？就是下一条指令地址 因为编译器为我们自动管理ra寄存器
  mycpu()->intena = intena;
}
~~~



~~~c
void
scheduler(void)
{
  //从sched中swtch到scheduler时，并不是从这里开始执行
  struct proc *p;
  struct cpu *c = mycpu();
  
  c->proc = 0;
  for(;;){
    // Avoid deadlock by ensuring that devices can interrupt.
    intr_on();

    for(p = proc; p < &proc[NPROC]; p++) {
       //不会获取两次锁
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // Switch to chosen process.  It is the process's job
        // to release its lock and then reacquire it
        // before jumping back to us.
        p->state = RUNNING;
        c->proc = p;
        swtch(&c->context, &p->context);
		
        //而是从这里开始执行，因为swtch是一个函数，因此编译器处理函数调用代码时会自动为我们管理ra寄存器，返回时直接执行这里swtch的下一条指令 c->proc = 0
        // Process is done running for now.
        // It should have changed its p->state before coming back.
        c->proc = 0;
      }
      //释放锁，因为此时p->lock是当前CPU唯一持有的锁，所以释放后会开启中断。
      release(&p->lock);
    }
  }
}

~~~

这里注意到yield中acquire，在另一个kernel thread（scheduler）中release。而不是在同一个thread中release。 usually the thread that acquires a lock is also responsible for releasing the lock, which makes it easier to reason about correctness. For context switching it is necessary to brea k this convention

 if p->lock were not held during swtch: a different CPU might decide to run the process after yield had set its state to RUNNABLE, and before swtch The result would be two CPUs running on the same stack, which would cause chaos.



而且还要注意到scheduler以及sched本质上是一个协程！！!

 Procedures that intentionally transfer control to each other via thread switch are sometimes referred to as **coroutines**



这里我们可以看到，在多核体系下，进程如何切换到不同的CPU上，每个CPU都有一个scheduler thread，它们维护一个存放进程信息的数据结构（链表、队列）

## mycpu & myproc

~~~c
// Per-CPU state.
struct cpu {
  struct proc *proc;          // The process running on this cpu, or null.
  //为了适用于multi-core场景，每个CPU管理各自的proc
  //如果是uni-core，可以使用一个全局变量来管理
  struct context context;     // swtch() here to enter scheduler().
  int noff;                   // Depth of push_off() nesting.
  int intena;                 // Were interrupts enabled before push_off()?
};
struct cpu cpus[NCPU];
~~~



~~~c
// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // wait_lock must be held when using this:
  struct proc *parent;         // Parent process

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};
struct proc proc[NPROC];
~~~



~~~c
// Must be called with interrupts disabled,
// to prevent race with process being moved
// to a different CPU.
// 注意要禁用中断，因为时钟中断可能将proc切换到别的CPU上执行，从而返回值无效
// 当不再用到返回值时，才可以释放锁
// 而且要确保在可能切换到别的CPU时（调用yield、sleep等），要重新调用该函数获取最新的CPUid
int
cpuid()
{
  int id = r_tp();
  return id;
}
~~~



~~~c
// Return this CPU's cpu struct.
// Interrupts must be disabled.
struct cpu*
mycpu(void)
{
  int id = cpuid();
  struct cpu *c = &cpus[id];
  return c;
}
~~~



~~~c
// Return the current struct proc *, or zero if none.
struct proc*
myproc(void)
{
  push_off();				//关中断
  struct cpu *c = mycpu();
  struct proc *p = c->proc;
  pop_off();				//可能开中断
  return p;
}
~~~

 RISC-V numbers its CPUs, giving each a **hartid**. Xv6 ensures that each CPU’s hartid is stored in that CPU’s `tp` register while in the kernel. usertrapret saves tp in the trampoline page, because the user process might modify tp. Finally, uservec restores that saved tp when entering the kernel from user space. The compiler guarantees never to use the tp register in kernel code.

注意只能在machine mode下获取CPU的hartid



  ## PV

semaphore是更高层次的抽象，用于更加特定的场景。

## Pipe 

## Sleep & Wakeup

~~~c
void
sleep(void *chan, struct spinlock *lk)
{
  struct proc *p = myproc();
  
  // Must acquire p->lock in order to change p->state and then call sched. Once we hold p->lock, we can be guaranteed that we won't miss any wakeup (wakeup locks p->lock), so it's okay to release lk.

  acquire(&p->lock);  			//这个锁的作用是保证在更改状态时不会丢失wakeup信号
  release(lk);

  // Go to sleep.
  p->chan = chan;				//注册到特定的channel中
  p->state = SLEEPING;

  sched();						//主动放弃CPU，在这里释放p->lock

  // Tidy up.
  p->chan = 0;

  release(&p->lock);			//这里释放的是scheduler中获取的锁
    
  //重新获取锁
  acquire(lk);
}
~~~



~~~c
// Wake up all processes sleeping on chan.
// Must be called without any p->lock.
//wakeup的工作原理很简单，就是根据chan匹配到要唤醒的thread, 将它的状态从SLEEPING修改到RUNNABLE即可

void
wakeup(void *chan)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    if(p != myproc()){
      acquire(&p->lock);
      if(p->state == SLEEPING && p->chan == chan) {
        p->state = RUNNABLE;
      }
      release(&p->lock);
    }
  }
}
~~~





为什么sleep要接受一个 struct spinlock *lk？ 我们来看一个例子：

~~~diff
void broken_sleep(void *chan)
{
  struct proc *p = myproc();
  acquire(&p->lock);
-  release(lk);
  p->chan = chan;
  p->state = SLEEPING;
  sched();
  p->chan = 0;
  release(&p->lock);
-  acquire(lk);
}
~~~

~~~c
void uartwrite(char buf[], int n)
{
    //条件变量确保在uartwrite之前不会丢失wakeup信号，
    //uart_tx_lock确保在uartwrite中不会丢失wakeup信号
    acquire(&uart_tx_lock);
    int i = 0;
    while (i < n) {
        while (tx_done == 0) {
            //sleep(&tx_chan, &uart_tx_lock); 这是正确的代码
            release(&uart_tx_lock);
            //RIGHT HERE --INTERRUPT
            broken_sleep(&tx_chan);
            acquire(&uart_tx_lock);
        }
    }
}

void uartinit(void)
{
    acquire(&uart_tx_lock);
    if (ReadReg(LSR) & LSR_TX_IDLE) {
        wakeup(&tx_chan);	//如果uartwrite在那里中断，由于释放了锁，uartinit可以在此时调用wakeup，但uartwrite尚未sleep，故这次wakeup将会丢失，可能造成死锁直至下一次wakeup
    }
}
~~~

> 注：此段代码在2022版中被移除了

## Wait、Exit

~~~c
// Wait for a child process to exit and return its pid.
// Return -1 if this process has no children.
int
wait(uint64 addr)
{
  struct proc *pp;
  int havekids, pid;
  struct proc *p = myproc();

  //这把锁的作用就是确保不会丢失信号
  acquire(&wait_lock);

  for(;;){
    // Scan through table looking for exited children.
    havekids = 0;
    for(pp = proc; pp < &proc[NPROC]; pp++){
      if(pp->parent == p){
        // make sure the child isn't still in exit() or swtch().
        acquire(&pp->lock);

        havekids = 1;
        if(pp->state == ZOMBIE){
          // Found one.
          pid = pp->pid;
          if(addr != 0 && copyout(p->pagetable, addr, (char *)&pp->xstate,
                                  sizeof(pp->xstate)) < 0) {
            release(&pp->lock);
            release(&wait_lock);
            return -1;
          }
          freeproc(pp);
          release(&pp->lock);
          release(&wait_lock);
          
          return pid;
        }
        release(&pp->lock);
      }
    }

    // No point waiting if we don't have any children.
    if(!havekids || killed(p)){
      release(&wait_lock);
      return -1;
    }
    
    // Wait for a child to exit.
    sleep(p, &wait_lock);  //DOC: wait-sleep        
  }
}
~~~



 ~~~c
 // Exit the current process.  Does not return.
 // An exited process remains in the zombie state
 // until its parent calls wait().
 void
 exit(int status)
 {
   struct proc *p = myproc();
 
   //省略了释放资源以及检查的代码
 
   acquire(&wait_lock);
 
   //当进程退出时，将它所有的children转交给init进程
   reparent(p);
 
   // Parent might be sleeping in wait().
   wakeup(p->parent);
   
   acquire(&p->lock);
 
   p->xstate = status;
   p->state = ZOMBIE;
 
   release(&wait_lock);
 
   // Jump into the scheduler, never to return.
   sched();    //在这里释放锁 p->lock
   panic("zombie exit");
 }
 
 void
 reparent(struct proc *p)
 {
   struct proc *pp;
 
   for(pp = proc; pp < &proc[NPROC]; pp++){
     if(pp->parent == p){
       pp->parent = initproc;
       wakeup(initproc);
     }
   }
 }
 
 //init基本上就是调用wait，处理ZOMBIE状态的子进程
 int
 main(void)
 {
     //只保留了关键代码
     for(;;){
         // this call to wait() returns if the shell exits,
         // or if a parentless process exits.
         wpid = wait((int *) 0);						
         if(wpid == pid){
             // the shell exited; restart it.
             break;
         } else if(wpid < 0){
             printf("init: wait returned an error\n");
             exit(1);
         } else {
             // it was a parentless process; do nothing.
         }
     }
 }
 
 ~~~





## Kill

~~~c
// Kill the process with the given pid.
// The victim won't exit until it tries to return
// to user space (see usertrap() in trap.c).
int
kill(int pid)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++){
    acquire(&p->lock);
    if(p->pid == pid){
      p->killed = 1;
      if(p->state == SLEEPING){
        // Wake process from sleep().
        p->state = RUNNABLE;
      }
      release(&p->lock);
      return 0;
    }
    release(&p->lock);
  }
  return -1;
}
~~~

为什么不直接杀死进程并回收资源呢？因为此时进程可能在执行多阶段的原子性操作，必须等待执行完毕（例如文件创建），但由于操作一般不能撤销，所以一般的实现就是标记为killed，在返回用户态的代码中，或者其他内核代码中，检查进程标志来判断是否要调用exit将其杀死。



~~~c
//in readpipe.c
while(pi->nread == pi->nwrite && pi->writeopen){  //DOC: pipe-empty
    if(killed(pr)){
          release(&pi->lock);
          return -1;
    }
    //在文件创建代码中，不应该判断是否执行kill，而是直接忽视kill状态，并等待文件创建操作执行完毕
    
    sleep(&pi->nread, &pi->lock); //DOC: piperead-sleep
}


//in trap.c
if(killed(p))
    exit(-1);

~~~

> 在linux中，kill还有权限检查
