# Host(Builtin) Functions

A WebAssembly (WASM) host function refers to a function that is not defined in the WebAssembly module itself but is provided by the environment in which the WebAssembly code is executed. These functions are crucial for WebAssembly modules to interact with the outside world, as WASM on its own is quite sandboxed and limited in what it can do directly.

Here are some key points about WASM host functions:

1. Interaction with the Environment: WASM is designed to be a low-level, portable bytecode, which runs in a sandboxed environment. By itself, it doesn't have direct access to the system's resources like file systems, network, or the DOM in web browsers. Host functions serve as the bridge between the WASM code and these external resources.

2. Provided by the Host Environment: The host environment (which could be a web browser, a server-side platform like Node.js, or any other system that supports running WebAssembly) defines and provides these host functions. For instance, in a web browser, the host functions might allow interactions with web APIs.

3. Imported into WASM Modules: WASM modules import these host functions and use them as if they were part of the module. This import/export mechanism is part of the WASM design, allowing for modular code and separation of concerns.

4. Custom Functionality: Host functions can be used to provide custom functionality to WASM modules that is not available in the standard WASM instruction set. This can include anything from accessing the file system to performing complex computations that are more efficiently done in the host language (like JavaScript in the case of web browsers).

In summary, WASM host functions are a crucial part of the WebAssembly ecosystem, enabling WASM modules to interact with their environment in a controlled and secure manner, thus extending the capabilities of WebAssembly beyond its sandboxed context.

In the ZKWASM, we provide a wide range of pre-defined host functions to ease the burden of building sophisticated applications. We divides them into three major categories

1. IO related host functions
2. Constraint related host functions.
3. Predefined host functions with external circuits support.
