# Chipdev series

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



4\) Gray code counter&#x20;

Build a circuit that generates a Gray code sequence starting from `0` on the output (`dout`). Gray code is an ordering of binary numbers such that two successive values only have one bit difference between them. For example, a Gray code sequence for a two bit value could be: `b00 b01 b11 b10` The Gray code sequence should use the standard encoding. In the standard encoding the least significant bit follows a repetitive pattern of 2 on, 2 off ( `... 11001100 ...` ); the next digit a pattern of 4 on, 4 off ( `... 1111000011110000 ...` ); the nth least significant bit a pattern of 2n on 2n off. When the reset-low signal (`resetn`) goes to `0`, the Gray code sequence should restart from `0`.

Input and Output Signals

* `clk` - Clock signal
* `resetn` - Synchronous reset-low signal
* `out` - Gray code counter value

Output signals during reset

* `out` - `0` when `resetn` is active

Here we will be calculating the grey code using xor current bit and next bit, and storing it in a temp variable and last bit of q is assigned to last bit of temp

{% code lineNumbers="true" %}
```verilog
//gray code calculation
for( int i=0; i<DATA_WIDTH-1; i++) begin
    temp_next[i] = q_next[i+1] ^ q_next[i];
end
    temp_next[DATA_WIDTH-1:0] = q_next[DATA_WIDTH-1:0];
    
```
{% endcode %}



{% code lineNumbers="true" %}
```verilog
module model #(parameter
  DATA_WIDTH = 4
) (
  input clk,
  input resetn,
  output logic [DATA_WIDTH-1:0] out
);

  logic [DATA_WIDTH-1:0] q, q_next;
  logic [DATA_WIDTH-1:0] temp, temp_next;
  
  always_ff @(posedge clk or posedge reset) begin
    if(reset) begin
      temp <= 0;
      q <= 0;
    end else begin
      q <= q_next;
      temp <= temp_next;
    end
  end
  
  always_comb begin
    q_next = q + 1;
    for(int i=0; i<DATA_WIDTH-1; i++) begin
      temp_next[i] = q_next[i] ^ q_next[i+1];
    end
    temp_next[DATA_WIDTH-1] = q_next[DATA_WIDTH-1];
  end
  
  assign out = temp;
  
  endmodule
      
```
{% endcode %}

Here design might not need of seperate comb and ff blocks, just ff could work, but for design clarity and learning to write seperate comb and ff logics we did it. so, in ff block we are giving the q\_next and temp\_next calculated inside the comb block.&#x20;



5\) Reversing Bits

Reverse the bits of an input value's binary representation.

Input and Output Signals

* `din` - Input value
* `dout` - Bitwise reversed value



Idea: we can decalre a temp variable and assign the bit from msb to lsb in temp

```verilog
for(int i=0; i<DATA_WIDTH; i++) 
    temp[i] = din[DATA_WIDTH-1 - i];
    
```

this line will copy elements from MSB of din into the LSB into temp, which indeed is reversing the bit.

complete code:

{% code lineNumbers="true" %}
```verilog
module reversed (parameter N=32) (
    logic [N-1:0] din,
    logic [N-1:0] dout);
    
    logic [N-1:0] temp;
    
    always_comb begin
        for(int i=0; i< N; i++) begin
            temp[i] = din[N-1 - i];
        end
    end
    assign dout = temp;
endmodule
```
{% endcode %}



6\) Edge Detector

Build a circuit that pulses `dout` one cycle after the rising edge of `din`. A pulse is defined as writing a single-cycle `1` as shown in the examples below. When `resetn` is asserted, the value of `din` should be treated as `0`. Bonus - can you enhance your design to pulse `dout` on the same cycle as the rising edge? Note that this enhancement will not pass our test suite, but is still a useful exercise.

Input and Output Signals

* `clk` - Clock signal
* `resetn` - Synchronous reset-low signal
* `din` - Input signal
* `dout` - Output signal

Output signals during reset

* `dout` - `0` when `resetn` is active



Idea :&#x20;



7\) PISO Shift register:

Build a circuit that takes the multi-bit input (`din`) and shifts the input value’s least significant bit (rightmost bit) to the single-bit output (`dout`) one bit at a time. The circuit should begin shifting the input’s least significant bit when the the input enable signal (`din_en`) goes high. In other words, the input enable signal going high indicates that this circuit should start shifting the current input signal from it’s least significant bit, regardless of which bits the circuit has already shifted. If all the input’s bits have been shifted to the output so that there are no more bits to shift, the output must output `0`. When reset (`resetn`) is active, the input value that is being shifted is treated as `0`. Even when reset goes back to being inactive, the input value will still be treated as `0`, unless the input enable signal makes the circuit begin shifting from the input again.

