+++
date = "2017-12-26T00:38:00"
draft = false
tags = ["Today I Learned", "Operating System"]
title = "Virtual Memory Hardware"
math = true
outurl = ""
summary = """

"""

[header]
image = ""
caption = ""

+++
{{% toc %}}
## Issues in sharing physical memory
![](/img/post/WX20171226-152306.png)

- Protection
    - A bug in one process can corrupt in another
    - Must somehow prevent process A from trashing B's memory
    - Also prevent A from even observing B's memory (ssh-agent)
- Transparency
    - A process shouldn't require particular physical memory bits
    - Yet processes often require large amounts of contiguous memory (for stack, large data structures, etc.)
- Resource exhaustion
    - Programmers typically assume machine has "enough" memory
    - Sum of sizes of all processes often greater than physical memory

## Virtual memory 
### goals
- Give each program its own *virtual* address space
    - At runtime, **Memory-Management Unit (MMU)** relocates each load/store
    - Application doesn't see physical memory addresses
- Also enforce protection
    - Prevent one app from messing with another's memory
- And allow programs to see more memory than exists
    - Somehow relocate some memory accesses to disk

### Virtual memory advantages
- Can relocate program while running
    - Run partially in memory, partially on disk
- Most of a process's memory may be idle (80/20 rule)
    - ![](/img/post/WX20171226-153927.png)
    - Write idle parts to disk until needed
    - Let other processes use memory of idle part
    - Like CPU virtualization: when process not using CPU, switch (Not using amemory region? switch it to another process)
- **Challenge: VM = extra layer, could be slow**

## Ideas
### Ideal 1: load-time linking
![](/img/post/WX20171226-154928.png)

- *Linker* patches addresses of symbols like printf
- Idea: link when process executed, not at compile time
    - Determine where process will reside in memory
    - Adjust all refernces within program (using addtion)
- Problems?
    - How to enforce protection?
    - How to move once already in memory? (consider data pointers)
    - What if no contiguous free region fits program?

### Ideal 2: base + bound register
![](/img/post/WX20171226-163437.png)

- Two special privileged registers: **base** snf **bound**
- On each load/store/jump:
    - Physical address = virtual address + base
    - Check 0 ≤ vitual address ≤ bound, else trap to kernel
- How to move process in memory?
    - Change base register
- What happens on context switch?
    - OS must re-load base and bound register

## Definition
- Programs load/store to `virtual addresses`
- Actual memory uses `physical addresses`
- VM Hardware is Memory Management Unit (`MMU`)
    - Usually part of CPU
    - Configured through privileged instructions (e.g., load bound reg)
    - Translates from virtual to physical addresses
    - Gives per-process view of memory called `address space`

## Base + bound
### Trade-offs
- Advantages
    - Cheap in terms of hardware: only two registers
    - Cheap in terms of cycles: do add and compare in parallel
    - Examples: Cray-1 used this scheme
- Disadvantages
    - Growing a process is expensive or impossible
    - No way to share code or data (E.g., two copies of bochs, both running pintos)
- One solution: Multiple segments
    - E.g., separate code, stack, data segments
    - Possibly multiple data segments

## Segmentation
![](/img/post/WX20171226-164915.png)

- Let processes have many base/bound regs
    - Address space built from many segments
    - Can share/protect memory at segment granularity
- Must specify segment as part of virtual address

### Segmentation mechanics
![](/img/post/WX20171226-191449.png)

- Each process has a segment table
- Each VA (Virtual Address) indicates a segment and offset:
    - Top bits of addr select segment, low bits select offset (PDP-10)
    - Or segment selected by instruction or operand (means you need wider "far" pointers to specify segment)

### Segmentation example
![](/img/post/WX20171226-192145.png)

- 2-bit segment number (1st digit), 12 bit offset (last 3)
    - Where is 0x0

### Trade-offs
- Advantages
    - Multiple segments per process
    - Allows sharing!
    - Don't need entire process in memory
- Disadvantages
    - Requires translation hardware, which could limit performance
    - Segments not completely transparent to program (e.g., default segment faster or uses shorter instruction)
    - $n$ byte segment needs *n contiguous* bytes of physical memory
    - Makes *fragmentation* a real problem

### Fragmentation
- Fragmentation => Inability to use free memory
- Over time:
    - Variable-sized pices = many small holes (external fragmentation)
    - Fixed-sized pieces = no external holes, but force internal waste (internal fragmentation)
![](/img/post/WX20171226-213655.png)

### Alternatives to hardware MMU
- Language-level protection (Java)
    - Single address space for different modules
    - Language enforces isolation
    - Singularity OS does this
- Software fault isolation
    - Instrument compiler output
    - Checks before ever ystore operation prevents modules from trashing each other
    - Google Native Client does this

## Paging
- Divede memory up into small *pages*
- Map virtual pages to physical pages
    - Each process has separate mapping
- Allow OS to gain control on certain operation
    - Read-only pages trap to OS on write
    - Invalid pages trap to OS on read or write
    - OS can change mapping and resume application
- Other features sometimes found:
    - Hardware can set "accessed" and "dirty" bits
    - Control page execute permission separately from read/write
    - Control caching or memory consistency of page

### Trade-offs
- Eliminates external fragmentation
- Simplifies allocation, free, and backing storage (swap)
- Average internal fragmentation of .5 pages per "segment"

### Simplified allocation
- Allocate any physical page to any process
- Can store idle virtual pages on disk

### Paging data structures
- Pages are fixed size, e.g., 4K
    - Least significant 12 ($log_{2}4K$) bits of address are page offset
    - Most significant bits are *page number*
- Each process has a page table
    - Maps *virtual page numbers (VPNs)* to *physical page numbers (PPNS)*
    - Also includes bits for protection, validity, etc.
- On memory access: Translate VPN to PPN, then add offset
![](/img/post/WX20171227-094654.png)

### Example: Paging on PDP-11
- 64K virtual memory, 8K pages
    - Separate address space for instructions & data
    - I.e., can't read your own instructions with a load
- Entire page table stored in registers
    - 8 Instruction page translation
    - 8 Data page translation
- Swap 16 machine registers on each context switch

