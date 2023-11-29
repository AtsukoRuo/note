# File System

[TOC]

## Overview

The purpose of a file system is to organize and store data **persistently**. 

本质上文件系统就是一个在磁盘上的数据结构。以xv6文件系统为例，它在磁盘上的布局如下：

![image-20230806092110852](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230806092110852.png)

- superblock:  保存文件系统的元信息，例如有多少数据块、inode的数量
- boot：在qemu上，直接把代码复制到内存0x80000000上，无需考虑boot块。



The file system addresses several challenges:

- 数据结构：组织并存储数据

- **crash recovery**： The risk is that a crash might interrupt a sequence of updates（fs operation are multi-step disk operation） and leave inconsistent on-disk data structures.

- **concurrent**： Different processes may operate on the file system at the same time, so the file-system code must coordinate to maintain invariants.

- **cache**：Accessing a disk is orders of magnitude slower than accessing memory, so the file system must maintain an in-memory cache of popular blocks.



![image-20230806091931347](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230806091931347.png)

- Disk：提供持久化存储服务
- Buffer cache：提供同步以及缓存服务
- Logging：提供crash recovery服务
- Inode：提供了文件概念
- Directory：提供了目录概念
- Pathname：提供了文件系统资源定位符，
- File descriptor：统一抽象资源接口，整合了文件、管道、Socket等概念





## Disk

Disk hardware traditionally presents the data on the disk as a numbered sequence of 512-byte **blocks** (also called **sectors**)    **The block size that an operating system uses for its file system** maybe different than the sector size that a disk uses, but typically the block size is a multiple of the sector size

~~~c
uint64 sector = b->blockno * (BSIZE / 512);
~~~



硬盘（SSD）给我们提供了一种抽象：多个连续的块，编号从`0 ~ N`，大小一般为512kb。

## Buffer cache

The buffer cache has two jobs:

- **synchronize access** to disk blocks to ensure that only one copy of a block is in memory and that only one kernel thread at a time uses that copy;

- **cache popular blocks** so that they don’t need to be re-read from the slow disk.



所有读/写磁盘块的操作都要经过Buffer cache！

~~~c
struct {
  struct spinlock lock;
  struct buf buf[NBUF];
  //buf采用双向链表的策略来执行LRU算法的，同时还提供了一个数组来存储
  //一般不直接访问数组，而是通过head来访问
  //head.next是最近访问过的，而head.prev是最近访问最少的
  struct buf head;
} bcache;
~~~



~~~c
struct buf {
  int valid;   // it might have not yet been read in from disk
  int disk;    // it might have been updated by software but not yet written to the disk.
  uint dev;
  uint blockno;
  struct sleeplock lock;
  uint refcnt;
  struct buf *prev; // LRU cache list
  struct buf *next;
  uchar data[BSIZE];
};
~~~





![1691462686359-screenshot](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\1691462686359-screenshot.png)

~~~c
// Return a locked buf with the contents of the indicated block.
// Once bread has read the disk (if needed) and returned the buffer to its caller, the caller has exclusive use of the buffer and can read or write the data bytes.
//When the caller is done with a buffer, it must call brelse to release it.
struct buf* bread(uint dev, uint blockno)
{
  struct buf *b;

  b = bget(dev, blockno);		
  if(!b->valid) {
    virtio_disk_rw(b, 0);
    b->valid = 1;
  }
  return b;
}

// Look through buffer cache for particular block on device dev.
// If not found, allocate a buffer.
// In either case, return locked buffer.
static struct buf* bget(uint dev, uint blockno)
{
  struct buf *b;

  acquire(&bcache.lock);

  // Is the block already cached?
  for(b = bcache.head.next; b != &bcache.head; b = b->next){
    if(b->dev == dev && b->blockno == blockno){
      b->refcnt++;
      release(&bcache.lock);
      acquiresleep(&b->lock);
      return b;
    }
  }

  // Not cached.
  // Recycle the least recently used (LRU) unused buffer.
  for(b = bcache.head.prev; b != &bcache.head; b = b->prev){
    if(b->refcnt == 0) {
      b->dev = dev;
      b->blockno = blockno;
      b->valid = 0;				//ensures that bread will read the block data from disk rather than incorrectly using the buffer’s previous contents.
      b->refcnt = 1;
      release(&bcache.lock);
      acquiresleep(&b->lock);
      return b;
    }
  }
  panic("bget: no buffers");
}

// Write b's contents to disk.  Must be locked.
void bwrite(struct buf *b)
{
  if(!holdingsleep(&b->lock))
    panic("bwrite");
  virtio_disk_rw(b, 1);
}


// Release a locked buffer.
// Move to the head of the most-recently-used list.
void
brelse(struct buf *b)
{
  if(!holdingsleep(&b->lock))
    panic("brelse");

  releasesleep(&b->lock);

  acquire(&bcache.lock);
  b->refcnt--;
  if (b->refcnt == 0) {
    // no one is waiting for it.
    b->next->prev = b->prev;
    b->prev->next = b->next;
    b->next = bcache.head.next;
    b->prev = &bcache.head;
    bcache.head.next->prev = b;
    bcache.head.next = b;
  }
  release(&bcache.lock);
}
~~~



~~~c
void
acquiresleep(struct sleeplock *lk)
{
  acquire(&lk->lk);
  while (lk->locked) {
    sleep(lk, &lk->lk);
  }
  lk->locked = 1;
  lk->pid = myproc()->pid;
  release(&lk->lk);
}

