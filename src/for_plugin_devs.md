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

# Setting up the script
```bash
function cimvr() {
    $HOME/Projects/chatimprovr/cimvr.py $@
}

export CIMVR_PLUGINS="$HOME/Projects/outwipe/target/wasm32-unknown-unknown/release"
```
