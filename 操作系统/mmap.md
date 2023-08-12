最后一个Lab简直太精彩了,特此记录一下.



## 要求

~~~c
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
~~~

- addr always be zero，man  state that If addr is NULL, then the kernel chooses the (page-aligned) address at which to create the mapping;
  
- `length` is the number of bytes to map

- `prot` indicates whether the memory should be mapped readable, writeable, and/or executable; you can assume that `prot` is `PROT_READ` or `PROT_WRITE` or both

- `flags` will be either `MAP_SHARED`, meaning that modifications to the mapped memory should be written back to the file, or `MAP_PRIVATE`, meaning that they should not.It's OK if processes that map the same `MAP_SHARED` file do **not** share physical pages.

-  `fd` is the open file descriptor of the file to map

- You can assume `offset` is zero (it's the starting point in the file at which to map).

- return the mapped address，or 0xffffffffffffffff (-1) if it fails



`munmap(addr, length)` should remove mmap mappings in the indicated address range.

An `munmap` call might cover only a portion of an mmap-ed region, but you can assume that it will either unmap at the start,  or the whole region (but not punch a hole in the middle of a region).





- Modify `exit` to unmap the process's mapped regions as if `munmap` had been called. Run `mmaptest`; `mmap_test` should pass, but probably not `fork_test`.
- Modify `fork` to ensure that the child has the same mapped regions as the parent





## 流程

~~~c
//void *addr, uint64 length, int prot, int flags, int fd, uint64 offset
struct VMA {
  void *addr;
  uint64 length;
  int prot;
  int flags;
  struct file *file;			//一开始这里使用的是int fd, 在访问时通过myproc()->ofile[VMA->fd]来获取文件指针, 但是在测试用例中,mmap完后调用unlink,这导致fd索引不正确,而指向空file或者另一个file1. 因此后来改用file,解决了此BUG
  uint64 offset;
};
~~~





### fork

~~~c
int
fork(void)
{
...
  // Allocate process.
  if((np = allocproc()) == 0){
    return -1;
  }

  struct VMA *VMA = p->VMA;
  int index = 0;
  for (int i = 0; i < NVMA; i++) {
    if (VMA[i].length != 0) {
      struct VMA *p_VMA = VMA + i;
      struct VMA *c_VMA = &np->VMA[index++];
      c_VMA->addr = p_VMA->addr;
      c_VMA->file = filedup(p_VMA->file);
      c_VMA->flags = p_VMA->flags;
      c_VMA->length = p_VMA->length;
      c_VMA->offset = p_VMA->offset;
      c_VMA->prot = p_VMA->prot;
    }
  }
  
  // Copy user memory from parent to child.
  if(uvmcopy(p->pagetable, np->pagetable, p->sz) < 0){
	...
~~~

### exit

~~~c
uint64
sys_exit(void)
{
  int n;
  struct VMA *VMA = myproc()->VMA; 
  for (int i = 0; i < NVMA; i++) {
    if (VMA[i].length != 0) {
      munmap((uint64)VMA[i].addr, VMA[i].length);
    }
  }
  argint(0, &n);
  exit(n);
  return 0;  // not reached
}
~~~



### trap

~~~c
else if (r_scause() == 15 || r_scause() == 13) {
//printf("page falut\n");
uint64 va = PGROUNDDOWN(r_stval());
if (va >= MAXVA || va >= p->sz) {
  p->killed = 1;
  goto err;
}
pte_t *pte = walk(p->pagetable, va, 0);
//printf("pte: %p %p\n", *pte, pte);
if (pte == 0) {
  p->killed = 1;
  goto err;
}
struct VMA *VMA = VA2VMA(va);				
if (VMA == 0) {
  p->killed = 1;
  goto err;
}
void *mem;
if ((mem = kalloc()) == 0) {    
  p->killed = 1;
  goto err;
}
memset((char*)mem, 0, PGSIZE);      //必须垃圾数据改写为0，这是第一个测试用例要求的
struct inode *inode = VMA->file->ip;
ilock(inode);
//printf("offset : %d\n", va - begin);
readi(inode, 0, (uint64)mem, va - (uint64)VMA->addr, PGSIZE);
iunlock(inode);
*pte = *pte | (uint64)mem >> 2 |              ////mem是以PGSIZE(12)对齐的，而PTE中是要求在10处保存，一个小bug
    (VMA->prot & PROT_READ ? PTE_R : 0) |
    (((VMA->prot & PROT_WRITE) || (VMA->flags & MAP_PRIVATE)) ?  PTE_W : 0);
*pte = *pte & ~PTE_C;
//printf("pte modified: %p %p\n", *pte, pte);
//vmprint(p->pagetable);
~~~



### SystemCall

~~~ c
uint64 sys_mmap(void) {
  uint64 len = 0;
  argaddr(1, &len);     //这里使用argaddr仅仅是为了获取uint64类型的参数
  int prot = 0;
  argint(2, &prot);
  int flags = 0;
  argint(3, &flags);
  int fd = 0;
  argint(4, &fd);
  uint64 offset = 0;
  argaddr(5, &offset);

  struct proc* proc = myproc();
  struct VMA *VMA = proc->VMA;
  int i = 0;
  for (; i < NVMA; i++) {
    if (VMA[i].length == 0) {			//找到一个空闲的VMA
      VMA = VMA + i;
      break;
    }
  }
  if (i == NVMA) return -1;
  
  uint64 oldsz = PGROUNDUP(proc->sz);
  uint64 newsz = oldsz + len;
  if (newsz > MAXVA) {
    return -1;
  }
  proc->sz = newsz;     

  struct file *file= proc->ofile[fd];
  //文件是不可读的，但是映射要求可读
  //文件是不可写的，但是映射要求可写，同时是共享的
  if ((file->readable == 0 && (prot & PROT_READ))
    ||(file->writable == 0 && (prot & PROT_WRITE) && (flags & MAP_PRIVATE) == 0)) {
    return -1;
  }

  VMA->file = filedup(file);
  VMA->length = len;
  VMA->flags = flags;
  VMA->offset = offset;
  VMA->prot = prot;
    
  for (uint64 va = oldsz; va < newsz; va += PGSIZE) {
    pte_t *pte = walk(proc->pagetable, va, 1);
    *pte = PTE_C | PTE_U | PTE_V; 
    //printf("pte : %p %p\n", *pte, pte);
  }
  VMA->addr = (void *)oldsz;
  //print_VMA(VMA);
  return oldsz;
}

uint64 sys_munmap(void) {
  uint64 addr = 0;
  uint64 len = 0;
  argaddr(0, &addr);
  argaddr(1, &len);
  return munmap(addr, len);
}

int munmap(uint64 addr, uint64 len) 
{
  struct VMA *VMA = VA2VMA(addr);
  if (VMA == 0) return -1;

  struct file *file = VMA->file;
  if (VMA->flags & MAP_SHARED) {
    filewrite(file, addr, len);
  }
  uvmdealloc(myproc()->pagetable, addr, addr + len);
  VMA->addr = (void *)(addr + len);
  VMA->length -= len;
  if (VMA->length == 0) {
    fileclose(file);
  }
  return 0;
}

struct VMA* VA2VMA(uint64 va) 
{
  struct VMA *VMA = myproc()->VMA;
  for (int i = 0; i < NVMA; i++) {
    uint64 begin = (uint64)(VMA[i].addr);
    uint64 end = begin + VMA[i].length;
    if (begin <= va && va < end) {
      return VMA + i;
    }
  }
  return 0;
}
~~~



特别注意一下这里的内存管理策略，这也是本Lab的精彩之处！

由于xv6并没有提供内核级的内存管理器，所以映射页面的内存管理有些棘手。最好最优秀的解决方案是自己写一个内存管理器，显然这对于我来说是不可能的:) 。因此我采取了一种简单的方案，就是只增加sbrk，而不减少。采用这种方案必须修改一些与内存相关的一些函数才能正常工作。我也在此debug（printf + vmprint + gdb）四个多小时之久，要知道我才一共写了五个多小时😥。



~~~diff
void
uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free)
{
  uint64 a;
  pte_t *pte;

  if((va % PGSIZE) != 0)
    panic("uvmunmap: not aligned");

  for(a = va; a < va + npages*PGSIZE; a += PGSIZE){
    if((pte = walk(pagetable, a, 0)) == 0)
      panic("uvmunmap: walk");
-    if((*pte & PTE_V) == 0)
-      panic("uvmunmap: not mapped");
+ 	 if((*pte & PTE_V) == 0) {
+      *pte = 0;
+      continue;
+    }
    if(PTE_FLAGS(*pte) == PTE_V)
      panic("uvmunmap: not a leaf");
    if(do_free){
      uint64 pa = PTE2PA(*pte);
+     if (pa == 0) {
+        *pte = 0;
+        continue;
+     }
      kfree((void*)pa);
    }
    *pte = 0;
  }
}
~~~

这是因为在unmap时调用了`uvmdealloc(myproc()->pagetable, addr, addr + len);`，将相应的最后一级PTE全覆写为0，并且释放掉物理页面了。这在堆中留下一些空洞，而释放进程会再次调用uvmunmap，这些空洞会使得pa为0以及PTE_V检查过不去，因此修改为直接跳过即可，这里保险起见设置pte为0。





~~~diff
int
uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
{
  pte_t *pte;
  uint64 pa, i;
  uint flags;
  char *mem;

  for(i = 0; i < sz; i += PGSIZE){
    if((pte = walk(old, i, 0)) == 0)
      panic("uvmcopy: pte should exist");
    if((*pte & PTE_V) == 0)
-      panic("uvmcopy: page not present");
+	   continue;
    pa = PTE2PA(*pte);
    flags = PTE_FLAGS(*pte);
+    if ((*pte & PTE_C)) {
+      mappages(new, i, PGSIZE, pa, flags);
+      continue;
+    }
    if((mem = kalloc()) == 0)
	...
}

~~~

这是因为在子进程拷贝父进程的映射页面时，由于采用COW策略，有些页面还未加载。为了处理这种情况，添加了以上代码。
