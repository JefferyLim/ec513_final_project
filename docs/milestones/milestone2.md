# Milestone 2

# Relevant Commits

Here are the contributions to this milestone.

https://github.com/JefferyLim/black-parrot/pull/2

https://github.com/JefferyLim/black-parrot-sim/blob/ec513/sw/virtual_memory.c


# Outcomes

We've been able to successfully cause exceptions to occur due to a user program attempting to read memory from an unprivileged area of memory. We should be able to track:

1. Memory accesses and their privilege levels
2. Detection of unprivileged accesses

We do not have a way of determining if an address is a privileged location unless we already know ahead of time.

## Code

We've created a [virtual_memory.c](https://github.com/JefferyLim/black-parrot-sim/blob/ec513/sw/virtual_memory.c) program that maps pages and sets the privilege level (User or Supervisor level). It's very simple for now, but has proven to work for our needs.

## Understanding of Faults

We've parsed the micro-architecture as follows:

1. A virtual address goes through the memory pipeline's MMU -> TLB
2. If unsuccessful, this is a dcache miss, and a page table walk must be executed
3. A page table walk is done at the end of the pipeline, so it occurs when the commit packet reaches the scheduler
4. Here, the system stalls until the page table walker is finished
5. The privileges are checked, and if they do not pass, an exception occurs
6. The pipeline is pushed through with an exception in the dispatch queue
7. The system pipeline's CSR will trigger an exception, causing a change in PC to the correct handler
