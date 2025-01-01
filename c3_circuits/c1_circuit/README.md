# ZKWASM Circuits 

A ZK Circuit in abstract is a matrix that each column represents a function $$f_i(w_j)$$ where $$j=1,2,3 \cdots$$ and a set of constraints $$C$$. A constraint in $$C$$ can be describe as a function equation $$H(f_i(w * g^j) = 0$$. Given an arbitrary instance of matrix, if all cloumn of the matrix satisfies the constraints $$C$$ then it is a valid instanation of the given circuit.

Futhermore, given a matrix, we have 3 different type of columns.

1. Advice columns. Value of advice columns can vary among different valid instantiations.
2. Fix columns. Value of fix columns must keep the same between different valid instantiations and the value of each fixed columns are provided at steup stage of the circuit.
3. Instance columns. Value of instance columns usually decides the other value of the matrix and is used as the input of the verifier. We call the process of deducing other values in the matrix via the instance column to be the synthesize process. 

## Circuits involved in ZKWASM
In ZKWASM, there are three different categories of circuits.
1. Guest Circuit. This is the main circuit of ZKWASM is the guest circuit which proves the execution of WASM bytecode and enforces the semantics of WASM specification.
2. Host Circuits. These are the precompiled circuits plugins for the guest circuit that provides pre-defined functions via the WASM hostop specification.
3. Proof Batching Circuits. These are circuits to connecting proofs together to form a compact proof.


## 5. Succinct Proof of the circuits
Compared with a standard WASM run-time, ZKWASM aims to provide a proof to prove that the output is valid so that it can be used in scenarios which require trustless and privacy computation. Moreover, the verifying algorithm needs to be simple in the sense of complexity to be useful inpractical. In zkWasm we have three different categories of circuits.

1. Guest circuits: enforces the semantics of slices of execution traces.
2. Host circuits: enforces the semantics of host API calls
3. Aggregation circuits: batch the guest circuits and host circuits and connect proofs of slices of execution.

A standard work flow of a execution proof contains the following steps:

1. Circuit Setup
Circuit Setup contains two phases. One is to compute the the values of all fixed columns and the other is to compute the values of all implicit columns derived from lookups and permutations. Once the circuit was setup, the commitment of each pre-calculated columns are stored into a vkey file and the other information of the circuit is stored in to a circuit data file which uniquely defines the circuit.

2. Circuit Synthesize
The synthesize process calculates all the witness of the circuit matrix based on the provided value of instance columns. Once a circuit instance has been synthesized, we can start proof generation of this instantiation.

3. Proof Generation
Proof is generated after the syntehsize stage via PCS(polynomial commitment scheme). In short, the proof contains the commit of constraint polynomials $$C(x)$$, the commitment of the vanish polynomial V(x) and the quotient polynomial $$h(x)$$. All these info are stored in a file called `proof transcript` (usually has file name `XXX.transcript.data`).

4. Proof Verification
The proof verification algorithm is simply open all the commintment of PCS to check that the polynomial constraints holds.
