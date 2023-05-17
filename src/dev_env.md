# Setting up a development environment
## Preparation
### Installing Rust
Go to [Rustup](https://rustup.rs/) and follow the instructions. Alternatively, start at the [rust documentation](https://www.rust-lang.org/learn/get-started).

### WASM target
Make sure you have the `wasm32-unknown-unknown` target installed;
```sh
rustup target add wasm32-unknown-unknown
```

### Dependencies on Ubuntu:
```sh
sudo apt install build-essential cmake libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev libspeechd-dev libxkbcommon-dev libssl-dev
```

## Repository
Clone the `chatimprovr` repository, available [here](https://github.com/ChatImproVR/chatimprovr).
```sh
git clone https://github.com/ChatImproVR/chatimprovr.git
```

## Compilation
Build the client, server, and example plugins like so:
```sh
pushd server
cargo build --release
popd

pushd client
cargo build --release
cargo build --release --features vr # (For VR/OpenXR support)
popd

pushd example_plugins
./compile_all.sh # (Linux)
# ./compile_all.ps1 # (Windows)
popd
```

You can compile all of the example plugins with the `compile_all.sh` script. If you're on windows, sorry! Please open a PR.

While most crates _are_ in a workspace, the client crate is unfortunately excluded due to an issue with the `openxr` crate. 

## Next steps
[Set up the helper script](Beginner_Tutorial/setting_up_plugin_development_environment.md#setting-up-the-helper-script) in order to easily launch the engine for local testing and development.


## Tips and tricks
### Sparse registries
Recently, the sparse protocol for cargo registries was stablized. This can help improve initial compile times. See [the rust blog](https://blog.rust-lang.org/2023/03/09/Rust-1.68.0.html#cargos-sparse-protocol).

### Logging
Wasmtime/Cranelift puts a bunch of junk in the log by default. To disable this, put the following in your RC file:
```sh
export RUST_LOG="debug,cranelift=OFF,wasmtime=OFF"
```
