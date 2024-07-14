# Constraints Questions:





Write rand constraint on a 3 bit variable with distribution 60% for 0 to 5 and 40% for 6,7. Write coverpoint for the same.

```verilog
class coverg
rand bit [2:0] value;

constraint val_c {value dist {[0:5] := 60, [6:7] := 40 }; }

covergroup c_group;
cp1 : coverpoint value { bins b1 = {[0:5]};
                         bins b2 = {[6:7]}; }
endgroup
endclass
                         
```

<mark style="color:red;">Implication operator</mark> :&#x20;

Relation between two variables, if LHS of the -> expression holds true, RHS constraint(expression/condition/limitation) is considered.&#x20;

`A -> B` means "if A is true, then B must be true"

constraint class\_c { (class == RED) -> (value < 50); }



constraint seen till now, ( if else ), ( if else-if else ), ( foreach ), inside, dist,&#x20;



<mark style="color:red;">DIST :</mark>&#x20;

dist is a weightage distribution during randomization.&#x20;

there are two ways to implement, :/ and :=&#x20;

while using :/ this operator, we can assign weighted distribution to specific value if one is mentioned, if there is a group, it is assigned like weighted distribution/number of values in group, like below example.

```verilog
constraint class_c { value dist {3:/4, [5:8]:/7}; }

result will be
value 3 weight 4 
value 5 weight 7/4 = 1.25
value 6 weight 7/4 
.
.

when we use := weight is distributed as below
constraint class_c {value dist {3:=4, [5:8]:=7}; }
result will be 
value 3 weight 4
value 5 weight 7
value 6 weight 7
....same for the rest
```

<mark style="color:red;">Write a constraint for two random variables such that one variable should not match with the other & the total number of bits toggled in one variable should be 5 w.r.t the other.</mark>



So, when we use XNOR, it returns 1 if both variables are the same; otherwise, it returns 0. Performing an XNOR operation on two variables and setting `$countones` to 5 will return the variables where exactly 5 bits are the same, and the rest are different.

```verilog
class a;
rand bit [7:0] n1;
rand bit [7:0] n2;

constraint con {$countones(n1 ~^ n2) == 5; }

endclass

module tb();
a a1();
initial begin
    a1 = new();
    repeat(10);
    a1.randomize();
    $display("var1= %0d, var2 = %0d", a1.n1, a1.n2);
    end
endmodule
```



<mark style="color:red;">Randomize a variable and make sure the current variable is not same as the previous variable</mark>

For this we use this constraint con { !(a inside {queue}); }

this constraint is for a inside queue should be false, thus we get variable which is not seen.

Think this is used when we dont want to use randc and generate random variable. maybe not as randc will be generating random variables without repeating until all values are seen.

```verilog
class a;
rand bit [3:0] val;
int queue[$:7];

constraint con { !(val inside {queue} ); }

function void post_randomize();
if(queue.size() >= 6)
    queue.pop_back();

queue.push_front(val);

$display("varaible = %0d", val);
$display("array = %0p", queue);
endfunction
endclass

module tb();
a a1;
initial begin
    a1 = new();
    repeat(10) begin
    a1.randomize();
    end
    end
    endmodule
```



