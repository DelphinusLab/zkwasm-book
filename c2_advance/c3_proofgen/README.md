# Proof Generation Architecture
When executing a WASM image, its generated trace can be very long (several billion instructions) while the ZKWASM guest circuits has a limit on how many instructions it can proof in a single execution segment. Thus we need a scheme to split the huge execution into small pieces and generate the proof of them separately. After that we need to add extra connecting proofs to connect those separated proofs and then batch them together to generate a single proof for the whole execution.

In ZKWASM we assume an application is just like a standard desktop application that have a lifecycle of open, run and close. We say a single lifecycle from open to close is a bundle of execution while within that bundle there is a huge execution trace that contains billions of instructions. Furthermore, we split the execution of a bundle into many small execution segments.

The overall proof generation architecture is divided into three stages depends on the application executing style.

1. Segment proof: guest and host circuit proof of a single execution segment.
2. Continuation proof: connecting the segment proof and batch them into a single bundle execution proof.
3. Bundle proof batching: combine the continuation proof of each bundle and generate a single proofs (This is usually for an application rollup which needs to submit the state different to a Blockchain with a execution proofs).


<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/Setup_Phase_wbg.png">
  <img alt="title image light / dark." src="./assets/images/Setup_Phase_wbg.png" width="400">
</picture>
</p>

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/Proof_Phase_wbg.png">
  <img alt="title image light / dark." src="./assets/images/Proof_Phase_wbg.png" width="600">
</picture>
</p>

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/Deploy_Phase_wbg.png">
  <img alt="title image light / dark." src="./assets/images/Deploy_Phase_wbg.png" width="500">
</picture>
</p>


