# Aggregate (Batch) proofs

Suppose that we have generated a guest proof with its loading info as follows:

The overall information of the proof is stored in the zkwasm.loadinfo.json which looks like the following
```
{
  "circuit": "zkwasm.circuit.data",
  "k": 18,
  "instance_size": [
    2
  ],
  "transcripts": [
    "zkwasm.0.transcript.data"
  ],
  "instances": [
    "zkwasm.0.instance.data"
  ],
  "param": "K18.params",
  "name": "zkwasm",
  "hashtype": "Poseidon"
}
```

Although the guest circuit proof is constant size, it is still big and its verify algorithm is a bit costy. In ZKWASM we have a structured layer of batching multi proofs into a simple small proof to reduce the cost of onchain verification. (Please refer to [proof batch](../../c2_advance/c2_batcher/README.md))

Here, for simplification, we use ZKWASM batcher to batch the above proof into a small one.

Suppose that your project directory is the following:
```
.
├── Makefile
├── output.wasm
├── params
  ├── K18.params
  ├── K21.params
  ├── zkwasm.circuit.data
  └── zkwasm.vkey.data
├── output
  ├── etable.json
  ├── external_host_table.json
  ├── imtable.json
  ├── itable.json
  ├── jtable.json
  ├── mtable.json
  ├── zkwasm.loadinfo.json
  ├── zkwasm.0.instance.data
  └── zkwasm.0.transcript.data
```

1. write a simple empty batch config `batchconfig.json`:
```
{
    "equivalents": [
    ],
    "expose": [
    ],
    "absorb": []
}
```

2. clone the repo https://github.com/DelphinusLab/continuation-batcher

3. run the continuation batcher in batch mode:
```
cargo run --release --features cuda -- --param ./params --output ./output batch -k 22 --challenge sha --info output/zkwasm.loadinfo.json  --name proofbatch --commits batchconfig.json
```

This will generates a batch proof `proofbatch.loadinfo.json`.

