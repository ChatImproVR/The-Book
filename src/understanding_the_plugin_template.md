# Plugin Template

Before we start getting creative with plugin development, lets dive deep into the code to gain a better understanding of what is going on.

## Plugin File Structure
The following list is the file structure of the plugin template:
- `.cargo`
    - `config.toml`
- `src`
    - `lib.rs`
- `.gitignore`
- `Cargo.toml`
- `README.md`

New rust developers will be primarily familiar with developing code in `main.rs`, however plugins are a bit different. Plugins are libraries, so we write code starting from `lib.rs` instead.

Before we get into the code itself, there is one thing need to change. This information can be found in the `README.md` file, but we will go over it here as well.

### Update Cargo.toml file
Because this code is a template, the code is not fully setup for development. Inside the `Cargo.toml` file, you will find the following code:
```toml
[package]
name = "template_plugin" # CHANGE ME!
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies] # IMPORTANT!
cimvr_common = { git = "https://github.com/ChatImproVR/iteration0.git", branch = "main" }
cimvr_engine_interface  = { git = "https://github.com/ChatImproVR/iteration0.git", branch = "main" } 
serde = { version = "1", features = ["derive"] }
```

We need to change the name of the plugin. In this tutorial, it will be **galaga**. If we decide not change the name, the plugin will be compiled as `template_plugin.wasm`. Not only that, but the name will become important later when we use the `pkg_namespace!()` macro! __Make sure to pick something descriptive, unique, and long.__

Note that in the future, ChatImproVR may be available on crates.io or change it's name in git, in which case you will need to update the `[dependencies]` section!

Therefore, if we are creating the **galaga** plugin, Cargo.toml should look like:
```toml
[package]
name = "galaga"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
cimvr_common = { git = "https://github.com/ChatImproVR/iteration0.git", branch = "main" }
cimvr_engine_interface  = { git = "https://github.com/ChatImproVR/iteration0.git", branch = "main" }
serde = { version = "1", features = ["derive"] }
```

## Understanding lib.rs

Before we go straight into the `lib.rs` file and read the code, let's talk about some concept such as difference between client and server and the behavior of the template plugin.

### Difference between Client and Server
The Client and Server play very different roles in plugin developement. Deciding what code belongs server-side and what code belongs client-side can be challenging.

A simple description of the difference in semantics between client and server, is that changes made client-side will only be visible for that client, and changes made server-side can be made visible to **all** connected clients.

For example, let say that a user presses the left key to move an object to the left. If we want that behavior to only be visible for that user, then that code should be part of the client side and we do not need to send a message to the server to change the object. However, if the user wants to move an object which is visible to everyone else in the Server, then that movement behavior should happen Server-side; The Client will send a command to the server requesting that the object must move left.

Therefore, it is worth thinking about the desired behaviors first; If you are planning to develop a plugin that multiple users will interact at the same time, then it is recommended to develop the bahavior of the event on the Server-side. If you are planning to develop a story mode game, for example, that only one user will interect with other objects, developing the code client side would make sense and likely result in simpler code.