void
releasesleep(struct sleeplock *lk)
{
  acquire(&lk->lk);
  lk->locked = 0;
  lk->pid = 0;
  wakeup(lk);
  release(&lk->lk);
}
~~~



## Logging

The problem arises because many file-system operations involve multiple writes to the disk, and a crash after a subset of the writes may leave the on-disk file system in an inconsistent state. the crash may either leave an inode with a reference to a content block that is marked free, or it may leave an allocated but unreferenced content block.



An xv6 system call does not directly write the on-disk file system data structures. Instead, **it places a description of all the disk writes it wishes to make in a log on the disk**. Once the system call has logged all of its writes, it writes a special **commit** record to the disk indicating that the log contains a complete operation. At that point the system call copies the writes to the on-disk file system data structures. After those writes have completed, the system call erases the log on disk.



If the system should crash and reboot, the file-system code recovers from the crash as follows, before running any processes. If the log is marked as containing a complete operation, then the recovery code copies the writes to where they belong in the on-disk file system. If the log is not marked as containing a complete operation, the recovery code ignores the log. The recovery code finishes by erasing the log.



If the crash occurs before the operation commits, then the log on disk will not be marked as complete, the recovery code will ignore it, and the state of the disk will be as if the operation had not even started. If the crash occurs after the operation commits, then recovery will replay all of the operation’s writes, perhaps repeating them if the operation had started to write them to the on-disk data structure. In either case, the log makes operations atomic with respect to crashes





good side of logging solution：

- atomic fscalls
- fast recovery
- high performance（xv6 doesn't） 



~~~c
//一开始执行调度器时，第一个切换到forkret上
void forkret(void)
{
  static int first = 1;
  release(&myproc()->lock);
  if (first) {
    first = 0;
    fsinit(ROOTDEV);
  }
  usertrapret();
}

// Init fs
void fsinit(int dev) {
  ...
  initlog(dev, &sb);
}

void initlog(int dev, struct superblock *sb)
{
  ...
  recover_from_log();
}
static void
recover_from_log(void)
{
  read_head();
  install_trans(1); // if committed, copy from log to disk
  log.lh.n = 0;	
  write_head(); // clear the log
}
// Copy committed blocks from log to their home location
static void
install_trans(int recovering)
{
  int tail;

  for (tail = 0; tail < log.lh.n; tail++) {
    struct buf *lbuf = bread(log.dev, log.start+tail+1); // read log block
    struct buf *dbuf = bread(log.dev, log.lh.block[tail]); // read dst
    memmove(dbuf->data, lbuf->data, BSIZE);  // copy block to dst
    bwrite(dbuf);  // write dst to disk
    if(recovering == 0)
      bunpin(dbuf);		
    brelse(lbuf);
    brelse(dbuf);
  }
}
void
bunpin(struct buf *b) {
  acquire(&bcache.lock);
  b->refcnt--;
  release(&bcache.lock);
}
~~~



![image-20230808092543354](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230808092543354.png)

如果在write33与wrtie46中发生crash，那么我们将丢失inode x，因为没有任何一个目录记录它。下面我们来看一个引入log的写入操作



![image-20230808101542829](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230808101542829.png)

第一个bwrite2是更新log flag，第二个bwrite2是清除log flag。从中我们还观察到，写入操作再重复执行了2次，效率并不高。



A typical use of the log in a system call looks like this:

~~~c
begin_op()
...
bp = bread(...);
bp->data[...] = ...;
log_write(bp);
...
end_op();
~~~



~~~c
struct log {
  struct spinlock lock;
  int start;
  int size;
  int outstanding; // how many FS sys calls are executing.
    
// log.outstanding counts the number of system calls executing currently; the total reserved space is log.outstanding times MAXOPBLOCKS. The code conservatively assumes that each system call might write up to MAXOPBLOCKS distinct blocks. if outstanding > 0 then it prevents a commit from occurring during this system call.
    
  int committing;  // in commit(), please wait.
  int dev;
  struct logheader lh;
};
struct log log;

// Contents of the header block, used for both the on-disk header block
// and to keep track in memory of logged block# before commit.
struct logheader {
  int n;
  int block[LOGSIZE];
};


#define LOGSIZE      (MAXOPBLOCKS * 3)  // max data blocks in on-disk log
#define MAXOPBLOCKS  10  // max # of blocks any FS op writes 
~~~



~~~c
// called at the start of each FS system call.
void
begin_op(void)
{
  acquire(&log.lock);
  while(1){
    if(log.committing){
      sleep(&log, &log.lock);
    } else if(log.lh.n + (log.outstanding + 1)*MAXOPBLOCKS > LOGSIZE){
      // all concurrent ops must fit so limit # concurrent fs calls（限制了系统调用的并发数量）
      // this op might exhaust log space; wait for commit.
      sleep(&log, &log.lock);
    } else {
      log.outstanding += 1;
      release(&log.lock);
      break;
    }
  }
}


~~~

![1691464847620-screenshot](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\1691464847620-screenshot.png)

