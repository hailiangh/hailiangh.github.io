# Viterbi Decoder on FPGA
## Introduction
For this lab, we need to design Viterbi decoder. Viterbi decoder can be used to decode the code encoded by convolutional coding.

### Convolutional coding
Given a bitstream, we can encode it by performing convolution in a window of length K. The output from each time convolution is carried out is the parity bit. In each clock cycle, we can do r convolutions and get r parity bits. 
For example, assume the parity bits are calculated by: (This is one example, not necessarily always true)
p0[n]=x[n]⊕x[n-1]⊕x[n-2]
p1[n]=x[n]⊕x[n-1]

In the above case, r=2 since there are two parity bits in each cycle, and K=3 since the largest length of the convolution window is 3 (from x[n]  to x[n-2]). Note that the “XOR” here is the mod 2 addition. 

Suppose a bitstream is 101110, where the leftmost bit is the earliest bit (x[0]). We assume x[-1] and x[-2] are 0, then the first two parity bits (p0[0] and p1[0]) are 1 each, which gives 11. The second is 11, the third is 01, all the way to the sixth, which is 01. Therefore, the length of the output encoding code is r ×length(input bitstream).

### Viterbi decoder
Given an encoded bitstream, Viterbi decoder is employed to decode it. First, we need to find a reliable way to represent our encoding system. For the encoding example detailed above, we can represent it by the circuit shown in Figure 1.

## Implementation

## Submission
