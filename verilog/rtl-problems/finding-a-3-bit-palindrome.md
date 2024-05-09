# Finding a 3 bit palindrome

three bit palindrome, similar to previous problem, we have a module where a incoming stream of bits are coming. and need to know the number is palindrome or not.

So, the idea here is we need to know a number is palindrome or not, we need to check the msb of number and lsb. if they are same its a palindrome.

for this we implement a shift register, which shifts the lsb bit to left and assign new x\_i bit to lsb. and a counter to count upto 2 to start checking palindrome\_o.

its a simple problem, \`\`\`

```
assign next_count = count_q[1] ? count_q : count_q + 2'b01;
assign next_shift_reg = {shift_reg_q[0], x_i};

assign palindrome_o = (shift_reg_q[1] == x_i) & count_q[1];
```

these are the logics to find the palindorme or not. [link](https://www.edaplayground.com/x/NcdF)