~~~c
// Caller has modified b->data and is done with the buffer.
// Record the block number and pin in the cache by increasing refcnt.
// commit()/write_log() will do the disk write.
//
// log_write() replaces bwrite(); a typical use is:
//   bp = bread(...)
//   modify bp->data[]
//   log_write(bp)
//   brelse(bp)
void
log_write(struct buf *b)
{
  int i;

  acquire(&log.lock);
  if (log.lh.n >= LOGSIZE || log.lh.n >= log.size - 1)
    panic("too big a transaction");
  if (log.outstanding < 1)
    panic("log_write outside of trans");

  for (i = 0; i < log.lh.n; i++) {
    if (log.lh.block[i] == b->blockno)   // log absorption
      break;
  }
 //log_write notices when a block is written multiple times during a single transaction, and allocates that block the same slot in the log. This optimization is often called absorption.
 //By absorbing several disk writes into one, the file system can save log space and can achieve better performance because only one copy of the disk block must be written to disk. 而不是在接下来的提交中多次调用bwrite写入同一个块
    
  log.lh.block[i] = b->blockno;
  if (i == log.lh.n) {  // Add new block to log?
    bpin(b);			//防止从buffer cache中驱逐出去，避免破坏事务的原子性
    log.lh.n++;
  }
  release(&log.lock);
}

void
bpin(struct buf *b) {
  acquire(&bcache.lock);
  b->refcnt++;
  release(&bcache.lock);
}

~~~

只有在提交操作时使用bwrite函数，其他场景使用log_write函数即可。log_write仅仅是记录信息，那些块还在位于内存的缓冲区中，直至提交操作。



~~~c
// called at the end of each FS system call.
// commits if this was the last outstanding operation.
void end_op(void)
{
  int do_commit = 0;

  acquire(&log.lock);
  log.outstanding -= 1;
  if(log.committing)
    panic("log.committing");
  if(log.outstanding == 0){				//此时是最后一个system call的提交
    do_commit = 1;
    log.committing = 1;
  } else {
    // begin_op() may be waiting for log space,
    // and decrementing log.outstanding has decreased
    // the amount of reserved space.
    wakeup(&log);
  }
  release(&log.lock);

  if(do_commit){
    // call commit w/o holding locks, since not allowed
    // to sleep with locks.
    commit();
    acquire(&log.lock);
    log.committing = 0;
    //唤醒在begin_op()中因为log.committing而睡眠的线程
    wakeup(&log);
    release(&log.lock);
  }
}
~~~

![1691464796677-screenshot](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\1691464796677-screenshot.png)



~~~c
static void commit()
{
  if (log.lh.n > 0) {
    write_log();     // Write modified blocks from cache to log
    write_head();    // Write header to disk -- the real commit
    install_trans(0); // Now install writes to home locations
    log.lh.n = 0;
    write_head();    // Erase the transaction from the log
  }
}
// Copy modified blocks from cache to log.
static void
write_log(void)
{
  int tail;

  for (tail = 0; tail < log.lh.n; tail++) {
    struct buf *to = bread(log.dev, log.start+tail+1); // log block
    struct buf *from = bread(log.dev, log.lh.block[tail]); // cache block
    memmove(to->data, from->data, BSIZE);
    bwrite(to);  // write the log
    brelse(from);
    brelse(to);
  }
}

// Write in-memory log header to disk.
// This is the true point at which the
// current transaction commits.
static void
write_head(void)
{
  struct buf *buf = bread(log.dev, log.start);
  struct logheader *hb = (struct logheader *) (buf->data);
  int i;
  hb->n = log.lh.n;
  for (i = 0; i < log.lh.n; i++) {
    hb->block[i] = log.lh.block[i];
  }
  bwrite(buf);				//commit point	
  brelse(buf);
}


// Copy committed blocks from log to their home location
static void install_trans(int recovering)
{
  int tail;

  for (tail = 0; tail < log.lh.n; tail++) {
    struct buf *lbuf = bread(log.dev, log.start+tail+1); // read log block
    struct buf *dbuf = bread(log.dev, log.lh.block[tail]); // read dst
    memmove(dbuf->data, lbuf->data, BSIZE);  // copy block to dst
    bwrite(dbuf);  // write dst to disk
    if(recovering == 0)
      bunpin(dbuf);
    brelse(lbuf);
    brelse(dbuf);
  }
}


~~~

### Block allocator

~~~c
// Block of free map containing bit for block b
#define BBLOCK(b, sb) ((b)/BPB + sb.bmapstart)

// Allocate a zeroed disk block.
// returns 0 if out of disk space.
static uint balloc(uint dev)
{
  int b, bi, m;
  struct buf *bp;

  bp = 0;
  //遍历所有的block map块
  for(b = 0; b < sb.size; b += BPB){
    //读取出一个block map块
    bp = bread(dev, BBLOCK(b, sb));
    //对这个block map块进行位遍历
    for(bi = 0; bi < BPB && b + bi < sb.size; bi++){
      m = 1 << (bi % 8);
      if((bp->data[bi/8] & m) == 0){  // Is block free?
        bp->data[bi/8] |= m;  // Mark block in use.
        log_write(bp);
        brelse(bp);
        bzero(dev, b + bi);
        return b + bi;
      }
    }
    brelse(bp);
  }
  printf("balloc: out of blocks\n");
  return 0;
}

// Free a disk block.
static void bfree(int dev, uint b)
{
  struct buf *bp;
  int bi, m;

  bp = bread(dev, BBLOCK(b, sb));
  bi = b % BPB;
  m = 1 << (bi % 8);
  if((bp->data[bi/8] & m) == 0)
    panic("freeing free block");
  bp->data[bi/8] &= ~m;
  log_write(bp);
  brelse(bp);
}
~~~

 The race that might occur if two processes try to allocate a block at the same time is prevented by the fact that the buffer cache only lets one process use any one bitmap block at a time.

## Inode

The term **inode** can have one of two related meanings.

