# Using the ZKWASM proof batcher

## Description of the circuit
A proof can not been generated without a predefined zk circuit. A circuit describes its params, gates, copy constraints and lookups. When a proof is generated, use a proof description file to describe the summary of it as follows:

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
