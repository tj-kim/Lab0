## Lab Overview
In Lab 0, we build a 4 bit adder using a 1 bit adder from a previous homework assignment all in verilog. We then program the FPGA to run the 4- bit adder.

## Test Strategy
![Test Table](https://github.com/tj-kim/Lab0/blob/master/test_table.png)

The above table shows the 16 test cases we used for this lab. The same test cases were used both for the test bench simulation and the FPGA testing, as we believed that the cases very pretty extensive and touched a lot of different scenarios.

We tested the following combination of scenarios.
* What happens if we add 2 numbers with different signs? (negative + negative, positive + positive, positive + negative)
* What happens if we add 2 numbers with the different magnitudes? (same magnitudes, different  magnitudes)
* What happens if we test for different cases for overflow and carryout? (overflow and carryout, no overflow and carryout, overflow and no carryout, neither overflow nor carryout)
* What happens when one or both of the values we are adding are zero?
* What happens if we keep the same values but switch the order we add them?

## Test Case Failures and Changes
Test Case 1

| A3 | A2 | A1 | A0 | B3 | B2 | B1 | B0 | OF | S3 | S2 | S1 | S0 |
|----|----|----|----|----|----|----|----|----|----|----|----|----|
| 0  | 0  | 0  | 0  | 1  | 1  | 1  | 1  | 0  | 1  | 1  | 1  | 1  |
| 0  | 0  | 0  | 1  | 0  | 0  | 0  | 0  | 0  | 0  | 0  | 0  | 1  |
| 0  | 0  | 0  | 1  | 0  | 0  | 0  | 1  | 0  | x  | x  | x  | 0  |
| 0  | 0  | 0  | 1  | 0  | 0  | 1  | 0  | 0  | 0  | 0  | 1  | 1  |
| 0  | 0  | 0  | 1  | 0  | 0  | 1  | 1  | 0  | x  | x  | x  | 0  |
| 0  | 0  | 0  | 1  | 0  | 1  | 0  | 0  | 0  | x  | x  | x  | 1  |

### Notes and Observations
* Here we used the exhaustive case for the initial test to get a bigger picture of where our problems were occurring. We only included a portion of that for this explanation (We did not read the part of the lab telling us that we wanted around 16 cases yet).
* The only times there seemed to be no errors was when one of the values added was ‘0000’ or when there was a lack of obvious carryout involved.
* However, in some cases there were no carryouts but still errors were present.
* We did not mention what the final carryouts were in this case.

### Fixes
* We were confused between what ‘carryout’ and what ‘overflow’ was, so reviewed the rules of 2’s complement to refine our understanding. We added both to the test case table for future iterations.
* We were not utilizing intermediate wires in our FullAdder4bit modules, and we were misusing the output ‘carryout’ as such. This was causing most of the errors.
```verilog
module FullAdder4bit
// Other code cut out for saving space in report
    // Your Code Here
  structuralFullAdder adder0 (sum[0], carryout, a[0], b[0], 0);        // adding 0th bit of a and b
  structuralFullAdder adder1 (sum[1], carryout, a[1], b[1], carryout); 
endmodule
```
* We added intermediate wires in the system to fix this problem, and assigned carryin/carryout values between the 1 bit adders.
```verilog
module FullAdder4bit
    // Your Code Here
  wire co0, co1, co2; // Intermediate wires that are carryouts, and then carryins
  structuralFullAdder adder0 (sum[0], co0, a[0], b[0], 0);        // adding 0th bit of a and b
  structuralFullAdder adder1 (sum[1], co1, a[1], b[1], co0); 
endmodule
```

Test Case 2
| A3 | A2 | A1 | A0 | B3 | B2 | B1 | B0 | CO | S3 | S2 | S1 | S0 | OF | Case |
|----|----|----|----|----|----|----|----|----|----|----|----|----|----|------|
| 0  | 0  | 0  | 1  | 0  | 0  | 0  | 1  | 0  | 0  | 0  | 1  | 0  | 1  | ++NO |
| 0  | 1  | 1  | 0  | 0  | 1  | 1  | 0  | 0  | 1  | 1  | 0  | 0  | 0  | ++O  |
| 1  | 1  | 1  | 1  | 1  | 1  | 1  | 1  | 1  | 1  | 1  | 1  | 0  | 1  | --NO |
| 1  | 0  | 1  | 0  | 1  | 0  | 1  | 0  | 1  | 0  | 1  | 0  | 0  | 0  | --O  |
| 0  | 1  | 0  | 1  | 1  | 1  | 0  | 0  | 1  | 0  | 0  | 0  | 1  | 0  | +-NO |

### Notes and Observations
* We improved our test case table as we added carryout and case informations to it. We also decreased the number of test cases to 5, which we decreased too much (we still did not read the part about the 16 test cases yet).
* The Case indicator in the table tells us what is the sign of the 2 values that are added, and whether overflow will happen or not (+ = positive, - = negative, NO = No Overflow, O = Overflow).
* Other than the positive + negative case, the overflow flag seems to be flipped. We are returning 1 for overflow for cases with no overflow and vice versa.

### Fixes
Pre-fix code
```verilog
module FullAdder4bit
// Other code cut out for saving space in report
    // Your Code Here
 structuralFullAdder adder1 (sum[1], carryout, a[1], b[1], carryout); 
// Other code here cut to save space in report
Wire n_overflow;
‘XOR Xor_3(n_overflow, sum[3], carryout);
`NOT N_Xor_3(overflow, n_overflow);  // overflow if sum[3] and carryout are different
endmodule
```
* We were getting the correct sum and carryout values so we checked the post 1-bit adder step in our code for the problem. The problem was that we were using NXOR to compute the overflow value instead of just XOR. This problem was caused by our lack of understanding of how to compute overflow in verilog, but we figured it out after looking at both the test case table and our code.
* Moreover, for the positive + negative case, we were not getting the correct overflow flag once the problem above was fixed as all overflow values were inverted. We solved this by fixing our way of computing overflow. Before this, we thought that overflow was found by comparing values of the leftmost bit of the sum and the final carryout. We now compare the final 2 carryout values and not the leftmost bit of the sum.
* Lastly, we expanded our test case table to 16 cases as we finally got to read through all of the lab. The reasoning and results of the test simulation are mentioned above, in the Test Case Strategy portion of this report.
Post-fix code
```verilog
module FullAdder4bit
// Other code cut out for saving space in report
    // Your Code Here
 structuralFullAdder adder1 (sum[1], carryout, a[1], b[1], carryout); 
// Other code here cut to save space in report
`XOR Xor_3(overflow, co2, carryout);  // determining overflow by comparing values of final 2 carry outs
endmodule
```

## Propagation Delays
Using the test cases described above, we generated the following waveforms in GTKWave:

![Waveforms Tests 1-8](https://github.com/tj-kim/Lab0/blob/master/waveform_1.png)
![Waveforms Tests 9-16](https://github.com/tj-kim/Lab0/blob/master/waveform_2.png)

By inspection, the worst propagation delay occurred when adding 0b0001 and 0b1111. As determined by the length of the bars, the sum was resolved in 11 gate delays, the carryout in 12 gate delays, and the overflow in 13 gate delays. This is consistent with our theoretical calculations for the propagation delay.

![One-bit adder diagram](http://www.circuitstoday.com/wp-content/uploads/2010/04/Full-Adder-Circuit.gif)

The worst-case propagation delays for our 1-bit adder (circuit diagram source: http://www.circuitstoday.com/half-adder-and-full-adder) are: 

|       | A               | B               | Cin      |
|-------|-----------------|-----------------|----------|
| S     | 2 XOR           | 2 XOR           | 1 XOR    |
| Cout  | 1 XOR, 2 AND/OR | 1 XOR, 2 AND/OR | 2 AND/OR |

For the 4-bit adder, we assumed XOR gates have a delay of 1 time step. However, they are actually made up of a combination of ANDs and ORs, so realistically, XORs should actually have 2 gate delays. Just to be consistent with our simulation results, we will assume that XORs have 1 gate delay in the propagation delay calculations below:

|    | AB0            | AB1     | AB2   | AB3 |
|----|----------------|---------|-------|-----|
| S0 | 3              | -       | -     | -   |
| C0 | 2              | -       | -     | -   |
| S1 | 3+2            | 3       | -     | -   |
| C1 | 3+3            | 2       | -     | -   |
| S2 | 3+3+2          | 3+2     | 3     | -   |
| C2 | 3+3+3          | 3+3     | 2     | -   |
| S3 | 3+3+3+2 = 11   | 3+3+2   | 3+2   | 3   |
| C3 | 3+3+3+3 = 12   | 3+3+3   | 3+3   | 2   |
| OF | 3+3+3+3+1 = 13 | 3+3+3=1 | 3+3+1 | 3+1 |

As expected, the worst-case delay for the sum is 11, the carryout is 12, and the overflow is 13.