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
