# Edge Capture

Problem Statement:&#x20;

Implement a edge detector circuit which captures any neg edge( transitioning from 1 to 0) on 32 bit input signal. Edge detection and capture must be performed on bit basis for all 32 bits.



Example: input is 0xAB

which is 1010\_1011

then output would be 0000\_0000

in next cycle let input be 0xCA

which is 1100\_1010, now checking previous input and current bit by bit to detect 1->0 transition

1010\_1011

1100\_1010

output would be 0010\_0001, 21. And these are sticky edge detector, meaning the output would change in next cycle unless reset is applied.&#x20;
