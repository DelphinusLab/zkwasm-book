# ZKWASM aggregation circuit

## Overview
Proof aggregation (or batching if there are no connections between the target circuits) in the context of Zero-Knowledge Proofs (ZKP) refers to a method where multiple proofs can be combined into a single, compact proof. This technique is particularly important for scaling blockchain networks and other systems that rely on cryptographic proofs, as it significantly reduces the amount of data that needs to be processed and verified.

Aggregating proofs from different systems or protocols can be challenging due to differences in cryptographic assumptions, algorithms, and security models. In this document, when we talk about proof aggregation, we refer to the process of connect or batch multiple proofs of same or different circuits with the same circuit size and same PCS (KZG in particular).

## Verifier circuit
The key concept of proof aggregation is how to verify multiple proofs in a single constaint size algorithm. If we relax the requirement of constant size of the verification algorithm, then we might endup with a huge verification cost. A very basic but fundamental idea of proof aggregation is to write the proof verify algorithm in circuit (verification circuit). This technique transform the verification of the initial proof in to verifying the proof of the varification circuit.

To verifying multiple proofs $P_i$ of different circuit $C_i$ we can write $V_i$ into circuit and combine them linearly (in circuit) to generate a new circuit that performs the following check.

```
V_0(P_0);
V_1(P_1);
V_2(P_2);
...
V_n(P_n);
```
(Please refer to (https://github.com/DelphinusLab/continuation-batcher) for automatically generate verifying-circuit for a given circuit)

## Combine multiple verifier circuits
As we know the basic idea of combining the verifier cicuits, we can now define the instances and witness (private instances) of $V_i$. Suppose that $V_i$ is the verification circuit of $C_i$ and $P_i$ is a proof of $C_i$. It follows that the commitment of the instance of $C_i$ becomes the public instance of $V_i$ and $P_i$ becomes the witness of $V_i$ and the verification circuit can be written in a more precise form

```
V_0(c_0, P_0);
```

## commitment arithment dsl

## batching related column for guest circuits

## batching related column for host circuits

## batching related column for itself
