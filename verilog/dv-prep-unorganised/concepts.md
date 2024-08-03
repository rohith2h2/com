---
description: >-
  Unorganised random interview questions, some have company names. Most of the
  questions are solved based on scenario approach
---

# Interview Questions(unorganised)

<details>

<summary><mark style="color:red;">write a uvm driver for a simple bus protocol(AMD)</mark></summary>

```verilog
interface intf(input bit clk, input bit rst);
    logic [6:0] Addr
    logic [31:0] WrData;

clocking MonCB @(posedge clk);
        input Addr;
        input WrData;
    endclocking
endinterface

Assume you already have virtual interface drv_if connected to 
the physical interface above;                                     

Please implement the run_phase of your driver. 

answer:
//You need to drive the Addr,, WrData generated sequence items from sequencer 
to virtual interface connecting DUT. For that we implemented the below driver 
logic, seq_item_port.get_next_item(), seq_item_port.try_next_item(), 
seq_item_port.item_done(), 

class my_driver extends uvm_driver #(my_transaction);
  `uvm_component_utils(my_driver)
  
  virtual intf drv_if;
  
  function new(string name, uvm_component parent);
    super.new(name,parent);
  endfunction
  
  //build phase as question already mentioned virtual interface drv_if connected to physical 
  //interface, so using uvm_config_db::get() to fetch the value of the virtual interface and 
  //assign it to configuration object property(drv_if).
  //getting interface handle using uvm_config_db
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    if(!uvm_config_db#(virtual intf)::get(this,"","drv_if", drv_if) )
      `uvm_fatal("NOVIF", "virtual interface is not found");
  endfunction
  
  //run phase
  task run_phase(uvm_phase phase);
    my_transaction tr;
    forever begin
      seq_item_port.get_next_item(tr); //get next transaction from sequencer
      
      //driving logic
      //drive transaction onto interface
      if(drv_if.rst) begin
        drv_if.Addr = '0;
        drv_if.WrData = '0;
      end
      
      drv_if.Addr = tr.Addr;
      drv_if_WrData = tr.WrData;
      
      @(posedge drv_if.clk); //wait for clock cycle to ensure values are driven
      
      seq_item_port.item_done(); //indicate transactio is done
      
    end
  endtask
  
endclass

```

</details>

<details>

<summary><mark style="color:red;">Cache Size is 64KB, Block size is 32B and the cache is Two-Way Set Associative. For a 32-bit physical address, find the bit number of Block Offset, Index and Tag.(AMD)</mark></summary>

calculation of tag bits = L - Set index - Block offset

l is lenght of address which is given in question 32 bits.&#x20;

Block offset is calculated with block size which is 2^5, 5 bits

set index is calculated with no of set in total cache, for that cache blocks should be known, which is cache size/block size = 2^11, we have 2 way set associative so, 2^11 / 2, which is 2^10, 10 bits

tag = 32 - 10 - 5 = 17 bits

</details>

Apple GPU DV

Computer architecture related questions. (covered all basic stuff from pipeline, cache, virtual memory, OOO)

<details>

<summary><mark style="color:red;">Pipeline Hazards :</mark> </summary>



Pipelining is a technique used to increase instruction throughput by overlapping the execution of multiple instructions.\
\
Imagine you're designing a 5-stage pipeline for a RISC processor. You notice that for certain instruction sequences, the pipeline stalls frequently. Can you explain what might be causing this, and propose a solution?

Frequent stalls in certain instruction sequences might be because of data dependencies between instructions.&#x20;

Consider this sequence:

ADD R1, R2, R3 #R1 = R2+R3

SUB R4, R1, R5 #R4 = R1 -R5

SUB instruction needs R1 value, but in 5 stage pipeline, result is not available in time.

this is a RAW hazard, pipeline needs to stall because second instruction can't proceed until first instruction is done. we can solve this by following methods, <mark style="color:green;">Forwarding</mark> - implementing the forwarding path that can route the ADD result directly to the SUB instruction without waiting for the writeback stage.&#x20;

<mark style="color:green;">Instruction reordering:</mark>&#x20;

possible rearrange instructions, to introduce independent instruction between dependent ones, allowing pipeling to continue without stalls.

<mark style="color:green;">Pipeline interlocking</mark> detect hazads in decode stage and introduce NOPs or stall pipeline when needed.

</details>

<details>

<summary><mark style="color:red;">Cache</mark></summary>

working on a system where the main bottleneck is memory access time. You've been asked to design a cache system to improve performance. What factors would you consider, and how would you approach this design?

Main bottleneck is memory access time, meaning cpu is frequently waiting for data from main memory.&#x20;

cache features -> larger caches store more data but increase access time and power.

cache line size larger line exploit spacial locality but may waste bandwidth if only small portions are used.

cache associativity tells how data is mapped and accessed within the cache

Replacement policy tells which cache line to evict when a new line is brought in. ( there are different methods for this lru, mru, .. )

OOO is a technique in modern processor utilize instruction level parallelism, ooo allows the instruction to differently from the program order,&#x20;

How it works: a) Instructions are fetched and decoded. b) They're placed in reservation stations, waiting for their operands to be ready. c) As soon as an instruction's operands are available, it can be executed, regardless of program order. d) Results are placed in the reorder buffer. e) Instructions are retired (committed) in program order to maintain correct program state.

