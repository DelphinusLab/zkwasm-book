# Batch ZKWASM proofs

## Description of the circuit
A circuit describes its params, gates, copy constraints and lookups. For any circuits involved in the DELPHINUS-ZKWASM system, we describe it using a circuit description file (usually with name circuitName.circuit.data). A circuit description info is usefull for a few ways.

1. used as a reference for batchers to batch proofs that related to the circuit
2. used as a reference for verifier generators to generate verifier based on the circuit
3. describes the `column name` of each circuit column for the commitment arithment circuit that connects different proofs.

### Example: Create a circuit description file for a sample circuit
```
git clone git@github.com:DelphinusLab/continuation-batcher.git
cargo test --release
```

This cargo test runs proofs of two sample circuits with circuit data `test1.circuit.data` and `test2.circuit.data`.

### Creating a circuit for your own circuit
Suppose that you have a halo2 circuit (defined in https://github.com/DelphinusLab/halo2-gpu-specific) and would like to generate a circuit data for it. You can use the following code:
```
fn store_info_full<E: MultiMillerLoop, C: Circuit<E::Scalar>>(
    params: &Params<E::G1Affine>,
    vkey: &VerifyingKey<E::G1Affine>,
    circuit: &C,
    cache_file: &Path,
) {
    log::info!("store vkey full to {:?}", cache_file);
    let mut fd = OpenOptions::new()
        .read(true)
        .write(true)
        .create(true)
        .truncate(true)
        .open(&cache_file)
        .unwrap();
    vkey.store(&mut fd).unwrap();
    store_pk_info(params, vkey, circuit, &mut fd).unwrap();
}
```
where `cache_file` is the path of your targeting circuit description file.

## Overview of the Batching Tool

### Generate batch proof from ProofLoadInfos

We support two modes of batching proofs. The rollup continuation mode and the flat mode. In both mode we have two options to handle the public instance of the target proofs when batching.
1. The commitment encode: The commitment of the target instance becomes the public instance of the batch proof.
2. The hash encode: The hash of the target instance become the public instance of the batch proof.

Meanwhile, we provide two openschema when batching proofs, the Shplonk and GWC and three different challenge computation methods: sha, keccak and poseidon. (If the batched proofs are suppose to be the target proofs of another round of batching, then the challenge method needs to be poseidon.)

### General Command Usage

The general usage is as follows:

```
cargo run --release -- --params [PARAMS_DIR] --output [OUTPUT_DIR] [SUBCOMMAND] --[ARGS]
```

where `[SUBCOMMAND]` is the command to execute, and `[ARGS]` are the args specific to that command.

The `--output` arg specifies the directory to write all the output files to and is required for all commands.
The `--params` arg specifies the directory to write all the params files to and is required for all commands.

### Batching Sub Command

```
USAGE:
    circuit-batcher batch [OPTIONS] --challenge <CHALLENGE_HASH_TYPE>... --openschema <OPEN_SCHEMA>...

OPTIONS:
    -a, --accumulator [<ACCUMULATOR>...]
            Accumulator of the public instances (default is using commitment) [possible values:
            use-commitment, use-hash]

    -c, --challenge <CHALLENGE_HASH_TYPE>...
            HashType of Challenge [possible values: poseidon, sha, keccak]

        --commits <commits>...
            Path of the batch config files

        --cont [<CONT>...]
            Is continuation's loadinfo.

    -h, --help
            Print help information

        --info <info>...
            Path of the batch config files

    -k [<K>...]
            Circuit Size K

    -n, --name [<PROOF_NAME>...]
            name of this task.

    -s, --openschema <OPEN_SCHEMA>...
            Open Schema [possible values: gwc, shplonk]
```

**Example:**

```
#! /bin/bash

params_dir="./params"
output_dir="./output"

if [ ! -d "$params_dir" ]; then
    # If it doesn't exist, create it
    mkdir -p "$params_dir"
else
    echo "./params exists"
fi

if [ ! -d "$output_dir" ]; then
    # If it doesn't exist, create it
    mkdir -p "$output_dir"
else
    echo "./output exists"
fi

# Get the resource ready for tests
cargo test --release --features cuda

# verify generated proof for test circuits
cargo run --release --features cuda -- --params ./params --output ./output verify --challenge poseidon --info output/test_circuit.loadinfo.json

# batch test proofs
cargo run --features cuda -- --params ./params --output ./output batch -k 22 --openschema shplonk --challenge keccak --info output/test_circuit.loadinfo.json --name batchsample --commits sample/batchinfo_empty.json


# verify generated proof for test circuits
cargo run --release --features cuda -- --params ./params --output ./output verify --challenge keccak --info output/batchsample.loadinfo.json

# generate solidity
cargo run --release -- --param ./params --output ./output solidity -k 22 --challenge keccak --info output/batchsample.loadinfo.json
```

## Tool Details
1. Describe circuits.
2. Generate the proofs of target circuits.
3. Define your batching policy via the batch DSL.
4. Execute the batching DSL and generate the batching circuit.
5. Generate the final solidity for your batching circuit.

### Proof Description
To describe a proof, we need to specify (in file name)
1. The circuit this proof related to.
2. The instance size of the proof.
3. The witness data if the proof have not been generated yet.
4. The proof transcript.

```
type ProofPieceInfo = {
  circuit: filename,
  instance_size: int,
  witness: filename,
  instance: filename,
  transcript: filename
}
```

### Description of a proof group
To batch a group of proofs together, the proofs themself need to be generated using the same param k (not necessarily the same circuit). When describing the group we provide the following fields:

```
type ProofGenerationInfo {
  proofs: Vec<ProofPieceInfo>
  k: int
  param: filename,
  name: string,
  hashtype: Poseidon | Sha256 | Keccak
}
```

This tool requires the target proofs (the proofs we want to batch) are all uses the **Poseidon** as hash functions for challenge generation. If the batch circuit is used to generate intermediate proofs for another batching circuit, then we need to specify the batch circuit to use the **Poseidon** hash for challenge generation as well. If the batch circuit is used to generate a final proof for verification on chain, then you should specify the challenge hash to be either Keccak or Poseidon.

### Handling the instances of target proofs
Suppose that we would like to batch our target proofs **T_i**, the batching circuit is **C_b**, the verifier of **C_b** is **V_b** and our batched proof is called proof **B**. If is important to understand the instance structure of **C_b**.

In this tool, we support two accumulator modes to pass the information of the instance of **T_i** to the instance of **C_b**, namely in **HashInstance** mode and **CommitInstance** mode.

![Alt text](./assets/images/prove-agg-instance-mode.png?raw=true "Two modes to carry the instances of target proofs")

## Description the batch schema when connecting proofs
When batch proofs, we are in fact writing the verifying function into circuits. Thus we need to specify the components of the circuits we used to construct the final verifying circuit. The main components of the verifing circuit contains the challenge circuit (the hash we use to generate the challenge), the ECC circuit (what is used to generate msm and pairing), the proof relation circuit (what is used to describe the relation between proofs, their instances, commitments, etc)

1. The challenge circuit has three different type
```
hashtype: Poseidon | Sha256 | Keccak
```

2. The ECC circuit has two options. One is to use the ECC circuit with lookup features. This circuit can do ECC operation with minimized rows and thus can be used to batch a relatively big amount of target circuits. The other option is to use a concise ECC circuit. This circuit do not use the lookup feature and thus generates a lot of rows when doing ECC operation. This ECC circuit is usually used at the last round of batch as the solidity for this circuit is much more gas effective.

3. The proof relation circuit is configurable. It can be described in a JSON with commitment arithmetic and the commitment arithmetic has three categories: equivalents, expose and absorb.

**Example: A simple proof relation sheet with commitment arithmetic**
```
{
    "equivalents": [
        {
            "source": {"name": "circuit_1", "proof_idx": 0, "column_name": "A"},
            "target": {"name": "circuit_2", "proof_idx": 0, "column_name": "B"}
        }
    ],
    "expose": [
        {"name": "test_circuit", "proof_idx": 0, "column_name": "A"}
    ],
    "absorb": []
}
```

### Specify the proof relation

There are a few scenarios we need to specify the constraints between commitments of different proofs.

#### Equivalents
Suppose that we have two circuits, **circuit_1** and **circuit_2**, they both have instances and witnesses namely **instances_1**, **instances_2**, **witness_1**, **witness_2**. It follows that, after batching the proofs, we lose the information of **witness_1** and **witness_2**. Thus to establish the connection between **witness_1** and **witness_2** we provide a configurable component in the batching circuit that allows user to specify equivalents between columns of **witness_1** and **witness_2**. when we put the following configuration into the proof relation sheet
```
{
    "equivalents": [
        {
            "source": {"name": "circuit_1", "proof_idx": 0, "column_name": "A"},
            "target": {"name": "circuit_2", "proof_idx": 0, "column_name": "B"}
        }
    ],
}
```
the batch will ensure the witness of column **A** of the first proof of **circuit_1** will equal to the witness of column **B** of the first proof of **circuit_2**.
<div><img src="./assets/images/commitment-equivalent.png?" width="60%"/></div>


#### Expose
Suppose that we have two groups of proofs that batched into **batch_proof_1** and **batch_proof_2** where **batch_proof_1** contains **proofA** and **batch_proof_2** contains **proofB**. It follows that we can not establish connections between the witness of **proofA** and **proofB** when batching **batch_proof_1** and **batching_proof_2** because **batch_proof_1** lost the track of witness of **proofA** and  **batch_proof_2** lost the track of witness of **proofB**. Thus to solve this problem, we provide the **expose** semantics when batching **batch_proof_1** and **batch_proof_2**. For example, if we want to constraint that witness column **A** of **proofA** is equal to the witness column **B** of **proofB**, we can first expose **A** of **proofA** in the proof relation sheet of **batch_proof_1** as follows
```
{
    "expose": [
        {
            {"name": "circuit_1", "proof_idx": 0, "column_name": "A"},
        }
    ],
}
```
and then expose **B** in the proof relation sheet of **batch_proof_2**. The expose of the witness will append three new instances to the instances of the batched proof which represents the commitment of the witness.
<div><img src="./assets/images/commitment-expose.png?" width="40%"/></div>

#### Absorb
Suppose that we have a batched proof **batch_proof_1** which contains **proof_1** and another proof **proof_2**. Then it follows that if we would like to establish a connection between witness **A** of **proof_1** and witness **B** of **proof_2**, we need not only expose **A** in the proof relation sheet of **batch_proof_1** but also provide a semantic for the batcher to ensure that the exposed commitment of **A** is equal to the commitment of **B** in **proof_2**. Since **proof_2** has not been batched yet, neither **equivalents** nor **expose** will work. Thus, we need a new semantic called **absorb** here.
```
    "absorb": [
	    {
		    "instance_idx": {"name": "single", "proof_idx": 1, "group_idx": 2},
		    "target": {"name": "single", "proof_idx": 0, "column_name": "post_img_col"}
	    }
    ]
```
<div><img src="./assets/images/commitment-absorb.png?raw=true" width="70%"/></div>


## Batching Circuit Generation
When generating a batch circuit, the basic ingredients are the verifying algorithms of each target cicruits. To describe a verifying algorithm we specify it into an algorithm AST (abstract syntax tree) and the basic operation in such AST is described as follows:
```
#[derive(Clone, Debug, Eq, PartialEq, Hash)]
pub enum EvalOps {
    TranscriptReadScalar(usize, EvalPos),
    TranscriptReadPoint(usize, EvalPos),
    TranscriptCommonScalar(usize, EvalPos, EvalPos),
    TranscriptCommonPoint(usize, EvalPos, EvalPos),
    TranscriptSqueeze(usize, EvalPos),

    ScalarAdd(EvalPos, EvalPos),
    ScalarSub(EvalPos, EvalPos),
    ScalarMul(EvalPos, EvalPos, bool),
    ScalarDiv(EvalPos, EvalPos),
    ScalarPow(EvalPos, u32),

    MSM(Vec<(EvalPos, EvalPos)>, EvalPos), // add last MSMSlice for dependence
    MSMSlice((EvalPos, EvalPos), Option<EvalPos>, usize), // usize: msm group

    CheckPoint(String, EvalPos), // for debug purpose
}

```

where the terms of the EvalOps are as follows:
```
pub enum EvalPos {
    Constant(usize),
    Empty,
    Instance(usize, usize),
    Ops(usize),
}

```
Thus verifying algorithm is a huge AST generated from the verification key of the target circuits. The motivation of doing so is to simplified the maintaince efforts of emitting the target verifying code. For example, if we would like to emit a native verifying binary, we can compile the AST into the native code. In the proof batching scenario, we compile the AST into circuits by providing all the circuits of EvalOps and then connecting them together based on the AST.

For example, https://github.com/DelphinusLab/halo2aggregator-s/blob/main/src/circuit_verifier/mod.rs implements a compiler that emits a halo2 circuits for the verifying algorithm. In addition to circuit generation, we can also use the solidity generator to generate a verifying contract by implementing a compiler like https://github.com/DelphinusLab/halo2aggregator-s/blob/main/src/solidity_verifier/codegen.rs. More over, you can generate an R1CS circuit for the verifying algorithm (useful in the last round batching to reduce gas fee) by implementing a compiler that targets circom as a backend target.



