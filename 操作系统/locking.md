[TOC]



并发执行从何而来？

- 每个核都独立执行指令流
- 在某个核中，切换到不同线程中
- 中断设备驱动代码与CPU代码（操作系统代码与用户代码）之间的切换



The word **concurrency** refers to situations in which multiple instruction streams are interleaved, due to multiprocessor parallelism, thread switching,or interrupts.



Strategies aimed at correctness under concurrency, and abstractions that support them, are called **concurrency control** techniques.



**竞争（race conditions）**可能会出现在**`RAW`（写后读）**、**`WAW`（写后写）**、**`WAR`（读后写）**操作序列中，而造成结果不正确。其根本原因在于破坏了**操作的不变式（Invariants）**





## 锁🔒

**锁（lock）**提供了**互斥（mutual exclusion）**语义，保证了在某一时刻只有一个CPU持有该🔒。



锁对性能的影响：

Although **locks are an easy-to-understand concurrency control mechanism** and is good for correctness, the downside of **locks is that they can limit performance, because they serialize concurrent operations.** 







We say that multiple processes **conflict** if they want the same lock at the same time, or that the lock experiences **contention**. 

 

The sequence of instructions between acquire and release on same lock is often called a **critical section（临界区）**.

~~~c
acquire(&listlock);
l->next = list;
list = l;
release(&listlock);
~~~

![image-20230801153602390](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230801153602390.png)



锁有三个好处：

- atomic
- avoid lost update
- maintain invariant

下面分别阐述这些好处：

When we say that a lock protects data, we really mean that the **lock protects some collection of invariants that apply to the data**. **Invariants are properties of data structures that are maintained across operations**. Typically, an operation’s correct behavior depends on the invariants being true when the operation begins. The operation may temporarily violate the invariants but must reestablish them before finishing. 

锁**串行化（serialize）**了临界区，因此维护了操作的不变式。所以可以将临界区中所有的操作视为**原子的（atomic）**，也就是说，只能观察到操作的完整更改，而不会出现部分更改。



### Deadlock

Four conditions need to hold for a deadlock to occur [C+71]:

• **Mutual exclusion:** Threads claim exclusive control of resources that they require (e.g., a thread grabs a lock).

• **Hold-and-wait:** Threads hold resources allocated to them (e.g., locks that they have already acquired) while waiting for additional resources (e.g., locks that they wish to acquire).

• **No preemption:** Resources (e.g., locks) cannot be forcibly removed from threads that are holding them.

• **Circular wait:** There exists a circular chain of threads such that each thread holds one or more resources (e.g., locks) that are being requested by the next thread in the chain.



![image-20230802151306904](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230802151306904.png)

solution

- order locks globally



### Implement

Hardware Support

![image-20230801174249824](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230801174249824.png)

### Spin-lock

自旋锁在多核等待时很有用，但是在单核切换时，会浪费CPU执行时间。

~~~c
// Mutual exclusion lock.
struct spinlock {
  uint locked;       // Is the lock held?

  // For debugging:
  char *name;        // Name of lock.
  struct cpu *cpu;   // The cpu holding the lock.
};


void
initlock(struct spinlock *lk, char *name)
{
  lk->name = name;
  lk->locked = 0;
  lk->cpu = 0;
}

// Acquire the lock.
// Loops (spins) until the lock is acquired.
void
acquire(struct spinlock *lk)
{
  push_off(); // disable interrupts to avoid deadlock. this will discuss on interrupt & locks
  if(holding(lk))
    panic("acquire");

  // On RISC-V, sync_lock_test_and_set turns into an atomic swap:
  // the sync_lock_test_and_set return old value of first parameter
  //   a5 = 1
  //   s1 = &lk->locked
  //   amoswap.w.aq a5, a5, (s1)
  while(__sync_lock_test_and_set(&lk->locked, 1) != 0)
    ;

  // Tell the C compiler and the processor to not move loads or stores
  // past this point, to ensure that the critical section's memory
  // references happen strictly after the lock is acquired.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize();

  // Record info about lock acquisition for holding() and debugging.
  lk->cpu = mycpu();
}

