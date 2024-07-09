# Concepts

1. Why we need both types of coverage(code and functional) ?

Getting to 100% code coverage, doesnt necessarily tell the complete story. Example, imagine a simple AND gate, to get to 100% code coverage, we only need two different types of input combinations - 00 and 11, these two will produce all the output combinations. Then the coverage tells all the code was exercised but we dont know whether our design works or not. all we know is input and outputs of our design are able to take on both logic 0 and logic 1. We couldn't certainly tell our design is an AND gate, it could also be an OR gate.  And this is where functional coverage comes in, So the first thing funtional coverage tells is have observed all the possible values on input. and checking the input and its output tells us which gate we are looking at.&#x20;



Also getting 100% functional coverage is not enough either, Imaginer we have two AND gates whos outputs are given to OR gate and output is taken there. Now we might simply attain 100% functional coverage by defining coverpoints for one AND gate and final output but not exercising the other AND gate.This is called coverage hole and This is where Code coverage becomes crucial, Cause it shows there is a part of code which is not exercised at all.&#x20;



And getting 100% on both code and functional coverage doesn't mean our design is bug free. But it gives confidence, that our design in thoroughlt exercised based on the specifications provided.&#x20;





