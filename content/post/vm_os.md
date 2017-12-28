+++
date = "2017-12-27T00:38:00"
draft = false
tags = ["Today I Learned", "Operating System"]
title = "Virtual Memory OS"
math = true
outurl = ""
summary = """

"""

[header]
image = ""
caption = ""

+++
{{% toc %}}

## Paging
![](/img/post/WX20171227-110059.png)

- Use disk to simulate larger virtual than physical memory

### Working set model
- Disk much, much slower than memory
    - Goal: run at memory speed, not disk speed
- 80/20 rule: 20% of memory gets 80% of memory accesses
    - Keep the hot 20% in memory
    - Keep the cold 80% on disk

### Paging challenge
- How to resume a process after a fault?
    - Need to save state and resume
    - Process might have been in the middle of an instruction!
- What to fetch from disk?
    - Just needed page or more?
- What to eject?
    - How to allocate physical pages amongst processes?
    - Wchich of a particular process's pages to keep in memory?

### Re-starting instructions
- Hardware provides kernel with information about page fault
    - Faulting virtual address (In `%cr2` reg on x86--may see it if you midify Pintos `page_fault` and use `fault_addr`)
    - Address of instruction that caused fault
    - Was the access a read or write? Was it an instruction fetch? Was it caused by user accss to kernel-only memory?
- Hardware mush allow resuming after a fault
- Idempotent instructions are easy
    - E.g., simple load or store instruction can be restarted
    - Just re-execute any instruction that only accesses one address
- Complex instructions must be re-started, too
    - E.g., x86 move string instructions
    - Specify src, dst, count in `%esi, %edi, %ecx` registers
    - On fault, registers adjusted to resume where move left off
        

## Eviction policies

## Thrashing

## Details of paging

## The user-level perspective

## Case study: 4.4 BSD