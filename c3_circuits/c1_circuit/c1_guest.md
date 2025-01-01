# Guest Circut

## Circuit Overview
The circuits of ZKWASM contains four global tables: execution table, image table, frame table and memory table, two assist tables: bit operation table and range table; and one host table which is used to connect host circuits that are implemented in separated external circuits.

### Global tables
1. The execution table represents the execution trace of the WASM bytecode. Thus the table is splited into sections and each section represents a transition function $$t_i$$. The semantics of each $$t_i$$ is decided by the opcode of the wasm byte code. Since there are more than 100 different opcodes in WASM specification, the opcode will be the indicator (switch) of which constraint group should be enabled for $$t_i$$.

2. The image table (pre & post) contains the snap shot of the memory when the execution trace start (the preimage) and finish (the postimage) executing.

3. The memory table contains the memory accessing record for each read and write of a memory or stack or image address. In WASM specification, memory, stack, image are in separated address spaces.

4. The frame table contains the record of function calling frames. In wasm specification, callers' instruction pointer is not recorded in the stack thus the semantics is enforced by the wasm runtime.

### Assist tables
1. Bitwise operation table: lookup table for bitwise opcodes operands and results.
2. Range table: enforces range check for u8, u16, etc ...

### Host tables
Host tables contains witness for host function calls. The operands and results are pure witnesses with no constraints. The constraints between inputs and outputs of host functions are proved in separated host (precompiled) circuits .

## IO of ZKWASM
The outermost function of ZKWASM is of type `zkmain(void): (void)`. To access the external IO, we use two builtin host functions: *zkwasm_input* and *zkwasm_output*.

At the circuit level, the public inputs/outputs are arranged in a separat column called instance column and the private inputs are pure witness.

Suppose that we have an column of instances [$$I_i$$] and a internal cursor $$k$$,  *zkwasm_input(1)* binds the $$k$$th instance to the top of the stack and similarly *zkwasm_output()* binds the current top of the stack with $$I_k$$. Both *zkwasm_input* and *zkwasm_output* increase the cursor $$k$$.

Unlike binding public instances, *zkwasm_input(0)* binds a value to the top of the stack with no constraints. Thus a common way to constraint those unconstrained witnesses is to hash them and enforces that the result of the hash is bind to some public instances (see the following pseudo code):
```
   let hash = Hash.push(zkwasm_input(0), ....);
   require(hash = [zkwasm_input(1), ...]);
```
