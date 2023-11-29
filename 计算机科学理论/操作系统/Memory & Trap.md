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

### Code ： Init

~~~c
//main.c
void main() {
	//...
    printf("xv6 kernel is booting\n");
    printf("\n");
    kvminit();       // create kernel page table
    kvminithart();   // turn on paging
    procinit();      // process table
	//...     
}

//kalloc.c

extern char end[]; // first address after kernel.
                   // defined by kernel.ld.

void kinit()
{
  initlock(&kmem.lock, "kmem");
  freerange(end, (void*)PHYSTOP);
  //这里PHYSTOP是物理内存的最大地址（不是整个物理地址空间的最大地址），在xv6上为0x88000000
  //MAXVA是虚拟内存的最大地址，在xv6上为80 0000 0000
}

//将内核代码后的第一行代码地址到物理内存最大地址之间的页面注册到kmem上
void freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
  //kfree以及kmem见下面memory allocate一节
}

//vm.c
typedef uint64 *pagetable_t; // 512 PTEs
/*
 * the kernel's page table.
 */
pagetable_t kernel_pagetable;

void kvminit(void)
{
  kernel_pagetable = kvmmake();
}

// Switch h/w page table register to the kernel's page table,
// and enable paging.
void kvminithart()
{
  // wait for any previous writes to the page table memory to finish.
  sfence_vma();
  w_satp(MAKE_SATP(kernel_pagetable));
  // flush stale entries from the TLB.
  sfence_vma();
}

// 内核页表采用直接映射的方式进行地址翻译
pagetable_t kvmmake(void)
{
  pagetable_t kpgtbl;

  //这里先分配一个物理页面，作为第一级页表！
  kpgtbl = (pagetable_t) kalloc();
  memset(kpgtbl, 0, PGSIZE);

  // uart registers
  kvmmap(kpgtbl, UART0, UART0, PGSIZE, PTE_R | PTE_W);

  // virtio mmio disk interface
  kvmmap(kpgtbl, VIRTIO0, VIRTIO0, PGSIZE, PTE_R | PTE_W);

  // PLIC
  kvmmap(kpgtbl, PLIC, PLIC, 0x400000, PTE_R | PTE_W);

  // map kernel text executable and read-only.
  kvmmap(kpgtbl, KERNBASE, KERNBASE, (uint64)etext-KERNBASE, PTE_R | PTE_X);

  // map kernel data and the physical RAM we'll make use of.
  kvmmap(kpgtbl, (uint64)etext, (uint64)etext, PHYSTOP-(uint64)etext, PTE_R | PTE_W);

  // map the trampoline for trap entry/exit to
  // the highest virtual address in the kernel.
  kvmmap(kpgtbl, TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);

  // allocate and map a kernel stack for each process.
  proc_mapstacks(kpgtbl);
  
  return kpgtbl;
}
~~~



### Code：Page Allocate & Free

~~~c
struct run {
  struct run *next;
};

struct {
  struct spinlock lock;
  struct run *freelist;
} kmem;


// Free the page of physical memory pointed at by pa,
// which normally should have been returned by a
// call to kalloc().  (The exception is when
// initializing the allocator; see kinit above.)
//释放一个物理页面
void kfree(void *pa)
{
  struct run *r;
  //传入地址的范围要合法并且pa是以PGSIZE对齐的
  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  //填入垃圾数据，这样解引用悬挂指针时可以报错
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;

  acquire(&kmem.lock);
  //归还页面，页面的前sizeof(kmem)个字节用于保存空闲链表的信息
  r->next = kmem.freelist;
  kmem.freelist = r;
  release(&kmem.lock);
}

// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.

//返回一个物理页面的基地址(以PGSIZE对齐的)，若没有页面可以分配，则返回0
void *kalloc(void)
{
  struct run *r;

  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r)
    kmem.freelist = r->next;
  release(&kmem.lock);

  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}

~~~



### Code：vm

~~~c
// add a mapping to the kernel page table.
// only used when booting.
// does not flush TLB or enable paging.
void kvmmap(pagetable_t kpgtbl, uint64 va, uint64 pa, uint64 sz, int perm)
{
  if(mappages(kpgtbl, va, sz, pa, perm) != 0)
    panic("kvmmap");
}


// Create PTEs for virtual addresses starting at va that refer to
// physical addresses starting at pa. va and size might not
// be page-aligned. Returns 0 on success, -1 if walk() couldn't
// allocate a needed page-table page.