// Release the lock.
void
release(struct spinlock *lk)
{
  if(!holding(lk))
    panic("release");

  lk->cpu = 0;

  // Tell the C compiler and the CPU to not move loads or stores
  // past this point, to ensure that all the stores in the critical
  // section are visible to other CPUs before the lock is released,
  // and that loads in the critical section occur strictly before
  // the lock is released.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize();

  // Release the lock, equivalent to lk->locked = 0.
  // This code doesn't use a C assignment, since the C standard
  // implies that an assignment might be implemented with
  // multiple store instructions.
  // On RISC-V, sync_lock_release turns into an atomic swap:
  //   s1 = &lk->locked
  //   amoswap.w zero, zero, (s1)
  __sync_lock_release(&lk->locked);

  pop_off();
}

// Check whether this cpu is holding the lock.
// Interrupts must be off.
int
holding(struct spinlock *lk)
{
  int r;
  r = (lk->locked && lk->cpu == mycpu());
  return r;
}
~~~



~~~c
// push_off/pop_off are like intr_off()/intr_on() except that they are matched:
// it takes two pop_off()s to undo two push_off()s.  Also, if interrupts
// are initially off, then push_off, pop_off leaves them off.

void
push_off(void)
{
  int old = intr_get();

  intr_off();
  if(mycpu()->noff == 0)
    mycpu()->intena = old;
  mycpu()->noff += 1;
}

void
pop_off(void)
{
  struct cpu *c = mycpu();
  if(intr_get())
    panic("pop_off - interruptible");
  if(c->noff < 1)
    panic("pop_off");
  c->noff -= 1;
  if(c->noff == 0 && c->intena)
    intr_on();
}

~~~



~~~c
// are device interrupts enabled?
static inline int
intr_get()
{
  uint64 x = r_sstatus();
  return (x & SSTATUS_SIE) != 0;
}
~~~



~~~c
// Per-CPU state.
struct cpu {
  struct proc *proc;          // The process running on this cpu, or null.
  struct context context;     // swtch() here to enter scheduler().
  int noff;                   // Depth of push_off() nesting.
  int intena;                 // Were interrupts enabled before push_off()?
};
~~~



### Sleep-lock



### re-entrance lock

It might appear that some deadlocks and lock-ordering challenges could be avoided by using **re-entrant locks**, which are also called **recursive locks**.



The idea is that if the lock is held by a process and if that process attempts to acquire the lock again, then the kernel could just allow this (since the process already has the lock), instead of calling panic,



但是这种重入锁会有一些问题：

~~~c
struct spinlock lock;
int data = 0; // protected by lock
f() {
    acquire(&lock);
    if(data == 0){
        call_once();
        g();
        data = 1;
    }
    release(&lock);
}

g() {
    aquire(&lock);
    if(data == 0){
        call_once();
        data = 1;
    }
    release(&lock);
}

~~~

But if re-entrant locks are allowed, and h happens to call g, call_once will be called *twice*.

If re-entrant locks aren’t allowed, then h calling g results in a deadlock, which is not great either. 在这种情景下，推荐使用这种方案，因为方便debug

### Interrupt

Some xv6 spinlocks protect data that is used by both threads and interrupt handlers. Suppose sys_sleep holds tickslock, and its CPU is interrupted by a timer interrupt. clockintr would try to acquire tickslock, see it was held, and wait for it to be released. **clockintr isn't scheduled**, so it never return. but sys_sleep will not continue running until clockintr returns. So the CPU will deadlock,



To avoid this situation, if **a** spinlock is used by an interrupt handler, a CPU must never hold that lock with interrupts enabled.

 Xv6 is more conservative: when a CPU acquires **any** lock, xv6 always disables interrupts on that CPU. Xv6 re-enables interrupts when a CPU holds no spinlock



