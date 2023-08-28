# Host Environment & Building

To build and run these projects require the Rust compiler. Install with:

```console
curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
```

Verify the installation with:
```console
rustc --version
```


Clone the [zkWasm repo](https://github.com/DelphinusLab/zkWasm) and move into the directory:
```console
git clone --recurse-submodules https://github.com/DelphinusLab/zkWasm.git
cd zkWasm
```

change into the `wasmi` directory from [Delphinus Labs](https://github.com/DelphinusLab/wasmi) and build using cargo:
```console
cd wasmi
cargo build --release
```

Build zkWasm:
```console
cd ..
cargo build --release
```
