# Setting up Plugin Development Environment

Plugins are typically developed in their own repositories or folders.

## Before plugin development
Before we start doing any plugin development, we need to set up the engine into your machine. Please refer to the [Installation](../installation.md) as beginning installation and [Development Environment](../installation.md#most-advanced-version-development-environment-version) to set up the development environment.

## Using the template
The easiest way to get started is to fork the 
[plugin template repository](https://github.com/ChatImproVR/template).

[![Use this template button](./images/use_this_template.png)](https://github.com/ChatImproVR/template)

If you're using another git service besides github, you can clone the repository and remove the default remote:
```sh
git clone git@github.com:ChatImproVR/template.git
cd template
git remote remove origin
```
## Modifying the helper script
We need to update the helpder script to locate where the plugin path is located. For the next section, we will help you out how to modify the helper script based on each terminal.
> Note that all the additional change is inserting line(s) outside of the `cimvr` function call.

### On Linux/Unix/MacOS (Bash)
let's say we want to develop a plugin called `foo`, that we're developing at `$HOME/Projects/foo`. Then we could add this to our `~/.bashrc`:
```bash
export CIMVR_PLUGINS="$HOME/Projects/foo"
```

If you are developing on several plugins at the same time, for example `foo` and `poo` and `foo` is located in the `$HOME/Projects/foo` whereas the `poo` is located in the `$HOME/Desktop/foo`, then it will be seperated by the `;` sign.
```bash
export CIMVR_PLUGINS="$HOME/Projects/foo;$HOME/Desktop/poo"
```

### On MacOS (Zsh)
let's say we want to develop a plugin called `foo`, that we're developing at `$HOME/Desktop/Rust/foo`. Then we could add this to our `~/.bashrc`

```zsh
export CIMVR_PLUGINS="$HOME/Desktop/Rust/foo"
```

If you are developing on several plugins at the same time, for example `foo` and `poo` and `foo` is located in the `$HOME/Projects/foo` whereas the `poo` is located in the `$HOME/Desktop/foo`, then it will be seperated by the `;` sign.
```zsh
export CIMVR_PLUGINS="$HOME/Desktop/Rust/foo;$HOME/Desktop/poo"
```

### On Windows
Let's say we want to develop a plugin called `foo`, that we're developing at `C:\Users\dunca\Documents\Projects\foo`. Then we could add this to our `$profile`:
```ps1
$Env:CIMVR_PLUGINS="C:\Users\dunca\Documents\Projects\foo"
```
If you are developing on several plugins at the same time, for example `foo` and `poo` and `foo` is located in the `C:\Users\dunca\Documents\Projects\foo` whereas the `poo` is located in the `C:\Users\Mr.Kangs\Desktop\poo`, then it will be seperated by the `;` sign.
```ps1
export CIMVR_PLUGINS="C:\Users\dunca\Documents\Projects\foo;C:\Users\Mr.Kangs\Desktop\galaga"
```

### Using the script to launch plugins
```bash
cimvr foo
```

This will start the client and the server with our plugin as arguments. Note that this name `foo` is determined via package name, the one we set earlier.

The `CIMVR_PLUGINS` environment variable is a semicolon-seperated list of search paths. We've set it to the path of the plugin under our own plugin, so that it can find `foo.wasm`.

> Note that the `example_plugins` directory will be looked for by default, so you don't need to add an environment variable for these.

## Tips and tricks
### Using the cargo-watch crate
Running e.g. `cargo watch -x 'build --release'` in your plugin's root will compile it automatically when you save the source. In turn, the engine will reload your plugin automatically. This means you can effectively save source or assets and see the result immediately!