</details>

<details>

<summary><mark style="color:red;">Gave spec for a design and asked to come up with a test plan for same. Then asked to implement the discussed checks in a scoreboard.</mark></summary>



</details>

<details>

<summary><mark style="color:red;">Implement checks in scoreboard</mark></summary>

```verilog
class my_scoreboard extends uvm_scoreboard;
    `uvm_utils_component(my_scoreboard)
    
    function new(string name="", uvm_component parent);
        super.new(name,parent);
    endfunction
//port to recieve packets from monitor
uvm_analysis_imp#(my_transaction, my_scoreboard) item_collected_export;

//array to store expected results
bit [31:0] expected_results[bit[31:0] ];

//instantiate analysis port
function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    item_collected_export = new("item_collected_export", this);
endfunction
//storing data or action to be taken when data is received from analysis port
virtual function void write(my_transaction trans);
    bit[31:0] expected_data;
    //check if we have expected results for this addr
    if(expected_results.exists(trans.addr)) begin
        expected_data = expected_results[trans.addr];
        
        //compare expected results and actual data
        if(trans.data == expected_data) begin
            `uvm_info(get_type_name(), $sformatf("PASS: Addr=%0h, Expected=%0h, Actual=%0h", 
                  trans.addr, expected_data, trans.data), UVM_LOW)
              end else 
              `uvm_error(get_type_name(), $sformatf("Unexpected transaction: Addr=%0h, Data=%0h", 
                 trans.addr, trans.data))
         end
 endfunction
 //we can write the comparing logic in write function or in run_phase
 
