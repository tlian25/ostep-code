# Memory Virtualization

## Address Space
- running program's view of memory in the system
- contains all of the memory state of the running program
- every program memory address is virtual; only the OS knows where the physical memory is

#### Isolation
- isolate processes from each other
- ensures running programs cannot affect the operation of the underlying OS
- some OS's wall off pieces of the OS from other pieces of the OS

#### Transparency
- OS should implement virtual memory in a way that is invisible to the running program


#### Efficiency
- both in terms of time and space


#### Protection
- when one process performs a load, a store, or an instruction fetch, it should not be able to access or affect in any way the memory contents of any other process or the OS itself


## Address Translation

**Hardware-based address translation**
- hardware transforms each memory access
- change the virtual address provided by the instruction to a physical address where the desired information is actually located


#### Dynamic (Hardware-Based) Relocation
- two hardward registers within each CPU
- base register - set to start of physical memory address location
- bounds register - ensures that addresses are within confines of the address space
- physical address = virtual address + base
- relocation of the address happens at runtime
- we can move address spaces even after the process has started running
- if process generates a virtual address that is greater than the bounds or one that is negative, the CPU will raise an exception and terminate the process
- base and bounds registers are hardware structures kept on the chip
- hardware provides special priviledged instructions to modify registers
- CPU must be able to generate exceptions in situations where a user program tries to access memory illegally
- OS must take action when a process is created, finding space for its address space in memory
- OS must reclaim memory when process is terminated or exited
- OS must save and restore the base-bounds pair in some per-process structure when it switches between processes
- OS must provide exception handlers, or functions to be called; OS installs these handlers at boot time


**Memory Management Unit (MMU)**
- part of the process that helps with address translation of the memory

**Static Relocation**
- a piece of software known as the loader takes an executable that is about to the run and rewrites its address to the desired offset in physical memory
- does not provide protection
- difficult to later relocated an address space to another location once placed

**Free List**
- OS must track which parts of free memory are not in use
- simplest data structure is a list of the ranges of the physical memory which are not in use



## Segmentation
- need to support a large address space with a lot of free space between the stack and the heap
- instead of having just one base/bounds pair in MMU, have a base/bounds pair per logical segment of the address space
- segment is a contiguous portion of the address space of a particular length
- segmentation allows  OS to place each of code, stack, heap in different parts of physical memory
- hardware uses segment registers during translation
- problem of compacting physical memory - expensive in memory and CPU processing


#### Code Sharing
- sometimes useful to share certain memory segments between address spaces
- basic support from hardware adds a few bits per segment, indicating whether or not a program can read or write a segment, or execute code that lies within the segment


#### Fine-grained Segmentation
- allow for address spaces to consist of a large number of smaller segments
- segment table to support creation of a large number of segments


## Free-Space Management
- external fragmentation - free space gets chopped into little pieces of different sizes
- subsequent requests may fail because there is no single contiguous space that can satisfy the request, even though the total amount of free space exceeds the size of the request
- internal fragmentation - if an allocator hands out chunks of memory bigger than requested, any unasked for space in a chunk is considered internal fragmentation



#### Tracking the size of allocated regions
- header block - extra information just before the handed-out chunk of memory, contains the size of the allocated region and additional pointers to speed up deallocation, magic number for integrity checking



#### Best Fit
1. search through free list and find chunks of free memory that are as big or bigger than requested size
2. Return the one that is the smallest in the group of candidates
- naive implementations pay a heavy performance penalty when performing an exhaustive search

#### Worst Fit
- find largest chunk and return the requested amount
- tries to leave big chunks free instead of lots of small chunks that can arise from best-fit
- full search of free space still required
- can lead to excess fragmentation while still having high overheads



#### First Fit
- find the first block that is big enough
- advantage of speed
- can pollute the beginning fo the free list with small bojects
- how the allocator manages the free list's order becomes an issue
- one approach is address-based ordering - keep list ordered by address of free space


#### Next Fit
- keeps an extra pointer to location within the list where one was looking last
- spread the searches for free space throughout the list more uniformly
- avoid splintering the beginning of the list



#### Segregated Lists
- if a particular application has a few popular-sized requests, keep a separate list just to manage ojbects of that size
- all other requests are forwarded to a more general memory allocator
- how much memory should one dedicate to the pool of specialized memory?



