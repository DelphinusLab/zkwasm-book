# IO Functions

## fetch inputs
Since a ZKP circuits distinguish between public inputs(instances) and private inputs(witness), ZKWASM supports two different way to fetch inputs from the user, **zkwasm_input(1)** reads an public instance from the user and **zkwasm_input(0)** reads a private input(witness).

```
extern "C" {
    zkwasm_input(bool) -> u64/i64
}
```

Suppose that a list of public inputs are specified via --public input1:i64 input2:i64 input3:i64 ..., then the following code should return an array of [input1, input2, input3]

```
fn get_inputs() -> [u64; 3] {
    [
        wasm_input(1),
        wasm_input(1),
        wasm_input(1),
    ]
}
```

## Input Witness Queue.
The ZKWASM run time supports multiple queue. When the ZKWASM runs an image, the input witness queues is initialzed by a sequence of u64 specified by the ZKWASM running API (See command line args --private). Once this queue is specified, we can not change it and all the witness can be fetched from the beginning (start from index 0) by **zkwasm_input(0)** one by one.

Suppose that a list of witness inputs are specified via --private input1:i64 input2:i64 input3:i64 ..., then the following code should return an array of [input1, input2, input3]

```
fn get_witness() -> [u64; 3] {
    [
        wasm_input(0),
        wasm_input(0),
        wasm_input(0),
    ]
}
```


During the execution, we have another two different kinds of witness queues. The default witness queue and the indexed witness queue.To insert and pop witness to default witness queue we can use **wasm_witness_insert** and **wasm_witness_pop**. To insert and pop indexed witness queue, we need to follow the convention of specifiying the witness queue index first (via **wasm_witness_set_index(index)**) and then calling **wasm_witness_indexed_pop** or **wasm_witness_indexed_insert**.


## Manipulate witness
extern "C" {
    /// inserts a witness at the current wasm_private inputs cursor
    pub fn wasm_witness_insert(u: u64);
    pub fn wasm_witness_pop() -> u64;
    pub fn wasm_witness_set_index(x: u64);
    pub fn wasm_witness_indexed_pop() -> u64;
    pub fn wasm_witness_indexed_insert(x: u64);
    pub fn wasm_witness_indexed_push(x: u64);
    pub fn require(cond: bool);
}

**Example add some witness to the default witness group**
```
unsafe {
    wasm_witness_set_index(i);
    wasm_witness_indexed_push(0x0);
    wasm_witness_indexed_push(0x1);
    wasm_witness_indexed_insert(0x2);
    let a = wasm_witness_indexed_pop();
    require(a == 0x1);
    let a = wasm_witness_indexed_pop();
    require(a == 0x0);
    let a = wasm_witness_indexed_pop();
    require(a == 0x2);
}
```

**Example add some witness to a witness group = i**
```
unsafe {
    wasm_witness_set_index(i);
    wasm_witness_indexed_push(0x0);
    wasm_witness_indexed_push(0x1);
    wasm_witness_indexed_insert(0x2);
    let a = wasm_witness_indexed_pop();
    require(a == 0x1);
    let a = wasm_witness_indexed_pop();
    require(a == 0x0);
    let a = wasm_witness_indexed_pop();
    require(a == 0x2);
}
```
