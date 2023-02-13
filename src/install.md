# Setting up a development environment
# Installation
See the project's [README.md](https://github.com/ChatImproVR/iteration0/blob/main/README.md) file!

# Preparation
Make sure you have the `wasm32-unknown-unknown` target installed;
```sh
rustup target add wasm32-unknown-unknown
```

Dependencies on Ubuntu:
```sh
sudo apt install build-essential cmake libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev libspeechd-dev libxkbcommon-dev libssl-dev
```

# Compilation
Build the client and server like so:
```sh
pushd server
cargo build --release
popd

pushd client
cargo build --release
popd
```

You can compile all of the example plugins with the `compile_all.sh` script. If you're on windows, sorry! Please open a PR.

While most crates _are_ in a workspace, the client crate is unfortunately excluded due to an issue with the `openxr` crate. 

# Hosting a server
You may host a server with
```sh
cargo run --release -- <plugins>
```

# Connection to a remote server
You may use the client to connect to a remote server like so:
```sh
cargo run --release -- --connect <ip>:5031 <plugins>
```

The default port is 5031, but this can be configured in the server with `--bind <bind addr>:<port>`


