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



Implementing Ripple carry counter in verilog and verifying functionality of code by simulating waveforms.

```
// toggle flip flop
module tff(q, clk,reset );
  output reg q;
  input clk, reset;
  
  always @ (posedge reset or negedge clk) begin
    if (reset) begin
      q <= 1'b0;
    end else begin
      q <= ~q;
    end
  end
endmodule

module ripple_carry (q, clk, reset);
  output [3:0] q;
  input clk,reset;
  
  tff tff0(q[0], clk, reset);
  tff tff1(q[1], q[0], reset);
  tff tff2(q[2], q[1], reset);
  tff tff3(q[3], q[2], reset);
  
endmodule
```

```
//testbench
module test;
  
  reg clk, reset;
  wire [3:0] q;
  
  //Instanstiate design
  ripple_carry rcc(q, clk, reset);
  
  initial begin
    $dumpfile("dump.vcd");
    $dumpvars(1, test);
    
    clk = 1'b0;
    reset = 1'b1;
    #10 reset =1'b0;
    
    #200
    reset = 1'b1;
    #10 reset = 1'b0;
    #50;
    $finish;
  end
  
  always #5 clk = ~clk;
  
endmodule

```

```
// Some code
module multiplexor #(parameter WIDTH = 5) (
  input [WIDTH-1:0] in0,
  input [WIDTH-1:0] in1, 
  input sel, 
  output [WIDTH-1:0] mux_out
);
  reg [WIDTH-1:0] muxout;
  
  always @(*) begin
    if (sel == 1'b0)
      muxout <= in0;
    else if (sel == 1'b1)
      muxout <= in1;
  end
  
  assign mux_out = muxout;
  
endmodule
```

```
// Some code
//testbench
module multiplexor_test;

  localparam WIDTH=5;

  reg              sel  ;
  reg  [WIDTH-1:0] in0  ;
  reg  [WIDTH-1:0] in1  ;
  wire [WIDTH-1:0] mux_out;

  multiplexor
  #(
    .WIDTH ( WIDTH )
   )
  multiplexor_inst
  (
    .sel     ( sel     ),
    .in0     ( in0     ),
    .in1     ( in1     ),
    .mux_out ( mux_out ) 
  );

  task expect;
    input [WIDTH-1:0] exp_out;
    if (mux_out !== exp_out) begin
      $display("TEST FAILED");
      $display("At time %0d sel=%b in0=%b in1=%b mux_out=%b",
               $time, sel, in0, in1, mux_out);
      $display("mux_out should be %b", exp_out);
      $finish;
    end
   else begin
      $display("At time %0d sel=%b in0=%b in1=%b, mux_out=%b",
               $time, sel, in0, in1, mux_out);
   end 
  endtask

  initial begin
    sel=0; in0=5'h15; in1=5'h00; #1 expect (5'h15);
    sel=0; in0=5'h0A; in1=5'h00; #1 expect (5'h0A);
    sel=1; in0=5'h00; in1=5'h15; #1 expect (5'h15);
    sel=1; in0=5'h00; in1=5'h0A; #1 expect (5'h0A);
    $display("TEST PASSED");
    $finish;
  end

endmodule
```



