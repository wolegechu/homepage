+++
date = "2018-01-01T00:38:00"
draft = false
tags = ["Today I Learned", "Operating System"]
title = "[OS Notes] Synchronization (Part I)"
math = true
outurl = ""
summary = """

"""

[header]
image = ""
caption = ""

+++
{{% toc %}}

##  Cache coherence (the hardware view)
- Important memory system properties
    - Coherence - concerns accesses to a single memory location
        - Must obey program order if access from only CPU
        - There is a total order on all updates
        - There is bounded latency before everyone sees a write
    - Consistency - concerns ordering across memory locations
        - Even with coherence, different CPUs can see the same write happen at different times
        - Sequential consistency is what matches our intuition (As if instructions from all CPUs interleaved on one CPU)
        - Many architectures offer weaker consistency can still be sufficient to implement thread API

### Multicore Caches
- Performance requires caches
    - Divided into chuncks of bytes called lines (e.g., 64 bytes)
    - Caches create an opportunity for cores to disagree about memory
- Bus-based approaches
    - "Snoopy" protocols, each CPU listens to memory bus
    - Use write-through and invalidate when you see a write bits
    - Bus-based schemes limit scalability
- Modern CPUs use networks (e.g., hypertransport, QPI)
    - CPUs pass each other messages about cache lines

### MESI coherence protocol
- Modified
    - One cache has a valid copy
    - That copy is dirty (needs to be written back to memory)
    - Must invalidate all copies in other caches before entering this state
- Exclusive
    - Same as Modified except the cache copy is clean
- Shared
    - One or more caches and memory have a valid copy
- Invalid
    - Doesn't contain any data
- Owned (for enhanced "MOESI" protocal)
    - Memory may contain stale value of data (like Modified state)
    - But have to broadcast modifications (sort of like Shared state)
    - Can have both one owned and mutiple shared copies of cache line

### Core and Bus Actions
- Core
    - Read
    - Write
    - Evict (modified? must write back)
- Bus
    - Read: without intent to modify, data can come from memory or another cache
    - Read-exclusive: with intent to modify, must invalidate all other cache copies
    - Writeback: contens put on bus and memory is updated

### cc-NUMA
- Old machines used *dance hall* achitectures
    - Any CPU can "dance with" any memory equally
- An alternative: Non-Uniform Memory Access
    - Each CPU has fast access to some "close" memory
    - Slower to access memory that is farther away
    - Use a directory to keep track of who is caching what
- Originally for esoteric machines with many CPUs
    - But AMD and then Intel integrated memory controller into CPU
    - Faster to access memory controlled by the local socket (or even local die in a multi-chip module)
- cc-NUMA = cache-coherent non-uniform memory access
    - Rarely see non-cache-coherent NUMA

### Real World Coherence Costs
- If another core in same socket holds line in modified state:
    - load: 109 cycles