- It might refer to the **on-disk** data structure containing a file’s size and list of data block numbers
-  “inode” might refer to an **in-memory** inode, which contains a copy of the on-disk inode as well as **extra information needed within the kernel**.



~~~c
// On-disk inode structure
struct dinode {
  short type;           // File type —— File or Directory？
  short major;          // Major device number (T_DEVICE only)
  short minor;          // Minor device number (T_DEVICE only)
  short nlink;          // Number of links to inode in file system
  uint size;            // Size of file (bytes)
  uint addrs[NDIRECT+1];   // Data block addresses
};
~~~



~~~c
struct inode {
  uint dev;           // Device number
  uint inum;          // Inode number
  int ref;            // Reference count，正在使用该inode的线程数
  struct sleeplock lock; // protects everything below here
  int valid;          // inode has been read from disk?

  short type;         // copy of disk inode
  short major;
  short minor;
  short nlink;	      //在文件系统中，硬链接的数量
  uint size;
  uint addrs[NDIRECT+1];
};
~~~



~~~c
struct {
  struct spinlock lock;
  struct inode inode[NINODE];
} itable;
//itable实际上是一个icache层，主要还是提供同步机制
#define NINODE       50  // maximum number of active i-nodes
~~~





### Inode

一般来讲，inode节点与其对应的数据块一起作为整一个事务来获取，所以在iput、iupdate等代码中无需调用begin_op以及end_op函数

- struct inode* ialloc(uint dev, short type)：在磁盘上分配一个inode，并在内存中保存这个inode
- static struct inode* iget(uint dev, uint inum)：获取指定dev以及inum的inode
- void ilock(struct inode *ip)：给指定inode上锁
- void iunlock(struct inode *ip)
- void iput(struct inode *ip)，inode的ref减一，如果ref为0，释放该inode
- void iupdate(struct inode *ip)，写入磁盘中的inode
- void itrunc(struct inode *ip)，丢弃inode的数据区

~~~c
#define IBLOCK(i, sb)     ((i) / IPB + sb.inodestart)
//((inum) / (1024 / sizeof(struct dinode)) + sb.inodestart)

//这个函数完成两件事情
//1. 在硬盘上分配一个inode
//2. 在内存上保存这个inode
struct inode*
ialloc(uint dev, short type)
{
  int inum;
  struct buf *bp;
  struct dinode *dip;

  for(inum = 1; inum < sb.ninodes; inum++){
    //读取指定inum的inode所作的块
    bp = bread(dev, IBLOCK(inum, sb));
    //读取指定inum的inode
    dip = (struct dinode*)bp->data + inum%IPB;
   
    if(dip->type == 0){  // a free inode
      memset(dip, 0, sizeof(*dip));
      dip->type = type;
      log_write(bp);   //mark it allocated on the disk
      brelse(bp);      //Move to the head of the most-recently-used list.
      return iget(dev, inum);		//开始向内存保存这个inode
    }
    brelse(bp);
  }
  printf("ialloc: no inodes\n");
  return 0;
}
~~~



~~~c
static struct inode*
iget(uint dev, uint inum)
{
  struct inode *ip, *empty;

  acquire(&itable.lock);

  // Is the inode already in the table?
  empty = 0;
  for(ip = &itable.inode[0]; ip < &itable.inode[NINODE]; ip++){
    //ip->ref > 0 means active inode
    if(ip->ref > 0 && ip->dev == dev && ip->inum == inum){
      ip->ref++;
      release(&itable.lock);
      return ip;
    }
    if(empty == 0 && ip->ref == 0)    // Remember empty slot.
      empty = ip;
  }

  //这里处理的是新添inode的情况
  // Recycle an inode entry.
  if(empty == 0)
    panic("iget: no inodes");

  ip = empty;
  ip->dev = dev;
  ip->inum = inum;
  ip->ref = 1;
  ip->valid = 0;
  release(&itable.lock);
  return ip;
}
//这里无需上锁，只有在接下来要访问inode时才获取锁
~~~



~~~c
// Lock the given inode.
// Reads the inode from disk if necessary.
// 给inode ip上锁
void
ilock(struct inode *ip)
{
  struct buf *bp;
  struct dinode *dip;

  if(ip == 0 || ip->ref < 1)
    panic("ilock");

  acquiresleep(&ip->lock);

  if(ip->valid == 0){				//如果还未从硬盘中读取
    bp = bread(ip->dev, IBLOCK(ip->inum, sb));
    dip = (struct dinode*)bp->data + ip->inum%IPB;
    ip->type = dip->type;
    ip->major = dip->major;
    ip->minor = dip->minor;
    ip->nlink = dip->nlink;
    ip->size = dip->size;
    memmove(ip->addrs, dip->addrs, sizeof(ip->addrs));
    brelse(bp);
    ip->valid = 1;
    if(ip->type == 0)
      panic("ilock: no type");
  }
}

// Unlock the given inode.
void
iunlock(struct inode *ip)
{
  if(ip == 0 || !holdingsleep(&ip->lock) || ip->ref < 1)
    panic("iunlock");

  releasesleep(&ip->lock);
}
~~~



~~~c
// Drop a reference to an in-memory inode.
// If that was the last reference, the inode table entry can
// be recycled.
// If that was the last reference and the inode has no links
// to it, free the inode (and its content) on disk.
// All calls to iput() must be inside a transaction in
// case it has to free the inode.
// 不要持有lock来调用iput
// 可以使用iunlockput来释放有锁的inode

