# Milestone 3

# Relevant Commits

Here are the contributions to this milestone.

https://github.com/JefferyLim/black-parrot/pull/4

https://github.com/JefferyLim/black-parrot-sim/blob/ec513/sw/virtual_memory.c


# Milestone 3

We've consolidated the trace files into a single trace file. [Lines 210 and down](https://github.com/JefferyLim/black-parrot/blob/8892cf1caa29051b54ed5e7c003a3cc70753d144/bp_top/test/common/bp_nonsynth_uarch_tracer.sv#L210) cover the new tracer file, named `uarch_0.trace`. 

After reviewing the microarchitecture and traces, we ultimately were unable to do any type of speculative execution, causing any microarchitectural leaks of privilege data. 

Ultimately, this is a single issue design. We find out if we've mispredicted a branch one cycle into the calculator pipeline, and thus, a memory request will be unable to send data out. Note, we know that the instruction cache is able to speculatively fetch, but we did not include it in our list of traces.=

## Tracers

We've combined the tracers into a single file in order to better understand the flow. Originally, we were going to have separate tracers and a python script to combine them, but decided to write the logic in the tracer file.

One other thing we did not discuss in thel ast milestone are the other tracer files that already exist. We can leverage some of these traces in addition to the ones we've created. We reviewed some of them to determine how useful they are.

* CMT_TRACE_P - prints each committed instruction along with register modifications
  * Very useful, prints the architectural changes
* DRAM_TRACE_P - prints each dram access
  * Very useful, prints the architectural changes
* NPC_TRACE_P - prints each (speculative) PC executed by the BE
  * Would be useful if it existed
* DCACHE_TRACE_P - prints each load/store
  * Very useful, can be used to compare the memory transactions in our tracer file
* VM_TRACE_P - prints each TLB fill
  * Very useful, can be used to compare the memory transactions in our tracer file
* PC_PROFILE_P - prints pc information
  * Shows frequency of PC that were ran. Not useful for us
* BRANCH_PROFILE_P - prints branch information
  * Not very useful
* CORE_PROFILE_P - prints a cycle-accurate stall trace
  * Useful in understanding what causes stalls 

### uarch traces
- A cycle counter to synchronize with other tracers
- Track issue, and dispatch packets (packets that enter the BE's scheduler and leave the issue queue)
- Track when instructions are squashed (poison_isd_i)
- Track all memory instructions
- Track committed packets that cause an exception 
- Track page faults that occur (from the memory pipeline)
- Track priv faults that occur (from the page table walker)
- Track whenever page table walks occur
- Track when pipelines are flushed (commit_pkt.npc_w_v)
- Track privilege modes (m, s, u)

### DCache traces
- cache_req_v_o: indicates a cache miss or uncached request is being sent out to the next level - LCE
- cache_req_yumi_i: signals that the LCE has accepted the outgoing request from cache_req_v_o
- data_mem_pkt_v_i: signals an incoming request from the LCE/engine, typically for filling data on a miss or handling writebacks/evictions.
- data_mem_pkt_yumi_o: indicates the dcache has accepted the incoming data memory request from data_mem_pkt_v_i.

We were unable to determine a great way of utilizing the following signals. We hope to see if we can do anything in the next day.
- cache_req_metadata_v_o: valid signal for metadata (like replacement way, dirty status) being sent out on a miss.  potentially interesting for info about the cache's internal state 
- stat_mem_pkt_v_i: signals an incoming request from the LCE/engine to read or modify the status memory (LRU, dirty bits). (external actions manipulating LRU might be found here)
- stat_mem_pkt_yumi_o: Indicates the dcache has accepted the incoming status memory request from stat_mem_pkt_v_i. Timing reflects readiness to handle state updates
- wbuf_v_li indicates a store hit is being written into the write buffer. maybe look for delays/potential bypass scenarios, affecting later load/store timing
- wbuf_v_lo: indicates a buffered store is ready to be written from the write buffer into the actual data memory
- wbuf_yumi_li : marks write buffer entry wbuf_v_lo completed


## Code

Last milestone, we created a [virtual_memory.c](https://github.com/JefferyLim/black-parrot-sim/blob/ec513/sw/virtual_memory.c) program that maps pages and sets the privilege level (User or Supervisor level). We've cleaned it up and updated it to include some other cases. The flow of software we tested was:

1. (Machine mode) Set up memory spaces and enable virtual memory. We allocate an address (0x8040_0000) to be user accessible.
2. (Machine mode) Write some data (0x8badf00ddeadbeef) into this address.
3. (Machine mode) Launch user test program
4. (User mode) Successfully reads from the address (0x8040_0000)
5. (User mode) Launches a Syscall
6. (Supervisor mode) Update PTE
6a. (Supervisor mode) Flushes the TLB
6b. (Supervisor mode) Does NOT flush the TLB

From here, the code behavior diverges. In the case of flushing the TLB

7. (User mode) Attempts to read again from the address, only to be blocked

In the case of not flushing the TLB

7. (User mode) Successfully reads from memory, and is able to continue writing to memory location.

## Interesting Discoveries

So far, we have been unable to generate any trace where we can see microarchitectural changes due to branch mispredictions. 

However, we have found an interesting behavior due to a software bug, where not flushing the TLB causes the user to continue reading from memory that is privileged. This is considered more of a software bug than hardware, as we read that it is the software that is responsible for flushing the TLB. We discovered this due to the Dromajo emulator and the verilator simulation showing different behavior. Dromajo shows that a privilege fault should occur, but the verilator simulation does not show that. We will be going over this example in our presentation.  

## Example Traces

Here is a trace of the steps that get taken during a fault. In purple, we've highlighted the dispatch of a read instruction by the user for addres 0x8040_0000. Note that this trace is one of the earlier traces. We're going through and cleaning up the trace file to be a bit cleaner.

Because the memory pipeline can't find the TLB entry, it is forced to do a page table walk (highlighted in blue). It goes through the 3 page tables, where it eventually retrieves the PTE highlighted in orange. This entry 0x201000cf has the U bit deasserted. We see that the dispatch of the same instruction goes through, but the ocmmit instruction has an exception, thus, a pipe flush occurs, and no other requests leave/enter the memory pipeline.

![image](https://github.com/user-attachments/assets/7c8d79cc-f7fc-4021-9c89-5336ba3bfbc9)


### Cleaned up Trace
Here is the assembly code:

The yellow represents the address of the privileged location. 

In orange, we update the PTE so we remove user permissions, but do not flush the TLB (`s_incorrect_flush`)

As a result, all the accesses after, until the `s_repage_secret` (green) function, will still work, unless the TLB gets updated with the correct PTE.

![image](https://github.com/user-attachments/assets/3eeba606-f5a3-41b5-a317-4fa12bbdc867)

Here is the cleaned up tracer:
![image](https://github.com/user-attachments/assets/7576b9b3-a1af-4d50-907b-61556112e398)


## Tracking the microarchitectural states

In general, we believe that tracking the requests and returns from the memory pipeline should give us enough information in THIS architecture to know if there are any transient information left over. If we were to have a more complex, or out-of-order design, we would have to better track all the information. 