Input and Output Signals

* `clk` - Clock signal
* `resetn` - Synchronous reset-low signal
* `din` - Input signal
* `din_en` - Enable signal for input data
* `dout` - Output signal

Output signals during reset

* `dout` - `0` when `resetn` is active



Idea : Here we need to give the din to temp variable, and this is done when din\_en is high, indicating the  we need to start shifting, when din\_en goes low, so once the din\_en goes high, even if din value changes, it wont affect our current output, until din\_en again goes high, reseting temp to new din value. If din\_en doesnt go low, we keep on giving the same number without any shifting in input.



For this we take to logic -> temp, next\_temp. temp hold din, next\_temp hold shifting of din. next\_temp is calculated in comb\_logic



{% code lineNumbers="true" %}
```verilog
//PISO
module PISO #(parameter N= 16) (
    input logic clk,
    input logic reset,
    input logic [N-1:0] din,
    input logic din_en,
    output logic [N-1:0] dout);
    
    logic [N-1:0] temp;
    logic [N-1:0] next_temp;
    
    always_ff @(posedge clk or posedge reset) begin
    if(reset) begin
    temp <= 0;
    end else if(din_en) begin
        temp <= din;
    end else begin
        temp <= next_temp;
        end
    end
    
    always_comb begin
        next_temp = temp >> 1;
        dout = temp[0];
    end
    endmodule
```
{% endcode %}

This should do the task.



8\) SIPO, now we need to design a circuit, which takes serial input and gives out parallel output,&#x20;



idea : so we need to have a register which takes the din serially and every cycle, it left shifts to place incoming bits at lsb by adding din to temp

temp <= (temp << 1) + din;

{% code lineNumbers="true" %}
```verilog
module sipo #(parameter n = 16) (
    input logic clk,
    input logic reset,
    input logic din,
    output logic [n-1:0] dout );
    
    logic [n-1:0] temp;
    always_ff @(posedge clk or posedge reset) begin
        if(reset) begin
            temp<= 0;
        end else begin
            temp <= (temp<<1) + din;
        end
    end
    assign dout = temp;
    endmodule
        
```
{% endcode %}



9\) Fibonacci Generator:&#x20;

Design a circuit which generates the Fibonacci sequence starting with `1` and `1` as the first two numbers. The Fibonacci sequence is a sequence of numbers where each number is the sum of the two previous numbers. More formally this can be expressed as: `F0 = 1` `F1 = 1` `Fn = Fn-1 + Fn-2` for `n > 1`. Following the definition of the Fibonacci sequence above we can see that the sequence is `1, 1, 2, 3, 5, 8, 13, etc`. The sequence should be produced when the active low signal (`resetn`) becomes active. In other words, the sequence should restart from `1` followed by another `1` (the Fibonacci sequence's initial condition) as soon as `resetn` becomes active.

Input and Output Signals

* `clk` - Clock signal
* `resetn` - Synchronous reset-low signal
* `out` - Current Fibonacci number

Output signals during reset

* `out` - `1` when `resetn` is active (the first `1` of the Fibonacci sequence)



The idea behind generating the Fibonacci sequence is that each number is the sum of the two preceding ones. The sequence starts with 0 and 1, and continues as 0, 1, 1, 2, 3, 5, 8, 13, and so on.

To generate this sequence in hardware, we use two registers, `prev` and `curr`, initialized to 0 and 1 respectively when the reset signal is asserted.

When the reset signal is deasserted, we calculate the next value in the sequence. To do this, we first update the `prev` register to hold the current value of `curr`. Then, we calculate the next Fibonacci number using a combinational logic block, adding the values of `prev` and `curr` to get `curr_next`.

Finally, the `curr` register is updated to hold the newly calculated Fibonacci number, `curr_next`, which is also the output of the circuit.

{% code lineNumbers="true" %}
```verilog
module fib_generator #(parameter n=16) (
    input logic clk,
    input logic rst,
    output logic [n-1:0] out);
    
    logic [n-1:0] prev;
    logic [n-1:0] curr;
    logic [n-1:0] curr_next;
    
    always_ff @(posedge clk or negedge rst) begin
        if(reset) begin
            prev <= 0;
            curr <= 1;
        end else begin
            prev <= curr;
            curr <= curr_next;
        end
    end
    
    //comb block to calculate curr_next, which is prev+curr values
    always_comb begin
        curr_next = prev + curr;
        out = curr;
    end
    endmodule
```
{% endcode %}

10\) Counting Ones



Given an input binary value, output the number of bits that are equal to `1`.

Input and Output Signals

* `din` - Input value
* `dout` - Number of `1`'s in the input value



Idea :&#x20;

