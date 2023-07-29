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
void require(int cond); // external host api describes a equal constraint.
unsigned long long wasm_input(int); // external host api for fetch user inputs.

char* string ="delphinus-service-example\0";

void zkmain() {
    int sum = wasm_input(1);
    for (int i=0; string[i]!='\0'; i++) {
       sum -= 1;
    };
    require(sum == 0);
}
```

Save the build application as `example.wasm` to a local directory on your machine.

## Submitting a New Application:

Using upper right task bar on [Delphinus Lab Service Explorer](https://zkwasm-explorer.delphinuslab.com/) click `Create New Application` and upload the `example.wasm` in the "Image ID (MD5)" field.


[d](./assets/images/2_new_application.png)

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/1_taskbar.png">
  <img alt="title image light / dark." src="./assets/images/1_taskbar.png" width="448">
</picture>
</p>

<p align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/images/2_new_application.png">
  <img alt="title image light / dark." src="./assets/images/2_new_application.png" width="700">
</picture>
</p>