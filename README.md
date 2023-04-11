# Question: The experimental results must include discussion on real-life/practical design bugs in the chosen designs as these bugs may be significantly different from the types of mutants considered in this paper.
# Response-to-Review: 
We take a buggy design from project *reed_solomon_decoder* as an example.
This buggy design is derived from the real bug **Misindexing**.
## Bug Description:
A misindexing bug occurs when a developer uses an incorrect index to extract information from a variable. For example, the IEEE754 (https://doi.org/10.1109/IEEESTD.1985.82928) standard defines the binary layout of 32-bit floating point, where the bits [22:0] are the fraction and the bits [30:23] are the exponent. However, in an implementation of floating point adder, the developer incorrectly extracted bits [23:0] as the fraction in a floating point adder, which lead to the wrong output value.
### Bug Source:
Real bugs from GitHub projects shows misindexing, and the commits details are presented below.
Verilog-axis(Verilog AXI Stream Components): https://github.com/alexforencich/verilog-axis/commit/76c805e4167c1065db0a7cdec711b30c1e11da91#diff-09b0ecbe0779c53e7a28b0d57be6ca8bd2f6224a339902399abed42ee0338d57

### Real-world code snippets
```verilog
// Line 228
assign int_s_axis_tready[m] = int_axis_tready[select_reg*M_COUNT+m] || drop_reg; 
//M_COUNT should be S_COUNT, causing misindexing of the information extraction from int_axis_tready

// Line 296
wire s_axis_tvalid_mux = int_axis_tvalid[grant_encoded*S_COUNT+n] && grant_valid; //S_COUNT should be M_COUNT
//S_COUNT should be M_COUNT, causing misindexing of the information extraction from int_axis_tvalid
```

## Bug Design:
Based on the above bug description, the buggy version in our benchmark is designed to reproduce this bug by implementing misindexing. For an assignment statement, we parses the set of variables on the right side of the statement and modify the index of information extraction for variables where information extraction occurs. Then we add or subtracts a value for the current information extraction index in practice, provided no buffer overflow occurs. We present a basic code snippet for simplicity from project reed_solomon_decoder/BM_lamda.v (https://github.com/hammad-a/verilog_repair/blob/master/benchmarks/opencores/reed_solomon_decoder/BM_lamda.v), which belongs to the benchmark in our paper. 

### Buggy code snippets in our benchmark:

```verilog
// Correct Code
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

//Buggy Code
if (cnt == 0)
begin
    D<= S[K+e_cnt];
  end
else if (cnt < 5)
  begin
    add_pow1<= L[cnt+2]; //Buggy statement, the L[cnt+1] is changed to L[cnt+2]
    add_pow2<= S[K+e_cnt-cnt];
    div1<=0;
    add_1<= pow1 + pow2;
    IS_255_1<=(&pow1 || &pow2)? 1:0;
  end
```
The buggy statement is line 192 of BM_lamda.v. We modifies the information extraction index of variable L from [cnt+1] to [cnt+2], thus causing the wrong information extraction for L and reproducing the misindexing bug.