**Slab Allocator**
- when kernel boots up, it allocates a number of object caches for kernel objects that are likely to be requested frequently
- locks, file-system inodes, etc
- when a given cache is running low on free space, it requests some slabs of memory from a more general memory allocator
- when reference counts of the objects within a given slab all go to zero, the general allocator can reclaim them from the specialized allocator
- keeps free objects on the lsits in a pre-initialized state as initialization and destruction is costly


**Binary Buddy Allocator**
- free memory conceptually thought of as one big space of size 2^N
- when request is made, search for free space recursively divides free space by two until a block that is big enough is found
- can suffer from internal fragmentation as we only give out power-of-two sized blocks
- when returning block to free list, allocator checks whether buddy is free, if so, it coalesces the two blocks into a larger block recursively 




## Paging

- flexibility - system will be able to support the abstraction of an address space effectively, regardless of how a process uses the address space
- simplicity - fixed sized chunks
- for every memory reference, paging requires us to perform one extra memory reference in order to first fetch the translation from the page table


#### Page Table
- a per-process data structure to record where each virtual page of the address space is placed in physical memory
- store address translations for each of the virtual pages of the address space
- store page table in memory somewhere because it can get very large


## Translation Lookaside Buffer (TLB)
- part of chip's MMU
- hardware cache of popular virtual-to-physical address translations
- hardware first checks TLB to see if the desired translation is held
- if so, the translation is performed quickly without having to consult the page table
- misses can be handled by software or hardware
- benefits of software are flexibility and simplicity
- context switches - OS must be careful to ensure that the next process does not accidentally use translations from some previously run process
- ASID - address space identifier, added field used to share TLB across context switches
- TLB access can easily become a bottleneck in the CPU pipeline


#### TLB Basic Algorithm
1. extract the virtual page number from the virtual address and check if TLB holds translation for this VPN
2a. If yes, we have a TLB hit and can extract the page frame number (PFN) from the relevant TLB entry, concat that onto the offeset from the original virtual address, and form the desired physical address
2b. If no, we have a TLB miss and must look in the page table to find the translation and write back to TLB


## Smaller Tables
- tradeoff of internal fragmentation


#### Combining paging and segments
- one page table per logical segment
- three page tables - code, heap, stack


#### Multi-level page tables
- turns linear page table into something like a tree
- page directory to store where a page of the page table table is


## Swap Space
- reserve some space on disk for moving pages back and forth
- OS will need to remember the disk address of a given page
- present bit - 1 means page is present in physical memory, 0 means on disk somewhere
- page fault - accessing a page that is not in physical memory
- while I/O is in flight, the process will be in the blocked state
- OS will be free to run other ready processes while the page fault is being serviced


#### Replacement
- High watermark and low watermark
- when OS notices that there are fewer than LW pages available, a background thread runs to free memory and evict pages until there are HW pages available
- called swap daemon or page daemon


**Average Memory Access Time (AMAT)**
- AMAT = T_M + (P_miss * T_D)
- T_M = cost of accessing memory
- T_D = cost of accessing disk
- P_miss = probability of cache miss

**Types of Cache Misses**
- Compulsory - cold-start miss
- Capacity - ran out of space and had to evict
- Conflict - limits on where an item can be placed in hardware cache

**Belady's Anomaly**
- in general we expect cache hit rate to increase when cache gets larger
- but with FIFO it gets worse

#### Optimal Replacement Policy
- leads to fewest number of misses overall
- replace the page that will be acccessed furthest in the future
- ideal but impossible to implement practically


#### Practical Policies
- FIFO - first in first out
- LRU - least recently used
- LFU - least frequently used
- Random


**Approximating LRU**
- scan resistance - try to avoid worst-case behavior of LRU
- use bit - set to 1 by hardware when a page is referenced
- clock algorithm 
    - loops through pages until finds one with use bit 0
    - for all pages checked with use bit 1, clear bit to 0
- dirty pages - consider whether a page has been modified or not while in memory
    - if page modified, it must be written back to disk to evict, which is expensive
    - prefer to evict clean pages over dirty pages
    - needs a dirty bit to support


#### Page selection policy
- OS has to decide when to bring a page into memory
- for most page, OS uses demand paging - on demand loading
- OS could guess that a page is about to be used - prefetch


#### Thrashing
- when memory is oversubscribed
- memory demands of the set of running processes simply exeeds the available physical memory
- system will be constantly paging
- mitigation
    - admission control - reduce set of working processes
    - out-of-memory kill - daemon chooses a memory-intensive process and kills it
