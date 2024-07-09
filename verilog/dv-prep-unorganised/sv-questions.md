# SV questions

<mark style="color:red;">-> Class and Object</mark>

Class: Class is a set of attributes and associated behaviour grouped together. Object is an instance or handle of class. Attributes in class can be data members and associated behaviour means functions or tasks. below is code&#x20;

```verilog
//accesing data member of class
class transaction;
bit [7:0] data;
int id;

function new(bit [32:0] m_data);
$display("inside constructor");
data = m_data;
endfunction

endclass

module example;
initial begin

transaction tr = new(10);
$display("Value of %d", tr.data);

end
endmodule
```



now we access the class methods ( functions and tasks)

```verilog
class transaction;
bit [31:0] data;
int id;

task value(bit [31:0] m_data, int m_id);
data = m_data;
id = m_id;
endtask

function a(transaction tr);
$display("values of data and id", %0d, %0d);
endfunction

endclass

module ex;
initial begin
transaction tr = new();
tr.value(1,2);
tr.a(tr);
end

endmodule

```

<mark style="color:red;">-> 2 state and 4 state variables</mark>

2 state variable can store 0 and 1. they are int, shortint, longint, bit and byte

4 state variable can store 0 , 1, X, Z. they are integer, logic, reg, time (differene between logic and reg is reg can only used in procedural statements whereas logic can be used in both continous assignments and procedural assignments)



\-> <mark style="color:red;">packed and unpacked arrays:</mark>

<figure><img src="../.gitbook/assets/Screenshot 2024-07-05 at 8.08.58 PM.png" alt=""><figcaption></figcaption></figure>

\-> <mark style="color:red;">Dynamic and associative arrays</mark>

Dynamic arrays needs to be allocated memory before using it.&#x20;

Elements of array have particular data types.



Associative arrays memory can be allocated while using it.

elements of array can be any data type, we can group together different data types.



<mark style="color:red;">What is queue ?</mark>&#x20;

Queue is a variable size and ordered collection of elements. Queues are two types bounded and unbounded.&#x20;

Bounded means they have specific size or limited entries. defined as <mark style="color:red;">queue \[\<size>];</mark>

Unbounded states they have non specific queue size with unlimited entries. <mark style="color:red;">queue \[$];</mark>

In queues, memory locations may not be contagious.&#x20;



<mark style="color:red;">Structure vs Union:</mark>

Structure is similar to union, as they contain different data types inside them, except unions data type member they all share same memory location, which restricts user to use one data member at once.&#x20;



<mark style="color:red;">pass by value and pass by reference</mark>

pass by value argument passing mechanism copies argument locally, and operate on those variables, any changes made to those variables in the functions are not visible outside the function.

Pass by reference argument passing mechanism does not copy arguments locally, but reference to original arguments is passed. means any changes to arguments inside subroutine affects the original values of subroutine.&#x20;

```verilog
//pass by value does this
class ex;

function p1(int q);
    q = q + 1;
    endfunction
endclass

module a();
 ex e1;
 int a;
  initial begin
      a = 5;
      e1 = new();
      e1.p1(a);
  end
  endmodule
  
  //for pass by value result will be q = 6 and a = 5
  //for pass by reference we need to add ref to function p1(ref int q)
  //then results will be q = 6 and a = 6

```



<mark style="color:red;">Module vs Program Block</mark>

Program block can't instantiate a module block, where as a module block instantiate another module block or program block.&#x20;

Initial block inside program block is scheduled in reactive region, where as initial block in module block is scheduled in active region.&#x20;



<mark style="color:red;">SUPER</mark>

super is used in sub class which points to the properties and methods of the base class.&#x20;

super.task(), would return task in the base class from the subclass which is extended from base class.

example code:

```verilog
class parent;
  bit [3:0] a;
  bit [3:0] b;
  
  function new(bit [3:0] c); //default new constructor will assign 0, with this we can 
  //control the value being passed
    this.a = c; //this keyword used to refer class properties, methods of current class
    b = 1001;
  endfunction
  
  task print();
    $display("values of a and b are : %b, %b", a, b);
  endtask
endclass

class pe1;
  task p();
    $display("parent");
  endtask
endclass

class p2 extends pe1;
  task p1();
    super.p(); // super is used to access the properties or methods of base class
    $display("child");
  endtask
endclass

module a1();
  parent p1;
  pe1 a;
  p2 b;
  initial begin
    a = new();
    b = new();
    p1 = new(111);
    p1.print();
    b.p1(); // calling p1 task i can access parent class properties with super
  end
endmodule
    
```



<mark style="color:red;">-> == and ===</mark>

\== returns 0, 1, X, output will be X if one of the variables or both are X.

\=== returns only 0, 1 and it is used to compare X and Z, where ambigous values cant be comapred with ==



<mark style="color:red;">Clocking block</mark>

To specify, synchronization scheme and timing requirement for an interface, clocking block is used to separate timing related information from tb &#x20;

clocking block is used to avoid race condition between design and tb.&#x20;



<mark style="color:red;">Cross coverage:</mark>

cross coverage allows having cross product between two or more variable or cover points within same covergroup.&#x20;

```verilog
bit [3:0] addr;
bit [3:0] data;

covergroup c_grp(@posedge clk);
cp1 : coverpoint addr;
cp2 : coverpoint data;
cp1xcp2 : cross cp1, cp2;
endgroup:c_grp
```

<mark style="color:red;">Difference between always\_comb and always@(\*) blocks</mark>

always\_comb is triggered once at time zero, whereas always@(\*) is triggered when any of the signal in sensitivity list changes.

always\_comb is sensitive to changes of contents in function.

always@(\*) is sensitive to arguments of functions when invoked inside it.



<mark style="color:red;">Struct and Class</mark>

<figure><img src="../.gitbook/assets/Screenshot 2024-07-06 at 2.41.11 PM.png" alt=""><figcaption></figcaption></figure>



<mark style="color:red;">Difference between static and automatic Functions:</mark>

Functions are autimatically declared as static unless they are declared inside a class scope. Functions declared inside a class behaves as automatic unless specifically mentioned as static.&#x20;

Variables declared inside static function are static unless declared as automatic, same goes with automatic functions.&#x20;

functions declared as static can be accessed with a null handler

static properties means suppose we declare a data member or property being static then one copy of property is shared between all the instances of the class, where as properties or data member usually declared as dynmaic by default, means each instance of the class have seperate copy of that dynamic property.&#x20;



<mark style="color:red;">Shallow copy and Deep copy</mark>

When shallow copy is used, class properties like data members are copied but not the objects, only insstance handle of object is copied, means changes made to instane of object will be reflected to original object.&#x20;

ex: pac p1 , p2;

p2 = p1.new();



Deep copy is same as shallow copy expect, objects are also copied using the copy custom method.

p2 = p1.copy();



<mark style="color:red;">Virtual :</mark>&#x20;

A virtual function from base class can be overridden by a method of its child class having same signature(function name or arguments)





























