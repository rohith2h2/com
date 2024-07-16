# Divide by 3 Module

Q : Divide by 3 circuit, we need to check the serial input given is divisible by 3 or not. Like lets assume theres a module, which takes serial input x and concatenates it to LSB position of currently seen inputs to form a new number. and we need to check this new number is divisible by 3 or not.

Problem statement: we start with 0 and with each new x\_i will concatenated with 0, which is {0, x\_i} .. this new is divisible by 3 or not need to be checked. here we cant use flops to store the number because there are millions of inputs coming into machine and we will be run out of flops, so the div\_o is given out at every change in x\_i.

For this we will be designing a state machine. there will be 3 states, 0 state, 1 state, 2 state, there are the remainders for the divisible by 3

```
FSM : 
starting with 0,
assume a%3 = 0, so initially 
now we get a new x_i input: rem_0
if x_i = 0, then {a, 0}, this is simply left shifting a and adding 0, so 2*a + 0, where remainder is 0 as a%3 remainder is 0, div_o is 1
if x_i = 1, then{a, 1}, 2*a + 1, remainder is 1, moves to next state (1)

rem_1
current number 2a + 1
if x_i = 0, {2a+1, 0}, then 4a+2, remiander is 2 moves to next state (2)
if x_i = 1, {2a+1, 1}, then 4a+3, remainder is 0, moves to zero state with div_o is 1

rem_2
current number 4a+2
if x_i = 0, {4a+2, 0}, then 8a+4, remainder will be 1, moves to state (1)
if x_i = 1, {4a+2, 1}, then 8a+5, remainder will be 2, moves to state (2)
```

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

[ans link here](https://www.edaplayground.com/x/8qq2)
