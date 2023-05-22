# 😀 Introduction to verilog:

Verilog is a type of HDL used in designing FPGAs and ASICs. Compared with traditional software languages, verilog works very differently. We will be learning verilog by implementing digital design concepts in verilog code. In this way, we will be learning faster and efficiently.

## Implementing AND in verilog:

For implementing AND gate in verilog we need three signals, 2 inputs and 1 output. we will use " & " operator for AND operation between input\_1 and input\_2.

For every verilog code we need to create a module where we will be writing all the logic.

```
module AND_gate(
    input input_1,
    input input_2,
    output output_AND
);
    assign output_AND = input_1 & input_2;
endmodule

```

Now we will implement D Flip Flop in verilog.

```
// D flip flop
module d_ff(d, clk, q, q_bar);
input d, clk;
output q,q_bar;
//wire d, clk;
reg q,q_bar;

always @(posedge clk)
begin 
    q <= d;
    q_bar <= !d;
end
endmodule
    
```

here we have implemented D flip flop in verilog, we have used 4 signals, two are inputs and rest are outputs. q, q\_bar are mentioned as reg because there is a procedural block where q and q\_bar is used. and assignment require left side to be reg type. We have used non-blocking assignment here as it is sequential logic. non blocking assignment means all the operation on the right side of assignments is performed first and assigning the values is performed last.(ex a <= b, c <= a ; here b is executed and next a is executed and b value is given to a and old value of a is given to c.&#x20;
