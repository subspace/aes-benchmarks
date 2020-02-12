##  AES Speed Benchmarks


* Compare Purism laptop (AES-NI) to Dell laptop (VAES-NI)
* Iteratively progress through different variants of AES
* For each mode compare both AES-128 and AES-256
* Test both CTR and CBC mode to see how many registers are used
* Add all results to google sheets tracker
* Need to setup debugger so that you can inspect the memory (if not expected)

1. OpenSSL

This test the standards implementation of AES using OpenSSL, which will use AES-NI by default. This will be limited to 16x XMM registers at 16 bytes each, though only one register will used in CBC mode. AES-NI can be manually disabled to see the pure software speed.

```
    # AES-128-CBC with AES-NI disabled
    OPENSSL_ia32cap="~0x200000200000000" openssl speed -elapsed -evp aes-128-cbc

    # AES-128-CBC with AES-NI enabled
	openssl speed -elapsed -evp aes-128-cbc

    # AES-256-CBC with AES-NI disabled
    OPENSSL_ia32cap="~0x200000200000000" openssl speed -elapsed -evp aes-256-cbc

    # AES-256-CBC with AES-NI enabled
	openssl speed -elapsed -evp aes-256-cbc
```

2. Rust AES crate at 1x and 8x blocks

This is a pure rust implementation of AES that also uses AES-NI by default. It can run in single block per instruction or eight blocks per instruction mode.

```
    git clone https://github.com/RustCrypto/block-ciphers.git
    cd block-ciphers
    cargo bench
```

3. Rust crypto-primitives 

This is rust bindings to C code that can take advantage of AVX-512 registers and VAES instructions.
Ensure gcc or clang are updated to latest version, else it will not build!
If you don't have Ice Lake architecture only the first half of benchmarks will run.

```
    git clone https://github.com/elalfer/rust-crypto-pimitives.git
    cd rust-crypto-pimitives
    cargo bench
```

4. Variable block width branch

This is a custom implementation that can be configured for any number of registers to be used per instruction. Though it may not take advantage of VAES instructions, need to check.
