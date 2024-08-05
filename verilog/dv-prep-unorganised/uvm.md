# UVM

<details>

<summary><mark style="color:red;">UVM_component vs UVM_object</mark></summary>

when uvm\_component is derived class object is created, it becomes part of testbench hierarchy, which persists until the end of simulation.

uvm\_object are created, referenced and destroyed once dereferenced.&#x20;

</details>

<details>

<summary><mark style="color:red;">uvm_transaction and uvm_sequence_item</mark></summary>

uvm\_transaction is inherited from uvm\_object that add additional information on timing, notification.

uvm\_sequence\_item class is derived from uvm\_transaction class that adds basic functionality for sequence and sequence items like get\_sequence\_id, set\_sequencer, get\_sequence&#x20;



The m\_sequencer handle is just a handle in every sequence that points to the se- quencer that is running that sequence and it was set when the start() method was passed the handle of the sequencer (env.vsqr in this case).

</details>

<details>

<summary>About uvm_config_db in great detail</summary>

Config database provides access to centralized database where type specific information can be stored and retrieved. resources can be stored in this database which can be retrieved based on name or type, db can be accessed globally.

uvm\_config\_db#(type T)::create(uvm\_cmpnt cntxt, string inst\_name, string field\_name, value);

uvm\_config\_db#(int)::set(null, "top.env", "my\_config", 42); In this case:

null as cntxt means the configuration is set at the top level of the entire testbench hierarchy. "top.env" as inst\_name means the configuration is accessible to the env component under top and all its children.

Your examples for inst\_name are correct and illustrate its flexibility:

"top.env.agent.monitor": Accessible only to this specific monitor. "top._": Accessible to all components directly under top. "top.env._.monitor": Accessible to all monitor components under top.env, regardless of their parent agent.







</details>

<details>

<summary>What testbench top consists</summary>

Testbench top consists of interface declaration, DUT instantiation, clock and reset generation, Test() generation or instantiation

```verilog
// Some code
//testbench top
module testbench_top;
  // Interface declaration
  my_if vif();

  // DUT instantiation
  my_dut dut(.clk(vif.clk), .rst_n(vif.rst_n), /* other ports */);

  // Clock generation
  initial begin
    vif.clk = 0;
    forever #5 vif.clk = ~vif.clk;
  end

  // Reset generation
  initial begin
    vif.rst_n = 0;
    #100 vif.rst_n = 1;
  end

  // Test initiation
  initial begin
    uvm_config_db#(virtual my_if)::set(null, "*", "vif", vif);
    run_test();
  end

  // Waveform dumping
  initial begin
    $dumpfile("dump.vcd");
    $dumpvars(0, testbench_top);
  end
endmodule
```





&#x20;

</details>



























