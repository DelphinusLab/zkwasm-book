# Setup ZKWASM circuit

With the wasm bytecode being generated, we can start setup the circuit of the VM. In the Setup Phase an application as a wasm image is used as input to zkWasm to generate a zkSNARK circuit for the application. The setup phase generates the output vkey.data for the image which is a circuit that commits its constant column.

There are two modes to setup ZKWASM, the uniform-circuit mode and the image specific mode.

## The unifirom-circuit mode
When the zkWASM is run in the uniform-mode, its circuit is designed for all WASM images and the bytecode of the WASM image are witness of a certain column of the ZKWASM guest circuits. When a proof is generated using the uniform ZKWASM circuit, one need to provide the image commitment to the verifier to verify that certain proof is generated for a particular ZKWASM image.

Within the directory `$WASMBIN` with the program build using [environment setup](./c1_install.md), run the setup zkWasm with the input .wasm image (`ouput.wasm`) from above as follows.

```
cargo run --release --features uniform-circuit -- --params ./params testwasm setup --host standard
```

This produces the output files in `./params/`
```
.
├── Makefile
├── output.wasm
├── params
  ├── K18.params
  ├── testwasm.circuit.finalized.data
  └── testwasm.zkwasm.data
└── zkmain.c/zkmain.rs
```

## The single image mode
When the zkWASM is run in the single image mode (which is the default mode), its circuit is generated for a specific WASM image and the bytecode of the WASM image are fixed values of a certain column of the ZKWASM guest circuits. When a proof is generated using the single mode ZKWASM circuit, one do not need to provide the image commitment to the verifier and only the proof for that particular image can be verified using the generated verifier.

Within the directory `$WASMBIN` with the program build using [environment setup](./c1_install.md), run the setup zkWasm with the input .wasm image (`ouput.wasm`) from above as follows.

```
cargo run --release -- --params ./params testwasm setup --host standard --wasm $PROJECT/output.wasm
```

This produces the output files in `./params/`
```
.
├── Makefile
├── output.wasm
├── params
  ├── K22.params
  ├── testwasm.circuit.finalized.data
  └── testwasm.zkwasm.data
└── zkmain.c/zkmain.rs
```

As the default mode is more friendly for beginers that might only focus on a particular application, the following content will assume the default mode is enabled. (However all examples should also works in uniform-mode).

## The continuation mode
When you are running a wasm image with unbounded execution trace, you need to use the continuation mode of zkWasm which will splits the execution trace into slices and generate proofs for each slice.
```
cargo run --release --features continuation --params ./params testwasm setup
```
This produces the output files in `./params/`
```
.
├── Makefile
├── output.wasm
├── params
  ├── K22.params
  ├── testwasm.circuit.finalized.data
  ├── testwasm.circuit.ongoing.data
  └── testwasm.zkwasm.data
└── zkmain.c/zkmain.rs
```


