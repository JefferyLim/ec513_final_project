# Milestone 2

# Relevant Commits

Here are the contributions to this milestone.

https://github.com/JefferyLim/black-parrot/pull/2

https://github.com/JefferyLim/black-parrot/pull/3

https://github.com/JefferyLim/black-parrot-sim/blob/ec513/sw/virtual_memory.c


# Outcomes

We've been able to successfully cause exceptions to occur due to a user program attempting to read memory from an unprivileged area of memory. We should be able to track:

1. Architectural memory reads and their privilege levels
2. Detection of unprivileged accesses
3. Track what read information gets retreived from cache

We do not have a way of determining if an address is a privileged location unless we already know ahead of time.

## Tracers

### uarch_tracer
- A cycle counter to synchronize with other tracers
- Track issue, and dispatch packets (packets that enter the BE's scheduler and leave the issue queue)
- Track when instructions are squashed (poison_isd_i, commit_pkt.npc_w_v)
- Track committed packets that cause an exception 
- Track page faults that occur (from the memory pipeline)
- Track priv faults that occur (from the page table walker)
- Track privilege modes (m, s, u)

### cache_tracer
### Data Cache:
- cache_req_v_o: indicates a cache miss or uncached request is being sent out to the next level - LCE
- cache_req_yumi_i: signals that the LCE has accepted the outgoing request from cache_req_v_o
- cache_req_metadata_v_o: valid signal for metadata (like replacement way, dirty status) being sent out on a miss.  potentially interesting for info about the cache's internal state 
- data_mem_pkt_v_i: signals an incoming request from the LCE/engine, typically for filling data on a miss or handling writebacks/evictions.
- data_mem_pkt_yumi_o: indicates the dcache has accepted the incoming data memory request from data_mem_pkt_v_i.
- stat_mem_pkt_v_i: signals an incoming request from the LCE/engine to read or modify the status memory (LRU, dirty bits). (external actions manipulating LRU might be found here)
- stat_mem_pkt_yumi_o: Indicates the dcache has accepted the incoming status memory request from stat_mem_pkt_v_i. Timing reflects readiness to handle state updates
- wbuf_v_li indicates a store hit is being written into the write buffer. maybe look for delays/potential bypass scenarios, affecting later load/store timing
- wbuf_v_lo: indicates a buffered store is ready to be written from the write buffer into the actual data memory
- wbuf_yumi_li : marks write buffer entry wbuf_v_lo completed

### Icache:
- icache_pkt_i: carries the incoming fetch request, including the virtual address (.vaddr) and crucially the speculation flag (.spec). tells you if and what address is being requested -speculatively.
- data_o: holds the instruction data returned on a hit
- cache_req_o: contains the request sent to the next level cache/memory on a miss or uncached access. shows if a speculative access that missed might have propagated
- data_mem_pkt_i: takes fill data coming back from the cache engine. confirms that a cache line was allocated/filled. maybe keep an eye for a speculative miss case.
- tag_mem_pkt_i:  tag and state updates coming back from the cache engine. it signals a state change possibly triggered speculatively?

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
