# Page-faultexceptions

RISC-V has three different kinds of page fault: 

- **load page faults** (when a load instruction cannot translate its virtual address)
- **store page faults** (when a store instruction cannot translate its virtual address)
-  **instruction page faults** (when the address for an instruction doesn’t translate)

The value in the `scause` register indicates the type of the page fault

the `stval` register contains the address that couldn’t be translated.



## COW fork

Xv6 implements fork with uvmcopy (kernel/vm.c:306), which allocates physical memory for the child and copies the parent’s memory into it.

The basic plan in COW fork is for the parent and child to initially share all physical pages, but for each to map them read-only (with the PTE_W flag clear). 



## Lazy Allocation

## Demand Paging

## Swap Area

## Memory Map Files



## Lab COW



