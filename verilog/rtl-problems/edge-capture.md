# Edge Capture

Problem Statement:&#x20;

Implement a edge detector circuit which captures any neg edge( transitioning from 1 to 0) on 32 bit input signal. Edge detection and capture must be performed on bit basis for all 32 bits.



Consider the following scenario:

* **Cycle 1:** Input is `0xAB` (binary 1010\_1011). Output should be `0000_0000` (no previous input to compare against).
*   **Cycle 2:** Input is `0xCA` (binary 1100\_1010). Comparing to the previous input:

    * `1010_1011` (previous)
    * `1100_1010` (current)

    The output should be `0010_0001` (representing edges on bits 21 and 25).

**Design Approach**

1. **Store Previous Input:** We'll use a register (`data_q`) to store the value of the input signal from the previous clock cycle.
2. **Bitwise Comparison:** On each clock cycle, we'll compare the current input (`data_i`) with the stored previous input (`data_q`). We'll use bitwise AND (`&`) with the inverted current input (`~data_i`) to isolate the positions where a 1-to-0 transition occurred.
3. **Sticky Capture:** The detected edges will be ORed (`|`) with a register (`edge_q`) that holds any previously captured edges. This ensures the edges remain sticky until reset.





idea: so we will be taking to instances of data value from current cycle and previous cycle, and check whether there's a transition from 1 to 0, and or it with edge\_q which is all the previous edge detections.

{% code lineNumbers="true" %}
```verilog
assign edge_seen[31:0] = (~data_i[31:0] & data_q[31:0] ) | edge_q[31:0];
assign edge_o[31:0] = edge_seen[31:0];
```
{% endcode %}

this code snippet which is comb\_block calculates the edge seen detection.



complete code:\


{% code lineNumbers="true" %}
```verilog
module edge_capture( input logic clk, input logic reset, input logic[31:0] data_i,
    output logic[31:0] edge_o);
    
    logic [31:0] edge_q;
    logic [31:0] data_q;
    logic [31:0] edge_seen;
    
    always_ff @(posedge clk or posedge reset) begin
        if(reset) begin
            data_q[31:0] <= 32'h0;
            edge_q[31:0] <= 32'h0;
        end else begin
            data_q[31:0] <= data_i[31:0];
            edge_q[31:0] <= edge_seen[31:0];
        end
    end
    
    //comb logic 
    assign edge_seen[31:0] = (~data_i[31:0] & data_q[31:0]) | edge_q[31:0];
    assign edge_o[31:0] = edge_seen[31:0];
    
endmodule
```
{% endcode %}
