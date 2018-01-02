+++
date = "2017-12-27T00:38:00"
draft = false
tags = ["Today I Learned", "Operating System"]
title = "[OS Notes] Virtual Memory OS"
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

### What to fetch
- Bring in page that caused page fault
- Pre-fetch surrounding pages?
    - Reading two disk blocks approximately as fast as reading one
    - As long as no track/head switch, seek time dominates
    - If application exhibits spacial locality, then big win to store and read multiple contiguous pages
- Also pre-zero unused pages in idle loop
    - Need 0-filled pages for stack, heap, anonymously mmapped memory
    - Zeroing them only on demand is slower
    - Hence, many OSes zero freed pages while CPU is idle

### Selecting physical pages
- May need to eject some pages
    - More on eviction policy in two slides
- May also have a choice of physical pages
- Direct-mapped physical caches
    - Virtual -> Physical mapping can affect performance
    - In old days: Physical adddredd A conflicts with $kC+A$ (where k is any integer, C is cache size)
    - Applications can conflict with each other or themselves
    - Scientific application benefit if consecutive virtual pages do not confilict in the cache
    - Many other applications do better with random mapping
    - There days: CPUs more sophisticated than kC + A

### Superpages
- How should OS make use of "large" mappings
    - x86 has 2/4 MB pages that might be useful
    - Alpha has even more choices: 8KB, 64KB, 512KB, 4MB
- Sometimes more pages in L2 cache than TLB entries
    - Don't want costly TLB misses going to main memory
- Or have two-level TLBs
    - Want to maximize hit rate in faster L1 TLB
- OS can transparently support superpages
    - "reserve" appropriate physical pages if possible
    - Promote contiguous pages to superpages
    - Does complicate evicting (esp. dirty pages) - demote

## Eviction policies
### Starw man: FIFO eviction
- Evict oldest fetched page in system
- Example-reference string 1,2,3,4,1,2,5,1,2,3,4,5
- 3 physical pages: 9 page faults


## Thrashing

## Details of paging

## The user-level perspective

## Case study: 4.4 BSD