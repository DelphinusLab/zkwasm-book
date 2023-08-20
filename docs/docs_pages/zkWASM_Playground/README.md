---
layout: default
---

# zkWASM Playground

The [Delphinus Lab Service Explorer](https://zkwasm-explorer.delphinuslab.com/) provides a convenient user interface to:

1. Submit Web Assembly images where zkWasm generates zkSNARK circuits for the image.
2. Prove a Web Assembly image execution.
3. Deploy a verification contract on an Ethereum Testnet.

To interact with Delphinus Lab service [install metamask](https://metamask.io/download/) as a browser extension and obtain some testnet coins.

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/0_metamask.png">
  <img alt="title image light / dark." src="./assets/images/0_metamask" width="363">
</picture>
</p>


Paste the below example C source in to [WasmFiddle](https://wasdk.github.io/WasmFiddle/) and hit the Build button in the top center of the page.


```C
unsigned long long wasm_input(int); // external host api for fetch user inputs.

int zkmain() {
  int sum = wasm_input(1);
  int b = sum + 2;
  return b + 1;
}
```

Save the build application as `example.wasm` to a local directory on your machine.

## Submitting a New Application:

Using upper right task bar on [Delphinus Lab Service Explorer](https://zkwasm-explorer.delphinuslab.com/) click `Create New Application` and upload the `example.wasm` in the "Image ID (MD5)" field.


### Setup Phase:

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/1_taskbar.png">
  <img alt="title image light / dark." src="./assets/images/1_taskbar.png" width="700">
</picture>
</p>




<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/2_new_application.png">
  <img alt="title image light / dark." src="./assets/images/2_new_application.png" width="700">
</picture>
</p>

You can see that the `TASK ID 64d09a94f0e3eee93f7e8e04` has moved from "Processing" to the "Done" state.

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_3_applicatioprocessing.png">
  <img alt="title image light / dark." src="./assets/images/test02_3_applicatioprocessing.png" width="700">
</picture>
</p>

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_4_applicationdone.png">
  <img alt="title image light / dark." src="./assets/images/test02_4_applicationdone.png" width="700">
</picture>
</p>



Inspecting the task in the explorer shows the Application Image Hash: `B246F9E85B9D392B0A33374974A2CA23`, and the Setup phase Status as "Done".

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_4_applicationdone2.png">
  <img alt="title image light / dark." src="./assets/images/test02_4_applicationdone2.png" width="700">
</picture>
</p>


### Submit a Proof:

At the top of the explorer, hit "Submit Prove Task", a new popup window will appear to create a new prove task. In the Image hash part input the application hash. In this example it is: `C0704FFA2384B360548789AD22911938`. With the image hash input the public input you can enter should be an integer 64-bit type, here for this example we are using the value 112, input: `112:i64`.

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_5_provedone.png">
  <img alt="title image light / dark." src="./assets/images/test02_5_provedone.png" width="700">
</picture>
</p>

You can view the "Proof Info" details which contains the inputs, transcripts, auxiliary data (for batch field dividing in smart contracts) and instances of the proof batching circuits:

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_15_proofinfo.png">
  <img alt="title image light / dark." src="./assets/images/test02_15_proofinfo.png" width="700">
</picture>
</p>




### Deploying a Verification Contract:

One the application has been Setup and has a Proof generated, a verification contract can be deployed on one of the Ethereum testnets. Hit the "Deploy Verification Contract" at the top of the explorer. Input the Image ID, in this case `C0704FFA2384B360548789AD22911938` and choose a test network to deploy to. Hit "Confirm".


<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_6_deployverification.png">
  <img alt="title image light / dark." src="./assets/images/test02_6_deployverification.png" width="700">
</picture>
</p>

Sign the transaction and wait for the verification contract to be deployed.

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_7_sign.png">
  <img alt="title image light / dark." src="./assets/images/test02_7_sign.png" width="380">
</picture>
</p>

In the main window of the explorer the deployment task can be seen as "Processing",

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_8_deployprocessing.png">
  <img alt="title image light / dark." src="./assets/images/test02_8_deployprocessing.png" width="700">
</picture>
</p>

and after a short amount of time the status of this task is "Done".

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_9_deploydone.png">
  <img alt="title image light / dark." src="./assets/images/test02_9_deploydone.png" width="700">
</picture>
</p>

The Deployment Info shows a "Details section" with the address of the contract,


<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_12_deploydone.png">
  <img alt="title image light / dark." src="./assets/images/test02_12_deploydone.png" width="700">
</picture>
</p>

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_13_deploymentinfo.png">
  <img alt="title image light / dark." src="./assets/images/test02_13_deploymentinfo.png" width="700">
</picture>
</p>

This can be seen on etherscan:

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_14_etherscan.png">
  <img alt="title image light / dark." src="./assets/images/test02_14_etherscan.png" width="1000">
</picture>
</p>

With the verification contract deployed we can verify the proof on chain with the "Verify on Chain" facility. Within the proof section of the image in the explorer, select the Task ID for the proof, to view the proof task, then select "View Proof Info".

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_16_complete.png">
  <img alt="title image light / dark." src="./assets/images/test02_16_complete.png" width="1000">
</picture>
</p>

Hit the Verify on Chain button, sign the transaction, in some time the transaction receipt should should produce a successful on chain verification.<br>


<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/test02_17_prooftranscripts.png">
  <img alt="title image light / dark." src="./assets/images/test02_17_prooftranscripts.png" width="700">
</picture>
</p>

`Verification transaction successful! Transaction hash: 0x64f0c91ff3687541c674d9324a5869c9734f4729799869217e5640cda353f255`


[back](./../../)