So, this is simple circuit where we need to return the number of ones in a input. So, binary input is 0 and 1, so for finding the 1s we just need to add each bit value, which will return the number of 1s.



{% code lineNumbers="true" %}
```verilog
module ones #(parameter n=16) (
    input logic [n-1:0] din,
    output logic [$clog2(n):0] dout);
    
    logic [$clog2(n):0] temp;
    always_comb begin
        for(int i=0; i<n; i++) begin
            temp = temp + din[i];
        end
        dout = temp;
    end
    
    endmodule
    
```
{% endcode %}



11\) Gray code to binary conversion



Given a value, output its index in the standard Gray code sequence. This is known as converting a Gray code value to binary. Each input value's binary representation is an element in the Gray code sequence, and your circuit should output the index of the Gray code sequence the input value corresponds to. In the standard encoding the least significant bit follows a repetitive pattern of 2 on, 2 off ( `... 11001100 ...` ); the next digit a pattern of 4 on, 4 off ( `... 1111000011110000 ...` ); the nth least significant bit a pattern of 2n on 2n off.

Input and Output Signals

* `gray` - Input signal, interpreted as an element of the Gray code sequence
* `bin` - Index of the Gray code sequence the input corresponds to



Idea : So, when converting, the MSB of gray code will be same as MSB of binary bit. and rest of the bit are result of xor of all the bits when performed right shift thus dropping the i lsb bits,&#x20;



{% code lineNumbers="true" %}
```verilog
module gray2binary #(parameter n = 16) (
    input logic [n-1:0] gray,
    output logic [n-1:0] binary);
    
    logic [n-1:0] temp;
    
    always_comb begin
        temp = 0;
        for(int i=0; i< n; i++) begin
            temp[i] = ^(gray >> i);
        end
        binary = temp;
    end
    
    endmodule

```
{% endcode %}



12 ) Trailing zeros

Find the number of trailing `0`s in the binary representation of the input (`din`). If the input value is all `0`s, the number of trailing `0`s is the data width (`DATA_WIDTH`)

Input and Output Signals

* `din` - Input value
* `dout` - Number of trailing `0`s

There are four trailing zeros in `0b1010000`. If we assume `DATA_WIDTH = 8`, there are `8` trailing zeros in `0b00000000`.



Idea : quite interesting question, so here what we can do is implement this logic inside for loop by iterating through each and every element, and if i encounter 1, loop would break, else dout will be incremented to represent the trailing zeros. If all the bits are zero in the din, then dout would be data widht of the din.



{% code lineNumbers="true" %}
```verilog
module trailingzeros #(parameter n = 16) (
    input logic [n-1:0] din,
    output logic [$clog2(n):0] dout);
    
    always_comb begin
        dout = 0;
        for(int i=0; i< n; i++) begin
            if(din[i] == 1) begin
                break;
            end
            dout = dout + 1;
        end
        if(din == 0) begin
            dout = n;
        end
    end
    endmodule
```
{% endcode %}



13\) One hot detector

One-hot values have a single bit that is a `1` with all other bits being `0`. Output a `1` if the input (`din`) is a one-hot value, and output a `0` otherwise.



Idea : declare a counter and keep on incrementing it until the loop, and check if the counter is 1, return onehot as 1 else 0.



{% code lineNumbers="true" %}
```verilog
module onehot #(parameter n = 16) (
    input logic [n-1:0] din,
    output logic onehot);
    
    logic [n-1:0] counter;
    
    always_comb begin
        counter = 0;
        for(int i=0;i<n; i++) begin
            if(din[i] == 1) begin
                counter = counter + 1;
            end
        end
        assign onehot = (counter == 1);
    end
endmodule
```
{% endcode %}



14 ) Build a module which controls a stopwatch timer. The timer starts counting when the start button (`start`) is pressed (pulses) and increases by `1` every clock cycle. When the stop button (`stop`) is pressed, the timer stops counting. When the reset button (`reset`) is pressed, the count resets to `0` and the timer stops counting. If count ever reaches `MAX`, then it restarts from `0` on the next cycle. `stop`'s functionality takes priority over `start`'s functionality, and `reset`'s functionality takes priority over both `stop` and `start`'s functionality.\
\
Idea :&#x20;

So, when start is asserted in a clock cycle, from that clock cycle we should start counting until count reached max, and if we see stop then, we need to stop counting. Priority status is, when we have both start and stop asserted, then we need to given priority to stop and stop counting. so to know which state is currently active, we declare a reg state, which asserts to 0, when stop is asserted, in this way, if we check start loop,we have this statement : if(start || state)  state <= 1; count + 1;\
so, if both start and state are 0, then count is same, and we need to write stop before start to give priority to stop.



Complete code&#x20;

