# Weird counter

Problem statement : This is a weird counter problem, where we start to count from 0 when reset is deasserted, and the pattern goes like this 0, 1, 2, 3, ,4 ,5, 6, 7, 6, 5, 4, 4, 3, ,2, 1. Once we see 7 we need to start down counting to 1, and we need to count 4 two times.&#x20;



Idea : here to solve this problem we will be using fsm. We will have two states, Up state and down state.&#x20;

When we are in UP state : we will check if the counter == 7, if yes, then change the state to down and count value to 6,

else we will be incrementing the counter.



In down state : here we will be checking if we have seen the 4 value, if yes, we will be down counting, else if we seen 4 for first time, we change the count to 4 again, and if we have not seen 4 then we down count the counter value.





{% code lineNumbers="true" %}
```verilog
module downcount (
    input logic clk,
    input logic reset,
    output logic [2:0] count );
    
    typedef enum logic[1:0] {up, down} state;
    state state_q;
    state state_next;
    logic next_count;
    logic seen4;
    
    always_ff @(posedge clk or posedge reset) begin
        if(reset) begin
            state_q <= up;
            count <= 0;
            seen4 <= 0;
        end else begin
            state_q <= next_state;
            count <= next_count;
        end
    end
    
    always_comb begin 
        next_state = state_q;
        next_count = count;
        
        case(state_q)
            
            up:
                if(count == 7) begin
                    next_state = down;
                    next_count = 6;
                end else begin
                    next_count = count + 1;
                end
            
            down:
                if(count == 4 && !seen4) begin
                    next_count = 4;
                    seen4 = 1;
                end else if(count == 4 && seen4) begin
                    next_count = count - 1;
                end else begin
                    next_count = count - 1;
                end
        endcase
        
    end
    
endmodule
                    
                
```
{% endcode %}

