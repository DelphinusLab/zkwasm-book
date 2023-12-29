# Quick Start

ZKWASM's philosophy is trying to provide a customizable code stack that users can interactive with the ZKWASM virtuial at different levels. This quick start is a tutorial that assumes the user runs ZKWASM VM as an integrated product that takes in a compiled WASM image, inputs for this image and then generates an execution proofs after running it.

Generating a ZK-proven application whose proof is deployment ready is broken down into three stages.

1. Setup Phase.
2. Proof Phase.
3. Deploy Phase.

In the Proof Phase a web assembly image is processed by zkWasm to generate a zkSNARK circuit for the application itself. The Proof Phase generates a zero knowledge proof corresponding to the applications execution against a guest and host virtual machine. Finally in the Deployment Phase a verification contract for the application is deployed on an EVM compatible network.<br>

Moreover, before we can generate and proof out of ZKWASM virtual machine, we need to prepare a WASM image that we are going to run. 
