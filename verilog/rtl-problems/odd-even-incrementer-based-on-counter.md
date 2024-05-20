# Odd Even incrementer based on counter

Write verilog for a counter that increments even numbers for 5 clk cycles and then odd numbers for 5 clk cycles&#x20;

Output : 0,2,4,6,8,9,11,13,15,17,18....&#x20;

Assume counter width is 5 bits, Given a module, how would you go about area optimisation and power optimisation





Idea:&#x20;

use a counter to count to 4, when count reaches 4,  we need to increment the number by one, ie, for 0 to 4 counts, number will be 0 2 4 6 8, then count reaches 4 hence we increment 8 to 9, and start to count to 4 again, so number will be 9, 11, 13, 15, 17, then again count reaches 4, and this continues.

In reset phase, we assign 0 to count and cycle\_counter. if reset is low, then comb logic is performed and assigned&#x20;

In comb logic, if cycle\_counter is 4, then reset the cycle\_counter and increment counter by one, else keep incrementing the cycle\_counter by 1 and increment count by 2.

in below code we will be separating logics in sequential and comb

{% code overflow="wrap" lineNumbers="true" %}
```verilog
always_ff @(posedge clk or negedge reset) begin
    if(reset) begin
        count <= 5'b0;
        cycle_counter <= 3'b0;
    end else begin
        count <= next_count;
        cycle_counter <= next_cycle_counter;
        even_phase <= next_even_phase;
    end
end
```
{% endcode %}

{% code overflow="wrap" lineNumbers="true" %}
```verilog
always_comb begin
    if(cycle_counter == 3'b100) begin
        next_cycle_counter = 3'b000;
        next_count = count + 1;
    end else begin
        next_cycle_counter = cycle_counter + 1'b1;
        next_count = count + 2;
    end
end
```
{% endcode %}

Complete code:

{% code lineNumbers="true" %}
```verilog
module evenodd ( 
    input logic clk,
    input logic reset,
    output logic [4:0] count );
    
    logic [2:0] cycle_counter;
    logic [2:0] next_cycle_counter;
    logic [4:0] next_count;
    
    always_ff @(posedge clk or posedge reset) begin
        if(reset) begin
            cycle_counter <= 3'b0;
            count <= 5'b0;
        end else begin
            cycle_counter <= next_cycle_counter;
            count <= next_count;
        end
    end
    
    always_comb begin
        if(cycle_counter == 3'd4) begin
            next_cycle_counter = 3'b0;
            next_count = count + 1;
        end else begin
            next_cycle_counter = cycle_counter + 1;
            next_count = count + 2;
        end
    end
    
endmodule

```
{% endcode %}

complete code with testbench : [https://www.edaplayground.com/x/bpaq](https://www.edaplayground.com/x/bpaq)

