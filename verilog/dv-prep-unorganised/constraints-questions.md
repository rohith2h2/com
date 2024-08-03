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



<details>

<summary>Write constraint such that sum of arr[10] is 100 without using .sum method</summary>

```verilog
//can be done like this
constraint sum_o{
arr[0] + arr[1] + arr[2] + ...+arr[9] == 100;
}

//or Assuming array elements inside [1:20], here we it only solves for 100
class sum;
rand int list[10];

constraint sum_o{
foreach(list[i]) { 
    ( (i%2) == 0 ) -> list[i] inside {[1:20]};
    ( (i%2) == 1 ) -> list[i] = 20-list[i-1];
}
}

// another version of above constraint is

constraint sum_o{
    foreach(arr[i]){ if(i%2==0) arr[i] + arr[i+1] == 20; }
    }

```

</details>

<details>

<summary>Write a constraint where e<a href="https://verificationacademy.com/forums/t/array-constraint-each-row-total-should-be-100-with-each-element-less-than-80-i-want-to-understand-how-to-do-this-using-only-sum-method-of-array/39998">ach row total should be 100, with each element less than 80. </a></summary>

```verilog
class array_sum;
rand bit [7:0] arr[3][4];

constraint arr_sum{
    foreach(arr[i]){
    arr[i].sum(item) with ( (item<80) ? item : 101) == 100;
    }
   }
```

</details>

<details>

<summary>Write constraint for an integer array with 10 elements such that exactly 3 of them are same and rest are unique</summary>

```verilog
constraint con{
    foreach(arr[i]) {
        arr[i] < 10;
        arr[i] != val3 -> arr.sum() with (int'(item == arr[i]) ) == 1;
    }
    arr.sum() with (int'(item == val3) ) == 3;
}
    

```



</details>

<details>

<summary>Write a constraint to randomize 3x3x3 array with unique elements</summary>

```verilog
class threeD;
rand int arr3D[3][3][3];

constraint unique_elements{
    foreach(arr3D[i,j,k]){
        arr3D[i][j][k] inside {[1:27]};
    }
    unique {arr3D};
}
function void display();
foreach(arr[i,j,k]) begin
    $display("array[%0d][%0d][%0d] = %0d", i,j,k,arr3D[i][j][k]);
end
endfunction
endclass

module test;
  initial begin
    Unique3DArray obj = new();
    repeat(1) begin
      if (obj.randomize()) begin
        obj.display();
        $display("");
      end else begin
        $display("Randomization failed.");
      end
    end
  end
endmodule
    
```

</details>

<details>

<summary>Write constraints – to pick a ball out of 10 different colored balls and that color should not be repeated for in next 3 draws</summary>

```verilog
class balls;
rand int x;
int q[$:2];

constraint con{
    x inside {[1:10]};
    unique(x, q);
}

function void post_randomize();
if(q.size() != 3) begin
    q.push_back(x);
end else begin
    q.pop_front();
    q.push_back(x);
end
endfunction

module tb;
balls b;

initial begin
    b = new();
    repeat(20) begin
    b.randomize();
    end
end
endmodule
```

</details>

<details>

<summary>Write a constraint to divide values of 1 queue into 3 queues so that all 3 queues have unique elements</summary>

```verilog
// Some code
class queuedivider;
rand int source_q[$];
rand int q1[$];
rand int q2[$];
rand int q3[$];

constraint source_size{
    source_q.size() inside {[1:10]};
 }
 
 constraint source_a{
     foreach(source_q[i]){
      source_q[i] inside {[1:100]};
      }
     }
  constraint limit_qs{
    q1.size() inside {[1:10]};
    q2.size() inside {[1:10]};
    q3.size() inside {[1:10]}; }
  
  constraint uniq{
    unique {q1}; unique {q2}; unique {q3}; }
    
  constraint common{
  q1.size() + q2.size() + q3.size() == source_q.size();
  foreach(source_q[i]){
    source_q[i] inside {q1, q2, q3};
  }
  }
  
  function void post_randomize();
    $display("Source Queue: ", source_queue);
    $display("Queue 1: ", queue1);
    $display("Queue 2: ", queue2);
    $display("Queue 3: ", queue3);
  endfunction
endclass

module test;
  initial begin
    QueueDivider qd = new();
    repeat(5) begin
      if (qd.randomize()) begin
        $display("Randomization successful");
      end else begin
        $display("Randomization failed");
      end
      $display("------------------------");
    end
  end
endmodule

//this code works but there are some errors but this is mainly the idea
    


```

</details>

<details>

<summary>Write a constraint to generate dynamic array of 300 elements. each element can have value 0/1/2/3/4 each of the above values should be present more than 40 times in the array. element 0 can be repeated while 1/2/3/4 are not allowed to repeat consecutively ex: 001342.. allowed(0 can be repeated) ex: 0122431.. not allowed(2 is repeated)</summary>



```verilog
```

</details>

<details>

<summary>generate a solved sudoku puzzle</summary>

