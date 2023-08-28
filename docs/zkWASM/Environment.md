# Host Environment & Building

To build and run these projects require the Rust compiler. Install with:

```console
curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
```

Verify the installation with:
```console
rustc --version
```


Clone the [zkWasm](https://github.com/DelphinusLab/zkWasm) repo and move into the directory:
```console
git clone https://github.com/DelphinusLab/zkWasm.git
cd zkWasm
```

initalize and update submodules for zkWasm:
```console
git submodule init
git submodule update
```

Build using cargo:
```console
cargo build --release
```

Build zkWasm:
```console
cd ..
cargo build --release
```