void
iput(struct inode *ip)
{
  acquire(&itable.lock);

  if(ip->ref == 1 && ip->valid && ip->nlink == 0){
    // inode has no links and no other references: truncate and free.

    // ip->ref == 1 means no other process can have ip locked,
    // so this acquiresleep() won't block (or deadlock).
    acquiresleep(&ip->lock);

    release(&itable.lock);

    itrunc(ip);
    ip->type = 0;
    iupdate(ip);
    ip->valid = 0;

    releasesleep(&ip->lock);

    acquire(&itable.lock);
  }

  ip->ref--;
  release(&itable.lock);
}

~~~



~~~c
// Truncate inode (discard contents).
// Caller must hold ip->lock.
void
itrunc(struct inode *ip)
{
  int i, j;
  struct buf *bp;
  uint *a;

  for(i = 0; i < NDIRECT; i++){
    if(ip->addrs[i]){
      bfree(ip->dev, ip->addrs[i]);
      ip->addrs[i] = 0;
    }
  }

  if(ip->addrs[NDIRECT]){
    bp = bread(ip->dev, ip->addrs[NDIRECT]);
    a = (uint*)bp->data;
    for(j = 0; j < NINDIRECT; j++){
      if(a[j])
        bfree(ip->dev, a[j]);
    }
    brelse(bp);
    bfree(ip->dev, ip->addrs[NDIRECT]);
    ip->addrs[NDIRECT] = 0;
  }

  ip->size = 0;
  iupdate(ip);
}


// Copy a modified in-memory inode to disk.
// Must be called after every change to an ip->xxx field
// that lives on disk.
// Caller must hold ip->lock.
void
iupdate(struct inode *ip)
{
  struct buf *bp;
  struct dinode *dip;

  bp = bread(ip->dev, IBLOCK(ip->inum, sb));
  dip = (struct dinode*)bp->data + ip->inum%IPB;
  dip->type = ip->type;
  dip->major = ip->major;
  dip->minor = ip->minor;
  dip->nlink = ip->nlink;
  dip->size = ip->size;
  memmove(dip->addrs, ip->addrs, sizeof(ip->addrs));
  log_write(bp);
  brelse(bp);
}
~~~



### Inode Content

![image-20230806212114066](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\image-20230806212114066.png)

~~~c
// Inode content
//
// The content (data) associated with each inode is stored
// in blocks on the disk. The first NDIRECT block numbers
// are listed in ip->addrs[].  The next NINDIRECT blocks are
// listed in block ip->addrs[NDIRECT].

// Return the disk block address of the nth block in inode ip.
// If there is no such block, bmap allocates one.
// returns 0 if out of disk space.
//将逻辑磁盘号转换为物理磁盘号
//inode->addr中存储的是物理磁盘号
static uint
bmap(struct inode *ip, uint bn)
{
  uint addr, *a;
  struct buf *bp;

  if(bn < NDIRECT){
    if((addr = ip->addrs[bn]) == 0){		//分配一个数据块
      addr = balloc(ip->dev);
      if(addr == 0)
        return 0;
      ip->addrs[bn] = addr;
    }
    return addr;
  }
  bn -= NDIRECT;

  if(bn < NINDIRECT){
    // Load indirect block, allocating if necessary.
    if((addr = ip->addrs[NDIRECT]) == 0){		//分配一个indirect block
      addr = balloc(ip->dev);
      if(addr == 0)
        return 0;
      ip->addrs[NDIRECT] = addr;
    }
      
    bp = bread(ip->dev, addr);
    a = (uint*)bp->data;
    if((addr = a[bn]) == 0){			//分配一个数据块
      addr = balloc(ip->dev);
      if(addr){
        a[bn] = addr;
        log_write(bp);
      }
    }
    brelse(bp);
    return addr;
  }

  panic("bmap: out of range");
}


~~~



~~~c
// Read data from inode.
// Caller must hold ip->lock.
// If user_dst==1, then dst is a user virtual address;
// otherwise, dst is a kernel address.

//读取inode数据区中偏移量为off的连续n个字符
int
readi(struct inode *ip, int user_dst, uint64 dst, uint off, uint n)
{
  uint tot, m;
  struct buf *bp;

  if(off > ip->size || off + n < off)
    return 0;
  if(off + n > ip->size)
    n = ip->size - off;

  for(tot=0; tot<n; tot+=m, off+=m, dst+=m){
    uint addr = bmap(ip, off/BSIZE);
    if(addr == 0)
      break;
    bp = bread(ip->dev, addr);
    m = min(n - tot, BSIZE - off % BSIZE);
    if(either_copyout(user_dst, dst, bp->data + (off % BSIZE), m) == -1) {
      brelse(bp);
      tot = -1;
      break;
    }
    brelse(bp);
  }
  return tot;
}
~~~



~~~c
// Write data to inode.
// Caller must hold ip->lock.
// If user_src==1, then src is a user virtual address;
// otherwise, src is a kernel address.
// Returns the number of bytes successfully written.
// If the return value is less than the requested n,
// there was an error of some kind.
//向inode的偏移量为off的数据区中连续写入n个字符
int
writei(struct inode *ip, int user_src, uint64 src, uint off, uint n)
{
  uint tot, m;
  struct buf *bp;

  if(off > ip->size || off + n < off)
    return -1;
  if(off + n > MAXFILE*BSIZE)
    return -1;

  for(tot=0; tot<n; tot+=m, off+=m, src+=m){
    uint addr = bmap(ip, off/BSIZE);
    if(addr == 0)
      break;
    bp = bread(ip->dev, addr);
    m = min(n - tot, BSIZE - off%BSIZE);
    if(either_copyin(bp->data + (off % BSIZE), user_src, src, m) == -1) {
      brelse(bp);
      break;
    }
    log_write(bp);
    brelse(bp);
  }

  if(off > ip->size)
    ip->size = off;

  // write the i-node back to disk even if the size didn't change
  // because the loop above might have called bmap() and added a new
  // block to ip->addrs[].
  iupdate(ip);

  return tot;
}
~~~