```verilog
class sudoko #(int M=3);
  localparam N=M*M;
  local int unsigned puzzle[N][N];
  rand int unsigned box[N][N];
  
  //value of each box must be between 1 and N
  
  constraint box_con{
    foreach( box[row,col] ) { box[row][col] inside { [1:N] }; } }
  
  //boxes on same row must have unique values
      constraint row_con{
        foreach(box[row, colA]){ //this takes row, and checks all the columns in that row
          foreach(box[ ,colB]){
            if(colA != colB){
              box[row][colA] != box[row][colB];
            }
          }
          }
          }
              //boxes on same column must have unique values
              constraint col_con{
                foreach(box[rowA,col]){
                  foreach(box[rowB, ]){
                    if(rowA != rowB) {
                      box[rowA][col] != box[rowB][col];
                    }
                  }
                  }
                  }
                      
                      //boxes in same MxM must have unique values
                      constraint block_con{
                        foreach(box[rowA, colA]){
                          foreach(box[rowB, colB]){
                            if( rowA/M == rowB/M && colA/M == colB/M && !(rowA == rowB && colA == colB) ){
                              box[rowA][colA] != box[rowB][colB];
                            }
                          }
                          }
                          }
                              
                              //box must have same value as puzzle if that exists in puzzle
                              constraint puzzle_con{
                                foreach(puzzle[row,col]){
                                  if(puzzle[row][col] != 0) {
                                    box[row][col] == puzzle[row][col];
                                  }
                                }
                                }
                                    
                                    function int solve_this( int unsigned puzzle[N][N] );
    this.puzzle = puzzle;
    return this.randomize();
  endfunction: solve_this
 
  // Print the solution.
 
  function void print();
    for ( int i = 0; i < N; i++ ) begin
      if ( i % M == 0 ) $write( "\n" );
      for ( int j = 0; j < N; j++ ) begin
        if ( j % M == 0 ) $write( " " );
        $write( "%3d", box[i][j] );
      end
      $write( "\n" );
    end
  endfunction: print 
      endclass
                                  
                                  
                                  module top;
  parameter  M = 3;
  localparam N = M * M;
  int unsigned puzzle[N][N];
  sudoko#( M ) s;
 
  initial begin
    puzzle = '{ '{ 0,2,3, 4,0,6, 7,8,0 }, // 0: blank box
                '{ 4,0,6, 7,0,9, 1,0,3 },
                '{ 7,8,0, 0,2,0, 0,5,6 },
 
                '{ 2,3,0, 0,6,0, 0,9,1 },
	        '{ 0,0,7, 8,0,1, 2,0,0 },
		'{ 8,9,0, 0,3,0, 0,6,7 },
 
                '{ 3,4,0, 0,7,0, 0,1,2 },
                '{ 6,0,8, 9,0,2, 3,0,5 },
                '{ 0,1,2, 3,0,5, 6,7,0 } };
    s = new;
    if ( s.solve_this( puzzle ) ) s.print();
    else $display( "cannot solve the puzzle" );
  end
endmodule

```

</details>

<details>

<summary>Write constraint for number to be power of 4.</summary>

```verilog
class PowerOfFourGenerator;
  rand int unsigned number;

//   // Constraint to ensure number is a power of 4
//   constraint power_of_four {
//     number inside {1, 4, 16, 64, 256, 1024, 4096, 16384, 65536, 262144, 1048576, 4194304};
//   }
  constraint tobe{
    number inside {[1:100]};
  }

  //Alternative constraint using bit manipulation
  constraint power_of_four {
    number != 0;
    (number & (number - 1)) == 0; // Ensures it's a power of 2
    (number & 'b01010101) != 0; // Ensures it's a power of 4 
  } //this checks if the set bits at odd position and with number odd position
  //power of 4 has its bit set at odd positions 1,3,5, and we check that

  function void post_randomize();
    $display("Generated number: %0d", number);
    $display("Is it a power of 4? %s", is_power_of_four(number) ? "Yes" : "No");
  endfunction

  // Helper function to verify if a number is a power of 4
  function bit is_power_of_four(int unsigned n);
    return (n != 0) && ((n & (n - 1)) == 0) && ((n & 'b01010101010101010101010101010101) != 0);
  endfunction
endclass

module test;
  initial begin
    PowerOfFourGenerator pof = new();
    repeat(10) begin
      if (pof.randomize()) begin
        $display("Randomization successful");
      end else begin
        $display("Randomization failed");
      end
      $display("------------------------");
    end
  end
endmodule
```

</details>

<details>

<summary> N queens</summary>

```verilog
class Q_class#(N=8);
  rand int queen[N];
  
  //constraint to ensure queen value is between 1 to N, this is basically queen position at what coloumn
  constraint q_po{
    foreach(queen[N])
      queen[N] inside {[1:N]};
  }
  
  //constrain to ensure two queen are not at same coloumn, so need to check two queens doesnt have same value
  constraint col_con{
    foreach(queen[i])
      foreach(queen[j])
        if(i!=j)
          queen[i] != queen[j];
  }
  
 //constrain to ensure queens are not in diagonal, for this we use subtraction between rows and check there values with column values, this places queens at different location and not attack diagonally
  constraint diagonal_con{
    foreach(queen[i])
      foreach(queen[j])
        if(i!=j)
          (queen[i] - queen[j]) != (i-j) && (queen[i] - queen[j]) != (j-i);
  }
endclass

program N_Queens;

  initial begin

    Q_class qc1; //N = 8 default value
    Q_class #(4) qc2; //N = 4
    qc1 = new();
    qc2 = new();
   
//     qc1.randomize();
//     $display("qc1.Q is %p", qc1.queen);
//     qc1.randomize();
//     $display("qc1.Q is %p", qc1.queen);

    qc2.randomize();
    $display("qc2.Q is %p", qc2.queen);

  end

endprogram
```

</details>

























