# Question: Could you provide a concrete example on how a real bug influence mutation operator design?
# Response-to-Review: 
We take the mutation operator **DMI** in **Group: Data Mis-Access** as an example.
This mutation operator DMI is derived from the real bug **Misindexing**.
## Bug Description:
A misindexing bug occurs when a developer uses an incorrect index to extract information from a variable. For example, the IEEE754 [https://doi.org/10.1109/IEEESTD.1985.82928] standard defines the binary layout of 32-bit floating point, where the bits [22:0] are the fraction and the bits [30:23] are the exponent. However, in an implementation of floating point adder, the developer incorrectly extracted bits [23:0] as the fraction in a floating point adder, which lead to the wrong output value.
### Bug Source:
Real bugs from GitHub projects shows misindexing, and the commits details are presented below.
Verilog-axis(Verilog AXI Stream Components): https://github.com/alexforencich/verilog-axis/commit/76c805e4167c1065db0a7cdec711b30c1e11da91#diff-09b0ecbe0779c53e7a28b0d57be6ca8bd2f6224a339902399abed42ee0338d57

### Synthetic Code
```verilog
// Line 228
assign int_s_axis_tready[m] = int_axis_tready[select_reg*M_COUNT+m] || drop_reg; 
//M_COUNT should be S_COUNT, causing misindexing of the information extraction from int_axis_tready

// Line 296
wire s_axis_tvalid_mux = int_axis_tvalid[grant_encoded*S_COUNT+n] && grant_valid; //S_COUNT should be M_COUNT
//S_COUNT should be M_COUNT, causing misindexing of the information extraction from int_axis_tvalid
```

## Mutation Operator Design:
Based on the above bug description, mutation operator DMO is designed to reproduce this bug by implementing buffer overflow. The parameter of DMO is a new offset size for buffer access, which means a new offset size that can implement buffer overflow. For a assignment statement (e.g., line 3), DMO trace back the define of the variable _V_ (e.g., line 1) on the left side of the statement to determine its predefined bits _B_ (e.g., [N-1:0] in line 1). Then DMO sets a new offset size (e.g., offset in line 3) greater than _B_ for accessing _V_, thus reproducing the buffer overflow.