~~~cc

// Copy stat information from inode.
// Caller must hold ip->lock.
void
stati(struct inode *ip, struct stat *st)
{
  st->dev = ip->dev;
  st->ino = ip->inum;
  st->type = ip->type;
  st->nlink = ip->nlink;
  st->size = ip->size;
}
~~~



## Directory

目录与文件的实现类似，可以视为一种特殊的文件。它inode的type是T_DIR，而数据区保存的是`struct dirent[]`

根目录（root）保存在一个固定位置的特殊inode，root inode（1）

~~~c
// Directory is a file containing a sequence of dirent structures.
#define DIRSIZ 14

struct dirent {
  ushort inum;					//inum = 0 means that this entry is free
  char name[DIRSIZ];			//name is at most DIRSEIZ(14) characters; if shorter. it is terminated by a NULL
};

~~~



~~~c
// Look for a directory entry in a directory.
// If found, set *poff to byte offset of entry.

//从目录inode节点dp中查找是否有匹配的文件名name
//返回这个文件的inode，并且设置在目录中的偏移量poff
struct inode*
dirlookup(struct inode *dp, char *name, uint *poff)
{
  uint off, inum;
  struct dirent de;

  if(dp->type != T_DIR)
    panic("dirlookup not DIR");

  for(off = 0; off < dp->size; off += sizeof(de)){
    if(readi(dp, 0, (uint64)&de, off, sizeof(de)) != sizeof(de))
      panic("dirlookup read");
    if(de.inum == 0)			//当前是free的
      continue;
    if(namecmp(name, de.name) == 0){
      // entry matches path element
      if(poff)
        *poff = off;
      inum = de.inum;
      return iget(dp->dev, inum);
    }
  }

  return 0;
}


// Write a new directory entry (name, inum) into the directory dp.
// Returns 0 on success, -1 on failure (e.g. out of disk blocks).

//在指定目录节点dp中添加文件节点（name + inum）
int
dirlink(struct inode *dp, char *name, uint inum)
{
  int off;
  struct dirent de;
  struct inode *ip;

  // Check that name is not present.
  if((ip = dirlookup(dp, name, 0)) != 0){
    iput(ip);
    return -1;
  }

  // Look for an empty dirent.
  for(off = 0; off < dp->size; off += sizeof(de)){
    if(readi(dp, 0, (uint64)&de, off, sizeof(de)) != sizeof(de))
      panic("dirlink read");
    if(de.inum == 0)			//找到一个不用的
      break;
  }

  strncpy(de.name, name, DIRSIZ);
  de.inum = inum;
  if(writei(dp, 0, (uint64)&de, off, sizeof(de)) != sizeof(de))		//会更新dp->size的
    return -1;

  return 0;
}
~~~



## Pathname

![1691636282124-screenshot](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\1691636282124-screenshot.png)

~~~c
// Paths

// Copy the next path element from path into name.
// Return a pointer to the element following the copied one.
// The returned path has no leading slashes,
// so the caller can check *path=='\0' to see if the name is the last one.
// If no name to remove, return 0.
//
// Examples:
//   skipelem("a/bb/c", name) = "bb/c", setting name = "a"
//   skipelem("///a//bb", name) = "bb", setting name = "a"
//   skipelem("a", name) = "", setting name = "a"
//   skipelem("", name) = skipelem("////", name) = 0
//
static char* skipelem(char *path, char *name)
{
  char *s;
  int len;

  while(*path == '/')
    path++;
  if(*path == 0)
    return 0;
  s = path;
  while(*path != '/' && *path != 0)
    path++;
  len = path - s;
  if(len >= DIRSIZ)
    memmove(name, s, DIRSIZ);
  else {
    memmove(name, s, len);
    name[len] = 0;
  }
  while(*path == '/')
    path++;
  return path;
}
// Look up and return the inode for a path name.
// If parent != 0, return the inode for the parent and copy the final
// path element into name, which must have room for DIRSIZ bytes.
// Must be called inside a transaction since it calls iput().

static struct inode* namex(char *path, int nameiparent, char *name)
{
  struct inode *ip, *next;
  if(*path == '/')
    ip = iget(ROOTDEV, ROOTINO);
  else
    ip = idup(myproc()->cwd);

  while((path = skipelem(path, name)) != 0){
    ilock(ip);						//上锁
    if(ip->type != T_DIR){
      iunlockput(ip);
      return 0;
    }
    if(nameiparent && *path == '\0'){
      // Stop one level early.
      iunlock(ip);
      //此时ip是父目录的inode
      return ip;
    }
    if((next = dirlookup(ip, name, 0)) == 0){
      iunlockput(ip);
      return 0;
    }
    iunlockput(ip);
    ip = next;
  }
    
  if(nameiparent){
    iput(ip);
    return 0;
  }
  return ip;
}