```

</details>

<details>

<summary><mark style="color:red;">Can you explain OOP concepts in SystemVerilog</mark></summary>

Inheritance: is the ability to create child class(derived class) from parent class(base class) to reuse properties and methods of base class.

Polymorphism:Ability to process objects differently based on their data members.

abstraction: is the ability to hide the information from different classes.

encapsulation: including or joining the data member and methods into a single unit like class.

</details>

<details>

<summary><mark style="color:red;">Implement (A+B+C+D). (A+B+C+E). (A+B+C+F) using one 4-input OR gate and one 3-input AND gate.</mark></summary>

As A+B+C is common across three expressions, we can write the whole expression like this

(A+B+C) + D . (A+B+C) + E . (A+B+C) + F = (A+B+C) + (D.E.F)

here D.E.F can be implemented using 3 input AND gate and lets assume X is output of this 3 input AND gate and we OR that with the above expression (A+B+C+X), which we need a 4 input OR gate to implement, solving the expression using 3 input AND and 4 input OR gate

</details>

<details>

<summary><mark style="color:red;">This was more like a discussion rather than an interview. Interviewer shared his screen with a design that had 3 masters, an arbiter, and an ALU. (ALU was on different clock domain, so there was a FIFO in between) Interviewer asked about the testbench architecture and many implementation specifics for the TB.</mark></summary>

So, UVM components needed for these are, 3 Master are there, so we need three agents for each of the master's, we need a monitor for arbiter, ALU, FIFO monitor, scoreboard, Virtual sequencer, ENV

</details>

<details>

<summary><mark style="color:red;">Can you explain setup and hold time violations, their impact on a design, and how you would address them?</mark></summary>

Setup time and hold time violations are crucial timing violations in digital design that lead to metastability and functional errors



Setup time violation: Occurs when data doesn't arrive early enough and stabilize at input of flop before the clock edge arrives. This causes flip flop to capture incorrect data or enter metastability state.

This can be fixed by reducing the clock frequency, thus making the data arrive faster, Optimize critical paths(gate sizing,..),&#x20;

Hold time violation: Occurs when the data changes too quickly after the clock edge, and before ff had any time to properly latch or capture the value. This causes ff to capture new data instead of intended data leading to race conditions.

This can be fixed by introducing buffer or delay elements into the pipelines or short paths, Optimize clock tree to reduce clock skew, &#x20;

</details>

<details>

<summary><mark style="color:red;">Explain the concepts of ideal path and false path constraints in synthesis. How and when would you use them?"</mark></summary>



Synthesis constraints are crucial for guiding the synthesis tool to optimize the design correctly.&#x20;

Ideal path: path assumed to have zero delays during timing analysis.&#x20;

False path: A false path is a logical path in a digital circuit that exists in the netlist or RTL description but is not relevant for timing analysis because it will never be functionally active or doesn't need to meet timing constraints. Essentially, it's a path that the signals will never actually traverse during normal operation of the circuit.

Key points about false paths:

1. They are usually specified to the synthesis and static timing analysis (STA) tools.
2. Identifying false paths can help relax timing constraints and improve overall design performance.
3. They're often found in complex control logic or multi-clock domain designs.

</details>

<details>

<summary><mark style="color:red;">Describe open page and closed page policies in DRAM memory systems. What are their trade-offs.</mark></summary>

Open page and closed page are memory management strategies in DRAM systems.&#x20;

Open page : keeps the Most recently accessed row(page) in sense amplifier, anticipating future access to same row.

Closed page : closes the page access after each access.

</details>

<details>

<summary><mark style="color:red;">Synchronous vs Asynchronous reset: Which is better?</mark></summary>

Synchronous Reset: In a synchronous reset, the reset signal is sampled on the active edge of the clock. The state elements (like flip-flops) change their state only on the clock edge when the reset is active.

Asynchronous Reset: An asynchronous reset can change the state of flip-flops immediately, regardless of the clock signal. It typically uses the asynchronous input of flip-flops.

Now, let's look at the pros and cons of each:

Synchronous Reset:

Pros:

1. Predictable timing: Since the reset occurs only on clock edges, the behavior is more predictable and easier to analyze in timing simulations.
2. Easier to implement in FPGAs: Many FPGA architectures are optimized for synchronous resets, making them easier to implement and potentially faster.
3. Reduced metastability risk: Because the reset is synchronous with the clock, there's less chance of metastability issues when the reset is released.

Cons:

1. Longer reset time: The system must wait for the next clock edge to reset, which can be an issue in time-critical applications.
2. More complex reset tree: Ensuring that the reset reaches all flip-flops at the same clock edge can be challenging in large designs, potentially leading to clock skew issues.
3. Higher power consumption: The clock tree must be active for the reset to propagate, which can increase power consumption during reset.

Asynchronous Reset:

Pros:

1. Immediate response: The system can be reset instantly, without waiting for a clock edge, which is crucial in some safety-critical systems.
2. Simpler reset distribution: The reset signal doesn't need to be synchronized with the clock, simplifying the reset tree design.
3. Can operate without a clock: Useful in systems where the clock might not be reliable or available immediately after power-up.

Cons:

1. Potential for metastability: If the reset is released close to a clock edge, it can cause metastability issues, leading to unpredictable behavior.
2. More challenging timing analysis: The asynchronous nature makes it harder to predict and analyze timing in complex systems.
3. Recovery time issues: Flip-flops have specific timing requirements (recovery time) after an asynchronous reset, which can be challenging to meet in high-speed designs.

In practice, the choice between synchronous and asynchronous reset often depends on the specific requirements of the system. For example:

* In a safety-critical system that needs to respond to faults immediately, an asynchronous reset might be preferred.
* In a high-speed, complex FPGA design, a synchronous reset might be more suitable for its predictability and ease of implementation.

Some designers even use a combination of both: an asynchronous reset for initial power-up, followed by synchronous resets for normal operation. This approach aims to combine the immediate response of asynchronous reset with the predictability of synchronous reset.

</details>

<details>

<summary><mark style="color:red;">Asynchronous FIFO:</mark></summary>

Used in data transfer between two different clock domains.&#x20;

</details>

<details>

<summary><mark style="color:red;">3 by 1 MUX using 2 by 1 MUX</mark></summary>

This can be done in cascaded approach, we can build 3:1 mux using two 2:1 muxes, mux1 has inputs a and b, whose output is one of the inputs to mux2 along with third input c, thus s0 and s1 are select lines for mux 1 and mux 2, thus creating a 3:1 mux.

</details>

<details>

<summary><mark style="color:red;">detect this sequence 110011</mark></summary>



</details>

<details>

<summary><mark style="color:red;">Asynchronous FIFO: How to design and problems faced?</mark></summary>

```verilog
//Asynchronous FIFO

```

Core of FIFO is basically a dual port RAM allowing simultaneous read and write operations, we also need separate read and write pointers operating in there clock domains, To safely cross clock domains we use gray counters for the pointers, Gray code changes only one bit at a time reducing metastability issues.&#x20;



</details>

<details>

<summary><mark style="color:red;">Constraint to generate gray code</mark></summary>

```verilog
class c;
rand [2:0] g[8];

