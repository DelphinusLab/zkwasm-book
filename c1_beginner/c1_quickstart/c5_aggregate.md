# Aggregate (Batch) proofs

Suppose that we have generated a guest proof with its loading info and stored in the following directory.
```
.
├── Makefile
├── output.wasm
├── params
  ├── K18.params
  ├── K21.params
  ├── testwasm.circuit.finalized.data
  └── testwasm.zkwasm.config
├── output
  ├── traces
  ├── testwasm.loadinfo.json
  ├── testwasm.0.transcript.data
  └── testwasm.0.instance.data
```

Also suppose the overall information of the proof is stored in the zkwasm.loadinfo.json which looks like the following
```
{
  "k": 18,
  "proofs": [
    {
      "circuit": "testwasm.circuit.finalized.data",
      "instance_size": 4,
      "witness": "testwasm.0.witness.json",
      "instance": "testwasm.0.instance.data",
      "transcript": "testwasm.0.transcript.data"
    }
  ],
  "param": "K18.params",
  "name": "testwasm",
  "hashtype": "Poseidon"
}
```

Although the guest circuit proof is constant size, it is still big and its verify algorithm is a bit costy. In ZKWASM we have a structured layer of batching multi proofs into a simple small proof to reduce the cost of onchain verification. (Please refer to [proof batch](../../c2_advance/c3_proofgen/c2_batch.md))

Here, for simplification, we use ZKWASM batcher to batch the above proof into a small one.

Suppose that your project directory is the following:
```
.
├── Makefile
├── output.wasm
├── params
  ├── K18.params
  ├── K21.params
  ├── testwasm.circuit.finalized.data
  └── testwasm.zkwasm.config
├── output
  ├── traces
  ├── testwasm.loadinfo.json
  ├── testwasm.0.transcript.data
  └── testwasm.0.instance.data
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
cargo run --release --features cuda -- --param ./params --output ./output batch -k 22 --challenge sha --info output/testwasm.loadinfo.json  --name proofbatch --commits batchconfig.json
```

This will generates a batch proof `proofbatch.loadinfo.json` and the file structure in your project becomse the following
```
├── Makefile
├── output.wasm
├── params
  ├── K18.params
  ├── K22.params
  ├── proofbatch.circuit.data
  ├── proofbatch.vkey.data
  ├── testwasm.circuit.finalized.data
  └── testwasm.zkwasm.config
├── output
  ├── proofbatch.0.aux.data
  ├── proofbatch.0.instance.data
  ├── proofbatch.0.transcript.data
  ├── proofbatch.loadinfo.json
  ├── testwasm.loadinfo.json
  ├── testwasm.0.transcript.data
  └── testwasm.0.instance.data
```
while `the proofbatch.circuit.data` is the verify circuit of the batcher and `proofbatch.loadinfo.data` describes the proof of the batching result. Since design a product ready rollup process is much more complicated than just batching a few proofs of a single circuit, please refer to [advanced topics of building a rollup application](../../c2_advance/README.md) for more details.


