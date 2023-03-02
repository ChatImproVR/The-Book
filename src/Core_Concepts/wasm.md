# WebAssembly
## What is WebAssembly?
WebAssembly (WASM for short) provides a sandboxed environment in which programs written in a variety of languages can interface with a host application. WASM is intended to provide a platform-agnostic and performant alternative to native code.

## Limitations and features
Here we will go over some important aspects of working with WebAssembly in Rust. Firstly, that the standard library is available. Kind of. **In general, functions of the stdlib that communicate with the outside world will not work**; this includes but is not limited to: system time, files, sockets. Instead, one must rely on the interfaces provided by ChatImproVR. One exception to this rule is that allocation _does_ work; one can allocate memory with reckless abandon, at least up to 4 GB.

## Tips and tricks
### Including content
If you need to include content with your plugin (e.g. models and textures), you may use the `include_bytes!()` macro provided in the standard library to embed these files in your WASM binary. For more advanced use-cases, consider the [include dir](https://docs.rs/include_dir/latest/include_dir/) crate.

## Details
We use the `wasm32-unknown-unknown` target.
