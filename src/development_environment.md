# Development Environment Version
If you want to develop new plugins or engine features with a version that might have some bugs, but has more features than the stable version, then you can clone our repository from [**HERE**](https://github.com/ChatImproVR/chatimprovr). To use this version, there are a few additional steps that need to be completed:

## Additional Software Download
There are two additional software to download to set up the development environment.
1. [Rust](https://www.rust-lang.org/tools/install)
2. [Cmake](https://cmake.org/download/)
<!-- 1. [Rust (for both Desktop and VR)](#rust)
2. [CMake (For VR only)](#cmake) -->

<!-- ### Rust
When you visit the page for Rust, it will greet you the screen as below.
![Rust Install Page](/images/rust_install_page.png)

Please follow the instructions as they stated. The page might look slighlty different based on operating system.

### CMake
When you visit the [page for Cmake](https://cmake.org/download/), it will greet you the screen as below. Please download the latest version of Cmake that has the option of Windows x64 Installer: the file extension is `.msi`.
![Cmake Install Page](/images/cmake_install_page.png)
In the image above, the version we are downloading is 3.26.4, but you can download and install a higher version if you want as well.

Once you download the installer, open the installer. The first screen should be similar as the following screen.

![Cmake Installer Part 1](/images/cmake_installer_part_1.png)

Select `Next` to continue the installation process.

The next screen will ask regarding the agreement of the program. Select agree to continue the installation process as the image below. You can save the agreement by selecting the `print` option if you want.

![Cmake Installer Part 2](/images/cmake_installer_part_2.png)

In the next screen, we are have an option to install on system path for current user, all users, or not at all. Choose either for the current user or all users to add the system PATH. The image below has selected for all users.

![Cmake Installer Part 3](/images/cmake_installer_part_3.png)

The next screen is selection the program path. The image below has selected the default path, but you can choose whatever path you want.

![Cmake Installer Part 4](/images/cmake_installer_part_4.png)

The next screen is an verifier on selecting all the correct options that you have selected. Once everything is all set, click the `install` button to proceed the installation.

![Cmake Installer Part 5](/images/cmake_installer_part_5.png)

Once the installation is complete, then you will see the following screen.

![Cmake Installer Part 6](/images/cmake_installer_part_6.png) -->

## WASM target
Make sure you have the `wasm32-unknown-unknown` target installed;
```sh
rustup target add wasm32-unknown-unknown
```

## Dependencies on Ubuntu:
```sh
sudo apt install build-essential cmake libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev libspeechd-dev libxkbcommon-dev libssl-dev
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

You can compile all of the example plugins with the `compile_all.sh` script. 
If you're on windows, you can either use `Git Bash` to run the `.sh` files or open a PR; sorry about it!

While most crates _are_ in a workspace, the client crate is unfortunately excluded due to an issue with the `openxr` crate.

## Setting up the helper script
The helper script is intended to make it easy to run the client, server, or both from a single command. The script requires Python 3.

### On Linux/Unix/MacOS (Bash)

If your MacOS system is using `bash` instead of `zsh`, then please follow this procedure. Otherwise, please follow the MacOS (Zsh) section.

Assuming you have a copy of `chatimprovr` somewhere (in this case, `$HOME/Projects/chatimprovr`), you can put the following in your `~/.bashrc`:

```bash
function cimvr() {
    $HOME/Projects/chatimprovr/cimvr.py $@
}
```
This will allow you to access the script as `cimvr` anywhere.

> *NOTE: If you do not have the .bashrc file, you need to create on in the $HOME directory.*

### On MacOS (Zsh)

Assuming you have a copy of `chatimprovr` somewhere (in this case,  `$HOME/Desktop/Rust/chatimprovr`), you can put the following in your `~/.zshrc`:

```zsh
function cimvr() {
    $HOME/Desktop/Rust/chatimprovr/cimvr.py $@
}
```
This will allow you to access the script as `cimvr` anywhere.

> *NOTE: If you do not have the .zshrc file, you need to create on in the $HOME directory.*

### On Windows
Assuming you have a copy of `chatimprovr` somewhere (in this case, `C:\Users\dunca\Documents\chatimprovr`), you can put the following in your `Microsoft.PowerShell_profile.ps1`.

```ps1
function cimvr() {
    $cimvr_path="C:\Users\dunca\Documents\chatimprovr"
    python $cimvr_path\cimvr.py $args
}
```
This will allow you to access the script as `cimvr` anywhere.

> *NOTE: If you cannot find the `Microsoft.PowerShell_profile.ps1`, you can find the file by typing `$profile` in Windows PowerShell. There is a chance that `Microsoft.PowerShell_profile.ps1` might not exist yet. In that case, you need to create a new file and the directory to match that path. In the image below, the file should be located in `Documents\WindowsPowerShell` under the file name as `Microsoft.PowerShell_profile.ps1`. If running scripts is disabled on your machine, consult the common fixes section.*

![$profile path](./images/profile_path.png)

### Using the script to launch the engine
After building both `chatimprovr`'s client and server as well as the example plugins, we could launch the cube example included with ChatImproVR using:
```bash
cimvr camera cube
```
in the terminal where you installed the helper script.

## Additional Tips and Tricks
### Sparse registries
Recently, the sparse protocol for cargo registries was stablized. This can help improve initial compile times. See [the rust blog](https://blog.rust-lang.org/2023/03/09/Rust-1.68.0.html#cargos-sparse-protocol).

### Logging
Wasmtime/Cranelift puts a bunch of junk in the log by default. To disable this, put the following in your RC file:
```sh
export RUST_LOG="debug,cranelift=OFF,wasmtime=OFF"
```