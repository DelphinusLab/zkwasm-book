# MISC Q&A

## Why pick WASM as the native bytecods our ZKVM

1. Wasm run time is native supported runtime in Browser and Serveless application. Given the support of WASM in zkWASM, web2 application runs in browser and selveless server can quickly establish a trustless rollup via Delphinus ZKWasm applification specific rollup as a service.
2. Wasm is the major alternative to ethereum bytecode. Wasmi for substrate, Wasmtimer for Cosmos. Give a zkWASM virtual machine, we can potentially turn a substrate blockchain into a etheurem rollup. This give existing ecosystem (substrate) a way to be adopted into another ecosystem (Etheurem)
3. Wasm has a verfy concise but flexible specification. The buildin host function give a way for application and rollup service a native way to extend the runtime. Delphinuslab application rollup service can provide specific wasm runtime (with differen host functions) to provide applicification service.
4. Wasm support a broad range of generic languages including C, C++, rust, assembly script, typescript, etc. Thus libraries written in different languages can be linked together via wasm module system which means it is more easiler to build an ecosystem than single backend language.
5. Wasm is a bytecode that has regular evolution managed by World Wide Web Consortium (W3C). It means that wasm adopts itself quick than stable bytecodes like evm to fits the need of morden web2 applications.

## Why support unmodified WASM specification is important.
Delphinus-ZKWASM follows and supports the unmodified standard WASM bytecode specification https://www.w3.org/TR/wasm-core-1/ which are supported as a standard backend target by most of the generic programming languages (C, C++, Rust, Go, Assembly Script, etc). By providing full compatiability, wasm generated from the existing compilers can be executed in zkWASM emulator without further adjustment. Further more, zkWASM also follows the definition of WASM host functions, which are functions expressed outside WebAssembly but passed to a module as an import.

Supporting the unmodified WebAssembly (WASM) bytecode specification has several advantages, particularly in terms of compatibility, security, performance, and cross-platform functionality. Here's a closer look at why it's beneficial to adhere to the standard, unmodified WASM specification:

1. Compatibility: One of the core strengths of WASM is its wide compatibility across different platforms and environments. By sticking to the unmodified specification, developers ensure that their WASM code can run consistently across various web browsers, server-side environments, and even embedded systems. This compatibility reduces the need for platform-specific adjustments and simplifies the development process.

2. Security: The WASM specification is designed with a strong focus on security, particularly in the context of web browsers. It operates in a sandboxed environment, which limits its access to the host system, thereby reducing potential security risks. Deviating from the standard specification could potentially introduce security vulnerabilities. For example, using WASM as the underlying ZKWASM bytecode natively prevents security problems like self-modified code.

3. Community and Ecosystem Support: Sticking to the standard WASM specification ensures access to the broader community and ecosystem support, including tools, libraries, and resources that are built around the standard WASM. Deviating from the specification might limit the ability to leverage these resources effectively.

4. Future-Proofing: The WASM specification is continuously evolving with contributions from a wide range of industry experts. By following the unmodified specification, developers can more easily adapt to future changes and enhancements in the WASM ecosystem.

5. Interoperability: WASM is designed to have native host api support, meaning it can support customization which allowing developers to seamlessly integrate code(or zk circuit) from external ecosystem. Delphinus-ZKWASM fully support WASM's native host api extension features thus can be used as an glue language for a board range of application specific circuits.

## Why not just use LLVM bitcode as a binary format?
The LLVM compiler infrastructure has a lot of attractive qualities: it has an existing intermediate representation (LLVM IR) and binary encoding format (bitcode). It has code generation backends targeting many architectures and is actively developed and maintained by a large community. In fact PNaCl already uses LLVM as a basis for its binary format. However, the goals and requirements that LLVM was designed to meet are subtly mismatched with those of WebAssembly.

WebAssembly has several requirements and goals for its Instruction Set Architecture (ISA) and binary encoding:

1. Portability: The ISA must be the same for every machine architecture.
2. Stability: The ISA and binary encoding must not change over time (or change only in ways that can be kept backward-compatible).
3. Small encoding: The representation of a program should be as small as possible for transmission over the Internet.
4. Fast decoding: The binary format should be fast to decompress and decode for fast startup of programs.
5. Fast compiling: The ISA should be fast to compile (and suitable for either AOT- or JIT-compilation) for fast startup of programs.
6. Minimal nondeterminism: The behavior of programs should be as predictable and deterministic as possible (and should be the same on every architecture, a stronger form of the portability requirement stated above).

LLVM IR is meant to make compiler optimizations easy to implement, and to represent the constructs and semantics required by C, C++, and other languages on a large variety of operating systems and architectures. This means that by default the IR is not portable (the same program has different representations for different architectures) or stable (it changes over time as optimization and language requirements change). It has representations for a huge variety of information that is useful for implementing mid-level compiler optimizations but is not useful for code generation (but which represents a large surface area for codegen implementers to deal with). It also has undefined behavior (largely similar to that of C and C++) which makes some classes of optimization feasible or more powerful, but can lead to unpredictable behavior at runtime. LLVM’s binary format (bitcode) was designed for temporary on-disk serialization of the IR for link-time optimization, and not for stability or compressibility (although it does have some features for both of those).

None of these problems are insurmountable. For example PNaCl defines a small portable subset of the IR with reduced undefined behavior, and a stable version of the bitcode encoding. It also employs several techniques to improve startup performance. However, each customization, workaround, and special solution means less benefit from the common infrastructure. We believe that by taking our experience with LLVM and designing an IR and binary encoding for our goals and requirements, we can do much better than adapting a system designed for other purposes.

Note that this discussion applies to use of LLVM IR as a standardized format. LLVM’s clang frontend and midlevel optimizers can still be used to generate WebAssembly code from C and C++, and will use LLVM IR in their implementation similarly to how PNaCl and Emscripten do today.
