# Prepare WASM Image
Since ZKWASM runs Web Assembly, applications that are suppose to run in ZKWASM have to be compiled into WASM. WebAssembly (WASM) is designed to be a portable compilation target for programming languages, enabling deployment on the web for client and server applications. A wide range of languages can be compiled into Wasm, with varying degrees of support. Some of the most notable languages include:

C/C++: These are the most well-supported languages for compiling to WebAssembly. Tools like Emscripten are used to compile C and C++ code to Wasm.

Rust: Rust has robust support for WebAssembly. It’s often chosen for Wasm projects due to its performance characteristics and safety guarantees.

AssemblyScript: A variant of TypeScript, AssemblyScript is designed to compile to WebAssembly. It’s a good choice for developers familiar with TypeScript or JavaScript.

Go: Go has experimental support for WebAssembly. While not as mature as C/C++ or Rust, it is still a viable option.

Python, Ruby, and PHP: These languages have experimental or early-stage support for WebAssembly. Tools and runtimes are being developed to compile these languages to Wasm, but they may not yet be suitable for production use.

Kotlin/Native: Kotlin can target WebAssembly, allowing Kotlin code to run in the browser.

The ability to compile a language to WebAssembly largely depends on the availability of a suitable compiler or toolchain. The WebAssembly ecosystem is rapidly evolving, and new tools and languages are constantly being added to the list.


## Generate WASM from C (or C++) code

Following is a trivial piece of C code "zkmain.c" that compares two inputs (one from the public input and the other from the private input) and requires that these two inputs are equal.

```C
#include <stdint.h>
uint64_t wasm_input(uint32_t);
extern void require(int cond);

static inline uint64_t wasm_public_input()
{
  return wasm_input(1);
}

static inline uint64_t wasm_private_input()
{
  return wasm_input(0);
}

__attribute__((visibility("default")))
int zkmain() {
    uint64_t public = wasm_public_input();
    uint64_t private = wasm_private_input();
    require(public == private);
    return 0;
}
```

To compile the above piece of C code into WASM, we use the clang-15 as our C compiler and set the target binary to be wasm32
```
LIBS  = -lkernel32 -luser32 -lgdi32 -lopengl32
CFILES = $(wildcard *.c)

ifeq ($(CLANG),)
CLANG=clang-15
endif

FLAGS = -flto -O3 -nostdlib -fno-builtin -ffreestanding -mexec-model=reactor --target=wasm32 -Wl,--strip-all -Wl,--initial-memory=131072 -Wl,--max-memory=131072 -Wl,--no-entry -Wl,--allow-undefined -Wl,--export-dynamic

all: output.wasm

output.wasm: $(CFILES)
	$(CLANG) -o $@ $(CFILES) $(FLAGS) $(CFLAGS)


clean:
	rm -f *.wasm *.wat
```

Put the above Makefile in the same folder as zkmain.c and type
```
make
```
will give the desired wasm file output.wasm. This file can be then used to run in the ZKWASM VM to generate proofs once inputs are provided. Also, more C to zkWASM program sdk can be find in Delphinus Lab's [zkWasm-C](https://github.com/DelphinusLab/zkWasm-C) repo.


## Generate WASM from Rust code

Here we use rust as a reference language, thus all examples are coded and compiled using wasm-bindgen. wasm-bindgen is a tool and library for facilitating high-level interactions between WebAssembly (Wasm) modules and JavaScript. It’s particularly prominent in the Rust ecosystem, but its principles can be applied to other languages that compile to WebAssembly.

To use wasm-bindgen in your rust project, you can simply put it in your Cargo.tom as follows:

```
[dependencies]
zkwasm-rust-sdk = { git = "https://github.com/DelphinusLab/zkWasm-rust.git" }
wasm-bindgen = "0.2.83"
```

Suppose that we use zkmain.rs as the main entry point of your application. We can start our application as follows:

```
use zkwasm_rust_sdk::wasm_output;
  use zkwasm_rust_sdk::wasm_input;
  use zkwasm_rust_sdk::wasm_dbg;
  use zkwasm_rust_sdk::require;

#[wasm_bindgen]
pub fn zkmain() -> {
    let mut hasher = Sha256::new();
    let commands_len = unsafe {wasm_input(0)};
    for _ in 0..commands_len {
        let command = unsafe {wasm_input(0)};
        hasher.update(command.to_le_bytes());
        step(command);
    }
    let msghash = hasher.finalize();
    zkwasm_rust_sdk::dbg!("Hello world application with command hash {:?}\n", msghash);
}
```

Simlarly the Makefile used to generate WASM from RUST is the following:
```
build:
	wasm-pack build --release --out-name rust-sdk-test.wasm --out-dir pkg --features wasmbind
	wasm-opt -Oz -o output.wasm pkg/rust-sdk-test.wasm

clean:
	rm -rf pkg output params
```

## File output after WASM code generation.
Once you have compiled your C (C++) and RUST code, the directory should look like the below:

```
.
├── Makefile
├── output.wasm
└── zkmain.c/zkmain.rs
```

The file `ouput.wasm` will be the input image for zkWasm's Setup Phase below.



