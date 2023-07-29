# Delphinus-Lab-Book

This repo contains a `mdBook` for documentation about the zkWASM, tutorials for zkWasm service and specifications for zkWASM host circuits.

This book is a WIP and being constantly updated.



## Building:
This book is built with mdbook.

For installation instructions consult:
[mdBook Documentation](https://rust-lang.github.io/mdBook/guide/installation.html)

Alternatively you can use Cargo to install:
```console
cargo install mdbook
```

Build the book with:
<br></br>
```console
mdbook build
```

## Serving the Book:
<br></br>
To build & serve:

```
mdbook serve --open
```

To build & serve at host address on specific port:
```
mdbook serve -p 3000 -n 0.0.0.0
```