constraint c{ foreach(g[i]) {
                                g[i] = {i[2], i[2]^i[1], i[1]^i[0] };
                                }
endclass
module a;
a i =new();
initial begin
i.randomize();
end
endmodule
```

</details>

<details>

<summary>Explain what a glitch is? when it can occur, and how to resolve it?</summary>

A Glitch in digital circuit is a short, unwanted transition of a signal, that occurs before the signal settles to its intended value. Glitches causes significant issues, mainly in synchronous design where they mess up the setup and hold times of FF.  these occurs in combinational logic, due to propagation delay through different paths. Some techniques to resolve, use inertial delay built in gates, which filters out pulses shorter than certain duration.&#x20;

</details>

<details>

<summary>Discuss static and dynamic power in digital circuits and ways to reduce both?</summary>



</details>

<details>

<summary>Why we use create method in UVM</summary>



```verilog
// As my_env_config_obj is extended from uvm_object, create method
// requires object name only.
m_env_cfg = my_env_config_obj::type_id::create("m_env_cfg");

// my_env is extended from uvm_component and it requires name as well as parent 
// type name, given by 'this'
m_env = my_env::type_id::create("my_env", this);
```

We use **create()** method because, if any overrides are registered with the factory, the create method returns object of override type(by type I mean type of class). So, basically we get child object on parent handle if overrides are registered. Whereas **new()** method returns object of type its being called on.

</details>

<details>

<summary>Difference between start_item() and start() ?</summary>

start\_item() is a method of an already running sequence - the sequencer was set when you started it. start\_item/finish\_item is used to send transactions to a driver, and thus _must_ be connected to a sequencer.

start() is a method of a sequence object you just created, and in turn calls its body() method. You pass in a sequencer to the start() method if you want the sequence body to call the start\_item() method.

</details>

<details>

<summary>Write a constraint where you have a 32 bit value bit [31:0] val where you’d want to randomize this to where every randomization would only allow 2 bits to differ from the previous randomization.</summary>

```verilog
class abc;
  rand bit [31:0] cur_val;
  constraint sel{
    $countones( const'(cur_val)^cur_val)==2;
  }
endclass
```

</details>

<details>

<summary>Connections in UVM</summary>

In driver, we have this seq\_item\_port.get\_next\_item(tr), which sends req to sequencer, and after completing we will send seq\_item\_port.item\_done(), to specify item is done.

In monitor, we have analysis port to send transactions to scoreboard, it is written as uvm\_analysis\_port#(transaction) send; and we add constructor in build\_phase like this    send = new("send", this); after getting the values from dut through vif, we use write method to send data to scoreboard.     send.write(tr);

In scoreboard, we receive data from monitor and compare data with predictor(golden predictor) data, for getting data from monitor, we have implementation port

uvm\_analysis\_imp#(transaction, scoreboard(currentscoreboard) ) recv;                     constructor in build\_phase -> recv = new("recv", this);                                                                    here in scoreboard we have write method which we used in monitor\
write method of the scoreboard will receive the transaction packet from the monitor, on calling write method from the monitor.&#x20;



In agent class, we use connect\_phase to connect the driver and sequencer in this way,  d.seq\_item\_port.connect(seqr.seq\_item\_export); we do this in connect\_phase function.

In evn class, we connect the driver and scoreboard in connect\_phase function as below, we will first call instance of agent and access mon through that instance like this, a.mon.send.connect(s.recv); send is monitor port, recv is scoreboard imp port, thus connection between monitor and scoreboard is done in env class

In test class, we instantiate all the env, and seqr classes, and use phase.raise\_objection(this) and seq call like start method, and phase.drop\_objection(this) , and in tb top, we call this test to start the verification, run\_test("test") starts the verification.



</details>

<details>

<summary>Isolate rightmost 1 bit in a word</summary>

```verilog
word_out = (word_in) & (-word_in);
```

</details>

<details>

<summary>All datatypes initialization</summary>

```verilog
//unpacked arrays
bit [5:0] unpacked [2:0]; //creates unpacked[0], unpacked[1] .. of 6 bit lenght
//foreach for multidimensional array is like this
foreach(unpacked[i,j]){ }
//packed array
bit [2:0] [5:0] a; //creates 3 a packed in 6
```

</details>

<details>

<summary>Why we should use non-blocking assignments in driver and blocking assignments in monitor?</summary>

Nonblocking assignments in the driver help avoid race conditions in the underlying DUT. They allow the driver to change signal values “just after the clock” such as

```verilog

task run_phase( uvm_phase phase );
  int top_idx = 0;
  
  // Default conditions:
  ADPCM.frame <= 0;
  ADPCM.data <= 0;
  forever begin
    @(posedge ADPCM.clk);
     ADPCM.frame <= 1; // Start of frame
    for(int i = 0; i < 8; i++) begin // Send nibbles
      @(posedge ADPCM.clk);
      ADPCM.data <= req.data[3:0];
      req.data = req.data >> 4;
    end
  
    ADPCM.frame <= 0; // End of frame
    seq_item_port.item_done(); // Indicates that the sequence_item has been consumed
  end
endtask: run
```

This allows the driver to interact with the DUT as if it were a “normal” SystemVerilog module. For monitors, on the other hand, we use blocking assignments because there is no worry about race conditions, since monitors typically sample values on the clock edge and then just do stuff with them:

```verilog

task run_phase( uvm_phase phase );
    wb_txn txn, txn_clone;
    txn = wb_txn::type_id::create("txn");  // Create once and reuse
    forever @ (posedge m_v_wb_bus_if.clk)
      if(m_v_wb_bus_if.s_cyc) begin // Is there a valid wb cycle?
        txn.adr = m_v_wb_bus_if.s_addr; // get address
        txn.count = 1;  // set count to one read or write
  ...//You get the idea
```

</details>

<details>

<summary>Reversing digits of number</summary>

```cpp
function rev();
int reverse = 0;
if(number > 0)
reverse = reverse* 10 + (number%10);
number = number / 10;

//so in this way we can find can reverse, so the loop continues until 
//number becomes 0
```

</details>

<details>

<summary>Static casting vs Dynamic casting</summary>

Static casting is unrelated to oops in sv, since sv doesnt have pointers, the only use static casting is to change the interpretation of value from one type to other type. It has the ability to preserve each bit while casting(chaning the shape of array or struct from one type to other).\
\
Dynamic casting is used to safely cast the child class handle to parent class handle. means parent class = child class; //this is valid no error is seen, but below assignment shows error, child class = parent class; //this is not valid no we will do this instead

child class c1();

parent class = child class;

$cast(c1, parent class); //this is valid

</details>

<details>

<summary>ALU verification plan and coverage bins</summary>

```
mindmap
[ALU Verification Plan]
│
├─ Functional Verification
│  ├─ Operation Testing
│  │  ├─ Arithmetic operations (ADD, SUB, MUL, DIV)
│  │  ├─ Logical operations (AND, OR, XOR, NOT)
│  │  └─ Shift operations (Left shift, Right shift)
│  │
│  ├─ Input Combinations
│  │  ├─ All possible opcode combinations
│  │  ├─ Various data input ranges
│  │  └─ Special cases (e.g., divide by zero)
│  │
│  └─ Flag Testing
│     ├─ Zero flag
│     ├─ Carry/Borrow flag
│     ├─ Overflow flag
│     └─ Sign flag
│
├─ Edge Case Testing
│  ├─ Boundary value analysis
│  ├─ Corner cases for each operation
│  └─ Invalid input handling
│
├─ Performance Verification
│  ├─ Timing analysis(We'll verify that the ALU meets the required clock frequency and latency specifications.)
│  ├─ Power consumption(We'll measure and verify that power usage is within acceptable limits.)
│  └─ Area utilization(We'll ensure the ALU fits within the allocated chip area)
│
├─ Verification Environment
│  ├─ Testbench architecture
│  ├─ Stimulus generation
│  └─ Response checking
│
└─ Verification Metrics
   ├─ Code coverage
   ├─ Functional coverage
   └─ Assertion coverage
   

coverage bins help us create seperate bins for each value in the given range of 
possible values of coverage point variable.

```

</details>

<details>

<summary>Can you explain the MESI protocol, particularly the O and F states?</summary>

MESI protocol Modified, Exclusive, Shared and Invalid is a cache coherency protocol used in mulitprocessor systems to maintain consistency of data among multiple cache. So we extend the current MESI with two more states, O means Owned and F means Forward state.  Now we will discuss about all the states, \
Modified(M) : Cache line is present only in current cache and is dirty(different from MainMem),&#x20;

Exclusive(E) : Cache line is present only in current cache and is clean

Shared (S) : Cache line maybe present in other caches and is clean

Invalid (I) : Cache line is not present in cache

Owned (O) : O is similar to S state, Cache line is potentially shared but is dirty, Only one cache hold the line in O state, rest all hold in S state, and cache with O state is responsible of writing back the data to main mem when line is evicted. This state can benefits like, It allows modified cache line to be shared without immediately writing back the data to MM, and other caches can read the most up to date data directly from cache, rather than MM.&#x20;

Forward (F) : Cache should forward data to other cache based on read request, Used in systems with distributed directories to optimize snoop filtering. Suppose in systems with many processors this reduces the number of caches needs to be snooped on read request.&#x20;

In practice, the O state is more commonly implemented than the F state. The O state is particularly useful in systems where data sharing is frequent but writes are less common, as it allows for efficient sharing of modified data without immediate write-backs. The F state, while less common, can be very beneficial in large-scale systems where reducing snoop traffic is crucial for performance and scalability.

</details>

<details>

<summary>You're tasked with verifying an asynchronous FIFO. Can you walk me through your verification plan, highlighting the key challenges and how you'd address them</summary>

I would start with verifying the Cross Domain Crossing Checks:  For example the gray code counters are correctly implemented for the read and write pointers. and Full and empty flags verification, like filling the fifo fast enough and checking whether the full flag is asserted or not at the right time. Check whether data written into fifo is read out in correct order without corruption.  let's consider a scenario where we're testing the full flag behavior:

1. We start with an empty FIFO.
2. We write data at the maximum possible rate on the write clock domain.
3. We read data at a slower rate on the read clock domain.
4. We monitor the full flag, ensuring it asserts exactly when the FIFO reaches its maximum capacity.
5. We continue writing and reading, verifying that no data is lost and the full flag remains asserted as long as the FIFO is full. This approach allows us to verify not just the functionality, but also the timing aspects of the full flag across clock domains.

```
[Asynchronous FIFO Verification]
|
|-- Clock Domain Crossing (CDC)
|   |-- Gray code counters
|   |-- Metastability checks
|
|-- Flag Verification
|   |-- Full flag
|   |-- Empty flag
|
|-- Data Integrity
|   |-- Write/Read patterns
|   |-- Order preservation
|
|-- Corner Cases
|   |-- Almost full/empty
|   |-- Simultaneous R/W
|
|-- Performance
    |-- Throughput
    |-- Different traffic patterns
```



</details>

<details>

<summary>Randc vs rand</summary>

Rand : generates purely random values without any limitations on repetitions.

Randc : also generates purely random values without repetitions unless all the values are seen or generated atleast once, then it repeats the values. Suppose we have to test a port variable, using rand bit \[2:0] port; when we use rand, we might see it generates some port number from 8 ports more frequently thus missing some ports, but when we use randc, then it generates all the 8 ports atleast once before repeating the ports.&#x20;

</details>

<details>

<summary>Can you explain register renaming and register forwarding in the context of processor design? How do these techniques improve performance?</summary>

Register renaming and register forwarding are crucial techniques in modern processor design that help improve performance by addressing data hazards and enabling more efficient instruction-level parallelism.

Register Renaming: This technique is used to eliminate false dependencies (WAR and WAW hazards) between instructions. It involves mapping architectural registers to a larger set of physical registers. Here's a scenario to illustrate:

Consider the following code:

```
CopyADD R1, R2, R3  // R1 = R2 + R3
SUB R4, R1, R5  // R4 = R1 - R5
MUL R1, R6, R7  // R1 = R6 * R7
DIV R8, R1, R9  // R8 = R1 / R9
```

Without register renaming:

* The MUL instruction must wait for the SUB to complete due to the WAW hazard on R1.
* The DIV instruction must wait for the MUL to complete to get the correct value of R1.

With register renaming:

1.  The processor might map these operations to physical registers:

    ```
    CopyADD P1, P2, P3  // P1 = P2 + P3 (R1 mapped to P1)
    SUB P4, P1, P5  // P4 = P1 - P5 (R4 mapped to P4)
    MUL P6, P7, P8  // P6 = P7 * P8 (R1 mapped to P6)
    DIV P9, P6, P10 // P9 = P6 / P10 (R8 mapped to P9)
    ```
2. Now, the MUL can execute in parallel with ADD and SUB, as it's writing to a different physical register.
3. The DIV correctly uses the result from MUL (P6) instead of the earlier value of R1 (P1).

This technique allows for more instructions to be executed in parallel, significantly improving performance.

Register Forwarding: This technique, also known as data forwarding, is used to reduce the impact of data dependencies (RAW hazards) by directly passing results between pipeline stages without waiting for them to be written back to the register file.

Scenario: Consider a simple 5-stage pipeline (Fetch, Decode, Execute, Memory, Writeback) and the following code:

```
CopyADD R1, R2, R3
SUB R4, R1, R5
```

Without forwarding:

1. The ADD instruction computes the result in the Execute stage.
2. The SUB instruction, needing R1, would have to stall for two cycles (Memory and Writeback of ADD) before it can proceed to its Execute stage.

With forwarding:

1. As soon as the ADD instruction computes the result in the Execute stage, it's immediately forwarded to the SUB instruction.
2. The SUB instruction can now proceed to its Execute stage in the very next cycle, saving two cycles of pipeline stall.

This technique significantly reduces pipeline stalls, improving overall processor performance.

In modern processors, these techniques are often used together. Register renaming provides more opportunities for out-of-order execution, while register forwarding ensures that the most recent data is always available to dependent instructions, minimizing stalls and maximizing throughput."

Mind Map for Register Renaming and Forwarding:

```
Copy[Register Optimization Techniques]
|
|-- Register Renaming
|   |-- Eliminates false dependencies
|   |-- Maps architectural to physical registers
|   |-- Enables out-of-order execution
|
|-- Register Forwarding
    |-- Reduces impact of true dependencies
    |-- Passes results between pipeline stages
    |-- Minimizes pipeline stalls
```

</details>

<details>

<summary>Timing path delays</summary>

* Clock-to-Q delay: Time for a flip-flop to output data after clock edge.
* Combinational logic delay: Propagation through gates and other logic elements.
* Interconnect delay: Time for signals to traverse wires between logic elements.
* Setup time: Required stable time before clock edge at the receiving flip-flop.
* Clock skew: Difference in arrival times of clock at source and destination.
* Jitter: Variations in clock period

</details>

<details>

<summary>How to tackle negative slack issues</summary>

Identify Critical paths : Use timing reports to pinpoint worst negative slack paths.Analyzed logic depth: Found paths with excessive logic levels.

* Optimized RTL: Rewrote complex combinational logic to reduce depth.
* Applied timing constraints: Tightened constraints on critical paths.
* Used faster cells: Replaced slow cells with faster alternatives in the library.
* Pipeline insertion: Added registers to break long paths where appropriate.
* Clock skew optimization: Adjusted clock tree to balance arrival times.

</details>

<details>

<summary>Explain why CDC (Clock Domain Crossing) checks are important in digital design.</summary>

The importance of performing CDC analysis is to identify metastability issues due to missing synchronizers.

1. Metastability prevention: Signals crossing clock domains can become metastable, leading to unpredictable behavior.
2. Data coherency: Ensure multi-bit signals are sampled correctly across domains.
3. Functional correctness: Prevent data loss or corruption due to timing violations.
4. Reliability: Avoid system failures caused by CDC-related issues.
5. Performance optimization: Identify and optimize critical CDC paths.

Without proper CDC checks, designs can suffer from intermittent failures that are hard to debug and can compromise system integrity

</details>

<details>

<summary>Why are synchronizers used</summary>

Synchronizers are used for several critical reasons:

1. Metastability resolution: They allow potentially metastable signals to settle.
2. Clock domain bridging: Safely transfer data between different clock domains.
3. Noise reduction: Filter out glitches or noise on asynchronous inputs.
4. Timing closure: Help meet setup and hold times for cross-domain signals.
5. Deterministic behavior: Ensure predictable operation in multi-clock systems.

By using synchronizers, we significantly reduce the risk of system failures due to metastability or timing issues in cross-domain communications

</details>

<details>

<summary>What does the arrival time in a synthesis timing report signify?</summary>

Data arrival time is the time required for data to propagate through source flip flop, travel through combinational logic and routing and arrive at the destination flip-flop before the next clock edge occurs. Arrival Time= Tclk-q+Tcombo

* The cumulative delay from the launch edge of the clock to the point where data arrives at the capture flip-flop.
* It includes clock-to-Q delay of the source flip-flop, combinational logic delay, and interconnect delay.
* Used in comparison with required time to calculate slack.
* A larger arrival time often indicates a longer or more complex path.
* Critical for identifying timing violations and optimizing paths.

</details>

<details>

<summary>What strategies would you employ to resolve negative slack in a design?</summary>

Optimize critical paths : reduce logic levels, and use faster library cells.&#x20;

Restructure RTL : break complex combinational logic



</details>

<details>

<summary>Describe methods to synchronize data when sending it from one clock domain to another</summary>

For synchronizing data across clock domains, we can use:

1. Double Flop Synchronizer:
   * For single-bit signals
   * Two flip-flops in destination domain
2. Gray Code Counter:
   * For multi-bit counters
   * Ensures only one bit changes at a time
3. FIFO:
   * For multi-bit data streams
   * Separate read and write clocks
4. Handshaking:
   * For sporadic data transfers
   * Uses request/acknowledge signals
5. MUX-based Synchronizer:
   * For mutually exclusive multi-bit buses
6. Pausible Clocking:
   * Temporarily pause one clock during transfer
7. Synchronous Design:
   * Use rationally related clocks where possible

</details>

<details>

<summary>How would you approach synchronizing large signal data across clock domains?</summary>



</details>

<details>

<summary>Explain the design of a handshake protocol using valid, acknowledge, and data bits</summary>

* Signals:
  * Valid: Asserted by sender when data is ready
  * Acknowledge: Asserted by receiver when data is accepted
  * Data: The actual data bits to be transferred
* Protocol sequence: a. Sender puts data on bus, asserts Valid b. Receiver samples Valid, reads data c. Receiver asserts Acknowledge d. Sender deasserts Valid e. Receiver deasserts Acknowledge
* Key design considerations:
  * Use double-flop synchronizers for Valid and Acknowledge
  * Ensure data stability when Valid is asserted
  * Implement timeout mechanism for hung transactions
  * Consider adding parity/ECC for data integrity
* State machine implementation:
  * Sender states: Idle, Data\_Valid, Wait\_Ack
  * Receiver states: Idle, Data\_Received, Send\_

</details>

<details>

<summary>Given a 2 bit arbiter, verification plan</summary>

[https://reverent-northcutt-8163cd.netlify.app/blog/markdown-test2/](https://reverent-northcutt-8163cd.netlify.app/blog/markdown-test2/)

</details>

<details>

<summary>Here's a diagram of a 4-bit Linear Feedback Shift Register (LFSR). Can you implement this in Verilog?</summary>

```verilog
module lsfr(
input logic clk,
input logic rst,
output reg [3:0] out );

always @(posedge clk) begin
if(rst) begin
    out <= 4'b1111;
end else begin
    out <= {out[2:0], out[3] ^ out[2]};
end

end
// so when we are lsfr, we xor, with 1111, after 14 steps we again see 1111 pattern

```

</details>

<details>

<summary>What are the main types of hazards in pipelined processors?</summary>

1. Data Hazards:
   * Occur when an instruction depends on the result of a previous instruction still in the pipeline
   * Types: RAW (Read After Write), WAR (Write After Read), WAW (Write After Write)
   * Solutions: Forwarding, stalling, register renaming
2. Control Hazards:
   * Occur due to branch instructions and changes in program flow
   * Impact: Pipeline flush if branch prediction is incorrect
   * Solutions: Branch prediction, delayed branching
3. Structural Hazards:
   * Occur when two instructions need the same hardware resource simultaneously
   * Example: Single memory for both instruction fetch and data access
   * Solutions: Resource duplication, pipeline stalling

Addressing these hazards is crucial for effective pipelining and maintaining instruction throughp

</details>

<details>

<summary>why active low reset is preferred, and also about active high reset and their codes</summary>

**Active High Reset**: This means that the reset signal is asserted (or active) when it is at a high voltage level (usually logic 1, which is typically around 3.3V or 5V in digital circuits). When the reset signal is high, it indicates that the system should be reset or initialized.

**Active Low Reset**: Conversely, an active low reset means that the reset signal is asserted when it is at a low voltage level (usually logic 0, which is near 0V in digital circuits). When the reset signal is low, it indicates that the system should be reset or initialized.

best practice nowadays is to to match the polarity of the standard cell library. Standard cells often have an active low reset signal, so you should match that. For example, if a std cell has an active low reset and you write your RTL to have an active high reset, the synthesis tool will have to infer an additional inverter somewhere along that path.\


for active low reset it would be like this -> @(posedge clk or negedge reset)

for active high reset it would be like this -> @(posedge clk or posedge reset)

and why reset is active low is preferred

* Noise immunity:
  * Less susceptible to noise-induced resets
  * High voltage noise more common than low voltage noise
* Power-on behavior:
  * Ensures reset during power-up when voltages are rising
  * Prevents unwanted state changes during power fluctuations
* Open-drain compatibility:
  * Multiple reset sources can be easily combined (wired-OR)
  * Simplifies system-level reset design
* Legacy compatibility:
  * Many older ICs use active low reset
  * Maintains consistency in mixed old/new designs
* Pull-up resistor usage:
  * Can be combined with a pull-up for default reset state
  * Useful in systems with optional reset connections
* Lower power consumption:
  * In CMOS, driving a signal low typically consumes less power



</details>

<details>

<summary>Write through vs Write back cache. For which of these is cache coherency protocol needed?</summary>

Write-through and write-back are two cache writing policies:

Write-through:

* Writes data to both cache and main memory simultaneously
* Ensures memory always has updated data
* Higher memory bandwidth usage
* Simpler to implement

Write-back:

* Writes data only to cache
* Updates main memory when cache line is evicted
* More efficient use of memory bandwidth
* Requires dirty bit to track modified cache lines

Cache coherency is needed for both, but it's more critical and complex for write-back caches. In write-back, other caches or processors may have outdated data until a cache line is written back to main memory. Write-through naturally maintains coherency with main memory but still needs protocols to maintain coherency between multiple caches in a multi-processor system.

</details>

<details>

<summary>4bit ripple carry adder/subtractor</summary>

* Basic structure:
  * 4 full adders connected in series
  * Carry out of each adder feeds into carry in of the next
* For subtraction:
  * Add a control signal (sub)
  * XOR this with B inputs and carry-in
  * When sub=0, performs addition
  * When sub=1, performs subtraction (A - B = A + (-B) = A + \~B + 1)
*   Key points:

    * Simple design, easy to implement
    * Delay increases linearly with bit width
    * Not suitable for high-speed, wide arithmetic unit

    [https://vlsiverify.com/verilog/verilog-codes/4-bit-adder-subtractor/](https://vlsiverify.com/verilog/verilog-codes/4-bit-adder-subtractor/)

</details>

<details>

<summary>Control Hazards in detail</summary>



</details>

<details>

<summary>Packages</summary>

Use packages for shared declarations, instead of $unit. Packages serve as containers for shared definitions and declarations, preventing inadvertent spaghetti code. Packages also have their ownname space, which will notcollide with definitions in other packages. There can still be name collision problems if two packages are wildcard imported into the same name space. This can be prevented by using explicit package imports and/or explicit package references, instead of wildcard imports

</details>

<details>

<summary>Transport and Inertial Delays</summary>

![](<../.gitbook/assets/image (3).png>)

1\.        nertial delay: Delay model that represents time it takes for signal to propagate through the gate, but with a minimum duration that must be met before the signal propagates.

Transport delay: Delay model that represents time it takes for the signal to propagate through the gate.

</details>

<details>

<summary>You have a block that has input clock of 50MHz and, one bit input called "X", the output is a clock of 100MHz that toggles if and only if x is asserted high, and the second output is called "Z", "Z" changes keeps changing when x is asserted high, its value doesn't matter, as long as it changes. output is valid after 600ns.</summary>

```verilog
//Below are the test cases I came up with
// test case1 : Z should be stable when X is 0
always@(posedge clk50Mhz) begin
if(!X) 
    assert($stable(Z) ) else $fatal("Z is changing when X is 0");
end 

//testcase2 : Z keeps changing when X is 1
if(X) begin
assert(!($stable(Z) ) else $fatal("Z is not changign when X is 1");


```

</details>

<details>

<summary>.⁠ ⁠you have to model how contacts behave in phone directory. you start with an empty list [] when a call is recieved, append into dict.</summary>

```python
contacts = []

def recieve_call(new_call):
    global contacts
    
    if new_call in contacts:
        contacts.remove(new_call)
    
    contacts.insert(0, new_call)
    
    print("current Contact list", contacts)

recieve_call("90920490")
recieve_call("43948959")
.
.
.
.


```

</details>

<details>

<summary>Generate random values of X such that x can be from 1 to 10, but not 5</summary>

```verilog
rand x;

constraint x_ran{
    x inside {[1:10]} && x! = 5;
    }
    
//FORK JOIN is used to run multiple threads simultaneously
initial begin
    fork //begin both tasks A and B concurrently
        task A();
        task B();
    join //join waits for both the tasks to complete before continuing 
end
```

</details>









