---
description: >-
  Unorganised random interview questions, some have company names. Most of the
  questions are solved based on scenario approach
---

# Interview Questions(unorganised)

1. Why we need both types of coverage(code and functional) ?

Getting to 100% code coverage, doesnt necessarily tell the complete story. Example, imagine a simple AND gate, to get to 100% code coverage, we only need two different types of input combinations - 00 and 11, these two will produce all the output combinations. Then the coverage tells all the code was exercised but we dont know whether our design works or not. all we know is input and outputs of our design are able to take on both logic 0 and logic 1. We couldn't certainly tell our design is an AND gate, it could also be an OR gate.  And this is where functional coverage comes in, So the first thing funtional coverage tells is have observed all the possible values on input. and checking the input and its output tells us which gate we are looking at.&#x20;



Also getting 100% functional coverage is not enough either, Imaginer we have two AND gates whos outputs are given to OR gate and output is taken there. Now we might simply attain 100% functional coverage by defining coverpoints for one AND gate and final output but not exercising the other AND gate.This is called coverage hole and This is where Code coverage becomes crucial, Cause it shows there is a part of code which is not exercised at all.&#x20;



And getting 100% on both code and functional coverage doesn't mean our design is bug free. But it gives confidence, that our design in thoroughlt exercised based on the specifications provided.&#x20;





<mark style="color:red;">Design FSM which detects three consecutive 1's with overlapping. use mealy machine.(AMD)</mark>

For combinational circuits, output value completely depends on input values, In sequential circuits output value depends on present input and also previously stored value. Sequentail circuits are FSM. There are two types Mealy and Moore.&#x20;

Mealy : Output depends on present state and current input. Requires Fewer states. Bit harder

Moore : Output depends on current state only independent of current input. Need more state. Easier

There are again two conditions overlapping and non overlapping.

Overlapping : Final bits of sequence can be start of another sequence.&#x20;

Non overlapping : Once sequence detection is completed, next sequence can be started without any overlap.&#x20;

Refer this for detail with diagrams: [https://yue-guo.com/2018/11/18/sequence-detector-1001-moore-machine-mealy-machine-overlapping-non-overlapping/](https://yue-guo.com/2018/11/18/sequence-detector-1001-moore-machine-mealy-machine-overlapping-non-overlapping/)

```verilog
//111
typedef enum {s0, s1, s2} state;
state state_q;
state state_next;

always_ff @(posedge clk) begin
    if(rst) state_q <= s0; else state_q <= state_next; end
always_comb begin
    case(state_q)
        s0:if(din == 1) state_next = s1; else state_next = s0;
        s1:if(din == 1) state_next = s2; else state_next = s0;
        s2:if(din == 1) begin state_next = s2; dout = 1; end else state_next = s0;
    endcase
end


```

<mark style="color:red;">Perl/Python coding question(AMD)</mark>

<figure><img src="../.gitbook/assets/Screenshot 2024-07-10 at 7.33.44 PM.png" alt="" width="103"><figcaption></figcaption></figure>

```python
data = [ {"ID" : 11, "value":55},  {"ID": 10, "value": 37}, {"ID": 1, "value": 28},
    {"ID": 11, "value": 78},{"ID": 0, "value": 33},{"ID": 1, "value": 25},{"ID": 11, "value": 67},
    {"ID": 1, "value": 90},{"ID": 10, "value": 58},{"ID": 11, "value": 102},
]

filtered_data = [record for record in data if record["ID"] == 11]
sorted_data = sorted(filtered_data, key = lambda x: x["value"] )
for record in sorted_data
    print(record)
```

```perl
use strict;
use warnings;
# Define the data as an array of hashes
my @data = (
    { ID => 11, value => 55 },
    { ID => 10, value => 37 },
    { ID => 1, value => 28 },
    { ID => 11, value => 78 },
    { ID => 0, value => 33 },
    { ID => 1, value => 25 },
    { ID => 11, value => 67 },
    { ID => 1, value => 90 },
    { ID => 10, value => 58 },
    { ID => 11, value => 102 },
);
# Filter records with ID == 11
my @filtered_data = grep { $_->{ID} == 11 } @data;
# Sort the filtered data by the 'value' key
my @sorted_data = sort { $a->{value} <=> $b->{value} } @filtered_data;
# Print the sorted data
foreach my $record (@sorted_data) {
    print "ID: $record->{ID}, value: $record->{value}\n";
}
```



<mark style="color:red;">write a uvm driver for a simple bus protocol(AMD)</mark>

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


```



You need to drive the Addr,, WrData generated sequence items from sequencer to virtual interface connecting DUT. For that we implemented the below driver logic, seq\_item\_port.get\_next\_item(), seq\_item\_port.try\_next\_item(), seq\_item\_port.item\_done(),&#x20;

```verilog
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



<mark style="color:red;">Cache Size is 64KB, Block size is 32B and the cache is Two-Way Set Associative. For a 32-bit physical address, find the bit number of Block Offset, Index and Tag.(AMD)</mark>

calculation of tag bits = L - Set index - Block offset

l is lenght of address which is given in question 32 bits.&#x20;

Block offset is calculated with block size which is 2^5, 5 bits

set index is calculated with no of set in total cache, for that cache blocks should be known, which is cache size/block size = 2^11, we have 2 way set associative so, 2^11 / 2, which is 2^10, 10 bits

tag = 32 - 10 - 5 = 17 bits



Apple GPU DV

Computer architecture related questions. (covered all basic stuff from pipeline, cache, virtual memory, OOO)

<mark style="color:red;">Pipeline Hazards :</mark>&#x20;

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

<mark style="color:red;">Cache</mark>

working on a system where the main bottleneck is memory access time. You've been asked to design a cache system to improve performance. What factors would you consider, and how would you approach this design?

Main bottleneck is memory access time, meaning cpu is frequently waiting for data from main memory.&#x20;

cache features -> larger caches store more data but increase access time and power.

cache line size larger line exploit spacial locality but may waste bandwidth if only small portions are used.

cache associativity tells how data is mapped and accessed within the cache

Replacement policy tells which cache line to evict when a new line is brought in. ( there are different methods for this lru, mru, .. )

OOO is a technique in modern processor utilize instruction level parallelism, ooo allows the instruction to differently from the program order,&#x20;

How it works: a) Instructions are fetched and decoded. b) They're placed in reservation stations, waiting for their operands to be ready. c) As soon as an instruction's operands are available, it can be executed, regardless of program order. d) Results are placed in the reorder buffer. e) Instructions are retired (committed) in program order to maintain correct program state.



