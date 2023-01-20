# Question: Could you provide a concrete example on how a real bug influence mutation operator design?
# Response-to-Review: 
We take the mutation operator **DMI** in **Group: Data Mis-Access** as an example.
This mutation operator **DMI** is derived from the real bug **Misindexing**.
## Bug Description:
A misindexing bug occurs when a developer uses an incorrect index to extract information from a variable. For example, the IEEE754 (https://doi.org/10.1109/IEEESTD.1985.82928) standard defines the binary layout of 32-bit floating point, where the bits [22:0] are the fraction and the bits [30:23] are the exponent. However, in an implementation of floating point adder, the developer incorrectly extracted bits [23:0] as the fraction in a floating point adder, which lead to the wrong output value.
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
Based on the above bug description, mutation operator DMI is designed to reproduce this bug by implementing misindexing. The parameter of DMI is a new index value for extracting information from a variable, which means a new offset size that can implement misindexing. For an assignment statement, DMI parses the set of variables on the right side of the statement and modify the index of information extraction for variables where information extraction occurs. DMI adds or subtracts a value for the current information extraction index in practice, provided no buffer overflow occurs. We present a basic code snippet for simplicity from project reed_solomon_decoder/BM_lamda.v (https://github.com/hammad-a/verilog_repair/blob/master/benchmarks/opencores/reed_solomon_decoder/BM_lamda.v), which belongs to the benchmark in our paper. 

```verilog
// Coeerct Code
if (cnt == 0)
begin
    D<= S[K+e_cnt];
  end
else if (cnt < 5)
  begin
    add_pow1<= L[cnt+1];
    add_pow2<= S[K+e_cnt-cnt];
    div1<=0;
    add_1<= pow1 + pow2;
    IS_255_1<=(&pow1 || &pow2)? 1:0;
  end

//Mutated Code
if (cnt == 0)
begin
    D<= S[K+e_cnt];
  end
else if (cnt < 5)
  begin
    add_pow1<= L[cnt+2]; \\DMI mutates the L[cnt+1] to L[cnt+2]
    add_pow2<= S[K+e_cnt-cnt];
    div1<=0;
    add_1<= pow1 + pow2;
    IS_255_1<=(&pow1 || &pow2)? 1:0;
  end
```
The mutated statement is line 192 of BM_lamda.v. DMI modifies the information extraction index of variable L from [cnt+1] to [cnt+2], thus causing the wrong information extraction for L and reproducing the misindexing bug.
