# Find Faster Clock

I came across this question from this website **rajesh52.blogspot.com**

Design a module that finds the faster clock between two different clocks. The output A\_faster\_than\_B asserts "1" when fA > fB and asserts "0" when fB > fA along with valid.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Followup questions:

\- How will you handle when clocks are almost the same or the same? How will you indicate the output in this case?

\- How will update the design if asked to find and indicate the frequency of the clocks?



Ideas :&#x20;

Approach, assign a counter to each clock to count the cycles.

{% code lineNumbers="true" %}
```verilog
//counter for clock A
always_ff @(posedge clk_A or posedge reset) begin
    if(reset) begin
        count_a <= '0;
    end else begin
        count_a <= count_a + 1;
    end
end
```
{% endcode %}

{% code lineNumbers="true" %}
```verilog
// counter for clock B
always_ff @(posedge clk_B or posedge reset) begin
    if(reset) begin
        count_b <= '0;
    end else begin
        count_b <= count_b + 1;
    end 
end 
```
{% endcode %}

now we will compare the both clock on a period, so we will suppose period is 100, then we will be counting the clock cycles until 100 and compare them.

{% code lineNumbers="true" %}
```verilog
// comparision
always_ff @(posedge clk_A or posedge reset) begin
    if(reset) begin
        cycle_counter <= '0;
        valid <= 0;
        A_faster_than_B <= 0;
    end else begin
        if(cycle_counter < PERIOD) begin
            cycle_counter <= cycle_counter + 1'b1;
        end else begin
            if(count_a > count_b + threshold)
                A_faster_than_B <= 1;
            else if(count_b > count_a + threshold)
                A_faster_than_B <= 0;
            else
                valid <= 0; //clocks are considered same or close
        end
    end
end

endmodule
```
{% endcode %}



complete code:

{% code lineNumbers="true" %}
```verilog
module fasterclock(    
    input logic clk_A,
    input logic clk_B,
    input logic reset,
    output logic A_faster_than_B,
    output logic valid);
    
    logic [15:0] count_a;
    logic [15:0] count_b;
    logic [15:0] cycle_counter;
    parameter integer period = 500;
    parameter integer threshold = 2;
    
    always_ff @(posedge clk_A or posedge reset) begin
        if(reset) 
            count_a <= 16'h0;
        else
            count_a <= count_a + 1;
    end
    
    always_ff @(posedge clk_B or posedge reset) begin
        if(reset)
            count_b <= 16'h0;
        else 
            count_b <= count_b + 1;
    end
    
    //comaparision
    always_ff @(posedge clk_A or posedge reset) begin
        if(reset) begin
            cycle_counter <= 16'b0;
            valid <= 0;
            A_faster_than_B <= 0;
        end else begin
            if(cycle_counter < period) begin
                cycle_counter <= cycle_counter + 1;
            end else begin
                valid <= 1;
                if(count_a > count_b + threshold)
                    A_faster_than_B <= 1;
                else if (count_b > count_a + threshold) 
                    A_faster_than_B <= 0;
                else 
                    valid <= 0;
                end
            end
        end
    endmodule
    
```
{% endcode %}



complete code with testbench : [https://www.edaplayground.com/x/ABFW](https://www.edaplayground.com/x/ABFW)

