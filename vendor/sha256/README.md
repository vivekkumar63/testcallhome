# sha256 crypto digest

[docsrs]: https://docs.rs/sha256

[![GitHub Actions](https://github.com/baoyachi/sha256-rs/workflows/check/badge.svg)](https://github.com/baoyachi/sha256-rs/actions?query=workflow%3Abuild)
[![Crates.io](https://img.shields.io/crates/v/sha256.svg)](https://crates.io/crates/sha256)
[![Docs.rs](https://docs.rs/sha256/badge.svg)](https://docs.rs/sha256)
[![Download](https://img.shields.io/crates/d/sha256)](https://crates.io/crates/sha256)


## Examples

#### sha256 digest string

```rust
use sha256::digest;

fn main() {
    let input = "hello";
    let val = digest(input);
    assert_eq!(val, "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824")
}
```

#### sha256 digest bytes

```rust
use sha256::digest_bytes;

fn main() {
    let input = b"hello";
    let val = digest_bytes(input);
    assert_eq!(val, "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824")
}
```

#### sha256 digest file

```rust
use sha256::digest_file;
use std::path::Path;

fn main() {
    let input = Path::new("./foo.file");
    let val = digest_file(input).unwrap();
    assert_eq!(val, "433855b7d2b96c23a6f60e70c655eb4305e8806b682a9596a200642f947259b1")
}
```
