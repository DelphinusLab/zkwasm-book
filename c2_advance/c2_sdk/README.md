# Writing ZKWASM Application
Here we using the previous template (can be find at "https://github.com/DelphinusLab/zkwasm-mini-rollup/blob/main/abi/src/lib.rs") to demonstrate the key concept of customizing your own rollup application.

As has said before, the core step of the service is ``handle_tx``, in which the server handles the user request and updates the global state (Merkle tree). Before ``handle_tx`` the server needs to check that the signature of the transaction is valid. Thus the core steps based on ``handle_tx`` will be the following three:

0. params preparation for the transaction
1. verification of the signature
2. handle the transaction

## Params preparation
In this tutorial, we assume that the inputs of a trasaction contains three parts. The first part is the user ID which is the publication key (``input[0..8]``). The second part is the signature of the transaction (``input[8..20]``) and the last part is the transaction inputs provided by the user (``input[20..]``). Since all the params are reading from the private inputs, it is read by ``zkwasm_input(0)`` as follows.

```
    for _ in 0..20 {
        params.push(unsafe {wasm_input(0)});
    }
    let command = unsafe {wasm_input(0)};
    let command_length = ((command & 0xff00) >> 8)  as usize;
    unsafe { zkwasm_rust_sdk::require(command_length < 16) };
    params.push(command);
    for _  in 0..command_length - 1 {
        params.push(unsafe {wasm_input(0)});
    }
```

## Signature verification
A reference implementation of ``verify_tx_signature`` can be found in https://github.com/DelphinusLab/zkwasm-mini-rollup in which the altjubjub curve is used. The convention is simple that the inputs(``input[20..]``) of the transaction are hashed to be the message of the signature.
```
#[wasm_bindgen]
pub fn verify_tx_signature(inputs: Vec<u64>) {
    let pk = BabyJubjubPoint {
        x: U256([inputs[0], inputs[1], inputs[2], inputs[3]]),
        y: U256([inputs[4], inputs[5], inputs[6], inputs[7]]),
    };

    let sig = JubjubSignature {
        sig_r: BabyJubjubPoint {
            x: U256([inputs[8], inputs[9], inputs[10], inputs[11]]),
            y: U256([inputs[12],inputs[13], inputs[14], inputs[15]]),
        },
        sig_s: [inputs[16], inputs[17], inputs[18], inputs[19]]
    };
    let commands = &inputs[20..];
    let msg = PoseidonHasher::hash(commands, true);
    sig.verify(&pk, &msg);
}
```

## Transaction handling
When handling the transaction, the handler changes the global state and produces a few settlement events. The changed state will be the initial state for the following transactions but the settlement events are accumulated until the preemption point is reached. Once the preemption point is reached, all settlement events will be flushed and hashend to form a pineed receipt. This receipt plus the merkle root before and after the execution will be the public input of the bundled execution.

1. handle a single transaction
```
pub fn handle_tx(params: Vec<u64>) -> Vec<u64> {
    let user_address = [params[0], params[1], params[2], params[3]];
    let sig_r = [params[16], params[17], params[18], params[19]];
    let command = &params[20..];
    let transaction = $T::decode(command);
    transaction.process(&user_address, &sig_r)
}
```

2. encode bytes of settlement events into wasm output
```
pub fn conclude_tx_info(data: &[u8]) -> [u64;4] {
    let mut hasher = Sha256::new();
    hasher.update(data);
    let result = hasher
        .finalize()
        .chunks_exact(8)
        .map(|x| {
            u64::from_be_bytes(x.try_into().unwrap())
        }).collect::<Vec<_>>();
    result.try_into().unwrap()
}
```

3. bind the inputs and outputs of the execution
```
pub fn zkmain() {

    // inputs 0, 1, 2, 3 are the merkel root of the initial state
    unsafe {
        initialize([wasm_input(1), wasm_input(1), wasm_input(1), wasm_input(1)].to_vec())
    }

    for _ in 0..tx_length {
        // prepare params
        params = inputs[0..];

        // verify signature
        verify_tx_signature(params.clone());

        // handle transaction
        handle_tx(params);
    }

    unsafe { zkwasm_rust_sdk::require(preempt()) };

    let bytes = finalize();
    let txdata = conclude_tx_info(bytes.as_slice());

    let root = merkle_ref.merkle.root;

    // inputs 4,5,6,7  are the merkel root of the final state
    unsafe {
        wasm_output(root[0]);
        wasm_output(root[1]);
        wasm_output(root[2]);
        wasm_output(root[3]);
    }

    // inputs 8,9,10,11 are the hash of the events receipt
    unsafe {
        wasm_output(txdata[0]);
        wasm_output(txdata[1]);
        wasm_output(txdata[2]);
        wasm_output(txdata[3]);
    }
}
```
