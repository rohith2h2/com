# Chipdev series ( 1 to 10 )

Problems from chipdev series, which are relatively easier. In this Page we will go through the first 10 problems.



Problems :&#x20;

1\) Simple Router&#x20;

Build a router circuit which forwards data from the input (`din`) to one of four outputs (`dout0`, `dout1`, `dout2`, or `dout3`), specified by the address input (`addr`). The address is a two bit value whose decimal representation determines which output value to use. Append to `dout` the decimal representation of `addr` to get the output signal name `dout{address decimal value}`. For example, if `addr=b11` then the decimal representation of `addr` is `3`, so the output signal name is `dout3`. The input has an enable signal (`din_en`), which allows the input to be forwarded to an output when enabled. If an output is not currently being driven to, then it should be set to `0`.

Input and Output Signals

* `din` - Input data.
* `din_en` - Enable signal for `din`. Forwards data from input to an output if `1`, does not forward data otherwise.
* `addr` - Two bit destination address. For example `addr = b11 = 3` indicates `din` should be forwarded to output value 3 (`dout3`).
* `dout0` - Output 0. Corresponds to `addr = b00`.
* `dout1` - Output 1. Corresponds to `addr = b01`.
* `dout2` - Output 2. Corresponds to `addr = b10`.
* `dout3` - Output 3. Corresponds to `addr = b11`.



{% code lineNumbers="true" %}
```verilog
module router(
    input logic din,
    input logic[1:0] addr,
    input logic den,
    output logic dout0,
    output logic dout1,
    output logic dout2,
    output logic dout3;
    );
    
    assign dout0 = (den & (addr == 00) ) ? d0 : 0;
    assign dout1 = (den & (addr == 01) ) ? d1 : 0;
    assign dout2 = (den & (addr == 10) ) ? d2 : 0;
    assign dout3 = (den & (addr == 11) ) ? d3 : 0;
    
endmodule:router
```
{% endcode %}

we wrote a ternary operater, which checks den is asserted and addr values, if both these are true input is assigned to dout, else 0 is assigned.



2\) Second largest

Given a clocked sequence of unsigned values, output the second-largest value seen so far in the sequence. If only one value is seen, then the output (`dout`) should equal `0`. Note that repeated values are treated as separate candidates for being the second largest value. When the reset-low signal (`resetn`) goes low, all previous values seen in the input sequence should no longer be considered for the calculation of the second largest value, and the output `dout` should restart from `0` on the next cycle.

Input and Output Signals

* `clk` - Clock signal
* `resetn` - Synchronous reset-low signal
* `din` - Input data sequence
* `dout` - Second-largest value seen so far

Output signals during reset

* `dout` - `0` when `resetn` is active



idea :&#x20;

So the logic here is to check the din and update max and max\_q which are largest and second largest.

* If `din` is larger than `max_q` but not `max`, it means `din` should be the new second largest value.
* Example:
  * Suppose the sequence of inputs is 5, 10, 7.
  * After 5, `max = 5` and `max_q = 0`.
  * After 10, `max = 10` and `max_q = 5`.
  * After 7, since 7 is less than 10 but greater than 5, `max_q` is updated to 7.
  * The values are correctly tracked as `max = 10` and `max_q = 7`.

{% code overflow="wrap" lineNumbers="true" %}
```verilog
always_ff @(posedge clk or posedge reset) begin
    if(reset) begin
        max <= 0;
        max_q <= 0;
    end else begin
        if(din > max) begin //this checks for largest, hence din in largest max //is updated, and max_q will be previous max
            max <= din;
            max_q <= max;
        end else if( din > max_q) //this here implies, din is not larger than max, but larger than max_q, hence update current max_q with din
            max_q <= din; 
```
{% endcode %}



complete code

{% code overflow="wrap" lineNumbers="true" %}
```verilog
module secondlargest (parameter N=32) (
    input logic clk,
    input logic reset,
    input logic [N-1:0] din,
    output logic [N-1:0] dout );
    
    logic [N-1:0] max;
    logic [N-1:0] max_q;
    
    always_ff @(posedge clk or posedge reset) begin
        if(reset) begin
            max <= 32'h0;
            max_q <= 32'h0;
        end else begin
            if(din > max) begin
                max <= din;
                max_q <= max;
            end else if(din > max_q) 
                max_q <= din;
            end
    end
    assign dout = max_q;
endmodule
```
{% endcode %}





3\) Rounding Division

Prompt

Divide an input number by a power of two and round the result to the nearest integer. The power of two is calculated using `2DIV_LOG2` where `DIV_LOG2` is a module parameter. Remainders of 0.5 or greater should be rounded up to the nearest integer. If the output were to overflow, then the result should be saturated instead.

Input and Output Signals

* `din` - Input number
* `dout` - Rounded result

What needs to be done : So, we will have a din we need to divide it with a power of two, and round it to nearest integer, if the remainder is 0.5 or more we need to round it up to nearest integer, else we need to saturate the result by original.&#x20;



so here for calculating the division, we use this

{% code lineNumbers="true" %}
```verilog
assign div_result = din >> DIV_LOG2;
```
{% endcode %}

now we need to calculate rounded\_div, for which we take din\[DIV\_LOG2-1:0] value and add it to the div\_result. The bit `din[DIV_LOG2-1]` is the most significant bit among the shifted-out bits.&#x20;

* **If it's 1:** The fractional part(the shifted-out bits) is 0.5 or greater. This means we need to round up the integer result (`div_result`) by adding 1.
* **If it's 0:** The fractional part is less than 0.5. We don't need to round up, and `div_result` is already the correct rounded-down integer.

{% code lineNumbers="true" %}
```verilog
assign rounded_res = div_result + din[DIV_LOG2-1:0];
```
{% endcode %}

for chceking the overflow condition:

```verilog
(rounded_res[OUT_WIDTH] == 1) ? div_result : rounded_res;
//if overflowing then, we assign div_result, else rounded_res 
```

complete code:\


{% code lineNumbers="true" %}
```verilog
module round (parameter DIV_LOG2 = 3, OUT_WIDTH = 32, IN_WIDTH = DIV_LOG2+OUT_WIDTH)
( 
    input logic [IN_WIDHT-1:0] din,
    input logic [OUT_WIDTH-1:0] dout );
    logic [OUT_WIDTH:0] div_result;
    logic [OUT_WIDTH:0] rounded_res;
    
    
    assign div_result = din>>DIV_LOG2;
    assign rounded_res = div_result + din[DIV_LOG2-1];
    assign dout = (rounded_res[OUT_WIDTH] == 1) ? div_result : rounded_res;
    
endmodule
```
{% endcode %}

this problem explores a lot about bit shifting.