~~~c
void
acquire(struct spinlock *lk)
{
  push_off();
  if(holding(lk))				//如果已经持有该锁，那么就panic，这说明xv6不支持重入锁
    panic("acquire");
  while(__sync_lock_test_and_set(&lk->locked, 1) != 0)
    ;
  __sync_synchronize();
  lk->cpu = mycpu();
}

void
push_off(void)
{
  int old = intr_get();			
  //1中断开启，0中断关闭
  //如果是1，此时被中断打断，那么返回时，中断还是开启的，语义保持
  //如果是0，那么就不会有中断，语义保持
    
  intr_off();
  if(mycpu()->noff == 0)			//如果这是CPU第一个所持有的锁
    mycpu()->intena = old;			//记录调用push_off前中断状态
  mycpu()->noff += 1;			    
}

static inline int
intr_get()
{
  uint64 x = r_sstatus();
  return (x & SSTATUS_SIE) != 0;
}


struct cpu {
  struct proc *proc;          
  struct context context;     
  int noff;                   // Depth of push_off() nesting.
  int intena;                 // Were interrupts enabled before push_off()?
};
~~~



~~~c
void
release(struct spinlock *lk)
{
  if(!holding(lk))
    panic("release");
  lk->cpu = 0;
  __sync_synchronize();
  __sync_lock_release(&lk->locked);
  pop_off();
}

void
pop_off(void)
{
  struct cpu *c = mycpu();
  if(intr_get())
    panic("pop_off - interruptible");
  if(c->noff < 1)
    panic("pop_off");
  c->noff -= 1;
  if(c->noff == 0 && c->intena)
    intr_on();
}
~~~

It is important that acquire call push_off strictly before setting lk->locked.  If the two were reversed, there would be a brief window when the lock was held with interrupts enabled, and an unfortunately timed interrupt would deadlock the system

Similarly, it is important that release call pop_off only after releasing the lock

## Real World

A major challenge in kernel design is avoidance of lock contention in pursuit of parallelism. In the list example, a kernel may maintain a separate free list per CPU and only touch another CPU’s free list if the current CPU’s list is empty and it must steal memory from another CPU.



## Lab Thread

线程是调度的基本单位，而进程是隔离、资源分配的基本单位

为什么线程开销比进程开销小？

- 通信代价少，简化了编程模型的复杂度
- 占用资源少，多个线程共享同一份进程资源
- 但是调度所需的时间与进程相比，是相差不多的



协程本质上是一个**用户态线程**，采用**协作式多任务**的模型。而我们平常所说的线程是**内核态线程**，采用抢占式任务模型。内核态线程的开销主要在于复杂的调度算法实现、内核态检查（例如判断是何种中断）以及保存更多的寄存器。



### pthread API

- condition variable

  - https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_broadcast.html
  - https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_wait.html

  条件变量应用在生成者消费者模型中：

  ~~~c
  //init
  pthread_mutex_t mutex;
  pthread_cond_t cond;
  int condition;
  
  pthread_mutex_init(&mutex, NULL)
  pthread_cond_init(&cond, NULL)
  ~~~

  ~~~c
  pthread_mutex_lock(&bstate.barrier_mutex);
  while (!condition) {
      pthread_cond_wait(&cond, &mutex);
  }
  pthread_mutex_unlock(&mutex);
  pthread_cond_broadcast(&cond);
  ~~~

  

- pthread
  - https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_create.html
- mutex
  - https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_init.html
  - https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_lock.html

### 协程的简单实现

~~~c
/* Possible states of a thread: */
#define FREE        0x0
#define RUNNING     0x1
#define RUNNABLE    0x2

#define STACK_SIZE  8192
#define MAX_THREAD  4

