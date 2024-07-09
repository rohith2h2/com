# UVM

<mark style="color:red;">UVM\_component vs UVM\_object</mark>

when uvm\_component is derived class object is created, it becomes part of testbench hierarchy, which persists until the end of simulation.

uvm\_object are created, referenced and destroyed once dereferenced.&#x20;



<mark style="color:red;">uvm\_transaction and uvm\_sequence\_item</mark>

uvm\_transaction is inherited from uvm\_object that add additional information on timing, notification.

uvm\_sequence\_item class is derived from uvm\_transaction class that adds basic functionality for sequence and sequence items like get\_sequence\_id, set\_sequencer, get\_sequence&#x20;
