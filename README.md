# Question: Could you provide a concrete example on how a real bug influence mutation operator design?
# Response-to-Review: 
We take the mutation operator DMO in Group: Data Mis-Access as an example.
This mutation operator DMO is derived from the real bug Buffer Overflow.
## Bug Description:
A buffer overflow in an FPGA design occurs when a buffer is accessed with an offset that is greater than the size of the buffer. We present a basic code snippet for simplicity.
```Verilog
1 reg mybuf [N-1:0];   // a buffer with N 1-bit elements 
2 always @(posedge clk) 
3   mybuf[offset] <= value;   // offset >= N
```
Line 1 defines a buffer named mybuf consisting of N single-bit elements; mybuf [N-1:0] can be legally indexed from 0 to N-1 (inclusive). On Line 3, the snippet uses offset to assign a bit of mybuf to a value; however, the value of offset is greater than N and therefore overflows mybuf.

## Mutation Operator Design:
Based on the above bug description, mutation operator DMO is designed to reproduce this bug by implementing buffer overflow. The parameter of DMO is a new offset size for buffer access, which means a new offset size that can implement buffer overflow. For a assignment statement (e.g., line 3), DMO trace back the define of the variable _V_ (e.g., line 1) on the left side of the statement to determine its predefined bits _B_ (e.g., [N-1:0] in line 1). Then DMO sets a new offset size (e.g., offset in line 3) greater than _B_ for accessing _V_, thus reproducing the buffer overflow.
