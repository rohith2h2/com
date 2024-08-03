# LRU

we are asked to implement LRU scheme of way replacement policy for N-way set associative cache.

We have three inputs to the LRU block which are ls\_valid\_i, ls\_op\_i, ls\_way\_i and outputs are lru\_valid\_o and lru\_way\_o.\
ls\_valid\_i is the valid the signal which tells when the ls\_op\_i and ls\_way\_i are valid. ls\_op\_i is the operation signal which has three operations STORE, LOAD, INVALIDATE. when STORE operation is given the lru output lru\_way\_o tells which is the victim way to which particular data should be stored. When the LOAD operation is given it gives the ls\_way\_i information to tell which particular way got hit when it was load, and when INVALIDATE operation is given it tells which particular way is invalidated.&#x20;



```verilog
//LRU 


```
