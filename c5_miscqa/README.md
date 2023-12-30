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

