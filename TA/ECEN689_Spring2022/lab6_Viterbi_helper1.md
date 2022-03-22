## Using a state machine for Viterbi decoding

### State 0
Take 2 bits of `codein` into the input port of the `bmu` module (`code`)  
Prep the input of the `pmu` module
Then jump to state 1

### State 1
Wait 2 cycles for the output of `pmu` to get valid  
Then jump to state 2

### State 2
Save the outputs of `pmu` to some registers  

If all 10 bits of `codein` are processed, jump to state 3
Otherwise, jump to state 0 for the next 2 bits of `codein`

### State 3
Check the last distance output of `pmu`, find the one with minimum PM  
Jump to state 4

### State 4
Based on the saved previous outputs in state 2, find the code word in each column, save it in the corresponding position in `codeout`.  
