# Question: Could you provide a concrete example on how a real bug influence mutation operator design?
# Response-to-Review: 
We take the mutation operator DMI in Group: Data Mis-Access as an example.
This mutation operator DMI is derived from the real bug Misindexing.
## Bug Description:
A misindexing bug occurs when a developer uses an incorrect index to extract information from a variable.
### Bug Source:
Verilog-axis(Verilog AXI Stream Components): https://github.com/alexforencich/verilog-axis/commit/76c805e4167c1065db0a7cdec711b30c1e11da91#diff-09b0ecbe0779c53e7a28b0d57be6ca8bd2f6224a339902399abed42ee0338d57
A misindexing bug occurs when a developer uses an incorrect index to extract information from a variable.

Line 1 defines a buffer named mybuf consisting of N single-bit elements; mybuf [N-1:0] can be legally indexed from 0 to N-1 (inclusive). On Line 3, the snippet uses offset to assign a bit of mybuf to a value; however, the value of offset is greater than N and therefore overflows mybuf.

## Mutation Operator Design:
Based on the above bug description, mutation operator DMO is designed to reproduce this bug by implementing buffer overflow. The parameter of DMO is a new offset size for buffer access, which means a new offset size that can implement buffer overflow. For a assignment statement (e.g., line 3), DMO trace back the define of the variable _V_ (e.g., line 1) on the left side of the statement to determine its predefined bits _B_ (e.g., [N-1:0] in line 1). Then DMO sets a new offset size (e.g., offset in line 3) greater than _B_ for accessing _V_, thus reproducing the buffer overflow.