//修改页表pagetable，将[va, va + size]映射到[pa, pa + size]，权限为perm
//必要时，会自动分配一个子级页表的
//va与size可以不用PGSIZE对齐
int mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
{
  uint64 a, last;
  pte_t *pte;

  if(size == 0)
    panic("mappages: size");
  
  a = PGROUNDDOWN(va);
  last = PGROUNDDOWN(va + size - 1);
  for(;;){
    if((pte = walk(pagetable, a, 1)) == 0)
      return -1;
    if(*pte & PTE_V)
      panic("mappages: remap");
    *pte = PA2PTE(pa) | perm | PTE_V;
    if(a == last)
      break;
    a += PGSIZE;
    pa += PGSIZE;
  }
  return 0;
}

// 64-bit virtual address
//   39..63 -- must be zero.
//   30..38 -- 9 bits of level-2 index.
//   21..29 -- 9 bits of level-1 index.
//   12..20 -- 9 bits of level-0 index.
//    0..11 -- 12 bits of byte offset within the page.

// PTE
// 10...53 --  PPN
// 0...9 -- Flags
// shift a physical address to the right place for a PTE.
#define PA2PTE(pa) ((((uint64)pa) >> 12) << 10)
#define PTE2PA(pte) (((pte) >> 10) << 12)
#define PTE_FLAGS(pte) ((pte) & 0x3FF)

// extract the three 9-bit page table indices from a virtual address.
#define PXMASK          0x1FF // 9 bits
#define PGSHIFT 12  // bits of offset within a page
#define PXSHIFT(level)  (PGSHIFT+(9*(level)))
#define PX(level, va) ((((uint64) (va)) >> PXSHIFT(level)) & PXMASK)

//根据va，返回最后一级页表中对应的PTE
//若alloc为1，则按需分配页表。并不分配到用户进程所使用的物理页面
//否则，当查找过程中某个pte不存在，则返回0
pte_t *walk(pagetable_t pagetable, uint64 va, int alloc)
{
  if(va >= MAXVA)
    panic("walk");

  for(int level = 2; level > 0; level--) {
    //PX提取出对应level级的index
    pte_t *pte = &pagetable[PX(level, va)];		
    if(*pte & PTE_V) {
      pagetable = (pagetable_t)PTE2PA(*pte);
    } else {
      //如果此页表还没有分配
      if(!alloc || (pagetable = (pde_t*)kalloc()) == 0)
        //没有物理页面了 或者 不需要分配，那么返回0
        return 0;
      memset(pagetable, 0, PGSIZE);
      //此时pagetable是新分配的、下一级的页表，不是当前level页表
      //更新PTE
      *pte = PA2PTE(pagetable) | PTE_V;
    }
  }
  return &pagetable[PX(0, va)];
}
~~~



### Code：uvm

> **注意还需要维护proc->sz这个不变式**

- void uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free)：将最后一级页表中虚拟地址[va, va + npages * PGSIZE]对应的PTE清理为0，若do_free不为0，那么释放相应的物理页面。va必须PGSIZE对齐，PGROUNDUP(oldsz);

- uint64 uvmalloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz, int xperm)：将虚拟地址[oldsz, newsz]映射到自动分配的物理页面上，自动分配并修改页表，权限为xperm | PTE_R | PTE_U。oldsiz与newsz可以不用PGSIZE对齐。

- uint64 uvmdealloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz)：将最后一级页表中虚拟地址[oldsz, newsz]对应的PTE清理为0，并释放相应的物理页面。

- void freewalk(pagetable_t pagetable)：将多级页表中的所有PTE清理为0。这段代码中有一个很有意思的一点：

  ~~~c
  // Recursively free page-table pages.
  // All leaf mappings must already have been removed.
  void freewalk(pagetable_t pagetable)
  {
    // there are 2^9 = 512 PTEs in a page table.
    for(int i = 0; i < 512; i++){
      pte_t pte = pagetable[i];
      //在walk中我们可以得知：用户进程的一、二级页表的权限位仅有PTE_V(内核页表具有所有权限)
      
      if((pte & PTE_V) && (pte & (PTE_R|PTE_W|PTE_X)) == 0){
        // this PTE points to a lower-level page table.
        uint64 child = PTE2PA(pte);
        freewalk((pagetable_t)child);
        pagetable[i] = 0;
     	//这个函数要求我们必须先释放最后一级的所有PTE，
      } else if(pte & PTE_V){
        panic("freewalk: leaf");
      }
    }
    kfree((void*)pagetable);
  }
  ~~~

  

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

