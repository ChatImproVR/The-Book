# Setting up a development environment
See the project's [README.md](https://github.com/ChatImproVR/iteration0/blob/main/README.md) file!

ChatImproVR will restart plugins whose WASM files changes, making it easy to see updates appear. Note that this is not true hotloading, unless the plugin saves all of the relevant state.

You may find it helpful to install [cargo-watch](https://github.com/watchexec/cargo-watch) in order to compile plugins each time you save. This way, you can save your changes and have them appear in-game immediately!

**Note**: If you want `rust-analyzer` to work in VSCode with a module like `Client/`, you should open VSCode within that module. Otherwise `rust-analyzer` will glitch out.
