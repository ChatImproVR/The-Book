# Getting started with plugin development
The easiest way to get started is to fork the 
[plugin template repository](https://github.com/ChatImproVR/template)

![Use this template button](./use_this_template.png)

If you're using another git service than github, you can simply clone the repository and remove the default remote:
```sh
git clone git@github.com:ChatImproVR/template.git
cd template
git remote remove origin
```

The next step is to change the name of your package in `Cargo.toml`:
```toml
[package]
name = "template_plugin" # CHANGE ME!
```

This will become important later when you use the `pkg_namespace!()` macro

# Setting up the helper script
The helper script requires Python 3.

Assuming you have a copy of `iteration0` somewhere (in this case, `$HOME/Projects/chatimprovr`), you can put the following in your `~/.bashrc`:

```bash
function cimvr() {
    $HOME/Projects/chatimprovr/cimvr.py $@
}
```
This will allow you to access the script as `cimvr` anywhere.

let's say we want to develop a plugin called `foo`, that we're developing at `$HOME/Projects/foo`. Then we could notify the script about this like so:
```bash
export CIMVR_PLUGINS="$HOME/Projects/foo/target/wasm32-unknown-unknown/release"
```

After building both `iteration0`'s client and server, and also building our own plugin we could launch our script with the following:

```bash
cimvr foo
```

This will start the client and the server with our plugin as arguments. Note that this name `foo` is determined via package name, the one we set earlier.

The `CIMVR_PLUGINS` environment variable is a semicolon-seperated list of search paths. We've set it to `target/wasm32-unknown-unknown/release` under our own plugin, so that it can find `foo.wasm`.
