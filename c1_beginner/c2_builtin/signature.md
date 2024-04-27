# Poseidon Signature
The Poseidon Signature we provide is implemented vi a combination of ECC api. Recall that ECDSA signature has the following pesudo code

```
impl JubjubSignature {
    pub fn verify(&self, pk: &BabyJubjubPoint, msghash: &[u64; 4]) {
        unsafe {
            let r = BabyJubjubPoint::msm(&[
                (pk, msghash),
                (&self.sig_r, &ONE.0),
                (&NEG_BASE, &self.sig_s),
            ]);
            require(r.x.is_zero() && r.y == ONE);
        }
    }
}
```

Thus to implement the signature on a particular curve, we only need to support the msm host for that curve.

## ECC host APIs for Poseidon
We provide the following build-in host APIs for poseidon curve.

```
extern "C" {
    pub fn babyjubjub_sum_new(x: u64);
    pub fn babyjubjub_sum_push(x: u64);
    pub fn babyjubjub_sum_finalize() -> u64;
}
```
And the reference implementation of msm is

```
impl BabyJubjubPoint {
    pub fn msm(points: &[(&BabyJubjubPoint, &[u64; 4])]) -> BabyJubjubPoint {
        let mut len = points.len();
        unsafe {
            babyjubjub_sum_new(1u64);
        }
        for (point, scalar) in points {
            unsafe {
                babyjubjub_sum_push(point.x.0[0]);
                babyjubjub_sum_push(point.x.0[1]);
                babyjubjub_sum_push(point.x.0[2]);
                babyjubjub_sum_push(point.x.0[3]);
                babyjubjub_sum_push(point.y.0[0]);
                babyjubjub_sum_push(point.y.0[1]);
                babyjubjub_sum_push(point.y.0[2]);
                babyjubjub_sum_push(point.y.0[3]);
                babyjubjub_sum_push(scalar[0]);
                babyjubjub_sum_push(scalar[1]);
                babyjubjub_sum_push(scalar[2]);
                babyjubjub_sum_push(scalar[3]);
                len -= 1;
                if len != 0 {
                    babyjubjub_sum_finalize();
                    babyjubjub_sum_finalize();
                    babyjubjub_sum_finalize();
                    babyjubjub_sum_finalize();
                    babyjubjub_sum_finalize();
                    babyjubjub_sum_finalize();
                    babyjubjub_sum_finalize();
                    babyjubjub_sum_finalize();
                    babyjubjub_sum_new(0u64);
                }
            }
        }
        unsafe {
            BabyJubjubPoint {
                x: U256([
                    babyjubjub_sum_finalize(),
                    babyjubjub_sum_finalize(),
                    babyjubjub_sum_finalize(),
                    babyjubjub_sum_finalize(),
                ]),
                y: U256([
                    babyjubjub_sum_finalize(),
                    babyjubjub_sum_finalize(),
                    babyjubjub_sum_finalize(),
                    babyjubjub_sum_finalize(),
                ]),
            }
        }
    }
}
```
Recall that the ZKWASM guest communicates with host circuits via a serial-like calling convention, thus the above implementation is a serial-like data transmit process as follows.
1. set the reset flat to the host circuits by **babyjubjub_sum_new(true)**
2. transfer the points x, y and scalars
3. finalize and get the results