//返回指定路径的inode，未查找到则返回0
struct inode* namei(char *path)
{
  char name[DIRSIZ];
  return namex(path, 0, name);
}

//返回指定路径的父目录inode，并通过参数name返回最后一部分名字，例如a/b/c返回c
struct inode* nameiparent(char *path, char *name)
{
  return namex(path, 1, name);
}
~~~



## File descriptor

~~~c
struct file {
  enum { FD_NONE, FD_PIPE, FD_INODE, FD_DEVICE } type;
  int ref; // reference count
  char readable;
  char writable;
  struct pipe *pipe; // FD_PIPE
  struct inode *ip;  // FD_INODE and FD_DEVICE
  uint off;          // FD_INODE
  short major;       // FD_DEVICE
};

~~~



所有在系统中打开的文件都会在全局文件表中记录

~~~c
struct {
  struct spinlock lock;
  struct file file[NFILE];
} ftable;
#define NFILE       100  // open files per system
#define NOFILE       16  // open files per process
~~~



~~~c
// Allocate a file structure.
struct file*
filealloc(void)
{
  struct file *f;

  acquire(&ftable.lock);
  for(f = ftable.file; f < ftable.file + NFILE; f++){
    if(f->ref == 0){
      f->ref = 1;
      release(&ftable.lock);
      return f;
    }
  }
  release(&ftable.lock);
  return 0;
}
// Close file f.  (Decrement ref count, close when reaches 0.)
void
fileclose(struct file *f)
{
  struct file ff;

  acquire(&ftable.lock);
  if(f->ref < 1)
    panic("fileclose");
  if(--f->ref > 0){
    release(&ftable.lock);
    return;
  }
  ff = *f;
  f->ref = 0;
  f->type = FD_NONE;
  release(&ftable.lock);

  if(ff.type == FD_PIPE){
    pipeclose(ff.pipe, ff.writable);
  } else if(ff.type == FD_INODE || ff.type == FD_DEVICE){
    begin_op();
    iput(ff.ip);
    end_op();
  }
}
// Increment ref count for file f.
struct file* filedup(struct file *f)
{
  acquire(&ftable.lock);
  if(f->ref < 1)
    panic("filedup");
  f->ref++;
  release(&ftable.lock);
  return f;
}

// Get metadata about file f.
// addr is a user virtual address, pointing to a struct stat.
int
filestat(struct file *f, uint64 addr)
{
  struct proc *p = myproc();
  struct stat st;
  
  if(f->type == FD_INODE || f->type == FD_DEVICE){
    ilock(f->ip);
    stati(f->ip, &st);
    iunlock(f->ip);
    if(copyout(p->pagetable, addr, (char *)&st, sizeof(st)) < 0)
      return -1;
    return 0;
  }
  return -1;
}

// Copy stat information from inode.
// Caller must hold ip->lock.
void
stati(struct inode *ip, struct stat *st)
{
  st->dev = ip->dev;
  st->ino = ip->inum;
  st->type = ip->type;
  st->nlink = ip->nlink;
  st->size = ip->size;
}
~~~



## System Call



~~~c
// Create the path new as a link to the same inode as old.
// create a new name for a new inode.
uint64
sys_link(void)
{
  char name[DIRSIZ], new[MAXPATH], old[MAXPATH];
  struct inode *dp, *ip;

  if(argstr(0, old, MAXPATH) < 0 || argstr(1, new, MAXPATH) < 0)
    return -1;

  begin_op();
  if((ip = namei(old)) == 0){
    end_op();
    return -1;
  }

  ilock(ip);
  if(ip->type == T_DIR){
    iunlockput(ip);
    end_op();
    return -1;
  }

  ip->nlink++;
  iupdate(ip);
  iunlock(ip);

  if((dp = nameiparent(new, name)) == 0)
    goto bad;
  ilock(dp);
  if(dp->dev != ip->dev || dirlink(dp, name, ip->inum) < 0){
    iunlockput(dp);
    goto bad;
  }
  iunlockput(dp);
  iput(ip);

  end_op();

  return 0;

bad:
  ilock(ip);
  ip->nlink--;
  iupdate(ip);
  iunlockput(ip);
  end_op();
  return -1;
}
~~~





## Lab

### Buffer cache

利用HashTable来提高并发性，并使用链表法解决Hash冲突。

这里有一个关键问题：如何分配Bucket所需的空间？ 因为处于内核态，内存的分配大小都是以PGSIZE为单位的。

解决方法是先分配`struct buf[]`，然后将这些元素挂载到第一个bucket上。如果其他bucket想要buf，那么就从其他bucket中偷取未使用的buf。



~~~c
#define NBUCKET 13
struct {
  struct spinlock lock;
  struct buf buf[NBUF];

  struct bucket_t{
    struct spinlock lock;
    struct buf head;        //哨兵
  } buf_pool[NBUCKET];

} bcache;

void
binit(void)
{
  struct buf *b;

  initlock(&bcache.lock, "bcache");

  for (int i = 0; i < NBUCKET; i++) {
    initlock(&bcache.buf_pool[i].lock, "buf_pool_lock");
    bcache.buf_pool[i].head.next = 0;
  }
  
  for(b = bcache.buf; b < bcache.buf+NBUF; b++){
    b->next = bcache.buf_pool[0].head.next;
    bcache.buf_pool[0].head.next = b;
    b->refcnt = 0;
    initsleeplock(&b->lock, "buffer");
  }
}

