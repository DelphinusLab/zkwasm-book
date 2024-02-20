# Debug Execution Trace Size of Your Code

## Motivation

Execution trace size is critical to the performance of generating zk proof. This doc provides a way to print out the execution trace size of any code when running prove.

## Example of code in [zkWasm-rust sdk](https://github.com/DelphinusLab/zkWasm-rust):

```
extern "C" {
    pub fn wasm_dbg(v:u64);
    pub fn wasm_trace_size() -> u64;
}
pub fn zkmain() {
  int traceSize = wasm_trace_size();
  // TODO: Insert your code here
  wasm_dbg(wasm_trace_size() - traceSize);
}
```
By compiling the code into a WASM image, you can follow [here](https://github.com/DelphinusLab/zkWasm?tab=readme-ov-file#command-line) to run prove on the WASM image. The trace size of your code will show up in cli output.

Alternatively, you can submit your prove task to [zkwasmhub](https://explorer.zkwasmhub.com/), and the `wasm_dbg` info will show up in the `Debug Logs` section in the task overview page. 

