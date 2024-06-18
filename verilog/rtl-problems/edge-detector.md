# Edge detector

This is a interview question, Asked in fpga design role

there are two outputs r and f, which should be 0 when reset is applied, if reset is deasserted, and din is high, then r should be high for three cycles, if When ‘rst’ is high (‘1’), both ‘r’ and ‘f’ output low (‘0’) and remain low after ‘rst’ goes low (‘0’), until ‘a’ changes state with a new edge. When ‘rst’ is low (‘0’), ‘r’ outputs high (‘1’) for 3 ‘clk’ periods on rising-edges of ‘a’, whereas ‘f’ outputs high (‘1’) for 5 ‘clk’ periods on falling-edges of ‘a’. Note that the output behavior of ‘r’ and ‘f’ are mutually exclusive of each other.



Idea : So, here we need a counter, for r to count to 3 and through all this time r should be high, and same for f. So here we write a fsm, with idle, r\_high, f\_high states.



{% code lineNumbers="true" %}
```verilog
module edged(
    input logic clk,
    input logic reset,
    input logic a,
    output logic r,
    output logic f);
    
    typedef enum{idle, r_high, f_high} state;
    state state_q;
    state state_next;
    
    logic[2:0] r_count_q;
    logic[2:0] f_count_q;
    logic[2:0] r_count_next;
    logic[2:0] f_count_next;
    
    logic a_d;
    logic a_rising = a & ~a_d;
    logic f_falling = ~a & a_d;
    logic f_falling_q;
    
    always_ff @(posedge clk or posedge reset) begin
        if(reset) begin
            f_falling_q <= 0;
        end else begin
            f_falling_q <= f_falling;
        end
    
    always_ff @(posedge clk or posedge reset) begin
        if(reset) begin
            state_q <= 0;
            r_count_q <= 0;
            f_count_q <= 0;
            a_d <= 0;
        end else begin
            state_q <= state_next;
            r_count_q <= r_count_next;
            f_count_q <= f_count_next;
            a_d <= a;
        end
    end 
    
    always_comb begin
        state_next = state_q;
        r =0;
        f=0;
        
        case(state_q) 
            idle:
                if(a_rising) begin
                    state_next = r_high;
                    r_count_n = 1;
                end else if(f_falling_q) begin
                    state_next = f_high;
                    f_count_n = 1;
                end else begin
                    state_next = idle;
                end
            
            r_high:
                if(r_count_q == 3'b011) begin
                    state_next = idle;
                    r_count_n = 0;
                end else begin
                    r= 1;
                    r_count_n = r_count_q + 1;
                    state_next  = r_high;
                end
                
             f_high:
                 if(f_count_q == 3'b101) begin
                     state_next = idle;
                     f_count_n = 0;
                 end else begin
                     f = 1;
                     f_count_n = f_count_q + 1;
                     state_next = f_high; 
                 end
                 
             default : state_next = idle;
         endcase
         end
endmodule
        
            
            
            
```
{% endcode %}

This code can be written in a different way without fsm.  I wrote it in fsm for clearer understanding.&#x20;