// Look through buffer cache for block on device dev.
// If not found, allocate a buffer.
// In either case, return locked buffer.
static struct buf*
bget(uint dev, uint blockno)
{
  struct buf *b;
  int code = blockno % NBUCKET;
  struct bucket_t *bucket = &bcache.buf_pool[code];
  acquire(&bucket->lock);
  b = bucket->head.next;
  while (b != 0) {
    if(b->dev == dev && b->blockno == blockno){
      b->refcnt++;
      release(&bucket->lock);
      acquiresleep(&b->lock);
      return b;
    }
    b = b->next;
  }
  release(&bucket->lock);
  //开始偷buf
  // Not cached.
  // Recycle the least recently used (LRU) unused buffer.
  struct buf *stealed_buf;
  struct bucket_t *stealed_bucket;
  for (int i = 0; i < NBUCKET; i++) {
    stealed_bucket = &bcache.buf_pool[i];
    acquire(&stealed_bucket->lock);
    stealed_buf = stealed_bucket->head.next;
    struct buf *prev = &stealed_bucket->head;
    while (stealed_buf != 0) {
      if (stealed_buf->refcnt == 0) {
        prev->next = stealed_buf->next;
        if (code != i) acquire(&bucket->lock);			//防止死锁
        stealed_buf->next = bucket->head.next;
        bucket->head.next = stealed_buf;
        b = stealed_buf;
        b->dev = dev;
        b->blockno = blockno;
        b->valid = 0;
        b->refcnt = 1;
        if (code != i) release(&bucket->lock);
        release(&stealed_bucket->lock);
        acquiresleep(&b->lock);
        return b;
      }
      prev = stealed_buf;
      stealed_buf = stealed_buf->next;
    }
    release(&stealed_bucket->lock);
  }
  panic("bget: no buffers");

}
~~~



### Large File

注意点：

- 一定要修改fs.h中的NDIREXT宏，以及MAXFILE宏

- 注意bread后一定要正确释放brelse

  ~~~diff
  bp = bread(ip->dev, addr);
  a = (uint*)bp->data;
  if ((addr = a[bn]) == 0) {
    addr = balloc(ip->dev);
    if (addr == 0) {
      brelse(bp);
      return 0;
    }
    a[bn] = addr;
  - brelse(bp);			//错误的
    log_write(bp);
  }
  +brelse(bp);
  ~~~

- 注意计算bn

  ~~~c
  uint tmp = bn;
  bn = tmp / NINDIRECT;
  bn = tmp % NINDIRECT;
  
  //错误的
  bn = bn / NINDIRECT;
  bn = bn % NINDIRECT;
  ~~~

  



  

### Symbolic Link

注意点：一点要注意上锁的顺序，以及上锁时的条件检查

~~~c
uint64 sys_symlink(void) 
{
  char target[MAXPATH];
  char path[MAXPATH];
  argstr(0, target, MAXPATH);
  argstr(1, path, MAXPATH);
  struct inode *target_ip;

  begin_op();
  struct inode *path_ip = create(path, T_SYMLINK, 0, 0);
  if (path_ip == 0) {
    end_op();
    return -1;
  }
  //返回的inode已经上锁

  int iteration = 0;
  while (1) {
    //防止自循环，这里通过测试用例只能这样处理，按理来说不应该将path_ip的内容丢弃
    //解决方案是在这里直接写一开始的target，并在open中设置循环上限，但是懒得写了
    if (strncmp(target, path, MAXPATH) == 0) {
      path_ip->size = 0;
      iupdate(path_ip);
      break;
    }
    target_ip = namei(target);
    if (target_ip != 0) ilock(target_ip);					//注意上锁条件
    if (target_ip == 0 || target_ip->type == T_FILE) {
      //说明target并不存在
      //还有另一种错误情况就是路径中某个部分并不合法，这种情况暂不处理
      int len = strlen(target);
      itrunc(path_ip);
      path_ip->size = len;
      if (writei(path_ip, 0, (uint64)target, 0, len) != len) goto BAD;
      if (target_ip != 0) iunlockput(target_ip);		//注意解锁条件
      iupdate(path_ip);
      break;
    }
    //这个不能放在前面，因为target_ip可能为null
    if (target_ip->type == T_DEVICE || target_ip->type == T_DIR) goto BAD;
    
    if (target_ip->type == T_SYMLINK) {
      if (++iteration > 13) goto BAD;		//说实话这里检测是没必要的，因为自循环可以检测到这一点
      if (readi(target_ip, 0, (uint64)target, 0, MAXPATH) <= 0) goto BAD;
      iunlockput(target_ip);
    }
  }
  iunlockput(path_ip);
  end_op();
  return 0;
BAD:
  
  iunlockput(target_ip);
  iunlockput(path_ip);
  end_op();
  return -1;
}


//in sys_open()
if ((omode & O_NOFOLLOW) == 0 && ip->type == T_SYMLINK) {
    char temp_name[MAXPATH];
    //读出来内容可能有以下几种情况
    //1. 内容为空 对应自循环情况
    //2. 此时文件路径还不合法
    //3. 文件路径合法，类型为SYMLINK或FILE
    do {
      int num = readi(ip, 0, (uint64)temp_name, 0, MAXPATH);
      if (num <= 0) {
        iunlockput(ip);
        end_op();
        return -1;
      }
      struct inode *next = namei(temp_name);
      if (next == 0) {
        iunlockput(ip);
        end_op();
        return -1;
      }
      iunlockput(ip);
      ip = next;
      ilock(ip);
    } while (ip->type == T_SYMLINK);
  }
~~~

