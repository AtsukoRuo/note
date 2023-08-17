# Paper

[TOC]

> 本Note是关于一些经典的Paper以及课上教授对这些Paper的讲解（精简）。实际上，我对于这些内容的理解处于如懂的状态:)  ，一定会有大量理解错误的地方。在以后实践中积累大量经验之后，再慢慢纠正这些错误吧。学习也正是如此，在不断的否定中，获得对于事物更加正确、全面的认识。



## Journaling the Linux ext2fs Filesystem

实际上，`ext3`就是在`ext2`的基础上增加了Journal特性而已。

> 注：logging与journal是同义的

### Introduce

当我们讨论文件系统**可靠性（Reliability）**时,有几个问题至关重要：

- **Preservation**： data which was stable on disk before the crash should never ever be damaged。 也就是说，恢复过程不应该破坏已经安全存储的数据
- **Predictability**：崩溃发生的原因应该是可预测的，以便我们可以可靠地进行恢复。
- **Atomicity**：恢复过程必须是原子性的，要么文件操作全部执行完毕，要么一个都不执行



可预测性要求文件系统必须以可预测的方式来执行写磁盘操作。

There are many ways of achieving this ordering between disk writes：

- **synchronous metadata update**：wait for the first writes to complete before submitting the next ones to the device driver。但是这种方案会严重降低性能
- **deferred ordered write**：为了解决上述方案的性能问题，我们可以maintain an ordering between the disk buffers in memory, and to ensure that when we do eventually go to write back the data, we never write a block until all of its predecessors are safely on disk
- **soft update**： 但是上述方案有循环依赖的问题，maintain an ordering between the disk buffers in memory, and to ensure that when we do eventually go to write back the data, we never write a block until all of its predecessors are safely on disk



但是这些方案在恢复过程时，需要扫描所有的block，然后确定出错的原因来执行相应恢复操作。其中扫描过程会花费大量的时间。为了解决这个问题，我们在现有基础上，增加了journal机制。

### Basic Idea & Implementation

journal的核心概念就是**transaction**。相比较数据库的transaction，filesystem的有以下不同点：

- **no transaction abort**
- **relatively short-lived**

因此，在实现事务机制时，无需为每个系统调用都分配一个事务，而是只实现一个**single system-wide compound transaction**。在事务提交之前，任何系统调用都可以进入该事务。



在事务提交时，新的系统调用就会被阻塞。为了解决这个问题，实际上，内核在一个background kernel thread中维护多个transaction，但只有一个transaction处于开启状态，其他的transaction要么在commit中，要么已经close了。

commit的过程如下：

- block new system call
- wait for outstanding systemcall calling end_trans()
- open a new transaction to receive new system call
- write DESC block
- write data blocks to log
- write commit block, this is commit point
- write to home locations
- close



journal的布局如下所示：

![1692019983056-screenshot](C:\Users\AtsukoRuo\Desktop\note\操作系统\assets\1692019983056-screenshot.png)



注意、commit block以及descriptor block都在开头以一个magic number（signature）标识，如果一个数据块恰巧与此冲突，那么在write to log时会用0来代替，并且在descriptor block中记录。在write to home location时，再复原这一magic number。

### Performance technique

- async system call： 系统调用无需等待块写入到journal，而是直接返回。这说明即使系统调用成功返回，也不能说明block已经安全地写回到磁盘中。这对于text editor、database来说是很致命的问题。因此系统提供了fsync系统调用来开启同步机制。异步调用可以充分利用IO concurrent
- batching more system call into single transaction，这由以下好处：
  - write absorb，减少写的次数
  - 尽可能地在一次提供中写入多个block，这有利于disk scheduling
- concurrency
  - Systemcall in concurrency
  - only one open transaction can receive system calls but the other are writing blocks in parallel



同步问题在其他层次（例如inode层、buffer层）来解决

