# Using the ZKWASM proof batcher

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

### Generate the circuit description file of ZKWASM.
The ZKWASM command line tool will generate the circuit info for your automatically after you have finished your setup.


A proof can not been generated without a predefined zk circuit.  When a proof is generated, use a proof description file to describe the summary of it as follows:

```
{
  "circuit": "test1.circuit.data",
  "k": 22,
  "instance_size": [
    1
  ],
  "transcripts": [
    "test1.0.transcript.data"
  ],
  "instances": [
    "test1.0.instance.data"
  ],
  "param": "K22.params",
  "name": "test1",
  "hashtype": "Poseidon"
}
```
