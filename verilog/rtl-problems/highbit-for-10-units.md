# Highbit for 10 units

Problem statement :&#x20;

A N bit vector where each bit should be high for 10 units, and it should be like this

0000\_0001 for 10 time units

0000\_0010 for 10 time units

..... like that.



For this to happen, we will be using a counter which counts from 0 to 9, and when it reaches 9 we do this two operations.

highbit is calculated, and it will be used in dout where the 00000001  value will be shifted based on highbit value. for every 10 cycles, high bit changes to 1, 2, 3, 4....

```verilog
assign n_count = (count == 'h9) ? 0 : count + 1;
assign n_highbit = (count == 'h9) ? ( (highbit) + { {$clog2(n)(1'b0),1'b1} } ) : highbit;

assign dout = {clog2(n)(1'b0)} << highbit;

```

complete code:

```verilog
module #(parameter n = 8) (
    input logic clk,
    input logic rst,
    output logic [n:0] out);
    
    logic [3:0] count;
    logic [3:0] next_count;
    logic [$clog2(n)-1:0] bitstoshift;
    logic [$clog2(n)-1:0] next_bitstoshift;
    
    always_ff @(posedge clk or posedge rst) begin
        if(rst) begin
            count <= 'h0;
            bitstoshift <= 'h0;
        end else begin
            count <= next_count;
            bitstoshift <= next_bitstoshift;
        end
    end
    
    always_comb begin
        next_count = (count == 'h9) ? (0) : (count + 1) ;
        next_bitstoshift = (count == 'h9) ? ( (bitstoshift) + { {$clog2(n)(1'b0),1'b1} } ) : bitstoshift;
        out = ({$clog2(n)}(1'b0)) << bitstoshift;
    end
    
 endmodule 
```

