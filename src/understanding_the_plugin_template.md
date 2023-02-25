# Plugin Template

Before we start getting creative with our plugin development, lets dive deep into the code to have a good understanding on what is going on.

## Plugin File Sturcture

The following list is the file sturcture of the plugin template.

- .cargo
    - config.toml
- src
    - lib.rs
- .gitignore
- Cargo.toml
- README.md

For early rust developers, majority of the code has been developed in the `main.rs` file. However, in this template does not contain at all because we are developing a feature/plugin as a library rather than entire new system (which will be the `main.rs`). 

## So Where Do I Code...?

As you might noticed that there is only one rust file which is `lib.rs` file. That is where we will spend the majority of our time. However, before we get into the code itself, there is one thing need to change. This information can be found in the `README.md` file, but we will go over here as well. 

### Update Cargo.toml file

Because this code is an template, the code is not fully setup for development. Inside the `Cargo.toml` file, you will find the following code.
```toml
[package]
name = "template_plugin" # CHANGE HERE
version = "0.1.0" # CHANGE HERE
edition = "2021" # CHANGE HERE

[lib]
crate-type = ["cdylib"]

[dependencies] # KEEP IN EYE
cimvr_common = { git = "https://github.com/ChatImproVR/iteration0.git", branch = "main" }
cimvr_engine_interface  = { git = "https://github.com/ChatImproVR/iteration0.git", branch = "main" } 
serde = { version = "1", features = ["derive"] }
```
There are few spots that need to change in this file as you can find the comments. 

1. We want to change the name of the plugin to whatever name you want to give. In this tutorial, it will be **galaga**. Without changing the name, you will not able to compile the plugin the name you want to have; in other words, if we decide not change the name, we need to compile the plugin as `template_plugin`. Not only that, but also this will become important later when you use the `pkg_namespace!()` macro! Make sure to pick something descriptive, unique, and long.

2. The version variable and edition veriables purposes are keeping track the plugin version itself. While there is no necessary need at the current moment, it is recommended to track the progess overtime.

There is a comment says `#KEEP IN EYE`. If there is a major change within ChatImproVR engine code such as the new name (instead of `iteration0`), those path needs to be change, but we will update that part in this tutorial/documentation as well.

Therefore, if we are creating a **galaga** plugin, the up-to-date Cargo.toml should be looking like below.

```toml
[package]
name = "galaga"
version = "0.1.0"
edition = "2023"

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

Client and server are very different based on their definition itself. However, what makes confusing to the developers will be what type of code falls into client versus server. 

Here is one simple sentence between the differences: Client will **only** show change on one user whereas server will show change on **every** users.

For example, let say that one user press the left key to move an object to the left. If we want that behavior that only the user want to see it while no one else in the same server wants to see it, then that code should be part of the client side and do not need to send a message to move that object. However, if the user wants to see that object movement to everyone else in the same server, then that movement behavior should happen in the server side whereas the client will send a command to the server saying that certain object must move left.

Therefore, it is worth thinking on what behaviors should happen for your plugin development. If you are planning to develop a plugin that multiple users will interact at the same time, then it is recommended to develop the bahavior of the event on the server side. If you are planning to develop a story mode game, for example, that only one user will interect with other objects, developing the code client side would make sense. 

TODO: maybe have the picture of the whiteboard we had from 2/24 meeting

### What Does the Template Plugin Do?
The following code in lib.rs should be looking similar the code below.
```rs
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


#[cfg(test)]
mod tests {
    #[test]
    fn im_a_test() {}
}
```