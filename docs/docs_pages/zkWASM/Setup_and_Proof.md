---
layout: default
---

# Generating a Proof

Generating a ZK-proven application whose proof is deployment ready is broken down into three stages.

1. Setup Phase.
2. Proof Phase.
3. Deploy Phase.

In the Proof Phase a web assembly image is processed by zkWasm to generate a zkSNARK circuit for the application itself. The Proof Phase generates a zero knowledge proof corresponding to the applications execution against a guest and host virtual machine. Finally in the Deployment Phase a verification contract for the application is deployed on an EVM compatible network.<br>

## Image Generation:

An application is written in a language of the developers choice and compiled to web assembly. In the following we compile a
 simple C program shown below using the sdk in Delphinus Lab's [zkWasm-C](https://github.com/DelphinusLab/zkWasm-C) repo. This web assimbly image will be used in the Setup Phase below.

```C
#include "zkwasmsdk.h"

__attribute__((visibility("default")))

int zkmain() {
  int sum = wasm_input(1);
  return sum + 1;
}
```

Creating a new directory called `simple_program` in `./zkWasm-C/`, then move in to `./zkWasm-C/simple_program/` and run the below. This creates the `.c` program and `Makefile`.

```console
cd ./simple_program/
cat > ./simple_program.c << EOF
#include "zkwasmsdk.h"

__attribute__((visibility("default")))

int zkmain() {
  int sum = wasm_input(1);
  return sum + 1;
}
EOF
cat > ./Makefile << 'EOF'
LIBS  = -lkernel32 -luser32 -lgdi32 -lopengl32
SDK_DIR = ../../sdk
CFLAGS = -Wall -I$(SDK_DIR)/c/sdk/include/ -I$(SDK_DIR)/c/hash/include/

# Should be equivalent to your list of C files, if you don't build selectively
CFILES = $(wildcard *.c)
ifeq ($(CLANG),)
CLANG=clang-15
endif
FLAGS = -flto -O3 -nostdlib -fno-builtin -ffreestanding -mexec-model=reactor --target=wasm32 -Wl,--strip-all -Wl,--initial-memory=131072 -Wl,--max-memory=131072 -Wl,--no-entry -Wl,--allow-undefined -Wl,--export-dynamic

all: output.wasm

sdk.wasm:
	sh $(SDK_DIR)/scripts/build.sh sdk.wasm

output.wasm: $(CFILES) sdk.wasm
	$(CLANG) -o $@ $(CFILES) sdk.wasm $(FLAGS) $(CFLAGS)


clean:
	sh $(SDK_DIR)/scripts/clean.sh
	rm -f *.wasm *.wat
EOF
```

Compile the `.c` application using `make` to create the corresponding Wasm image. The directory should look like the below:

```
.
├── Makefile
├── output.wasm
├── sdk.wasm
└── simple_program.c
```

The file `ouput.wasm` will be the input image for zkWasm's Setup Phase below.


## Setup Phase:

In the Setup Phase an application as a wasm image is used as input to zkWasm to generate a zkSNARK circuit for the application. The setup phase generates the output vkey.data for the image which is a circuit that commits its constant column. In other words it is a program that has been fixed. In following proof is submissions, zkWasm will open and check the commitment of this vkey to make sure the same program is being executed. This is the combination of the .param and vkey.data files.<br>

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/Setup_Phase_wbg.png">
  <img alt="title image light / dark." src="./assets/images/Setup_Phase_wbg.png" width="400">
</picture>
</p>

Within the directory `./zkWasm/` with the program build using [environment setup](./Environment.md), run the setup stage of zkWasm with the input .wasm image (`ouput.wasm`) from above.


```
mkdir output_simple_program
cargo run --release -- --function zkmain --output ./output_simple_program --wasm $HOME/zkWasm-C/tests/simple_program/output.wasm setup
```

This produces the output files in `./output_simple_program/`
```
.
├── K18.params
├── K21.params
└── zkwasm.0.vkey.data
```



## Proof Phase:

In the Proof Phase the web assembly interpreter (wasmi) outputs two traces; the web assembly bytecode as the execution trace & the host API call trace. These record
the order of the host API calls and their output. The combination of the two traces ensures the host call trace is in the same order as the execution trace (including the same input arguments).<br>



<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/Proof_Phase_wbg.png">
  <img alt="title image light / dark." src="./assets/images/Proof_Phase_wbg.png" width="600">
</picture>
</p>


To generate a single proof using zkWasm using the wasm image from above `/simple_program/output.wasm` but this time we add a public input to the application `--public 112:i64`.

Within zkWasm generate a single proof:

```console
cargo run --release -- --function zkmain --output ./output_simple_program --wasm $HOME/zkWasm-C/tests/simple_program/output.wasm single-prove --public 112:i64
```

This produces the following output in the output directory `--output ./output_simple_program`:

```
.
├── etable.json
├── external_host_table.json
├── imtable.json
├── itable.json
├── jtable.json
├── K18.params
├── K21.params
├── mtable.json
├── zkwasm.0.instance.data
├── zkwasm.0.transcript.data
└── zkwasm.0.vkey.data
```

The process of proof generation requires reading the `vkey.data` and `K.params` files generated in the previous Setup Phase, which is used as input to zkWasm which creates a proof outputting guest `transcript.data` (the proof) and `instance.data` (the public input). For the Gust virtual machine. Along side the Host virtual machine creates its own proof using the `host.vkey` and Host call trace, outputting host `transcript.data`.<br>

The two proofs (`transcript.data` & `host-transcript.data`) are processed by a setup round in a batcher.


### Verification:

The verification of a proof generated by zkWasm is done using the `single-verify` command line arguement. In the below bash script which is execuated from within `output_simple_program` a single verification run can be done by:

```
cat > ./single_verify.sh << 'EOF'
#!/bin/bash

ZKWASM_DIR="$HOME/zkWasm"
WASMIMAGE_DIR="$HOME/zkWasm-C/tests/simple_program"
OUTPUT_DIR="$ZKWASM_DIR/output_simple_program"

set -e
set -x

# Single test
$ZKWASM_DIR/target/release/cli -k 18 \
    --function zkmain \
    --output $OUTPUT_DIR \
    --wasm $WASMIMAGE_DIR/output.wasm \
    single-verify \
    --proof $OUTPUT_DIR/zkwasm.0.transcript.data \
    --instance $OUTPUT_DIR/zkwasm.0.instance.data
EOF
chmod +x single_verify.sh
./single_verify.sh
```

Where:

`ZKWASM_DIR` is the directory of zkWasm.
`WASMIMAGE_DIR` is the directory of the simple_program output.wasm image (generated and used above).
`OUTPUT_DIR` is the output directory `./output_simple_program`.




## Deploy Phase:

In the Deployment Phase a verification contract is deployed on an Ethereum testnet.

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/Deploy_Phase_wbg.png">
  <img alt="title image light / dark." src="./assets/images/Deploy_Phase_wbg.png" width="500">
</picture>
</p>





## Poseidon Example:

In the following we use the poseidon example from Delphinus Lab's zkWasm-C [repo](https://github.com/DelphinusLab/zkWasm-C)

Using the make file generate `output.wasm` from within

[back](./../../)