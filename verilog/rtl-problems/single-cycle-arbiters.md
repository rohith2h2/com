# Single Cycle Arbiters

we will be learning about arbiters. What are arbiters? arbiter is digital circuit which selects one of different requesters available, using some predefined priority scheme. In a fixed priority arbiter, highest priority requester gets access first. If multiple requesters have same priority, then arbiter selects one requester in round robin fashion.

Now we will be seeing questions on arbiters:

\--> Single Cycle Arbiter:

```
You are leading a complex SoC which interacts with a lot of peripherals. As a result there is a need for an arbitration scheme to be used across the SoC. You decide to design a parameterized fixed priority arbitration scheme which grants a winner every cycle.

Design the parameterized single cycle arbiter module with fixed priority arbitration scheme. All the flops (if any) should be positive edge triggered with asynchronous resets.
### Interface Requirements

- Arbiter should grant a single winner (if any) every cycle
- The arbiter should use a fixed priority arbitration scheme
- Port[0] has the highest priority and priority decreases with incrementing port numbers

SOl:
So, there is a reg with req_i which is parameterized and gnt_o 
here we need to find which gnt_o should be high.

so we declare a new register new_req to hold the priority order. and the bit in new_req will 1, if there exists a higher priority than the current bit in req_i.
Its a bit confusing lets look at it with an example.

suppose req_i is 4'b1011, now the new_req value should be, here new_req[0] will always be 0, because there is no bit which has highest priority than this.
so 4'b1011       4'b1011
   4'b   0, next 4'b1110, because there exists a bit which has highest priority than req_i[1] which is req_i[0], hence new_req[1] is 1. This is how new_req is calculated. 
   Now gnt_o is calculated based on the ~(new_req) & req_i, when we do this 
   ~(new_req)   =  4'b0001
   & with req_i =  4'b1011
   gnt_o will be=  4'b0001
   which shows exactly in a single cycle position [0] requester should need to be prioritized.

```

[sol link here](https://www.edaplayground.com/x/DehS)
