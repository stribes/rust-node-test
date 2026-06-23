# wasm-rust-bridge

A minimal proof-of-concept showing how to write functionality in Rust, compile it to WebAssembly, and consume it from Node.js

Theoretically, this approach works with any language that can compile to WebAssembly and provide a JavaScript interoperability layer.

# Limitations

WebAssembly is not a replacement for native bindings in every situation.

This approach works best for:
- Pure computation
- Data processing
- Parsers
- Algorithms
- Shared business logic

For direct access to system resources, files, sockets, or native OS APIs, a native binding approach such as N-API may be more appropriate.

# How It Works

```
Rust Code
    |
    v
wasm-bindgen
    |
    v
WebAssembly Module (.wasm)
    |
    v
Generated JavaScript Bindings
    |
    v
Node.js
```

`wasm-bindgen` generates the JavaScript wrapper code needed to call WebAssembly functions as if they were normal Javascript functions.

# Why This Is Interesting

This experiment demonstrates how Rust can be used as a shared implementation layer while JavaScript remains the primary developer-facing API.

Rather than maintaining seperate implementations across multiple languaes, functionally can be written once in Rust, compiled to WebAssembly, and consumed from Node.js with minimal glue code.

Potential benefits include:

- A single source of truth for core business logic
- The ability to expose the same functionality to multiple runtimes
- Improved performance for compute-heavy operations
- Strong compile-time guarantees from Rust
- Easier experimentation with polyglot SDK architectures

For projects such as an SDK library, where you might want multiple languages covered, this approach can reduce duplication while preserving a familiar JavaScript developer experience.

This repository serves as a minimal proof-of-concept for that workflow.

# Guide

## Prerequisites

I am on Debian so these commands may vary depending on your operating system.

### Rust

Install Rust:
`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

Verify: 
`rustc --version`
`cargo --version`

### Node.js

It is recommended to update package information before installing new software:
`sudo apt update`

Install Node.js and npm:
`sudo apt install nodejs npm`

Verify: 
`node --version`
`npm --version`

### wasm-pack

Install:
`cargo install wasm-pack`

Verify:
`wasm-pack --version`

## Create the Library

`cargo new package_name --lib`

### Cargo.toml

```toml
[package]
name = "package_name"
version = "0.1.0"
edition = "2024"

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
```

### src/lib.rs

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

### Build for Node.js

`cd rust`
`wasm-pack build --target nodejs`

This generates:
```
pkg/
- package.json
- package_name.js
- package_name.d.ts
- package_name_bg.wasm
- package_name_bg.wasm.d.ts
```

### Calling in Node.js

Create index.js:
```js
const { add } = require("./pkg");

console.log(add(2, 3));
````

Run:
`node index.js`

Expected output:
`5`