<mark style="color:red;">Gave spec for a design and asked to come up with a test plan for same. Then asked to implement the discussed checks in a scoreboard.</mark>



Implement checks in scoreboard



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

<mark style="color:red;">Can you write a SystemVerilog code to generate the Fibonacci series</mark>

```verilog
module fibonacci(
    input logic clk,
    input logic rst,
    output logic o);
    
    logic first;
    logic second;
    logic curr;
    
    always_ff @(posedge clk) begin
        if(rst) begin
            first <= 1;
            second <= 1;
        end else begin
            first <= second;
            second <= curr
        end
    end
    always_comb begin
        curr = first+second;
        out = curr;  
    end
    
    endmodule
  
            
```

<mark style="color:red;">Can you explain OOP concepts in SystemVerilog</mark>

Inheritance: is the ability to create child class(derived class) from parent class(base class) to reuse properties and methods of base class.

Polymorphism:Ability to process objects differently based on their data members.

abstraction: is the ability to hide the information from different classes.

encapsulation: including or joining the data member and methods into a single unit like class.\
\


<mark style="color:red;">Implement (A+B+C+D). (A+B+C+E). (A+B+C+F) using one 4-input OR gate and one 3-input AND gate.</mark>

As A+B+C is common across three expressions, we can write the whole expression like this

(A+B+C) + D . (A+B+C) + E . (A+B+C) + F = (A+B+C) + (D.E.F)

here D.E.F can be implemented using 3 input AND gate and lets assume X is output of this 3 input AND gate and we OR that with the above expression (A+B+C+X), which we need a 4 input OR gate to implement, solving the expression using 3 input AND and 4 input OR gate



<mark style="color:red;">This was more like a discussion rather than an interview. Interviewer shared his screen with a design that had 3 masters, an arbiter, and an ALU. (ALU was on different clock domain, so there was a FIFO in between) Interviewer asked about the testbench architecture and many implementation specifics for the TB.</mark>

So, UVM components needed for these are, 3 Master are there, so we need three agents for each of the master's, we need a monitor for arbiter, ALU, FIFO monitor, scoreboard, Virtual sequencer, ENV

<mark style="color:red;">Can you explain setup and hold time violations, their impact on a design, and how you would address them?</mark>

Setup time and hold time violations are crucial timing violations in digital design that lead to metastability and functional errors



Setup time violation: Occurs when data doesn't arrive early enough and stabilize at input of flop before the clock edge arrives. This causes flip flop to capture incorrect data or enter metastability state.

This can be fixed by reducing the clock frequency, thus making the data arrive faster, Optimize critical paths(gate sizing,..),&#x20;

Hold time violation: Occurs when the data changes too quickly after the clock edge, and before ff had any time to properly latch or capture the value. This causes ff to capture new data instead of intended data leading to race conditions.

This can be fixed by introducing buffer or delay elements into the pipelines or short paths, Optimize clock tree to reduce clock skew, &#x20;



