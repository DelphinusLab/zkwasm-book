# Setup & Build the ZKWASM binary

To build and run these projects requires the Rust compiler. Install rustup with:

```console
curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
```

Verify the installation with:
```console
rustc --version
```


Clone the [zkWasm](https://github.com/DelphinusLab/zkWasm) repo and move into the directory:
```console
git clone --recurse-submodules https://github.com/DelphinusLab/zkWasm.git
```

Build using cargo (CPU version):
```console
cargo build --release
```

Build GPU version (The recommended way of running zkWASM prover):
```console
cargo build --release --features cuda
```
