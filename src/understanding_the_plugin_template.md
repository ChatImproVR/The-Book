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

Before we get into the code itself, there is one thing we need to change. This information can be found in the `README.md` file, but we will go over it here as well.

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

```rust
use cimvr_engine_interface::{make_app_state, prelude::*, println};

// All state associated with client-side behaviour
struct ClientState;

impl UserState for ClientState {
    // Implement a constructor
    fn new(_io: &mut EngineIo, _sched: &mut EngineSchedule<Self>) -> Self {
        println!("Hello, client!");

        // NOTE: We are using the println defined by cimvr_engine_interface here, NOT the standard library!
        cimvr_engine_interface::println!("This prints");
        std::println!("But this doesn't");

        Self
    }
}

// All state associated with server-side behaviour
struct ServerState;

impl UserState for ServerState {
    // Implement a constructor
    fn new(_io: &mut EngineIo, _sched: &mut EngineSchedule<Self>) -> Self {
        println!("Hello, server!");
        Self
    }
}

// Defines entry points for the engine to hook into.
// Calls new() for the appropriate state.
make_app_state!(ClientState, ServerState);
```