struct thread {
  uint64 fp;
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
  uint64 ra;
  uint64 sp;
  char       stack[STACK_SIZE]; /* the thread's stack */
  int        state;             /* FREE, RUNNING, RUNNABLE */
};
struct thread all_thread[MAX_THREAD];
struct thread *current_thread;
extern void thread_switch(uint64, uint64);
              
void 
thread_init(void)
{
  // main() is thread 0, which will make the first invocation to
  // thread_schedule().  it needs a stack so that the first thread_switch() can
  // save thread 0's state.  thread_schedule() won't run the main thread ever
  // again, because its state is set to RUNNING, and thread_schedule() selects
  // a RUNNABLE thread.
  current_thread = &all_thread[0];
  current_thread->state = RUNNING;
}

//非常简单的调度算法
void 
thread_schedule(void)
{
  struct thread *t, *next_thread;

  /* Find another runnable thread. */
  next_thread = 0;
  t = current_thread + 1;
  for(int i = 0; i < MAX_THREAD; i++){
    if(t >= all_thread + MAX_THREAD)
      t = all_thread;
    if(t->state == RUNNABLE) {
      next_thread = t;
      break;
    }
    t = t + 1;
  }

  if (next_thread == 0) {
    printf("thread_schedule: no runnable threads\n");
    exit(-1);
  }

  if (current_thread != next_thread) {         /* switch threads?  */
    next_thread->state = RUNNING;
    t = current_thread;
    current_thread = next_thread;
    //注意 thread_switch是一个函数，那么编译器会为我们自动将thread_switch的下一条指令的地址放入到ra中，然后我们直接在thread_switch中保存ra寄存器即可，那么在切换到新的协程后从thread_switch返回时，会直接执行thread_switch的下一条指令。
    thread_switch((uint64)t, (uint64)current_thread);
  } else
    next_thread = 0;
}

void thread_create(void (*func)())
{
  struct thread *t;

  for (t = all_thread; t < all_thread + MAX_THREAD; t++) {
    if (t->state == FREE) break;
  }
  t->state = RUNNABLE;
  t->ra = (uint64)func;
  t->sp = (uint64)&t->stack[STACK_SIZE - 1];
  t->fp = (uint64)&t->stack[STACK_SIZE - 1];
  //fp is s0
  //而且栈是从高地址往低处增长的！！！！
}

void 
thread_yield(void)
{
  current_thread->state = RUNNABLE;
  thread_schedule();
}

volatile int a_started, b_started, c_started;
volatile int a_n, b_n, c_n;

int 
main(int argc, char *argv[]) 
{
  a_started = b_started = c_started = 0;
  a_n = b_n = c_n = 0;
  thread_init();
  thread_create(thread_a);
  thread_create(thread_b);
  thread_create(thread_c);
  thread_schedule();
  exit(0);
}

~~~



~~~asm
thread_switch:
	sd fp,  0(a0)
	sd s1,  8(a0)
	sd s2,  16(a0)
	sd s3,  24(a0)
	sd s4,  32(a0)
	sd s5,  40(a0)
	sd s6,  48(a0)
	sd s7,  56(a0)
	sd s8,  64(a0)
	sd s9,  72(a0)
	sd s10,  80(a0)
	sd s11,  88(a0)
	sd ra,  96(a0)
	sd sp,  104(a0)
	
	ld fp,  0(a1)
	ld s1,  8(a1)
	ld s2,  16(a1)
	ld s3,  24(a1)
	ld s4,  32(a1)
	ld s5,  40(a1)
	ld s6,  48(a1)
	ld s7,  56(a1)
	ld s8,  64(a1)
	ld s9,  72(a1)
	ld s10,  80(a1)
	ld s11,  88(a1)
	ld ra,  96(a1)
	ld sp,  104(a1)
	ret    /* return to ra */

~~~



## Note

 rename("d1/x", "d2/y");

~~~c
lock d1
    erase x;
release d1
lock d2
    add y
release d2
~~~

这是错误的，正确做法是

~~~c
lock d1 + d2
    earse x
    add y
release d2 + d1
~~~

