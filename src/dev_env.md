# Setting up a development environment
## Preparation
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
Clone the `iteration0` repository, available [here](https://github.com/ChatImproVR/iteration0).
```sh
git clone https://github.com/ChatImproVR/iteration0.git
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
Recently, the sparse protocol for cargo registries was stablized. This can help improve initial compile times. See [the rust blog](https://blog.rust-lang.org/2023/03/09/Rust-1.68.0.html#cargos-sparse-protocol).
