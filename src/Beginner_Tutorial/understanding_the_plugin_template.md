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

Let's switch to the main component, `lib.rs`. The following code should look similar to `lib.rs` you have. 

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




Let's go every line in detail what it means in general.

### Packages

```rust
use cimvr_engine_interface::{make_app_state, prelude::*, println};
```

The `cimvr_engine_interface` facilitates communication between the plugin and the host. It does not include any interfacing with the specific features of the client or server; instead these datatyes are relegated to the `common` crate. The third line does not print out since it is part of the standard library that cimvr is not using. In simple terms, the `cimvr_engine_interface` is the connector between plugins and the engine itself. 

```rust
make_app_state!(ClientState, ServerState);
```
The `make_app_state!` will run and compile the plugin code. In other words, it is the main function of the plugin in order to load into the engine itself.

### Client 

If you are not familiar with the idea of Client versus Server, please refer to this [page](../Core_Concepts/client_and_server.md) before continue reading this part of the code. At the same time, the implementation utilize the idea of ECS. If you are not familiar with ECS, please refer to this [page](../Core_Concepts/core_concepts.html#entity-component-system).

Before we start implemeting the feature/functionality to the client, we need to declare the client entity. Because there are no required parameter to attached with the client, it will be an empty struct with the name of ClientState. The line below is the method how we will declare it.

```rust
struct ClientState;
```

Now we are going to implement the client side feature/functionality by implementing the idea of UserState. The UserState is define as below.
```rust
pub trait UserState: Sized {
    /// Constructor for this state; called once before the **Init** stage.
    fn new(io: &mut EngineIo, sched: &mut EngineSchedule<Self>) -> Self;
}
```
All we need to do is to modify the `new` function with the same parameter. If we want to add more features that correlates to the client side, we can create other functions that are directly implemented to the ClientState entity, but we will walk over when adding behavior to objects and such. At the same time, we will talk more detail regarding the parameter of the new function.

Within the `new` function, the template contains the following code.

```rust
println!("Hello, client!"); // Line 1
cimvr_engine_interface::println!("This prints"); // Line 2
std::println!("But this doesn't"); // Line 3

Self // Line 4
```
The first line will print out in the terminal as how any Rust language will print out: the standard `println!` statement.
The second line will print out in the terminal but differently. While the first line prints out on the terminal/client side, the second line prints the text on the engine side itself. 

TODO: explain better with the cimvr_engine_interface part/the line above.

The last line is the returning the client UserState as the updated the version for the client.

Which makes up the entire code for the client side itself.

```rust
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
```

### Server

If you are not familiar with the idea of Client versus Server, please refer to this [page](../Core_Concepts/client_and_server.md) before continue reading this part of the code. At the same time, the implementation utilize the idea of ECS. If you are not familiar with ECS, please refer to this [page](../Core_Concepts/core_concepts.html#entity-component-system).


If you have read the client section, the server side present the same idea itself: printing out on the terminal.

```rust
// All state associated with server-side behaviour
struct ServerState;

impl UserState for ServerState {
    // Implement a constructor
    fn new(_io: &mut EngineIo, _sched: &mut EngineSchedule<Self>) -> Self {
        println!("Hello, server!");
        Self
    }
}
```
In conclusion, the template prints out text in the terminal for both client and server side. In the next section, we will discuss on how to create object in the space and focus on creating both 3D and 2D objects.
