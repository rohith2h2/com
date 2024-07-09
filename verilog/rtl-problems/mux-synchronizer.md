# Mux Synchronizer

Problem statement;

Desing a mux synchronizer. There will be mux whose control input will be from a 2 ff synchronizer, and mux out will be given to a flop, and feedbacked into mux. and output of flop is mux synched value.



```verilog
module mux_synchronizer(
    input logic clk,
    input logic rst,
    input logic din,
    input logic cntrl,
    output logic sync_mux );
    
    logic q1, q2, mux_out;
    
    always_ff @(posedge clk or posedge rst) begin
     if(rst) begin
         q1 <= 0;
         q2 <= 0;
     end else begin
         q2 <= q1;
         q1 <= cntrl;
     end 
     end
     
     assign mux_out = (q2) ? din : sync_mux;
     
     always_ff @(posedge clk or posedge rst) begin
         if(rst) begin
             sync_mux <= 0;
         end else begin 
             sync_mux <= mux_out;
         end
     end
     endmodule
```
