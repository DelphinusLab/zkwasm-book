# State Related Functions

## State Model of ZKWASM
We support a build-in set of merkle tree manipulating APIs as follows.
```
extern "C" {
    pub fn merkle_setroot(x: u64);
    pub fn merkle_address(x: u64);
    pub fn merkle_set(x: u64);
    pub fn merkle_get() -> u64;
    pub fn merkle_getroot() -> u64;
}
```

The build-in merkle tree is of depth 32 and the leafs are of index from 0 to 2^32-1. Also the host APIs follows a io convention. For instance, to get the data at leaf [index=0] with merkle root where root is of type [u64; 4], we need
1. set the leaf index we would like to query
2. set the merkle root which indicates which merkle we are querying
3. call **merkle_get** to get query the data
4. get the merkle root again (this is for the convention of the merkle host circuits)

```
    /// Get the raw leaf data of a merkle subtree
    pub fn get_simple(&self, index: u32, data: &mut [u64; 4]) {
        unsafe {
            merkle_address(index as u64); // build in merkle address has default depth 32

            merkle_setroot(self.root[0]);
            merkle_setroot(self.root[1]);
            merkle_setroot(self.root[2]);
            merkle_setroot(self.root[3]);

            data[0] = merkle_get();
            data[1] = merkle_get();
            data[2] = merkle_get();
            data[3] = merkle_get();

            //enforce root does not change
            merkle_getroot();
            merkle_getroot();
            merkle_getroot();
            merkle_getroot();
        }
    }
```

Similarly, when set a leaf of a given merkle tree (specified by the merkel root of type [u64; 4] we need to follow the following convention:

```
    /// Set the raw leaf data of a merkle subtree but does enforced the get/set pair convention
    pub unsafe fn set_simple_unsafe(&mut self, index: u32, data: &[u64; 4]) {
        unsafe {
            // perform the set
            merkle_address(index as u64);

            merkle_setroot(self.root[0]);
            merkle_setroot(self.root[1]);
            merkle_setroot(self.root[2]);
            merkle_setroot(self.root[3]);

            merkle_set(data[0]);
            merkle_set(data[1]);
            merkle_set(data[2]);
            merkle_set(data[3]);

            self.root[0] = merkle_getroot();
            self.root[1] = merkle_getroot();
            self.root[2] = merkle_getroot();
            self.root[3] = merkle_getroot();
        }
    }
```

Please refer to https://github.com/DelphinusLab/zkWasm-rust/blob/main/src/merkle.rs for more details about using the Merkle APIs to construct a sparse Merkle tree of depth 256.

It is high recommended that instead of using the raw Merkle APIs, using the structured implementation for your data storage.
