# Status Bits

Problem statement :&#x20;

This problem is about checking to see whether we understand the problem statement or not.

We need to output a 10 bit number which has stream of input bits from \[7:0], and&#x20;

bit \[9] is the indicative of xor of the \[5:0], which is last 6 bits streamed in.

bit\[8] is set if the addition of bit\[7:6] and bit\[5:4] is greater than \[3:0] bits.

And we need to output this 10 bits, and also we need to mask the output to 0, unless atleast one status bit is set.



Idea : I explained the problem statement clearly, but when i first read its a bit confusing. lets start solving, so we take a shift reg and keep on adding the input stream. and we set the status bits according to logic, and we check the status bits for masking.

{% code lineNumbers="true" %}
```verilog
module stausbits (
    input logic clk,
    input logic reset,
    input logic din,
    output logic [9:0] dout );
    
    logic [7:0] shift_reg;
    logic status8;
    logic status9;
    
    always_ff @(posedge clk or posedge reset) begin
        if(reset) begin
            shift_reg <= '0;
        end else begin
            shift_reg <= {shift_reg[6:0], din};
        end
    end
    
    assign status9 = ^(shift_reg[5:0]);
    assign status8 = ({2'b0, shift_reg[7:6]} + {2'b0, shift_reg[5:4]}) > shift_reg[3:0];
    assign dout = (status9 || status8) ? {status9, status8, shift_reg} : '0;
    
endmodule
```
{% endcode %}

we used 2'b0, cause the width of two bits. in dout , we used or to check atleast one status bit is set to concatenate the 10 bit output, else mask to 0.