{% code lineNumbers="true" %}
```verilog
module stopwatch #(parameter n =16,MAX = 99) (
 input logic clk,
 input logic start,
 input logic stop,
 input logic reset,
 output logic [n-1:0] count);
 
 logic [n-1:0] temp;
 logic running;
 
 always_ff @(posedge clk) begin
  if(reset) begin
    temp <= 0;
    running <= 0;
  end else if(stop) begin
    temp <= temp;
    running<= 0;
  end else if(start || running) begin
    running <= 1;
    temp <= (temp == MAX) ? 0 : 1;
  end
  end
  assign count = temp;
  endmodule
```
{% endcode %}





15 ) Sequence Detector - 1010



Problem : we have to output 1 if we detect this sequence 1010.



Idea, we will have a state machine with 5 states, 0 state, 1 state this the first element, and from here we will be checking the elements and keep on updating the state machine states.



So when ever we write state machine its good practise to have two always blocks, one is sequential which stores the states and one is comb which updates the states.  and when we returning dout condition, its better to write a assign block and check the dout assert condition there. this eliminates any errors.



below code is good practise on how to write the state machine\


{% code lineNumbers="true" %}
```verilog
module s1010(
    input logic clk,
    input logic reset,
    input logic din,
    output logic dout);
    
    //state declaration
    typedef enum {S0, S1, S10, S101, S1010} state;
    state state_q;
    state state_next;
    
    //sequential to store current state 
    always_ff @(posedge clk) begin
        if(!reset) begin
        state_q <= S0;
        end else begin
        state_q <= state_next;
        end
    end
    
    always_comb begin
        dout = 0;
        
        case(state_q)
        
        S0 : 
        if(din == 1) begin
        state_next = S1;
        end else begin
        state_next = S0;
        end
        
        S1://1010
        if(din == 1) begin
        state_next = S1;
        end else begin
        state_next = S10;
        end
        
        S10: //1010
        if(din == 1) begin
        state_next = S101;
        end else begin
        state_next = S0;
        end
        
        S101: //1010
        if(din == 0) begin
        state_next = S1010;
        end else begin
        state_next = S1;
        end
        
        S1010: 
        if(din == 0) begin
        state_next = S0;
        end else begin
        state_next = S101;
        end
        
        endcase
        end
    assign dout = (state_q == S1010) ? 1 : 0;
    endmodule
        
```
{% endcode %}



16\) divisible by 3

Design a circuit that determines whether an input value is evenly divisible by three. The input value is of unknown length and is left-shifted one bit at a time into the circuit via the input (`din`). The circuit should output a `1` on the output (`dout`) if the current cumulative value is evenly divisible by three, and a `0` otherwise. When `resetn` is asserted, all previous bits seen on the input are no longer considered. The `0` seen during reset should not be included when calculating the next value. This problem is tricky, so it may help to think in terms of modulus and remainder states.



Idea : We can solve this by writing a state machine. We will have 4 states, reset state, where basically we change to next state based on din, and from there we will be having three states which are remainder of 3 as rem0, rem1, rem2. when we get into rem0, thats where we found a number divisible by 3. We have a seperate blog about divide by 3 module on this site, go through that for state machine diagram.



{% code lineNumbers="true" %}
```verilog
module div3 (
    input logic clk,
    input logic resetn,
    input logic din,
    output logic dout);
    
    typedef enum {rr, rem0, rem1, rem2} state;
    state state_q;
    state state_next;
    
    always_ff @(posedge clk) begin
        if(!resetn) begin
            state_q <= rr;
        end else begin
            state_q <= state_next;
        end
    end
    
    always_comb begin
        dout = 0;
        
        case(state_q) 
        
        rr: 
        if(din == 0) begin
            state_next = rem0;
        end else begin
            state_next = rem1;
        end
        
        rem0:
        if(din == 1) begin
            state_next = rem1;
        end else begin
            state_next = rem0; 
        end
        
        rem1: //1_
        if(din == 1) begin
            state_next = rem0;
        end else begin
            state_next = rem2;
        end
        
        rem2: //10
        if(din == 1) begin
            state_next = rem2;
        end else begin
            state_next = rem2;
        end
        
        default : state_next = rr;
        
        endcase
        
        end
        
        assign dout = (state_q == rem0) ? 1 : 0;
    endmodule
```
{% endcode %}



17 ) Divisible by 5



Design a module that determines whether an input value is evenly divisible by five. The input value is of unknown length and is left-shifted one bit at a time into the module via the input (`din`). The module should output a `1` on the output (`dout`) if the current cumulative value is evenly divisible by five and a `0` otherwise. When `resetn` is asserted, all previous bits seen on the input are no longer considered. The `0` seen during reset should not be included when calculating the next value. This problem is tricky, so it may help to think in terms of modulus and remainder states.



