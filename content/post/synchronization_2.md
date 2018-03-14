+++
date = "2018-01-01T00:38:00"
draft = false
tags = ["Today I Learned", "Operating System"]
title = "[OS Notes] Synchronization (Part II)"
math = true
outurl = ""
summary = """

"""

[header]
image = ""
caption = ""

+++
{{% toc %}}

## Overview of previous and current lectures
- Locks create serial code
    - Serial code gets no speedup from multiprocessor
- Test-and-set spinlock has additional disadvantages
    - Lots of traffic over memory bus
    - Not fair on NUMA machines
- Idea 1: Avoid spinlocks
    - We saw lock-free algorithms last lecture
    - Discussing RCU very quickly last time
- Idea 2: Design better spinlocks
    - Less memory traffic, better fairness
- Idea 3: Hardware turns coarse-into fine-grained locks!
    - While also reducing memory traffic for lock in common case

## RCU (Read-copy update)
- Some data is read way more often than written
    - Routing tables consulted for each forwarded packet
    - Data maps in system with 100+ disks (updated on disk failure)
- Optimize for the common case of reading without lock
    - E.g., global variable: `routing_table *rt;`
    - Call `lookup (rt, route);` with no lock
- Update by making copy, swapping pointer
    ```C
    routing_table *newrt = copy_routing_table (rt);
    update_routing_table (newrt);
    atomic_thread_fence (memory_order_release);
    rt = newrt;
    ```

### Is RCU really safe?
- Consider the use of global _rt_ with no fences: `loolup (rt, route);`
- Could a CPU read new pointer but then old contents of *rt?
- Yes on alpha, No on all other existing architectures
- We are saved by *dependency ordering* in hardware
    - Instruction B depends on A if B uses result of A
    - Non-alpha CPUs won't re-order dependent instructions
    - If wwriter uses release fence, safe to load pointer then just use it
- This is the point of `memory_order_consume`
    - Should be quivalent to acquire barrier on alpha
    - But should compile to nothing (be free) on other machines
    - Active area of discussion for C++ committee

### Preemptible kernels
- Recall *kernel process context*
    - When CPU in kernel mode but excuting on behalf of a process (e.h., might be in system call or page fault handler)
    - As opposed to interrupt handlers or context switch code
- A *preemptible kernel* can preempt process context code
    - Take a CPU core away from kernel process context code between any two instructions
    - Give the same CPU core to kernel code for a different process
- Don't confuse with
    - Interrupt handlers can always preempt process context code
    - Preemptive threads (always have for multicore)
    - Process context code running concurrently on other CPU cores
- Sometimes want or need to disable preemption
    - E.g., might help performance while holding a spinlock

### Garbage collection
- When can you free memory of old routing table?
    - When you are guaranteed no one is using it —— how to determine
- Definitions:
    - *temporary variable* - short-used (e.g., local) variable
    - *permanent variable* - long lived data (e.g., global *rt* pointer)
    - *quiescent state* - when all a thread's temporary variables dead
    - *quiescent period* - time during which every thread has been in quiesceent state at least once
- Free old copy of updated data after quiescent period
    - How to determine when quiescent period has gone by?
    - E.g., keep count of syscalls/context on each CPU
    - Can't hold a pointer across context switch or user mode
    - Must disable preemption while consuming RCU data structure

## Improving spinlock performance
### Useful macros
- Atomic compare and swap: CAS (mem, old, new)
    - In C11: ```atomic_compare_exchange_strong```
    - On x86: `cmpxchg` instruction provides this (with *lock* prefix)
    - ``` if *mem == old, then swap *mem<->new and return true, else false ```
- Atomic swap: XCHG (mem, new)
    - C11 `atomic_exchange`, can implement with xchg on x86
    - Atomically exchanges `*mem<->new`
- Atomic fetch and add: FADD (mem, val)
    - C11 `atomic_fetch_add`, can implement with *lock* add on x86
    Atomically sets `*mem += val` and returns *old* value of `*mem`
- Atomic fetch and subtract: FSUB (mem, val)
- Note all atomics return previous value (like x++, not ++x)
- All behave like sequentially consistent fences, too
    - Unlike `_explicit` versions, which take a `memory_order` argument

### MCS lock
- Idea 2: Build a better spinlock
- Lick designed by Mellor-Crummey and Scott
     - Goal: reduce bus traffic on cc machines, improve fairness
- Each CPU has a *qnode* structure in local memory
```C
typedef struct qnode {
    _Atomic (struct qnode *) next;
    atomic_bool locked;
} qnode;
```
- A lock is a *qnode* pointer: `typedef _Atomic (qnode *) lock;`
    - Construct lisk of CPUs holding or waiting for lock
    - *lock* itself points to taiil of list list
- While waiting, spin on your local *locked* flag

### MCS Acquire
```C
acquire (lock *L, qnode*I){
    I->next = NULL;
    qnode *preprocessor = I;
    XCHG (*L, predecessor);
    if (predecessor != NULL){
        I->locked = true
        predecessor->next = I;;
        while (I->locked);
    }
}
```
![](/img/post/QQ20180123-033259@2x.pnd)

- If unlocked, L is NULL
- If locked, no waiters, L is owner's qnode
- If waiters, *L is tail of waiter list

### MCS Release with CAS
```C
release (lock* L, qnode * I) {
    if (!I->next)
        if (CAS (*L, I, NULL))
            return;
    while (!I->nxt);
    I->next->locked false
}
```

