# Computer architecture and resume detailed descriptions

\-> <mark style="color:red;">RISC vs CISC</mark>

RISC has less instructions and less hardware, where CISC has complex instructions and complex hardware to decode them. RISC requires one clock cycle to finish, CISC takes multiple clock cycles to finish depending on complexity of instructions.  So pipelining is possible in RISC. RISC aims to reduce number of cycles per instruction to improve performance, CISC does this by reducing the number of instructions per program.&#x20;

\-> <mark style="color:red;">difference between wrtie thru and write back cache?</mark>

In write thru cache, every write operation to cache also written to main memory. disadvantage is memory bandwidht is always consumed with writes.&#x20;

In write back cache, every write operation to cache is only written to cache.&#x20;



\-> <mark style="color:red;">Inclusive and Exclusive Caches, these properties applies to cache when there are multiple levels</mark>

If there are L1 and L2 caches, then  if all address present in L1 cache are also designed to present in L2 cache then we can say L1 is inclusive cache.&#x20;

If addresses are present in at most only one cache, then we can say its exclusive cache. Advantage here is multiple caches can together store more data.&#x20;

Advantage with inclusive cache, if a cache line is removed from processor cache, then it has to check in only one of the cache, where as for exclusive it needs to check both L1 and L2.&#x20;



\-> <mark style="color:red;">algorithms used in cache line replacement of set way associative cache</mark>

LRU - This algorithm keeps track of when a cache line is used by associating age bits along with cache line, discards least recently used one when needed.&#x20;

MRU - Same as LRU, most recently used cache line in terms of age bits gets replaced.



\-> <mark style="color:red;">problem of cache coherency</mark>

cache coherence  is critical in SMP to ensure all the processors have consistent view of memory.

In a shared multiprocessor system (SMP), there are multiple processors with own caches, and its possible that same data exist in different caches simultaneously, If each processor allowed to update caches, there is chance of inconsistency among memory.&#x20;



\-> <mark style="color:red;">Snoop based and Directory based coherence protocols</mark>

Snoop based : request for data from processor is send to all processor, that are in Shared system, then every other processor snoops this request and checks if they have this data copy and responds accordingly, thus every processor tries to have coherent view of memory.

Directory based : Directory is used to track, where a cache block is located, and the requesting processor sends request to directory, and we get to point to point access with the processor which has the data.&#x20;



\-> <mark style="color:red;">MESI</mark>&#x20;

MESI stands for&#x20;

Modifies ( cache line is present only in current cache, and is dirty, its modified from the value in main memory),&#x20;

E - Exclusive ( Cache line is present in only current cache and is clean, value matches with main mem)

S - Shared ( Cache line is present in more than one CPU cache and is clean, same as main memory value)

I - Invalid ( cache line is not present in the cache)



\-> <mark style="color:red;">Pipelining</mark>&#x20;

Pipelining is technique that implements  a form of parallelism, called Instruction level Parallelism.

Basic Instruction set is divided into a blocks of steps. Rather than proceeding each block sequentially, with pipelining, the  each instruction is split into a sequence and is executed parallel.&#x20;



<mark style="color:red;">Structural and Data hazards:</mark>

Structural hazards occurs because there is not enough duplication of resources. or resource conflict that prevents overlapped execution. &#x20;

solutions are to wait ( must detect hazard, must have mechanism to stall, Increase CPI)&#x20;

solution 2 is to throw more hardware&#x20;

Data hazards are

RAW : Instructions needs data produced by previous instruction.

WAR : Occurs when an instruction writes to a register which is source for a previous instruction.

WAW : occurs when an instruction writes to register which is also written by previous instruction.



\-> <mark style="color:red;">MOESI protocol and Dragon protocol.</mark>

Dragon Protocol is write-update cache coherence protocol, it involves multiple states for each cache lines to manage how data is shared and updated across caches. Idea is to update all copies of cache line on a write operation, rather than invalidating them as in the MESI protocol.

states in dragon protocol:&#x20;

E : cache block is clean and is only in one cache

M : cache block is exclusively owned but may be different from main memory(dirty)

Sm(shared modified) : cache block is in multiple caches and is possibly dirty

Sc(shared clean) : cache block is in multiple caches and is clean.



<mark style="color:red;">MOESI :</mark>&#x20;

In this we add an extra state to MESI protocol to improve efficiency, O (owned), means the line is possibly dirty and is in more than one cache, A cache line in owned state holds the most recent and correct copy of the data, Only one core can hold the data in owned state, rest of the caches hold it in shared state.



<mark style="color:red;">Prefetching:</mark>&#x20;

Fetch data into cache before its needed. there will be hardware prefetching and software prefetching.

Helps in reducing cache misses, improves performance.&#x20;



<mark style="color:red;">Impact of cache parameters :</mark>&#x20;

Size : larger caches reduce miss rate, but increases access time and power consumption.

Block size : means more adjacent words will be fetched on each miss, and improves hit rate(spacial locality).&#x20;

Associativity: Higher associativity reduces conflict misses but increases lookup complexity. &#x20;





