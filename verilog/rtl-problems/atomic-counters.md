# Atomic Counters

Q : Design a 64 bit event counter which would be interfaced with a 32 bit bus controlled by a microcontroller. 64 bit counter will be incremented whenever a trigger input is seen. Given that counter is read by 32 bit bus, we need two read requests to completely read the 64 bit counter. and also these two read access should be single copy atomic. First request would return lower 32 bit of counter and next request would return the upper 32 bit of counter. controller would ultimately concatenate to get 64 bit.

A : This is a counter problem. Problem statement : we need to read a 64 bit counter with a 32 bit peripheral bus which is controlled by a controller which want to read the entire 64 bit, while 32 bit cant do this in a single request we need 2 req.

Here goes the link : [click here](https://www.edaplayground.com/x/subt)

Firstly we will write logic for count, we use ff and for flopped count value, we will assign next count value which is current count value + \{{63{1'b0\}}, trig\_i}; next we will ack\_o value, as in the waveforms, ack\_o should be 1 clock cycle after req\_i, so we will use assign ack\_o = req\_q; which is flopped version of req\_i.

now logic is for holding the upper 32 bits in count\_msb whenever atomic\_q is seen. atomic\_q gets high then we give out the lower 32 bits and next atomic\_q goes low, in this we give higher bits. we use as a way to when to stop and give the count, use atomic\_q value to store the upper bits.

finally when using out we implement this

```
assign output_o[31:0] = req_q ? atomic_q ? count_q[31:0] : count_msb[31:0] : 32'h0;
```
